---
title:  "[UE5] 추출 슈터 2-6. 애니메이션 시스템"
excerpt: "CombatComponent"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-03-01
last_modified_at: 2025-03-01
---

🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

## 개요

**이 포스팅에서 다루는 것:**
- Lyra 애니메이션을 MetaHuman 스켈레톤에 리타게팅하는 방법
- Lyra 스타일 Linked Anim Layer 구조로 무기별 애니메이션 교체
- FABRIK 왼손 IK + 크로스헤어 HUD

**왜 이 구조인가:**
- Lyra: Epic 공식 샘플, 라이선스 문제 없음, 퀄리티 높은 Rifle/Pistol/Shotgun/UnArmed 애니메이션 보유
- Linked Anim Layer: 무기 추가 시 새 무기 AnimBP만 만들면 됨. 메인 AnimBP 수정 불필요

---

## 구현 내용

### 1. Lyra 애니메이션 리타게팅

**IK Retargeter 2단계:**

1. 메타휴먼 IK Rig 생성

![IKRig.png](https://github.com/user-attachments/assets/86f3c77b-c57a-4719-9968-3240afd5745b)

- 메타휴먼의 스켈레탈 메시를 통해 IK Rig를 생성했다.
- 이후, 리타깃 체인을 자동 생성해주었다.

2. IK Retargeter

![Retargeter.png](https://github.com/user-attachments/assets/a68a6401-5cf1-4a02-a684-3a97f63fef76)

- 기존 Lyra 프로젝트에 존재하던, RTG_Mannequin을 사용했다.
- 해당 리타기터를 통해, 애니메이션 시퀀스들을 메타휴먼에 맞게 익스포트를 진행하였다.

**BlendSpace / AimOffset**
- AimOffset
  - AimOffset의 경우는 리타기터에서 뜨지 않았다.
  - `에셋 우클릭 → 리타깃 애니메이션` 을 통해 리타깃을 진행하였다.
- BlendSpace
![BS_RifleIdleJog.png](https://github.com/user-attachments/assets/2238def7-3878-4db0-9647-b214711a2c6f)
  - 현재로서는 간단하게 구현하기 위해 블렌드 스페이스를 이용하였다.
  - 이후, Lyra와 동일하게 진행할 것이다.

---

### 2. Linked Anim Layer 전체 구조

```
┌───────────────────────┐
│ ABP_EPCharacter (메인 AnimBP)                │
│  Parent C++: UEPAnimInstance                 │
│  Implements: ALI_EPWeapon                    │
│                                              │
│ ┌─ Locomotion StateMachine ─────┐    │
│ │  IdleWalkJog ←→ Crouch             │   │
│ │       ↕ JumpSources / Jump          │   │
│ └───────────────────┘   │
│                                              │
│ ┌─ Linked Anim Layer 슬롯 (빈 슬롯) ─┐   │
│ │  FullBody_IdleWalkRun  ← 빈 슬롯    │   │
│ │  FullBody_Crouch       ← 빈 슬롯    │   │
│ │  FullBody_Jump         ← 빈 슬롯    │   │
│ └───────────────────┘   │
│                                              │
│  AimOffset: AimPitch / AimYaw                │
│  AdditiveHitReact 슬롯 (HitReact 몽타주)     │
└───────────────────────┘
              ↑ LinkAnimClassLayers()
┌───────────────────────┐
│ ABP_EPWeaponAnimLayersBase                   │
│  Parent C++: UEPWeaponAnimInstance           │
│  Implements: ALI_EPWeapon                    │
│                                              │
│  각 레이어 함수에 Sequence/BS Player 노드    │
│  → 임시적으로 각 레이어에 맞는              │
│    애니메이션 시퀀스 변수 할당               │
└───────────────────────┘
              ↑ 상속
┌───────────────────────┐
│ ABP_RifleAnimLayers (무기 AnimBP)            │
│                                              │
│  AnimGraphOverrides (에셋 오버라이드 에디터) │
│  FullBody_IdleWalkRun → BS_IdleWalkJog      │
│  FullBody_Crouch      → Anim_Crouch         │
│  FullBody_Jump        → Anim_Jump           │
└───────────────────────┘
```

**3계층 설계의 핵심:**
- `ABP_EPCharacter`: 슬롯만 정의, 에셋 모름
- `ABP_EPWeaponAnimLayersBase`: 레이어 구현 구조 정의
- `ABP_RifleAnimLayers`: 실제 에셋만 꽂음
  - 새 무기 추가 시 이 파일만 새로 만들면 됨

**무기 교체 한줄로 가능:**
```cpp
// OnRep_EquippedWeapon 함수
Owner->GetMesh()->LinkAnimClassLayers(NewWeapon->WeaponDef->WeaponAnimLayer);
```

---

### 3. Locomotion 상태 머신

![Locomotion.png](https://github.com/user-attachments/assets/eeb70a47-a854-43e8-9cf8-24dc211c040c)

상태 4개:

| State | 호출 레이어 | 전환 조건 |
|-------|------------|----------|
| `IdleWalkJog` | `FullBody_IdleWalkRun` | 기본 상태 |
| `Crouch` | `FullBody_Crouch` | bIsCrouching |
| `JumpSources` | — | 점프 입력 |
| `Jump` | `FullBody_Jump` | 공중 상태 |

- 레이어 내부 BlendSpace가 Speed로 Idle→Walk→Jog 블렌딩

---

### 4. AnimGraph 노드 흐름

![AnimGraph.png](https://github.com/user-attachments/assets/1f4df2a2-d0bc-4de2-8a2d-33bda70b1f3f)

```
[Locomotion State Machine]
    ↓
[Cache Pose: "Locomotion"]
    ↓
[Layered Blend Per Bone: spine_01]  ← 상체에만 AimOffset 적용
  Base: Cached Locomotion
  Blend 0: AimOffset (AimPitch / AimYaw)
    ↓
[FABRIK: 왼손 IK]
    ↓
[AdditiveHitReact 슬롯]       ← 피격 몽타주 (임시)
    ↓
[Output Pose]
```

- 현재는 임시적으로 배치

---

### 5. Property Access (Thread-Safe)

- ABP_RifleAnimLayers AnimGraph:
  - Get Main Anim Blueprint Thread Safe
  → (ABP_EPCharacter로 캐스트)  
  → .Speed     → BlendSpace X축  
  → .Direction → BlendSpace Y축  

- 중복 변수 선언 없이 워커 스레드 안전하게 값 복사 가능

---

### 6. FABRIK 왼손 IK

- 현재는 왼손만 FABRIK으로 무기 소켓 위치로 보정

```cpp
NativeUpdateAnimation():
  ...
  FTransform WorldLeftHandIK = WeaponMesh->GetSocketTransform(FName("LeftHandIK"));
  FTransform HandR_World = Character->GetMesh()->GetBoneTransform(FName("hand_r"));

  LeftHandIKTransform = WorldLeftHandIK.GetRelativeTransform(HandR_World);
  ...
```

![FABRIK_Detail.png](https://github.com/user-attachments/assets/9439b914-234d-446f-b00c-2919390ed8a3)

- 팁 본은 폴리싱 단계에서 수정할 예정이다.

---

### 7. 크로스헤어 HUD

```cpp
// EPPlayerController.cpp — IsLocalController() 체크 필수
// PlayerController는 서버에도 존재 → 서버에서 HUD 생성 방지
if (IsLocalController())
{
    CrosshairWidget = CreateWidget<UEPCrosshairWidget>(this, CrosshairWidgetClass);
    CrosshairWidget->AddToViewport();
}
```

![Crosshair.png](https://github.com/user-attachments/assets/72b8746b-bca0-4283-8652-ce622cd1a5c9)

- 손이 총 안으로 파고들어가 있는 이유:
  - 팁 본이 왼손 중지로 설정됨
  - 무기별로 오프셋 값이 설정되어야 하지만, 아직 없음
  - 현재는 당장 수정할 필요가 없음

---

## 결과

- 이동 방향에 맞는 애니메이션
- 점프 애니메이션
- 웅크리기 애니메이션
- 상하 시야에 따라 상체(팔/총)만 AimOffset 적용
- 무기 장착 시 왼손이 LeftHandIK 소켓으로 이동
- 화면 중앙 크로스헤어 표시