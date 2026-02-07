---
title:  "[UE5] 추출 슈터 1-2. Enhanced Input으로 FPS 캐릭터 구현"
excerpt: "Enhanced Input Step 2"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-02-07
last_modified_at: 2025-02-07
---

📌 EmploymentProj의 GameplayFramework에 대해 알아보는 포스트  
🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

## 개요
- Enhanced Input 시스템 소개
- 구현 목표 (이동, 시점, 점프, 달리기)

## 입력 구조 설계
- InputAction을 PlayerController에 둔 이유
  - Move, Look, Jump, Sprint와 같은 입력은 모든 캐릭터가 동일하다.
  - 모든 캐릭터가 PlayerController를 가지도록 하여, 기본적인 동작을 공유하도록 한다.
- MappingContext 등록 시점
  - AEPPlayerController.BeginPlay()에서 UEnhancedInputLocalPlayerSubsystem에 MappingContext를 등록한다.

## 캐릭터 이동
- Input_Move 구현
  - 현재 PlayerController의 Yaw를 가져온다.
  - 현재 기준에서 X/Y 축으로의 각도를 선형 대수 함수를 통해 계산한다.
```cpp
const FRotator YawRotation(0.0, Controller->GetControlRotation().Yaw, 0.0);
const FVector ForwardDir = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
const FVector RightDir = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
```
- AddMovementInput 작동 방식
  - 컨트롤러를 기준으로하여, 움직이도록 함수를 호출한다.
```cpp
AddMovementInput(ForwardDir, Input.X);
AddMovementInput(RightDir, Input.Y);
```


## 시점 & 점프
- Input_Look
  - 입력으로 들어오는 Vector2의 X/Y를 더해주는 방식으로 시점을 전환한다.
```cpp
AddControllerYawInput(Input.X * Sensitivity);
AddControllerPitchInput(Input.Y * Sensitivity);
```
- ETriggerEvent 차이 (Started vs Triggered)
  - 꾹 누르면 점프를 계속 하게 하도록 하였다.
  - Started는 점프 버튼을 눌렀을때의 엣지(0→1)를 관찰한다.
  - Triggered는 Input Level이 1이라면 지속적으로 이벤트를 발생시킨다.

```cpp
EnhancedInput->BindAction(
  PC->GetJumpAction(),
  ETriggerEvent::Triggered, this,
  &AEPCharacter::Input_Jump
);
```

- 버니합 방지
  - 점프를 지속적으로 누르는 경우, 앞으로 한번만 이동해도  
  속도가 줄어들지 않았다.
  - 움직이지 않을때 속도를 감쇠시키도록 하기 위해, `BrakingDecelerationFalling` 변수를 사용하였다.
  - 또한 `InputAction`의 DeadZone을 조절하여, 일정 속도 이하인 경우 움직이지 않도록 하였다. 
```cpp
UCharacterMovementComponent* Movement = GetCharacterMovement();
Movement->JumpZVelocity = 420.f;
Movement->AirControl = 0.1f;

Movement->BrakingDecelerationFalling = 700.f;
// Movement->FallingLateralFriction = 0f; // 공중 마찰력
```

## 달리기
- MaxWalkSpeed 전환 방식
  - 달리기를 추가하여, 쉬프트 누름 여부에 따라 MaxWalkSpeed가 변경되도록 하였다.

```cpp
EnhancedInput->BindAction(
  PC->GetSprintAction(),
  ETriggerEvent::Triggered, this,
  &AEPCharacter::Input_StartSprint
);
EnhancedInput->BindAction(
  PC->GetSprintAction(),
  ETriggerEvent::Completed, this,
  &AEPCharacter::Input_StopSprint
);
```

## 결과
![movement.gif](https://github.com/user-attachments/assets/2a366694-bad2-47f5-bc7d-b4d74fae6517)

## 다음 편 예고
→ 매치 흐름 & 복제 시스템
