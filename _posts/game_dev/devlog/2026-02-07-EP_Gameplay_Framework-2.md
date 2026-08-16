---
title:  "[UE5] 추출 슈터 1-2. Enhanced Input으로 FPS 캐릭터 구현"
excerpt: "입력을 어디에 둘 것인가, 그리고 공중 관성 조율"

categories:
  - DevLog
tags:
  - [UE5, C++, GameplayFramework]

toc: true
toc_sticky: true

mermaid: true

date: 2026-02-07
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 1단계 Gameplay Framework** 두 번째 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 1-1. 아키텍처 설계](/devlog/EP_Gameplay_Framework-1)
{: .notice--info}

## 개요

Enhanced Input으로 이동 / 시점 / 점프 / 달리기를 붙인다.

기능 자체는 어렵지 않다. 이 글에서 실제로 판단한 건 두 가지이다.

1. **InputAction 에셋을 누가 소유할 것인가**: PlayerController냐 Pawn이냐
2. **공중에서 관성을 얼마나 남길 것인가**: 추출 슈터의 이동 감각

---

## 입력 구조: InputAction을 PlayerController에 뒀다

```cpp
// EPPlayerController.h
UPROPERTY(EditDefaultsOnly, Category = "Input")
TObjectPtr<UInputMappingContext> DefaultMappingContext;

UPROPERTY(EditDefaultsOnly, Category = "Input")
TObjectPtr<UInputAction> MoveAction;
// Look / Jump / Sprint / ADS / Crouch / Fire / Reload ...
```

```cpp
// EPCharacter.cpp: 캐릭터는 PC에게서 에셋을 빌려 바인딩만 한다
AEPPlayerController* PC = GetController<AEPPlayerController>();
EnhancedInput->BindAction(PC->GetMoveAction(), ETriggerEvent::Triggered,
                          this, &AEPCharacter::Input_Move);
```

MappingContext 등록은 `AEPPlayerController::BeginPlay()`에서
`UEnhancedInputLocalPlayerSubsystem`에 한다.

### 이건 정답이 하나가 아닌 선택이다

Lyra는 반대로 한다. `ULyraInputConfig` DataAsset을 **Pawn 쪽 컴포넌트**가 들고 있다.
그래서 어느 쪽이 맞는지 따져볼 가치가 있다.

| | PlayerController 소유 (이 프로젝트) | Pawn/Component 소유 (Lyra) |
|---|---|---|
| 캐릭터가 죽고 다시 스폰 | 에셋 참조 유지 | 새 Pawn마다 다시 지정 |
| 캐릭터마다 다른 입력 | **어렵다**. PC를 갈아야 함 | 쉽다 |
| 키 리바인딩 UI | PC 한 곳만 보면 됨 | Pawn 순회 필요 |
| 관전 모드 전환 | 자연스러움 | Pawn 소멸 시 정리 필요 |

**PlayerController를 고른 이유:** 이 게임은 플레이어가 조종하는 클래스가 **하나**이다.
장비만 바뀌지 Pawn 종류가 바뀌지 않는다. 그러면 입력 정의를 캐릭터마다 들고 다닐 이유가 없고,
죽고 리스폰해도 입력 에셋 참조가 그대로 남는 쪽이 단순하다.

**되돌려야 할 조건:** 탈것이 생기거나, 조작 체계가 다른 Pawn(드론, 포탑)이 생기는 순간
이 결정은 뒤집어야 한다. 그때는 Lyra 방식이 맞다.
지금 굳이 그 구조를 미리 만들지는 않았다. 두 번째 Pawn이 없기 때문이다.

---

## 이동: Pitch와 Roll을 버리는 게 핵심

```cpp
void AEPCharacter::Input_Move(const FInputActionValue& Value)
{
    const FVector2D Input = Value.Get<FVector2D>();

    // Yaw만 남기고 Pitch/Roll을 버린다 ← 이 줄이 전부다
    const FRotator YawRotation(0.0, Controller->GetControlRotation().Yaw, 0.0);

    const FVector ForwardDir = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
    const FVector RightDir   = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

    AddMovementInput(ForwardDir, Input.X);
    AddMovementInput(RightDir,   Input.Y);
}
```

`FRotationMatrix(...).GetUnitAxis(EAxis::X)`는 각도를 계산하는 게 아니라,
**회전 행렬을 만들어서 그 기저축(앞 방향)을 꺼내는** 코드이다.
각도는 이미 `ControlRotation.Yaw`로 주어져 있고, 여기서 하는 일은 **각도 → 방향 벡터** 변환이다.

진짜 중요한 건 `FRotator(0.0, Yaw, 0.0)`의 **0 두 개**이다.

| | Pitch를 남기면 | Pitch를 버리면 |
|---|---|---|
| 하늘을 보고 W | 위로 걸어가려 함 (`ForwardDir.Z != 0`) | **지면 평면 위에서만 전진** |
| 땅을 보고 W | 바닥으로 파고들려 함 | 동일하게 전진 |

FPS 이동의 기본 규칙이고, 이 한 줄이 그걸 만든다.

### `AddMovementInput`은 컨트롤러를 보지 않는다

```cpp
AddMovementInput(ForwardDir, Input.X);
```

이 함수는 `ControlInputVector`에 `Dir * Scale`을 **누적**할 뿐이다.
컨트롤러 기준이 되는 건 **바로 위에서 우리가 `ControlRotation`으로 `Dir`을 만들었기 때문**이지,
함수가 컨트롤러에 묶여 있어서가 아니다.

덕분에 나중에 AI가 같은 캐릭터를 조종할 때 월드 방향을 그대로 넣으면 된다.

---

## 시점: 왜 RPC가 필요 없는가

```cpp
void AEPCharacter::Input_Look(const FInputActionValue& Value)
{
    const FVector2D Input = Value.Get<FVector2D>();
    AddControllerYawInput(Input.X * Sensitivity);
    AddControllerPitchInput(Input.Y * Sensitivity);
}
```

두 줄이지만 네트워크 관점에서 짚어둘 게 있다.

시점 변경에는 **Server RPC를 쓰지 않는다.** 그럴 필요가 없다.
`ControlRotation`은 클라 로컬에서 즉시 반영되고,
서버에는 **이동 패킷에 실려서** 간다. `FSavedMove_Character`가
`SavedControlRotation`을 필드로 들고 있기 때문이다.

> 이 아이디어가 다음 단계의 전부이다.
> **"이동 패킷에 이미 자리가 있는데 왜 별도 RPC를 만드나."**
> [2-1편](/devlog/EP_Replication-1)에서 Sprint를 정확히 이 논리로 옮긴다.

---

## 점프: 오토 버니합을 켜두고 관성만 깎았다

### 트리거 선택

```cpp
EnhancedInput->BindAction(
    PC->GetJumpAction(),
    ETriggerEvent::Triggered, this,   // Started가 아니라 Triggered
    &AEPCharacter::Input_Jump
);
```

| | `Started` | `Triggered` |
|---|---|---|
| 발화 시점 | 눌리는 순간의 엣지(0→1) **1회** | 눌려 있는 동안 **매 프레임** |
| 점프 동작 | 한 번 누르면 한 번 점프 | 누르고 있으면 착지 즉시 재점프 |

`Triggered`를 골랐다. 매 프레임 `Jump()`가 호출되면 `bPressedJump`가 계속 참이라
착지하자마자 다시 뛴다. **오토 버니합이다.**

**의도한 건다.** 추출 슈터에서 이동은 생존 수단이고,
연타로 손가락을 혹사시키는 조작은 실력이 아니라 부담이다.
"누르고 있으면 계속 뛴다"가 이 게임의 규칙이다.

### 공중 관성: 이게 실제로 조율한 부분

문제는 다른 데서 나왔다. **앞으로 한 번 입력하고 손을 떼도 속도가 그대로 유지**됐다.
공중에서 무한 활강이 되는 셈이다.

```cpp
UEPCharacterMovement* Movement = Cast<UEPCharacterMovement>(GetCharacterMovement());
Movement->JumpZVelocity              = 420.f;
Movement->AirControl                 = 0.5f;
Movement->BrakingDecelerationFalling = 700.f;   // ← 이 값
// FallingLateralFriction 은 기본값이 0.f: 공중 마찰은 없다
```

엔진이 이렇게 처리한다.

```cpp
// UCharacterMovementComponent::PhysFalling
const float MaxDecel = GetMaxBrakingDeceleration();   // MOVE_Falling → BrakingDecelerationFalling
CalcVelocity(timeTick, FallingLateralFriction, false, MaxDecel);
```

그리고 `CalcVelocity`는 **가속도가 0일 때만** 브레이킹을 건다.
즉 `BrakingDecelerationFalling`은 *공중에서 입력을 놓았을 때만* 속도를 깎는다.
입력을 유지하면 그대로 날아간다.

정확히 원하던 동작이었다. **조작하면 관성이 살고, 놓으면 죽는다.**

`FallingLateralFriction`이 기본값 0이라는 게 전제이다. 여기에 값을 주면
입력 중에도 감속이 걸려서 공중 조작이 둔해진다. 그래서 건드리지 않았다.

### 값을 이렇게 정한 근거

| 값 | 무엇을 보고 정했나 |
|---|---|
| `JumpZVelocity = 420` | 기본 허리 높이(약 90cm) 장애물을 걸리지 않고 넘는 최소치 |
| `AirControl = 0.5` | 0.1은 공중에서 조작이 거의 안 먹혀 답답. 1.0은 공중 방향 전환이 자유로워 총격전이 우스워짐 |
| `BrakingDecelerationFalling = 700` | 입력을 놓고 약 0.5초 내에 수평 속도가 사라지는 값 |
| DeadZone (InputAction 모디파이어) | 게임패드 스틱 드리프트 차단, 놓아도 0.02가 남아 캐릭터가 미끄러졌음 |

> DeadZone은 **입력 값**을 자르는 모디파이어이다. 캐릭터 속도와는 무관하다.
> 입력 파이프라인(`하드웨어 축 → Modifier → Trigger → Value`)의 앞쪽에 있고,
> 속도는 그 결과물이라 볼 수도 없다.

---

## 달리기: 그리고 여기 다음 편의 씨앗이 있다

```cpp
EnhancedInput->BindAction(PC->GetSprintAction(), ETriggerEvent::Triggered, this,
                          &AEPCharacter::Input_StartSprint);
EnhancedInput->BindAction(PC->GetSprintAction(), ETriggerEvent::Completed, this,
                          &AEPCharacter::Input_StopSprint);
```

`Triggered`는 **누르고 있는 동안 매 프레임** 발화한다.

이 시점의 `Input_StartSprint` 구현은 이랬다.

```cpp
void AEPCharacter::Input_StartSprint(const FInputActionValue& Value)
{
    Server_SetSprinting(true);      // UFUNCTION(Server, Reliable)
}
```

> 🚨 **쉬프트를 누르고 있는 동안, 매 프레임 Reliable Server RPC를 쐈다는 뜻이다.**
> 60fps면 초당 60회. Reliable이라 ACK와 재전송 부담까지 붙는다.
>
> 이걸 [2-1편](/devlog/EP_Replication-1)에서 CMC `CompressedFlags`로 옮긴다.
> 옮긴 이유는 흔히 말하는 "체감 지연" 때문만이 아니라, **여기서 만든 이 구조 때문**이다.
>
> (트리거만 `Started`/`Completed` 쌍으로 바꿔도 RPC 횟수는 줄어든다.
> 하지만 그건 증상 완화지 해결이 아니다. 이동 상태를 이동 패킷 밖으로 보내는 구조 자체가 문제였다.)

---

## 결과

![movement.gif](https://github.com/user-attachments/assets/2a366694-bad2-47f5-bc7d-b4d74fae6517)

- WASD 이동 / 마우스 시점 / 오토 버니합 / 쉬프트 달리기 동작 확인
- 공중에서 입력을 놓으면 약 0.5초에 걸쳐 수평 속도가 사라짐

---

## 이 글 이후 바뀐 것

- `AirControl` 0.1 → **0.5** (공중 조작이 너무 둔했음)
- Sprint가 Server RPC → **CMC `CompressedFlags`** ([2-1편](/devlog/EP_Replication-1))
- ADS / Crouch / Fire / Reload는 이후 `ETriggerEvent::Started`로 바인딩

---

## 다음 편
→ [1-3. 서버 권한형 매치 흐름과 상태 복제](/devlog/EP_Gameplay_Framework-3)
