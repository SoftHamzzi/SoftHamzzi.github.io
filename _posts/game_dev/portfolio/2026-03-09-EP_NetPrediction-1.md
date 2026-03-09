---
title:  "[UE5] 추출 슈터 3-1. 본 단위 히트박스"
excerpt: "BoneHitbox"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-03-09
last_modified_at: 2025-03-09
---

🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

## 개요

**이 포스팅에서 다루는 것:**
- 캡슐 콜리전 기반 히트 판정의 구조적 한계
- Physics Asset으로 본별 히트박스를 구성하는 방법
- 전용 트레이스 채널과 스냅샷 구조체 설계

**왜 이렇게 구현했는가 (설계 의도):**
- FPS는 본 단위 대미지 배율이 핵심 전투 요소 — 캡슐로는 구현 불가능
- 본 단위 히트박스는 나중에 GAS 어빌리티로 판정 방식이 바뀌어도 인프라(Physics Asset + TraceChannel)는 그대로 재사용
- TraceChannel 분리로 환경(지형, 벽)과 히트박스 판정을 완전히 격리

---

## 구현 내용

### 1. 단일 캡슐 방식의 구조적 한계

![CapsuleComponent.png](https://github.com/user-attachments/assets/d0d19f2c-cf64-40f9-939d-0045b09924db)
- 단일 캡슐

![PhysicsAsset.png](https://github.com/user-attachments/assets/6b755951-2d9a-4bc4-a7f6-588d85e17a49)
- 피직스 에셋 내부의 Primitive

- Physics Asset은 각 본에 독립 바디 → `FHitResult.BoneName`에 어느 본을 맞았는지 기록

### 2. 에디터 설정 — Physics Asset

설정할 본 목록과 바디 형태:

| 본 이름 | 바디 형태 | 비고 |
|---|---|---|
| head | Sphere | 헤드샷 배율 적용 (약점 PhysicalMaterial) |
| neck_01 | Capsule |  |
| pelvis | Capsule | 하체 중심 |
| spine_04 / spine_02 | Capsule | 상체 |
| clavicle_l/r | Capsule | 어깨 |
| upperarm_l/r | Capsule |  |
| lowerarm_l/r | Capsule |  |
| hand_l/r | Capsule | 손 |
| thigh_l/r | Capsule | 허벅지 |
| calf_l/r | Capsule |  |
| foot_l/r | Capsule | 발 |

- head 본에는 약점 판정을 위해, PhysicalMaterial을 오버라이드하였다.
- 대미지 계산식은 현재 `BaseDamage(총기 기본 대미지) * BoneMultiplier(본 별 배율) * MaterialMultiplier(PM별 배율)`로 되어있다.
- 현재는 본 별 배율도 있지만, 추후 GAS + PM만 사용할 예정이다.

### 3. 스냅샷 구조체 설계

`Public/Types/EPTypes.h`에 추가:

```cpp
// 본 하나의 월드 Transform 스냅샷
USTRUCT()
struct FEPBoneSnapshot
{
  GENERATED_BODY()
  UPROPERTY() FName BoneName;
  UPROPERTY() FTransform WorldTransform; // GetUnrealWorldTransform() 기록값
};

// 캐릭터 전체 히트박스 스냅샷 (서버 특정 시각 기준)
USTRUCT()
struct FEPHitboxSnapshot
{
  GENERATED_BODY()
  UPROPERTY() float ServerTime = 0.f;
  UPROPERTY() FVector Location = FVector::ZeroVector; // Broad Phase용 루트 위치
  UPROPERTY() TArray<FEPBoneSnapshot> Bones; // Narrow Phase 리와인드용
};

// Physics Asset 바디 채널
static constexpr ECollisionChannel EP_TraceChannel_Weapon = ECC_GameTraceChannel1;
```

**Location을 함께 저장하는 이유:**
- Broad Phase(후보 선정)는 루트 위치만으로 충분하다.
  - Bone Transform 계산은 생략
- Narrow Phase 직전에만 본 Transform으로 리와인드한다.

### 5. VisibilityBasedAnimTickOption 설정

```cpp
// AEPCharacter 생성자
GetMesh()->VisibilityBasedAnimTickOption =
  EVisibilityBasedAnimTickOption::AlwaysTickPoseAndRefreshBones;
```

**왜 필요한가:**
- `ACharacter::ACharacter(const FObjectInitializer& ObjectInitializer)`에서 AlwaysTickPose로 초기화한다.
  - 애니메이션 틱만 실행하고, 본 업데이트는 하지 않는다.
- 그러므로 이 옵션이 없으면 사망 시, 서버에서 본 Transform이 갱신되지 않는다.  
→ `GetBodyInstance(BoneName)->GetUnrealWorldTransform()`이 사망 직전 포즈로 고정됨

---

## 4. 한계 및 향후 개선

- 손가락, 발가락 등 세밀한 본은 배제한다.
  - 퍼포먼스 대비 판정 기여도가 낮음
- Physics Asset 바디 수가 많을수록 ConfirmHitscan 비용 상승한다.
  - 최소한의 본만 설정하거나, CapsuleComponent를 여러개 두는 방식으로 수정한다.

## 5. 다음 편 예고
→  서버 사이드 리와인드