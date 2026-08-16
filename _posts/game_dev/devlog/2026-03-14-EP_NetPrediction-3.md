---
title:  "[UE5] 추출 슈터 3-3. 예측 이펙트, 그리고 죽어 있던 코드 한 줄"
excerpt: "체감은 클라가, 판정은 서버가: 3단계 마무리"

categories:
  - DevLog
tags:
  - [UE5, C++, LagCompensation, Networking]

toc: true
toc_sticky: true

mermaid: true

date: 2026-03-14
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 3단계 지연 보상** 마지막 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 3-2. 서버 사이드 리와인드](/devlog/EP_NetPrediction-2)
{: .notice--info}

## 판정을 연결한다

[3-2편](/devlog/EP_NetPrediction-2)에서 만든 SSR을 사격 경로에 붙인다.

```cpp
void UEPCombatComponent::HandleHitscanFire(
    AEPCharacter* Owner, const FVector& Origin,
    const TArray<FVector>& Directions, float ClientFireTime)
{
    if (!Owner || !Owner->GetServerSideRewindComponent()) return;

    // ── [Rewind Block] SSR에 완전 위임 ──────────────────
    // Broad Phase, 리와인드, Narrow Trace, 복구, 디버그 전부 SSR 내부
    TArray<FHitResult> ConfirmedHits;
    Owner->GetServerSideRewindComponent()->ConfirmHitscan(
        Owner, EquippedWeapon, Origin, Directions, ClientFireTime, ConfirmedHits);

    // ── [Damage Block] GAS 전환 시 여기만 교체된다 ──────
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

**블록을 둘로 나눈 게 요점이다.**
*"어디를 맞았는가"*(SSR)와 *"얼마나 아픈가"*(대미지)는 다른 문제이다.
4단계 GAS로 가면 `ApplyPointDamage` 자리가 `GameplayEffectSpec`으로 바뀌는데,
**Rewind Block은 손대지 않아도 된다.**

*(실제로 그렇게 됐다. 지금 코드의 대미지 경로는 GAS이고 SSR은 그대로이다.)*

---

## 부위별 대미지: 두 경로를 만들었고, 하나는 죽어 있었다

이 절이 이 글을 다시 쓴 가장 큰 이유이다.

### 만든 것: 두 개의 배율

**① 본 이름 기반**

```cpp
float UEPCombatComponent::GetBoneMultiplier(const FName& BoneName) const
{
    if (EquippedWeapon && EquippedWeapon->WeaponDef)
        if (const float* Found = EquippedWeapon->WeaponDef->BoneDamageMultiplierMap.Find(BoneName))
            return *Found;

    UE_LOG(LogTemp, Verbose, TEXT("[BoneHitbox] Unknown bone: %s"), *BoneName.ToString());
    return 1.0f;
}
```

**② PhysicalMaterial 기반**

```cpp
UCLASS()
class UEPPhysicalMaterial : public UPhysicalMaterial
{
public:
    UPROPERTY(EditDefaultsOnly, Category = "Damage")
    bool bIsWeakSpot = false;

    UPROPERTY(EditDefaultsOnly, Category = "Damage",
              meta = (EditCondition = "bIsWeakSpot", ClampMin = 1.0f))
    float WeakSpotMultiplier = 2.0f;
};
```

![Set_PM.png](https://github.com/user-attachments/assets/00bd6fc0-2e22-40bd-b7f8-78c319a0c2f9)

Physics Asset에서 `head` 프리미티브를 고르고 Physical Material 슬롯에 약점 PM을 할당한다.

```cpp
// 트레이스에서 반드시 켜야 한다
FCollisionQueryParams Params;
Params.bReturnPhysicalMaterial = true;    // 안 켜면 Hit.PhysMaterial이 항상 무효
```

### 죽어 있던 쪽

원래 이 글에는 *"`BoneDamageMultiplierMap`은 DA_AK74 에셋에서 에디터로 설정"*이라고 썼다.
**불가능한 이야기였다.**

```cpp
// EPWeaponDefinition.h:43-44
// 부위별 대미지(GAS 이후 태그 기반으로 수정)
TMap<FName, float> BoneDamageMultiplierMap;      // ← UPROPERTY가 없다
```

`UPROPERTY`가 없으면:

- **에디터 디테일 패널에 나타나지 않는다** → 값을 넣을 방법이 없음
- **직렬화되지 않는다** → 저장·로드해도 항상 빈 맵

그리고 저장소 전체를 뒤져도 **이 맵을 채우는 코드가 없다.**
선언 한 줄과 `Find` 한 줄이 전부이다.

> **`GetBoneMultiplier`는 항상 1.0f를 반환하고 있었다.**

더 뼈아픈 건, **내 구현 문서에 이미 적혀 있었다**는 것이다.

```
DOCS/Notes/03_BoneHitbox_Implementation.md:1093
> BoneDamageMultiplierMap은 UPROPERTY가 없으므로 에디터 노출 없이
> C++ 또는 DataAsset 로직으로 채운다.
```

"채운다"라고 써놓고 채우지 않았다.
같은 문서의 문제 해결 표에는 이런 줄까지 있다.

```
| 데미지가 항상 BaseDamage 그대로 | GetBoneMultiplier가 1.0만 반환 |
  BoneDamageMultiplierMap 채워져 있는지 확인 |
```

**증상까지 예상해뒀는데 확인을 안 했다.**

### 그런데 헤드샷은 잘 됐다

이게 못 알아챈 이유이다.

| 부위 | Base | Bone | Material | Final |
|---|---|---|---|---|
| `head` | 35 | **1.0** | 2.0 (약점 PM) | **70.0** |
| `upperarm_l` | 35 | **1.0** | 1.0 | **35.0** |

**헤드샷 2배는 정상 동작했다**. 본 배율이 아니라 **PhysicalMaterial** 덕분이다.
결과가 맞으니 경로를 의심하지 않았다.

> 원래 이 글의 §결과에는 *"팔 히트 시 Final=26.25 (35 × 0.75)"*라고 적었다.
> **소스 어디에도 0.75가 없다.** 본 배율은 늘 1.0이었고,
> `UEPPhysicalMaterial::WeakSpotMultiplier`는 기본값이 2.0이다.
> 팔의 PhysicalMaterial에 `bIsWeakSpot = true, WeakSpotMultiplier = 0.75`를
> 굳이 넣었다면 나올 수는 있는데, **약점 배율에 0.75를 넣을 이유가 없다.**
>
> 결과를 눈으로 확인하지 않고 설계 표의 계획값을 그대로 적은 것이다. 실제로는 35.0이 찍혔을 거다.

그리고 로그는 **범인을 계속 가리키고 있었다.**

```cpp
UE_LOG(LogTemp, Log,
    TEXT("[BoneHitbox] Bone=%s PM=%s Base=%.1f Bone*=%.2f Mat*=%.2f Final=%.1f"), ...);
```

`Bone*=1.00`이 매번 찍혔을 거다.
**각 배율을 따로 찍도록 로그를 만들어놓고 읽지 않았다.**
로그를 설계한 판단은 좋았는데, 그걸 보는 습관이 없었다.

### 결말: 두 경로가 애초에 겹쳤다

`head` 본에 ×2.0을 주는 것과 `head` 바디의 약점 PM에 ×2.0을 주는 것은 **같은 말**이다.
같은 개념에 진실의 원천이 둘이었다.

그리고 둘 중 하나는 명백히 약한다.

| | 본 이름 맵 | PhysicalMaterial |
|---|---|---|
| 설정 위치 | 무기 DataAsset (`TMap<FName,float>`) | Physics Asset 바디 |
| 스켈레톤이 바뀌면 | **조용히 깨짐** (`FName` 문자열) | 영향 없음 (에셋 참조) |
| 오타 | 컴파일 통과, 런타임 무효 | 불가능 |
| 무기별 차이 | 표현 가능 | 어려움 |

본 이름은 **문자열 키**라 스켈레톤을 손보거나 리타깃하면 조용히 무효가 된다.
[2-6편](/devlog/EP_Replication-6)에서 겪은 것과 같은 종류의 취약함이다.

**결국 PhysicalMaterial 쪽으로 통일했다.**
현재 코드에 `BoneDamageMultiplierMap`은 **아예 없다.**
GAS 이후에는 PhysicalMaterial이 `GameplayTagContainer`를 들고,
`Zone.Weakspot` 같은 태그로 대미지 실행이 분기한다.

**죽어 있던 쪽을 지우고 살아 있던 쪽으로 모은 것**이 이 이야기의 결말이고,
살아남은 쪽이 원래 가려던 방향이었다.

---

## 예측 이펙트: 이 글의 본론

### 문제

```
[클라] 클릭 ──► [서버] Server_Fire ──► Multicast ──► [클라] 총구 화염
              (핑/2)                  (핑/2)
```

**쏜 본인도 Multicast를 기다린다.** 핑 80ms면 80ms 뒤에 불이 난다.
판정은 3-2편에서 정확해졌는데, **손맛은 여전히 느렸다.**

### 해결

```cpp
void UEPCombatComponent::RequestFire(const FVector& Origin, const FVector& Direction, float ClientFireTime)
{
    // ... 로컬 검증 (탄약, 연사 속도) ...

    AEPCharacter* Owner = GetOwnerCharacter();

    // ① 즉시 재생: 서버를 기다리지 않는다
    if (Owner && Owner->IsLocallyControlled())
    {
        const FVector MuzzleLocation =
            (EquippedWeapon->WeaponMesh && EquippedWeapon->WeaponMesh->DoesSocketExist(TEXT("MuzzleSocket")))
            ? EquippedWeapon->WeaponMesh->GetSocketLocation(TEXT("MuzzleSocket"))
            : EquippedWeapon->GetActorLocation();
        PlayLocalMuzzleEffect(MuzzleLocation);
    }

    // ② 그 다음에 서버로 요청
    Server_Fire(Origin, Direction, ClientFireTime);

    // ③ 반동은 요청 '뒤'에 적용
    if (Owner && Owner->IsLocallyControlled())
    {
        float Pitch = EquippedWeapon->GetRecoilPitch();
        float Yaw   = FMath::RandRange(-EquippedWeapon->GetRecoilYaw(), EquippedWeapon->GetRecoilYaw());
        Owner->AddControllerPitchInput(-Pitch);
        Owner->AddControllerYawInput(Yaw);
    }
}

void UEPCombatComponent::Multicast_PlayMuzzleEffect_Implementation(const FVector_NetQuantize& Loc)
{
    AEPCharacter* OwnerChar = GetOwnerCharacter();
    if (OwnerChar && OwnerChar->IsLocallyControlled()) return;   // 발사자는 이미 봤다
    PlayLocalMuzzleEffect(Loc);
}
```

핵심은 **재생 함수를 분리한 것**이다.
`PlayLocalMuzzleEffect()`를 만들어두면 로컬 예측과 Multicast가 **같은 코드**를 쓴다.
쏜 사람과 보는 사람이 다른 이펙트를 보는 일이 없다.

### ③의 순서가 중요하다

반동을 `Server_Fire` **뒤에** 적용한다.
그래야 서버로 보내는 `Direction`이 **반동 적용 전 조준 방향**이다.

순서를 바꾸면 플레이어가 조준했던 곳이 아니라 **반동으로 튄 방향**으로 총알이 나간다.
첫 발부터 위로 빗나간다.

---

## 예측의 대가: 아직 안 갚았다

예측을 넣으면 반드시 따라오는 질문이 있다.
**"예측이 틀렸을 때 무엇을 되돌리는가?"**

```cpp
// 클라: 로컬 검증 통과 → 이펙트 재생
if (EquippedWeapon->CurrentAmmo <= 0) return;
if (CurrentTime - LocalLastFireTime < FireInterval) return;
PlayLocalMuzzleEffect(...);                     // ← 이미 재생됐다

// 서버: 독립적으로 다시 검증
if (!EquippedWeapon || !EquippedWeapon->CanFire()) return;   // ← 여기서 거부되면?
```

**총구 화염만 나고 총알은 안 나간다.** 클라이언트는 그 사실을 통보받지 못한다.

이게 이론적인 이야기가 아닌 이유는 [2-4편](/devlog/EP_Replication-4)에서 짚은
**두 개의 시계** 때문이다.

```cpp
// 클라: RequestFire
float CurrentTime = GetWorld()->GetTimeSeconds();     // 클라 로컬 시계
// 서버: AEPWeapon::CanFire
float LastFireTime;                                   // 서버 로컬 시계
```

두 시계의 연사 속도 판정이 경계에서 갈리면 **연사 중 간헐적으로** 이 현상이 난다.
탄약도 마찬가지이다. 클라의 `CurrentAmmo`는 복제된 값이라 한 발 늦을 수 있다.

최소한의 답은 거부를 알려주는 것이다.

```cpp
UFUNCTION(Client, Unreliable)
void Client_FireRejected();      // 이펙트 정리 + 탄약 재동기화
```

**아직 없다.** GAS로 가면 이건 어빌리티 **예측 키(prediction key)**와
서버 확정/거부에 따른 롤백으로 처리된다.
*"예측은 넣었는데 롤백은 없다"*, 그게 4단계로 넘어가는 동기 중 하나이다.

### 그리고 데디케이티드 서버에서도 이펙트를 만들고 있다

```cpp
if (OwnerChar && OwnerChar->IsLocallyControlled()) return;
PlayLocalMuzzleEffect(Loc);
```

Multicast RPC는 **서버에서도 실행된다**(서버가 자기 자신에게도 호출).
데디케이티드 서버에서 `IsLocallyControlled()`는 항상 거짓이라 **가드를 통과**하고,
나이아가라 스폰과 사운드 재생이 시도된다.

렌더링이 없어 대부분 무의미하지만 공짜는 아니다.

```cpp
if (GetNetMode() == NM_DedicatedServer) return;
```

### 탄착 이펙트는 히트마다 나간다

```cpp
for (const FHitResult& Hit : ConfirmedHits)
{
    ...
    Multicast_PlayImpactEffect(Hit.ImpactPoint, Hit.ImpactNormal);   // 히트당 1회
}
```

지금은 히트스캔 1발이라 괜찮지만, `Directions`를 **배열로 받는** 구조
(= 산탄총을 염두에 둔 설계)에서 펠릿 8개면 **Multicast 8회**이다.
배열로 묶어 한 번에 보내야 한다.

---

## `ClientFireTime`: 시계를 맞춘다

```cpp
void AEPCharacter::Input_Fire(const FInputActionValue& Value)
{
    if (!CombatComponent) return;

    const AGameStateBase* GS = GetWorld()->GetGameState<AGameStateBase>();
    const float ClientFireTime = GS
        ? GS->GetServerWorldTimeSeconds()      // 서버 기준 시각
        : GetWorld()->GetTimeSeconds();

    CombatComponent->RequestFire(
        FirstPersonCamera->GetComponentLocation(),
        FirstPersonCamera->GetForwardVector(),
        ClientFireTime);
}
```

| | `GetWorld()->GetTimeSeconds()` | `GS->GetServerWorldTimeSeconds()` |
|---|---|---|
| 기준 | 각자의 로컬 시계 | 서버 시계 (복제 보정) |
| 클라/서버 차이 | 원점이 다름 | 동기화됨 |
| 리와인드 정확도 | 핑에 따라 오차 | 정확 |

SSR의 `HitboxHistory`도 같은 시계로 기록하므로 **되돌릴 시각이 일치**한다.

> **다만 남은 질문이 하나 있다.**
> [3-2편](/devlog/EP_NetPrediction-2)에서 확인한 대로,
> `GetServerWorldTimeSeconds()`는 **한 프레임 안에서도 부르는 위치에 따라 다른 값**을 준다.
>
> - **발사 시각**은 `Input_Fire`: 액터 틱(`TG_PrePhysics`, `LevelTick.cpp:1721`).
>   `TimeSeconds += DeltaSeconds`(:1581) **이후**이다.
> - **스냅샷 시각**은 `OnMovementUpdated`: `TickDispatch`(:1545).
>   `TimeSeconds` 증가 **이전**이다.
>
> 클라와 서버가 각자 자기 월드에서 읽으므로 단순히 한 프레임 차이라고 단정할 수는 없지만,
> **같은 종류의 편향이 남아 있을 가능성**은 있다.
> 실측 오차 2.3cm 안에 묻혀 있는 것으로 보이고, 남은 오차의 첫 번째 용의자로 보고 있다.

---

## 래그돌 사망 시 머리카락이 하늘로 솟았다

**관찰:** `SetSimulatePhysics(true)`로 래그돌이 되는 순간
Groom(머리카락·눈썹·수염)이 위로 날아갔다.

**조치:** 사망 시 Groom을 숨긴다.

```cpp
// Multicast_Die
if (FaceMesh)
{
    TArray<USceneComponent*> FaceChildren;
    FaceMesh->GetChildrenComponents(false, FaceChildren);
    for (USceneComponent* Child : FaceChildren)
        Child->SetVisibility(false, true);
}
```

> **원인은 확정하지 못했다.** 이 글의 초판에는
> *"LeaderPose가 `BoneSpaceTransforms`를 복사하는데 물리 시뮬 중에는 그게 갱신되지 않아서"*라고
> 적었는데, **엔진 소스와 맞지 않다.**
>
> ```cpp
> // SkinnedMeshComponent.cpp:2187-2189  GetBoneTransform()
> if(LeaderBoneIndex >= 0 && LeaderBoneIndex < NumLeaderTransforms)
> {
>     return LeaderPoseComponentInst->GetComponentSpaceTransforms()[LeaderBoneIndex] * LocalToWorld;
> }
> ```
>
> 팔로워는 리더의 **`ComponentSpaceTransforms`**를 읽는다.
> 물리가 갱신하는 것도 그 배열이다. 즉 Face는 래그돌을 **정상적으로 따라간다.**
> 내 설명대로라면 얼굴이 통째로 멈춰야 하는데, 실제로는 그렇지 않다.
>
> Groom은 `USkeletalMeshComponent`가 아니라 **별도 시뮬레이션**을 도는 컴포넌트라
> 래그돌 전환 시 바인딩 트랜스폼이 급변하면서 큰 속도를 얻는 쪽이 더 그럴듯하지만,
> **확인하지 못했다.** 추정으로 남긴다.

이 문제는 결국 [2-2편](/devlog/EP_Replication-2)에서 이야기한 **`AEPCorpse` 별도 액터**로 넘어간다.
시체를 캐릭터와 분리하면 Groom을 아예 안 붙이면 되고, 문제가 사라진다.

---

## 3단계 정리

세 편에서 만든 것을 한 장으로 줄이면 이렇다.

```
[클라이언트]  클릭
   ├─ 총구 이펙트 즉시 재생          ← 체감 (3-3)
   ├─ 반동 즉시 적용                 ← 체감 (3-3)
   └─ ClientFireTime(서버 시계) 첨부  ← 정확성의 열쇠 (3-3)
                 │
                 ▼
[서버]  ConfirmHitscan
   ├─ Broad Phase : Location으로 후보 추리기       (3-1)
   ├─ 리와인드     : SetBodyTransform으로 과거 복원 (3-2)
   ├─ Narrow Phase: 본 단위 LineTrace              (3-1)
   ├─ 복구         : 즉시 원위치                    (3-2)
   └─ 대미지       : Base × Bone × Material         (3-3)
                 │
                 ▼
[모든 클라] 탄착 이펙트                             (3-3)
```

> **체감은 클라이언트가, 판정은 서버가.**

3단계 전체가 이 한 문장이다.
그리고 그 둘을 잇는 게 **하나의 시계**(`GetServerWorldTimeSeconds`)와
**하나의 좌표계**(리와인드된 히트박스)이다.

숫자로는 이렇다.

| | 값 |
|---|---|
| 리와인드 위치 오차 | 242cm → **2.3cm** |
| 총구 이펙트 지연 | 핑만큼 → **0** |
| 최대 되돌림 | 0.5초 (게임 디자인 결정) |
| 서버 스냅샷 | 본 20개 × 약 17장 / 캐릭터 |

### 남겨둔 것

정직하게 적어둔다. 전부 4단계 이후의 숙제이다.

- **`Origin`·`ClientFireTime` 미검증**: 클라이언트 주장을 그대로 신뢰
- **예측 실패 시 롤백 없음**. GAS 예측 키로 해결 예정
- **본 배율 맵 제거**: PhysicalMaterial + GameplayTag로 통일
- **데디 서버 이펙트 가드** / **탄착 Multicast 묶기**

---

## 다음 단계

3단계까지는 **직접 만든 시스템**이었다.
4단계에서는 그걸 **GAS 위에 다시 세운다**.
`Server_Fire`는 `GA_Item_PrimaryUse`로, `TakeDamage`는 `GameplayEffect`로,
그리고 여기서 못 한 **예측 롤백**이 어빌리티 예측 키로 들어온다.

SSR은 그대로 남는다. 처음부터 그러라고 컴포넌트로 떼어놨다.

---

## 참고

- 엔진 소스: `SkinnedMeshComponent.cpp:2187` / `LevelTick.cpp:1545, 1581, 1721`
- 프로젝트 문서: `DOCS/Notes/03_BoneHitbox_Implementation.md`
