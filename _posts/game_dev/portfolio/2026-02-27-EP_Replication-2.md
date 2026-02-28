---
title:  "[UE5] 추출 슈터 2-2. MetaHuman을 AEPCharacter에 통합"
excerpt: "MetaHuman"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-02-27
last_modified_at: 2025-02-27
---

📌 EmploymentProj의 GameplayFramework에 대해 알아보는 포스트  
🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

## 개요

**이 포스팅에서 다루는 것:**
- MetaHuman Creator에서 만든 캐릭터를 ACharacter C++ 클래스(EPCharacter)에 통합하는 방법
- 기본 마네킹 대신 MetaHuman을 쓸 때 생기는 구조적 차이 설명

**왜 이렇게 구현했는가 (설계 의도):**
1. **현실적인 에셋 선택**: 원래 서브컬쳐 스타일 게임이 목표였지만, 포트폴리오 프로젝트라는 여건 속에서 직접 제작 에셋 없이 빠르게 구현 가능한 MetaHuman을 선택. 실제로 MetaHuman은 상용 타이틀에서도 많이 활용되므로, 이 기회에 써보는 것 자체가 목적.

2. **캐릭터 구조 학습**: MetaHuman이 Body/Face/Outfit/Groom을 어떻게 나눠 관리하는지 뜯어보면서 UE5 캐릭터 컴포넌트 구조를 실제로 이해하는 것이 목적.

3. **향후 확장 가능성**: 의상 교체(Modular Character), 커스터마이징 등 MetaHuman 에코시스템을 나중에 활용할 수 있는 기반.

**기술적으로 해결한 것:**
- MetaHuman은 AActor 기반 BP로 제공 → ACharacter에 통합하려면 구조 분석 + 재구성 필요
- LeaderPoseComponent로 Face/Outfit이 Body 포즈를 따라가도록 처리

---

## 구현 전 상태 (Before)

- 기존 캐릭터: 기본으로 제공하는 TutorialTPP 스켈레탈 메시
- ACharacter의 기본 `GetMesh()` 하나로만 캐릭터 표현
- MetaHuman은 Body/Face/Outfit/Groom/... 이 분리된 멀티 메시 구조

**왜 단순 메시 교체만으로는 안 되는가:**
- MetaHuman Face는 별도 SkeletalMeshComponent로 분리되어 있음
- Face가 Body 본을 따라가지 않으면 목/머리 연결이 끊어짐

---

## 구현 내용

### 1. MetaHuman BP 구조 분석 및 적용

![MetaHuman.png](https://github.com/user-attachments/assets/2de84845-fd5f-47ca-8ff3-dc068627df10)

MetaHuman Creator 임포트 후 생성되는 BP

- Body, Face, SkeletalMesh
  - 각각 팔/다리, 머리, 옷 스켈레탈 메시를 담당한다.

- Face 하위 컴포넌트
  - Groom 컴포넌트로 수염, 눈썹, 머리카락 등등을 담당한다.

- [LODSync](https://dev.epicgames.com/documentation/ko-kr/metahuman/lodsync-component-for-unreal-engine)
  - 모든 메타 휴먼 컴포넌트의 LOD를 관리하여, LOD 간에 부드럽게 트랜지션 되도록 한다.

- [MetaHuman 컴포넌트](https://dev.epicgames.com/documentation/ko-kr/metahuman/the-metahuman-component-for-unreal-engine)
  - 각 LOD 레벨에 대해 애니메이션 기능을 토글할 수 있다.

![EPCharacter.png](https://github.com/user-attachments/assets/dda709d9-0e48-48f6-90d9-239314f53ff3)

- MetaHuman BP와 동일한 구조를 적용했다.
  - 기존 스켈레탈 메시는 Body를 담당하게 하였다.
  - 하위에 Face, Outfit 스켈레탈 메시를 추가하여 할당했다.
  - Groom 컴포넌트는 이미 할당된 값들이 있기에, 복사 붙여넣기하였다.

### 2. LeaderPoseComponent 패턴

```cpp
// EPCharacter.cpp 생성자
FaceMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Face"));
FaceMesh->SetupAttachment(GetMesh());
FaceMesh->SetLeaderPoseComponent(GetMesh());  // Body 본 포즈를 따라감

OutfitMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Outfit"));
OutfitMesh->SetupAttachment(GetMesh());
OutfitMesh->SetLeaderPoseComponent(GetMesh());
```

- Face/Outfit은 자체 AnimInstance를 실행하지 않는다.  
→ CPU 절약
- Body(GetMesh())에 붙은 AnimBP 하나가 전체를 구동한다.

### 3. FPS 시점 처리

- 자신의 Face에 카메라가 가려지는 상황이 발생하지 않아야 함.

- `FaceMesh에 bOwnerNoSee = true;`를 설정하였음.

### 4. 사망 처리 — 셀프 래그돌

별도 Corpse 액터를 스폰하지 않고, 캐릭터 자체를 래그돌 처리:

```cpp
void AEPCharacter::Multicast_Die_Implementation()
{
    GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
    GetMesh()->SetCollisionProfileName(TEXT("Ragdoll"));
    GetMesh()->SetSimulatePhysics(true);
}
```

**Corpse 액터 대신 셀프 래그돌을 선택한 이유:**
- 배틀그라운드와 같이 루트 상자를 스폰시키는 것이 이닌, 타르코프와 동일한 형태를 쓰고자 했음
- 별도 Actor 스폰 + 메타휴먼의 메시를 복사하는 작업이 네트워크 복제 복잡도 증가
- 앞서 적용한 LeaderPoseComponent로 인해, Body 하나만 래그돌 처리해도 Face/Outfit 모두 따라감

### 5. 다음 편 예고
→ 아이템 시스템