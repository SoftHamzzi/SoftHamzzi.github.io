---
title:  "[UE5] 추출 슈터 3-3. 부위 대미지와 클라 예측 이펙트"
excerpt: "판정 연결 + 부위 대미지 + 클라 예측 이펙트"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-03-14
last_modified_at: 2025-03-14
---

🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

---

## 개요

**이 포스팅에서 다루는 것:**
- `HandleHitscanFire` 함수를 통한 SSR 위임 구조와 Damage Block 분리 설계
- 부위별 데미지 배율 (`GetBoneMultiplier` + `UEPPhysicalMaterial`)
- `RequestFire`에서 총구 이펙트 즉시 예측 재생
- `Multicast_PlayMuzzleEffect` 중복 방지 패턴
- 래그돌 사망 시 Groom(머리카락) 처리 문제와 해결

**왜 이렇게 구현했는가 (설계 의도):**
- Damage Block을 SSR 위임과 분리 → GAS 단계에서 `ApplyPointDamage`만 `GameplayEffectSpec`으로 교체하면 끝
- 총구 이펙트를 즉시 예측 실행한다.
  - 딜레이가 조작감을 크게 해친다.  
  → 클라이언트 체감 우선
- Groom은 Physics Asset 래그돌과 궁합이 최악이었다.
  - 현실적 우선순위에서 숨기기 선택

---

## 구현 내용

### 1. HandleHitscanFire — SSR 위임 패턴

```cpp
// EPCombatComponent.cpp (private)
void UEPCombatComponent::HandleHitscanFire(
    AEPCharacter* Owner, const FVector& Origin,
    const TArray<FVector>& Directions, float ClientFireTime)
{
    if (!Owner || !Owner->GetServerSideRewindComponent()) return;

    // [Rewind Block] SSR에 완전 위임
    // Broad Phase, 리와인드, Narrow Trace, 복구, 디버그 전부 SSR 내부
    TArray<FHitResult> ConfirmedHits;
    Owner->GetServerSideRewindComponent()->ConfirmHitscan(
        Owner, EquippedWeapon, Origin, Directions, ClientFireTime, ConfirmedHits);

    // [Damage Block] GAS 전환 시 GameplayEffectSpec으로 교체
    for (const FHitResult& Hit : ConfirmedHits)
    {
        if (!Hit.GetActor()) continue;

        const float BaseDamage         = EquippedWeapon ? EquippedWeapon->GetDamage() : 0.f;
        const float BoneMultiplier     = GetBoneMultiplier(Hit.BoneName);
        const float MaterialMultiplier = GetMaterialMultiplier(Hit.PhysMaterial.Get());
        const float FinalDamage        = BaseDamage * BoneMultiplier * MaterialMultiplier;

        UGameplayStatics::ApplyPointDamage(
            Hit.GetActor(), FinalDamage,
            (Hit.ImpactPoint - Origin).GetSafeNormal(),
            Hit, Owner->GetController(), Owner, UDamageType::StaticClass());

        Multicast_PlayImpactEffect(Hit.ImpactPoint, Hit.ImpactNormal);
    }
}
```

- 대미지 블록은 GAS 이후, GameplayEffectSpec으로 교체한다.

### 2. 부위별 데미지 배율

**GetBoneMultiplier:**

```cpp
float UEPCombatComponent::GetBoneMultiplier(const FName& BoneName) const
{
    // WeaponDefinition에 TMap<FName, float> BoneDamageMultiplierMap 저장
    // DA_AK74 에셋에서 에디터로 설정
    if (EquippedWeapon && EquippedWeapon->WeaponDef)
        if (const float* Found = EquippedWeapon->WeaponDef->BoneDamageMultiplierMap.Find(BoneName))
            return *Found;

    // 목록에 없는 본 → 기본 배율 1.0 + Verbose 로그
    UE_LOG(LogTemp, Verbose, TEXT("[BoneHitbox] Unknown bone: %s"), *BoneName.ToString());
    return 1.0f;
}
```

**DA_AK74 에셋 배율 예시:**

| 본 | 배율 | 비고 |
|----|------|------|
| head | 2.0 | 헤드샷 2배 |
| neck_01 | 1.5 | |
| spine_04 / spine_02 | 1.0 | 기본 |
| upperarm / lowerarm | 0.75 | 팔 감소 |
| thigh / calf | 0.75 | 다리 감소 |

- 현재는 맞은 본 이름을 통해 대미지를 정하지만, GAS 이후 PhyscalMaterial로 옮길 예정이다.

**UEPPhysicalMaterial:**

```cpp
// Public/Combat/EPPhysicalMaterial.h
UCLASS()
class UEPPhysicalMaterial : public UPhysicalMaterial
{
    GENERATED_BODY()
public:
    // Physics Asset에서 head 바디에 이 PM 할당
    UPROPERTY(EditDefaultsOnly, Category = "Damage")
    bool bIsWeakSpot = false;

    UPROPERTY(EditDefaultsOnly, Category = "Damage",
        meta = (EditCondition = "bIsWeakSpot", ClampMin = 1.0f))
    float WeakSpotMultiplier = 2.0f;
};
```

**최종 데미지 계산:**

- FinalDamage = BaseDamage × BoneMultiplier × MaterialMultiplier
- 예: AK-74 BaseDamage=35, head 본 (BoneMultiplier=2.0), 약점 PM (MaterialMultiplier=2.0)  
→ 35 × 2.0 × 2.0 = 140 (즉사)

- 예: 팔 (BoneMultiplier=0.75), 일반 PM (MaterialMultiplier=1.0)
→ 35 × 0.75 × 1.0 = 26.25

- 이후, Lyra와 동일한 방식으로 대미지를 계산할 예정이다.

**추가 설정:**

```cpp
// LineTrace 시 반드시 bReturnPhysicalMaterial = true로 설정해야 한다.
FCollisionQueryParams Params;
Params.bReturnPhysicalMaterial = true;
```

- Hit.PhysMaterial에 바디에 할당된 PM이 채워진다.
- PhysicalMaterial 없으면 Hit.PhysMaterial.IsValid() == false

![Set_PM.png](https://github.com/user-attachments/assets/00bd6fc0-2e22-40bd-b7f8-78c319a0c2f9)

- Physics Asset에서 약점을 적용할 프리미티브를 선택한다.
- Physical Material 슬롯에 약점용 PM 할당한다.

### 3. 서버 검증 로그

```cpp
UE_LOG(LogTemp, Log,
    TEXT("[BoneHitbox] Bone=%s PM=%s Base=%.1f Bone*=%.2f Mat*=%.2f Final=%.1f"),
    *Hit.BoneName.ToString(),
    Hit.PhysMaterial.IsValid() ? *Hit.PhysMaterial->GetName() : TEXT("None"),
    BaseDamage, BoneMultiplier, MaterialMultiplier, FinalDamage);
```

- 로그를 남겨 적용된 대미지를 확인할 수 있게 하였다.

### 4. 클라이언트 예측 이펙트

**문제: Multicast만 쓰면 RTT만큼 딜레이**

1. 클라이언트: 발사 → Server_Fire RPC 전송
2. 서버: RPC 수신 → Multicast_PlayMuzzleEffect
3. 클라이언트: Multicast 수신 → 이펙트 재생

- 이전에는 서버에게서 응답을 받아야 이펙트가 재생되었다.

**해결: RequestFire에서 즉시 로컬 재생**

```cpp
void UEPCombatComponent::RequestFire(const FVector& Origin, const FVector& Direction, float ClientFireTime)
{
    // ... 검증 ...

    AEPCharacter* Owner = GetOwnerCharacter();
    if (Owner && Owner->IsLocallyControlled())
    {
        // RTT 없이 즉시 재생 — Multicast보다 RTT만큼 빠름
        const FVector MuzzleLocation =
            (EquippedWeapon->WeaponMesh && EquippedWeapon->WeaponMesh->DoesSocketExist(TEXT("MuzzleSocket")))
            ? EquippedWeapon->WeaponMesh->GetSocketLocation(TEXT("MuzzleSocket"))
            : EquippedWeapon->GetActorLocation();
        PlayLocalMuzzleEffect(MuzzleLocation);
    }

    Server_Fire(Origin, Direction, ClientFireTime);

    if (Owner && Owner->IsLocallyControlled())
    {
        // 반동도 즉시 예측 — 서버 확인 전에 적용
        float Pitch = EquippedWeapon->GetRecoilPitch();
        float Yaw = FMath::RandRange(-EquippedWeapon->GetRecoilYaw(), EquippedWeapon->GetRecoilYaw());
        Owner->AddControllerPitchInput(-Pitch);
        Owner->AddControllerYawInput(Yaw);
    }
}

void UEPCombatComponent::Multicast_PlayMuzzleEffect_Implementation(const FVector_NetQuantize& MuzzleLocation)
{
    AEPCharacter* OwnerChar = GetOwnerCharacter();
    if (OwnerChar && OwnerChar->IsLocallyControlled()) return; // 발사자는 이미 재생됨

    PlayLocalMuzzleEffect(MuzzleLocation);
}
```

- 로컬에서 재생할 수 있는 PlayLocal*Effect() 함수를 만들어 즉시 실행하도록 하였다.
  - Multicast는 발사자 본인도 다시 받게 되는데,  
  로컬 플레이어가 조종하는 캐릭터라면 실행하지 않는다.

### 5. ClientFireTime — 시간 기준 통일

```cpp
// AEPCharacter::Input_Fire (EPCharacter.cpp)
void AEPCharacter::Input_Fire(const FInputActionValue& Value)
{
    if (!CombatComponent) return;

    // GS->GetServerWorldTimeSeconds(): GameState가 복제하는 서버 기준 시간
    // SSR HitboxHistory도 동일 기준으로 기록 → 리와인드 시각이 정확히 일치
    const AGameStateBase* GS = GetWorld()->GetGameState<AGameStateBase>();
    const float ClientFireTime = GS
        ? GS->GetServerWorldTimeSeconds()
        : GetWorld()->GetTimeSeconds();

    CombatComponent->RequestFire(
        FirstPersonCamera->GetComponentLocation(),
        FirstPersonCamera->GetForwardVector(),
        ClientFireTime);
}
```

**두 시간 함수의 차이:**

| | GetWorld()->GetTimeSeconds() | GS->GetServerWorldTimeSeconds() |
|---|---|---|
| 기준 | 로컬 시계 | 서버 시계 (복제) |
| 클라/서버 차이 | RTT/2만큼 차이 | 동기화됨 |
| 리와인드 정확도 | 핑에 따라 오차 발생 | 정확 |

### 6. 래그돌 사망 시, Groom 처리

**문제**
- 래그돌 활성화 시 머리카락이 하늘로 올라가는 현상이 발생하였다.

1. `GetMesh()->SetSimulatePhysics(true)` 호출
2. 물리 시뮬레이션은 `ComponentSpaceTransforms`를 갱신
3. `LeaderPose(SetLeaderPoseComponent)`는 `BoneSpaceTransforms`를 복사
4. 물리 시뮬 중 `BoneSpaceTransforms`는 갱신 안됨 → 사망 직전 포즈로 고정
5. Groom 가이드 커브 루트가 `FaceMesh 소켓`에 바인딩 → 소켓이 고정된 위치에 남음
6. Groom 시뮬레이션이 가이드 커브 위치를 기준으로 날아감 → 머리카락 상승

**해결:**
- 임시적으로 사망 시 Groom을 숨기도록 하였다.

---

## 결과

- 헤드샷 시 `Bone=head, Final=70.0` (BaseDamage=35 × 2.0) 로그 출력
- 팔 히트 시 `Bone=upperarm_l, Final=26.25` (35 × 0.75) 로그 출력
- 사격 시 총구 이펙트가 버튼 누름과 동시에 즉시 재생
- 다른 클라이언트에서 보는 사격 이펙트가 정상적으로 재생 확인

**한계 및 향후 개선:**
- Groom 래그돌 완전 지원: Corpse 액터 스폰으로 전환 (GAS 단계 이후)
- `UEPPhysicalMaterial` MaterialTags → GAS 단계에서 `GameplayTagContainer` 기반으로 교체 예정