---
title:  "[UE5] 추출 슈터 2-6. 애니메이션, 그리고 남에게 보이는 것"
excerpt: "조준 피치는 2바이트로 온다: 애니메이션 상태의 동기화 경로 전부"

categories:
  - DevLog
tags:
  - [UE5, C++, Animation]

toc: true
toc_sticky: true

mermaid: true

date: 2026-03-01
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 2단계 Replication** 마지막 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 2-5. 복제 설계 전부](/devlog/EP_Replication-5)
{: .notice--info}

## 이 글의 질문

애니메이션을 붙이는 것 자체는 검색하면 나온다.
멀티플레이 게임에서 진짜 질문은 이거다.

> **이 애니메이션 상태가 다른 플레이어 화면에는 어떻게 도착하는가?**

혼자 테스트하면 전부 잘 된다. 내 캐릭터는 모든 값을 로컬로 알고 있으니까.
**2인으로 붙이는 순간 절반이 안 움직였다.**

먼저 결론 표부터 놓겠다. 이 글은 이 표를 설명하는 글이다.

| AnimInstance 필드 | 어디서 오나 | 다른 클라에서 보이나 |
|---|---|---|
| `Speed` | `GetVelocity()`: 이동 복제 | ✅ |
| `Direction` | 속도 + 액터 회전 | ✅ |
| `bIsCrouching` | `ACharacter::bIsCrouched` (엔진 복제 변수) | ✅ |
| `bIsFalling` | CMC 이동 모드 (복제됨) | ✅ |
| `AimPitch` | **`APawn::RemoteViewPitch16`** (`uint16`, `COND_SkipOwner`) | ✅ |
| `AimYaw` | 액터 회전에 편승 | ✅ |
| `bIsSprinting` | CMC 플래그, **복제 안 됨** | ❌ |
| `bIsAiming` | CMC 플래그, **복제 안 됨** | ❌ |
| 피격 몽타주 | `Multicast_PlayHitReact` (Unreliable) | ✅ |

마지막 두 줄이 이 글에서 발견한 버그이다.

---

## 발견 ①: 남은 절대 조준 자세를 취하지 않는다

```cpp
// EPAnimInstance.cpp: NativeUpdateAnimation
bIsSprinting = Character->GetIsSprinting();
bIsAiming    = Character->GetIsAiming();
```

```cpp
// EPCharacter.cpp
bool AEPCharacter::GetIsAiming() const
{
    if (UEPCharacterMovement* CMC = Cast<UEPCharacterMovement>(GetCharacterMovement()))
        return CMC->bWantsToAim;
    return false;
}
```

`bWantsToAim`은 **`UPROPERTY`가 아니다.**
[2-1편](/devlog/EP_Replication-1)에서 **일부러** 그렇게 만들었다.
프로퍼티 복제 대신 `CompressedFlags`로 보내려고.

그 값이 채워지는 경로를 따라가 보면 이렇다.

| 머신 | 채워지는 경로 |
|---|---|
| 소유 클라이언트 | `Input_StartADS()`가 직접 대입 |
| 서버 | `UpdateFromCompressedFlags()`가 복원 |
| **다른 클라이언트 (Simulated Proxy)** | **없음. 영원히 `false`** |

`CompressedFlags`는 **소유 클라 → 서버 단방향**이다.
서버에서 다른 클라로 나가는 경로가 **아예 없다.**

**즉 내 화면 속 다른 플레이어는 절대 조준하지 않고, 절대 질주 애니메이션을 재생하지 않는다.**

이동은 정상이다. `Speed`는 `GetVelocity()`에서 오고 속도는 이동 복제로 오니까.
**남이 질주하면 빠르게 움직이긴 하는데, 자세는 걷기이다.**

### 2-1편의 결론이 절반만 맞았다

[2-1편](/devlog/EP_Replication-1)에서 나는 이렇게 썼다.
*"CompressedFlags로 옮겼으니 복제 변수가 필요 없다."*

**같은 상태인데 필요한 방향이 두 개였다.**

| 용도 | 필요한 방향 | 수단 |
|---|---|---|
| 이동 예측 (속도 계산) | 소유 클라 ↔ 서버 | `CompressedFlags` |
| **시각 표현** (남에게 보이는 자세) | 서버 → 전 클라 | **복제 변수** |

`CompressedFlags`는 첫 번째만 해결한다.
시각 표현을 위해서는 결국 이런 게 필요하다.

```cpp
UPROPERTY(Replicated)          // 시뮬레이티드 프록시 표시용
uint8 bRepIsAiming : 1;
```

중복 같지만 아니다. **역할이 다른 두 값**이다.
Lyra도 이동 예측 상태와 표시용 상태를 분리해서 다룬다.

아직 안 고쳤다. **"CompressedFlags면 다 되는 줄 알았다"**가 이 단계의 정직한 상태이다.

---

## 발견 ②: 조준 피치는 엔진이 이미 2바이트로 보내고 있었다

```cpp
FRotator AimRotation = Character->GetBaseAimRotation();
AimPitch = FMath::ClampAngle(AimRotation.Pitch, -90.f, 90.f);
```

여기서 궁금해졌다.
**남의 조준 피치는 어떻게 내 화면까지 오지?** 컨트롤 로테이션은 로컬 값인데.

`APawn`을 열어보니 답이 있었다.

```cpp
// Pawn.h:118
uint16 RemoteViewPitch16;

// Pawn.cpp:304
RemoteViewPitch16 = FRotator::CompressAxisToShort(NewRemoteViewPitch);

// Pawn.cpp:1289
DOREPLIFETIME_CONDITION(APawn, RemoteViewPitch16, COND_SkipOwner);
```

**엔진이 조준 피치를 압축해서, 소유자를 제외한 전원에게 복제한다.**

- `float`(4바이트) 대신 **`uint16`(2바이트)**: 각도를 65536단계로 양자화.
  조준 자세를 표현하는 데는 차고 넘친다.
  (UE 5.6 이전에는 `uint8` **1바이트**, 256단계였다. 그걸로도 충분했다는 뜻이다.)
- **`COND_SkipOwner`**: 소유자는 자기 조준 방향을 이미 안다. 보낼 이유가 없다.

[2-5편](/devlog/EP_Replication-5)에서 내가 세운 두 원칙을
**엔진이 이 한 줄에서 동시에 지키고 있었다.**

> "이미 아는 것은 보내지 않는다" + "필요한 정밀도만큼만 보낸다"

2-5편의 조건표에 `COND_SkipOwner`를 적어놓고 *"(미사용)"*이라고 썼는데,
**엔진이 내 캐릭터에 이미 쓰고 있었다.**

### Yaw는 왜 안 보내는가

`AimYaw`는 별도로 복제되지 않는다. **액터 회전 자체가 복제되기 때문**이다.

```cpp
bUseControllerRotationYaw = true;      // 2-1편
```

캐릭터 몸통이 조준 방향으로 돌고, 그 회전은 이동 복제에 실려 온다.
그래서 `AimYaw`는 **복제된 액터 회전에서 다시 계산**하면 된다.

```cpp
AimYaw = FMath::ClampAngle(
    FRotator::NormalizeAxis(AimRotation.Yaw - Character->GetActorRotation().Yaw), -90.f, 90.f);
```

**Pitch는 따로 보내고 Yaw는 안 보낸다.** 이유는 하나이다.
Yaw는 이미 다른 것에 실려 있고, Pitch는 어디에도 실려 있지 않기 때문이다.
(캐릭터는 위아래로 기울지 않으니까.)

---

## Lyra 애니메이션 리타게팅

에셋을 직접 만들 수 없으니 Lyra의 것을 쓴다.
Epic 공식 샘플이라 라이선스 문제가 없고, Rifle/Pistol/Shotgun/UnArmed가 다 있다.

**IK Retargeter 2단계**

1. **메타휴먼 IK Rig 생성**

![IKRig.png](https://github.com/user-attachments/assets/86f3c77b-c57a-4719-9968-3240afd5745b)

메타휴먼 스켈레탈 메시로 IK Rig를 만들고 리타깃 체인을 자동 생성했다.

2. **IK Retargeter**

![Retargeter.png](https://github.com/user-attachments/assets/a68a6401-5cf1-4a02-a684-3a97f63fef76)

Lyra의 `RTG_Mannequin`을 사용해 시퀀스를 메타휴먼용으로 익스포트했다.

**막힌 곳, AimOffset은 리타기터에 안 뜬다**

AimOffset은 애니메이션 시퀀스가 아니라 **블렌드 에셋**이라
IK Retargeter의 익스포트 목록에 나오지 않는다.
`에셋 우클릭 → 리타깃 애니메이션 에셋`으로 따로 처리해야 했다.
(내부 시퀀스들이 먼저 리타깃돼 있어야 한다.)

BlendSpace는 지금은 직접 만들었다.

![BS_RifleIdleJog.png](https://github.com/user-attachments/assets/2238def7-3878-4db0-9647-b214711a2c6f)

Lyra는 BlendSpace 대신 Distance Matching + Orientation Warping을 쓰지만,
그건 지금 단계에서 과하다. 나중에 갈 방향으로만 적어둔다.

---

## Linked Anim Layer: 무기를 데이터가 고른다

```
┌─────────────────────────────────────────┐
│ ABP_EPCharacter  (메인)                  │
│  Parent C++: UEPAnimInstance             │
│                                          │
│  Locomotion StateMachine                 │
│  Linked Anim Layer 슬롯 (빈 슬롯)        │
│    FullBody_IdleWalkRun / Crouch / Jump  │
│  AimOffset (AimPitch / AimYaw)           │
│  AdditiveHitReact 슬롯                   │
└─────────────────────────────────────────┘
              ↑ LinkAnimClassLayers()
┌─────────────────────────────────────────┐
│ ABP_EPWeaponAnimLayersBase               │
│  레이어 함수의 구현 구조만 정의           │
└─────────────────────────────────────────┘
              ↑ 상속
┌─────────────────────────────────────────┐
│ ABP_RifleAnimLayers                      │
│  FullBody_IdleWalkRun → BS_IdleWalkJog   │
│  FullBody_Crouch      → Anim_Crouch      │
│  FullBody_Jump        → Anim_Jump        │
└─────────────────────────────────────────┘
```

| 계층 | 담당 | 새 무기 추가 시 |
|---|---|---|
| `ABP_EPCharacter` | 슬롯 정의. **에셋을 모름** | 손 안 댐 |
| `ABP_EPWeaponAnimLayersBase` | 레이어 구현 구조 | 손 안 댐 |
| `ABP_RifleAnimLayers` | 실제 에셋만 꽂음 | **이것만 새로 만듦** |

**[2-3편](/devlog/EP_Replication-3)의 3계층과 정확히 같은 사고**이다.
*"누가 바꾸나 / 언제 바뀌나가 다르면 나눈다."*

그리고 교체는 한 줄인다.

```cpp
// OnRep_EquippedWeapon / EquipWeapon 양쪽에서
Owner->GetMesh()->LinkAnimClassLayers(NewWeapon->WeaponDef->WeaponAnimLayer);
```

**여기가 2-3편과 물리적으로 이어지는 지점**이다.

```cpp
// UEPWeaponDefinition
TSubclassOf<UAnimInstance> WeaponAnimLayer;
```

**데이터가 애니메이션을 고르다.** C++ 어디에도 "Rifle"이라는 문자열이 없다.
새 무기는 DataAsset 하나 + AnimBP 하나면 끝이다.

그리고 이 호출이 `EquipWeapon`(서버)과 `OnRep_EquippedWeapon`(클라) **양쪽에** 있는 게 중요하다.
복제 변수 기반이라 **나중에 접속한 사람도 초기 복제로 `OnRep`을 받아** 같은 레이어를 링크한다.
[2-2편](/devlog/EP_Replication-2)의 `Multicast_Die`가 못 하던 그것이다.

---

## Locomotion 상태 머신과 AnimGraph

![Locomotion.png](https://github.com/user-attachments/assets/eeb70a47-a854-43e8-9cf8-24dc211c040c)

| State | 호출 레이어 | 전환 조건 |
|---|---|---|
| `IdleWalkJog` | `FullBody_IdleWalkRun` | 기본 |
| `Crouch` | `FullBody_Crouch` | `bIsCrouching` |
| `JumpSources` | - | 점프 입력 |
| `Jump` | `FullBody_Jump` | 공중 |

`bIsCrouching`은 `ACharacter::bIsCrouched`에서 온다. **엔진 복제 변수**이다.
[2-5편 결정 2](/devlog/EP_Replication-5)에서 *"Crouch는 엔진이 다 해준다"*고 했는데,
그 "다"에는 **다른 클라이언트에 보이는 것까지** 포함돼 있었다.
내가 직접 만든 Sprint/ADS와 갈린 지점이 정확히 여기이다.

![AnimGraph.png](https://github.com/user-attachments/assets/1f4df2a2-d0bc-4de2-8a2d-33bda70b1f3f)

```
[Locomotion State Machine]
    ↓
[Cache Pose: "Locomotion"]
    ↓
[Layered Blend Per Bone: spine_01]   ← 상체에만 AimOffset
  Base    : Cached Locomotion         (하체 = 이동)
  Blend 0 : AimOffset (Pitch/Yaw)     (상체 = 조준)
    ↓
[FABRIK: 왼손 IK]
    ↓
[AdditiveHitReact 슬롯]              ← Multicast_PlayHitReact 몽타주
    ↓
[Output Pose]
```

**`spine_01` 하나로 몸을 반으로 가른다.**
FPS 캐릭터 애니메이션의 기본 구조이다.
다리는 어디로 가는지, 상체는 어디를 보는지를 **따로** 표현해야 하기 때문이다.
달리면서 뒤를 조준하는 자세가 이걸로 나온다.

`AdditiveHitReact` 슬롯이 [2-5편 결정 7](/devlog/EP_Replication-5)의
`Multicast_PlayHitReact`가 재생되는 자리이다. Additive라 이동 중에도 겹친다.

---

## 스레드: 무엇을 어디서 읽는가

§Property Access를 쓰면 워커 스레드에서 안전하다고들 하는데,
**왜 안전한지**가 중요하다.

```cpp
void UEPAnimInstance::NativeUpdateAnimation(float DeltaSeconds)   // ← 게임 스레드
{
    ...
    FTransform WorldLeftHandIK = WeaponMesh->GetSocketTransform(FName("LeftHandIK"));
    FTransform HandR_World     = Character->GetMesh()->GetBoneTransform(FName("hand_r"));
    LeftHandIKTransform = WorldLeftHandIK.GetRelativeTransform(HandR_World);
}
```

`GetSocketTransform` / `GetBoneTransform`은 **다른 컴포넌트를 건드린다.**
워커 스레드에서 부르면 안 된다. 그래서 이 코드는 게임 스레드에 있어야 맞다.

전체 구조는 이렇다.

```
[게임 스레드] NativeUpdateAnimation
    └ 외부 객체(캐릭터·CMC·무기)를 읽어 AnimInstance 필드에 채운다
              ↓  값 확정
[워커 스레드] AnimGraph 평가
    └ Property Access로 그 '필드'만 읽는다. 외부 객체는 건드리지 않는다
```

> **외부 객체 접근은 게임 스레드에서, 그래프는 필드만 읽는다.**

무기 AnimBP에서 `Get Main Anim Blueprint Thread Safe`로 메인 AnimBP의
`Speed`/`Direction`을 읽는 것도 같은 원리이다.
**이미 확정된 필드를 복사할 뿐**이라 안전하다. 변수를 중복 선언할 필요가 없다.

순수 계산만 있다면 `NativeThreadSafeUpdateAnimation`으로 옮겨
게임 스레드 부담을 더 줄일 수 있다.
지금은 소켓 조회 때문에 그럴 수 없다. **그건 선택이 아니라 제약이다.**

---

## FABRIK 왼손 IK: 그리고 남은 문제 세 개

```cpp
if (UEPCombatComponent* Combat = Character->GetCombatComponent())
{
    if (AEPWeapon* Weapon = Combat->GetEquippedWeapon())
    {
        UMeshComponent* WeaponMesh = Weapon->WeaponMesh;
        if (WeaponMesh)
        {
            FTransform WorldLeftHandIK = WeaponMesh->GetSocketTransform(FName("LeftHandIK"));
            FTransform HandR_World     = Character->GetMesh()->GetBoneTransform(FName("hand_r"));
            LeftHandIKTransform = WorldLeftHandIK.GetRelativeTransform(HandR_World);
        }
    }
}
```

무기의 `LeftHandIK` 소켓을 **오른손 본 기준 상대 좌표**로 바꿔서 넘긴다.
오른손이 총을 쥐고 있으니, 그 기준으로 왼손 목표를 잡으면
무기가 어디로 움직이든 왼손이 따라붙는다.

![FABRIK_Detail.png](https://github.com/user-attachments/assets/9439b914-234d-446f-b00c-2919390ed8a3)

**① 무기를 버리면 왼손이 허공을 잡는다.**
`else`가 없어서 `LeftHandIKTransform`이 **직전 무기의 값에 고정**된다.
`FTransform::Identity`로 되돌리거나 `bHasWeapon` 플래그로 FABRIK 알파를 꺼야 한다.

**② 손이 총에 파고든다.**

![Crosshair.png](https://github.com/user-attachments/assets/72b8746b-bca0-4283-8652-ce622cd1a5c9)

FABRIK의 팁 본이 `hand_l`이 아니라 **왼손 중지 본**으로 잡혀 있다.
그리고 무기마다 그립 위치가 다른데 오프셋이 없다.

들어갈 자리는 정해져 있다.

```cpp
// UEPWeaponDefinition에 필드 하나
UPROPERTY(EditDefaultsOnly, Category = "Weapon|Anim")
FTransform LeftHandGripOffset;
```

[2-3편](/devlog/EP_Replication-3)에서 만든 Definition에 필드를 더하는 일이다.
**어디에 무엇을 넣어야 하는지는 이미 알고 있고, 폴리싱 단계에서 처리한다.**

**③ 쓰이지 않는 필드가 셋 있다.**

```cpp
UPROPERTY(BlueprintReadOnly, Category = "IK") FTransform RightHandIKTransform;  // 미사용
UPROPERTY(BlueprintReadOnly, Category = "IK") FVector RightElbowWorldPos;       // 미사용
UPROPERTY(BlueprintReadOnly, Category = "IK") FVector LeftElbowWorldPos;        // 미사용
```

선언만 하고 한 번도 대입하지 않는다. 항상 기본값이다.
블루프린트에서는 멀쩡히 보이므로, 실수로 연결하면 **팔이 원점으로 날아간다.**
지우는 게 맞다.

---

## 크로스헤어: `IsLocalController()`가 필요한 이유

```cpp
// EPPlayerController.cpp
// PlayerController는 서버에도 존재 → 서버에서 HUD 생성 방지
if (IsLocalController())
{
    CrosshairWidget = CreateWidget<UEPCrosshairWidget>(this, CrosshairWidgetClass);
    CrosshairWidget->AddToViewport();
}
```

[1-1편](/devlog/EP_Gameplay_Framework-1)에서 정리한
*"서버에는 모든 플레이어의 PlayerController가 있다"*의 직접적인 귀결이다.

이 검사가 없으면 데디케이티드 서버에서 **접속자 수만큼 위젯이 생성**된다.
서버는 화면이 없으니 눈에 안 보이고, 메모리만 조용히 샌다.

`HasAuthority()`가 아니라 `IsLocalController()`인 것도 중요하다.
리슨 서버에서는 호스트의 컨트롤러가 **둘 다 참**이고, 나머지는 권한만 참이다.
**"이 머신에서 실제로 화면을 보는가"**를 묻는 건 `IsLocalController()`뿐이다.

---

## 2단계를 마치며

이 여섯 편에서 만든 것을 한 줄로 줄이면:

> **"무엇을, 누구에게, 어떤 방향으로 보낼 것인가"를 매번 판단하는 습관.**

그리고 이 마지막 글에서 **그 판단을 한 번 놓쳤다는 걸 발견했다.**
Sprint/ADS는 이동 예측 방향(소유 클라 → 서버)만 생각하고
시각 표현 방향(서버 → 전 클라)을 빼먹었다.

3단계는 여기서 한 발 더 나간다.
**"서버가 보는 현재"와 "클라가 보던 과거"가 다르다는 문제**,
지연 보상이다.

---

## 다음 편
→ [3-1. 본 단위 히트박스](/devlog/EP_NetPrediction-1)
