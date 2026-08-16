---
title:  "[UE5] 추출 슈터 3-1. 본 단위 히트박스: 서버는 아무것도 렌더링하지 않는다"
excerpt: "히트박스를 만들기 전에, 서버가 캐릭터의 포즈를 아는지부터 확인해야 했다"

categories:
  - DevLog
tags:
  - [UE5, C++, LagCompensation, Networking]

toc: true
toc_sticky: true

mermaid: true

date: 2026-03-09
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 3단계 지연 보상** 첫 번째 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 2-6. 애니메이션 시스템](/devlog/EP_Replication-6)
{: .notice--info}

## 목표

헤드샷을 만들려면 **어디를 맞았는지** 알아야 한다.

지금은 캡슐 하나이다.

![CapsuleComponent.png](https://github.com/user-attachments/assets/d0d19f2c-cf64-40f9-939d-0045b09924db)

캡슐은 "맞았다"까지만 안다. 머리인지 발인지는 모른다.
이 단계에서 **판정 인프라**를 깐다. 실제 지연 보상은 [다음 편](/devlog/EP_NetPrediction-2)이다.

---

## Physics Asset: 본마다 바디

![PhysicsAsset.png](https://github.com/user-attachments/assets/6b755951-2d9a-4bc4-a7f6-588d85e17a49)

Physics Asset은 **본마다 독립적인 콜리전 바디**를 갖는다.
트레이스가 맞으면 `FHitResult.BoneName`에 어느 본인지 기록된다.

| 본 | 바디 | 예정 배율 | 비고 |
|---|---|---|---|
| `head` | Sphere | ×2.0 | 약점 PhysicalMaterial 오버라이드 |
| `neck_01` | Capsule | ×1.5 | |
| `spine_04` / `spine_02` | Capsule | ×1.0 | 상체 |
| `pelvis` | Capsule | ×1.0 | 하체 중심 |
| `clavicle_l/r` | Capsule | ×0.9 | 어깨 |
| `upperarm_l/r` | Capsule | ×0.8 | |
| `lowerarm_l/r` | Capsule | ×0.7 | |
| `hand_l/r` | Capsule | ×0.5 | |
| `thigh_l/r` | Capsule | ×0.8 | |
| `calf_l/r` | Capsule | ×0.7 | |
| `foot_l/r` | Capsule | ×0.5 | |

손가락·발가락은 뺐다. **바디 수는 곧 트레이스 비용**이고,
손가락에 맞고 안 맞고가 게임을 바꾸지 않는다.

> **배율은 "예정"이다.** 이 글 시점의 대미지 코드는 이게 전부였다.
>
> ```cpp
> UGameplayStatics::ApplyDamage(Hit.GetActor(), EquippedWeapon->GetDamage(), ...);
> ```
>
> `ApplyDamage`라서 `Hit.BoneName`이 대미지 경로로 넘어가지도 않는다.
> 배율 계산은 [3-3편](/devlog/EP_NetPrediction-3)에서 붙인다.
> 그리고 거기서 **한 쪽 배율이 조용히 죽어 있었다는 것**을 발견한다.

---

## 전용 트레이스 채널이 필요한 이유

```cpp
// EPTypes.h
static constexpr ECollisionChannel EP_TraceChannel_Weapon = ECC_GameTraceChannel1;
```

"환경과 격리하려고"라고만 생각하기 쉬운데, **진짜 이유는 따로 있다.**

`ACharacter`에는 히트박스보다 **훨씬 큰 것**이 하나 있다. 이동용 캡슐이다.

```
       ┌─┐  ← 캡슐 (반지름 34cm, 높이 88cm)
   ┌───┼─┼───┐
   │   │●│   │  ← head Sphere (반지름 10cm 남짓)
   └───┼─┼───┘
```

같은 채널로 트레이스하면 **캡슐이 항상 먼저 막는다.**
`FHitResult.BoneName`은 비어 있고, **본에는 영원히 닿지 않는다.**

전용 채널이 필요한 이유는 이걸 표현해야 하기 때문이다.

| | `ECC_Pawn` (이동·오버랩) | `EP_TraceChannel_Weapon` |
|---|---|---|
| 캡슐 콜리전 | **Block** | **Ignore** |
| Physics Asset 바디 | Ignore | **Block** |
| 지형·벽 | Block | Block |

**같은 물체가 용도에 따라 다르게 반응해야 한다**. 그게 채널을 나누는 이유이다.
"환경 격리"는 부수 효과이다.

---

## 그런데 서버가 캐릭터의 포즈를 모르고 있었다

여기가 이 글에서 가장 중요한 부분이다.

```cpp
// AEPCharacter 생성자
GetMesh()->VisibilityBasedAnimTickOption =
    EVisibilityBasedAnimTickOption::AlwaysTickPoseAndRefreshBones;
```

**이 한 줄이 없으면 위에서 만든 모든 것이 무의미하다.**

### 왜인가

`USkeletalMeshComponent`의 기본값은 원래 `AlwaysTickPoseAndRefreshBones`이다.
그런데 `ACharacter`가 **일부러 한 단계 낮춘다.**

```cpp
// Character.cpp:124
Mesh->VisibilityBasedAnimTickOption = EVisibilityBasedAnimTickOption::AlwaysTickPose;
```

두 값의 차이는 주석에 그대로 있다.

```cpp
// SkinnedMeshComponent.h:95-98
/** Always Tick and Refresh BoneTransforms whether rendered or not. */
AlwaysTickPoseAndRefreshBones,
/** Always Tick, but Refresh BoneTransforms only when rendered. */
AlwaysTickPose,
```

**"렌더링될 때만 본 트랜스폼을 갱신한다."**
캐릭터는 화면 밖에 있을 때가 많으니 합리적인 최적화이다.

판정 코드를 보면 명확하다.

```cpp
// SkinnedMeshComponent.cpp:1615-1618
bool USkinnedMeshComponent::ShouldUpdateTransform(bool bLODHasChanged) const
{
    return (bRecentlyRendered ||
            (VisibilityBasedAnimTickOption == EVisibilityBasedAnimTickOption::AlwaysTickPoseAndRefreshBones));
}
```

> **데디케이티드 서버는 아무것도 렌더링하지 않는다.**
> `bRecentlyRendered`는 **항상 거짓**이다.

그러면 남는 조건은 하나뿐이다. 그 옵션을 켜는 것.

### 켜지 않으면 어떻게 되나

**서버의 모든 캐릭터가 레퍼런스 포즈(T 포즈)에 고정된다.**
살아 있을 때도, 죽었을 때도, 처음부터 끝까지요.

그 상태로 `GetBodyInstance(BoneName)->GetUnrealWorldTransform()`을 읽으면
**T 포즈 기준 좌표**가 나온다.
웅크린 적의 머리를 조준해도, 서버는 그 머리가 서 있는 자세의 위치에 있다고 믿는다.

> **처음엔 이걸 "사망 시 문제"로 이해했다.** 틀렸다.
> 리슨 서버로 테스트하면 호스트 화면에 보이는 캐릭터는 `bRecentlyRendered`가 참이라
> 정상 동작한다. 그래서 렌더링이 끊기는 상황에서만 증상이 눈에 띄었던 건다.
> **데디케이티드 서버에서는 처음부터 전부 깨져 있다.**

한 가지 예외가 더 있다.

```cpp
// SkeletalMeshComponent.cpp:1711-1712
const bool bShouldUpdateTransform = Super::ShouldUpdateTransform(bLODHasChanged) ||
        (GetAnimInstance() && GetAnimInstance()->IsAnyMontagePlaying() ...
```

**몽타주 재생 중에는 갱신된다.**
피격 몽타주가 도는 동안만 히트박스가 맞고 평소엔 T 포즈,
증상이 간헐적으로 보이는 이유가 될 수 있다.

### 대가

이 옵션은 최적화를 **끄는** 것이다.
서버가 모든 캐릭터의 본을 매 틱 갱신한다.

**그런데 서버는 어차피 그 정보가 필요하다.** 히트 판정을 하니까.
클라이언트에서는 여전히 기본값이 유효하다. 안 보이는 캐릭터는 갱신할 필요가 없다.

**"클라이언트에서 옳은 최적화가 서버에서는 버그가 된다."**
데디케이티드 서버 모델에서 자주 반복되는 형태이고,
[2-2편](/devlog/EP_Replication-2)의 메시 구조가 여기서 전제로 작동한다.

---

## 스냅샷 구조체

과거를 되짚으려면 과거를 저장해야 한다.

```cpp
// Public/Types/EPTypes.h

// 본 하나의 월드 트랜스폼
USTRUCT()
struct FEPBoneSnapshot
{
    GENERATED_BODY()
    UPROPERTY() FName      BoneName;
    UPROPERTY() FTransform WorldTransform;    // GetUnrealWorldTransform() 기록값
};

// 특정 시각의 캐릭터 전체
USTRUCT()
struct FEPHitboxSnapshot
{
    GENERATED_BODY()
    UPROPERTY() float   ServerTime = 0.f;
    UPROPERTY() FVector Location   = FVector::ZeroVector;  // Broad Phase용 루트 위치
    UPROPERTY() TArray<FEPBoneSnapshot> Bones;             // Narrow Phase 리와인드용
};
```

### `Location`을 따로 두는 이유

판정을 두 단계로 나누기 위해서이다.

| 단계 | 쓰는 데이터 | 비용 |
|---|---|---|
| **Broad Phase**: 후보 추리기 | `Location` 하나 | 거리 계산 몇 번 |
| **Narrow Phase**: 실제 판정 | `Bones` 전부 | 바디 이동 + 트레이스 |

총알 선분에서 멀리 떨어진 캐릭터는 **본을 볼 필요조차 없다.**
루트 위치 하나로 걸러낸다.

> 🚩 **그런데 이 `Location`을 "언제" 찍느냐가 다음 편의 주제이다.**
> *"틱이 도는 시점의 위치"*와 *"그 이동이 실제로 처리된 시점의 위치"*는 다르다.
> 그 차이가 [3-2편](/devlog/EP_NetPrediction-2)에서 **242cm의 오차**로 나타난다.

### 비용을 세어보면

| | 값 |
|---|---|
| 판정 본 수 | 약 20개 |
| 보관 기간 ÷ 간격 | 0.5초 ÷ 0.03초 = **약 17장** |
| 캐릭터당 `FTransform` | 20 × 17 = **340개** |
| 8인 매치 | **약 2700개** |

`FTransform`은 회전·위치·스케일을 담아 작지 않다.
**이게 매 틱 갱신되는 서버 메모리**이다.

줄일 여지는 있다.

| 방법 | 효과 |
|---|---|
| 스케일 제외, 위치+회전만 저장 | 구조체 크기 감소 |
| 판정 본만 저장 (손·발 제외) | 20 → 12개 |
| 이동이 없으면 스냅샷 생략 | 정지한 캐릭터 비용 0 |

지금 규모에서는 손대지 않았다. **다만 무엇이 비싼지는 알고 간다.**

---

## 알고 넘어가는 것: 판정 바디와 래그돌 바디가 같다

Physics Asset의 바디는 두 곳에서 동시에 쓰인다.

1. 이 글의 **히트 판정**
2. [2-2편](/devlog/EP_Replication-2)의 **래그돌**

```cpp
// Multicast_Die
GetMesh()->SetCollisionProfileName(TEXT("Ragdoll"));
GetMesh()->SetSimulatePhysics(true);
```

죽는 순간 그 바디들이 **물리 시뮬레이션에 의해** 움직이기 시작한다.
그리고 다음 편에서 만들 리와인드는 **그 바디를 강제로 과거 위치로 옮긴다.**

```cpp
BodyInstance->SetBodyTransform(PastTransform, ETeleportType::TeleportPhysics);
```

**시뮬레이션 중인 바디를 텔레포트시키는 것**이라, 죽은 캐릭터를 리와인드하면
래그돌이 튈 수 있다. 지금은 사망 즉시 판정 대상에서 빠져 문제가 없다.

많은 슈터가 Physics Asset 대신 **별도 콜리전 컴포넌트 여러 개**로 히트박스를 만드는데,
이유가 정확히 이거다. **판정과 물리를 분리**하려는 것이다.
지금 규모에서는 Physics Asset 재사용이 이득이 크지만, 한계는 여기에 있다.

---

## 다음 편

인프라는 깔렸다. 그런데 아직 **가장 큰 문제**가 남아 있다.

> 클라이언트가 "머리를 맞혔다"고 확신한 그 순간,
> **서버 화면에서 그 머리는 이미 다른 곳에 있다.**

→ [3-2. 서버 사이드 리와인드](/devlog/EP_NetPrediction-2)
