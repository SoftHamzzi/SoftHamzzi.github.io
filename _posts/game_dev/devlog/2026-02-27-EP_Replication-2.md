---
title:  "[UE5] 추출 슈터 2-2. MetaHuman을 ACharacter에 이식하기"
excerpt: "표정을 포기하고 얻은 것: LeaderPoseComponent의 거래"

categories:
  - DevLog
tags:
  - [UE5, C++, Animation, MetaHuman]

toc: true
toc_sticky: true

mermaid: true

date: 2026-02-27
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 2단계 Replication** 두 번째 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 2-1. CMC 확장](/devlog/EP_Replication-1)
{: .notice--info}

## 문제

MetaHuman은 **`AActor` 기반 블루프린트**로 온다.
내 캐릭터는 **`ACharacter` 기반 C++ 클래스**이다.

이 둘은 상속 계보가 달라서 "MetaHuman BP를 상속받아 쓰기"가 안 된다.
캡슐 콜리전, `UEPCharacterMovement`([2-1편](/devlog/EP_Replication-1)),
`GetMesh()`를 전제로 도는 애님BP, 이게 전부 `ACharacter` 쪽에 있다.

그래서 **MetaHuman BP를 뜯어보고, 필요한 것만 `AEPCharacter`에 재구성**했다.

> 왜 MetaHuman인가는 짧게만 적겠다.
> 원래는 서브컬쳐풍 캐릭터가 목표였지만 직접 만들 시간이 없었고,
> MetaHuman은 상용 타이틀에서도 쓰이니 이 기회에 구조를 뜯어보자는 판단이었다.
> 이 글에서 다룰 가치가 있는 건 **에셋 선택이 아니라 그 다음에 내린 결정들**이다.

---

## MetaHuman BP는 무엇으로 되어 있나

![MetaHuman.png](https://github.com/user-attachments/assets/2de84845-fd5f-47ca-8ff3-dc068627df10)

| 컴포넌트 | 담당 |
|---|---|
| Body (SkeletalMesh) | 몸통·팔·다리 |
| Face (SkeletalMesh) | 머리. 얼굴 리그 본 수백 개 |
| SkeletalMesh (Outfit) | 의상 |
| Groom × N | 머리카락·눈썹·수염. **Face의 자식** |
| [LODSync](https://dev.epicgames.com/documentation/ko-kr/metahuman/lodsync-component-for-unreal-engine) | 모든 메시의 LOD를 동기 전환 |
| [MetaHuman 컴포넌트](https://dev.epicgames.com/documentation/ko-kr/metahuman/the-metahuman-component-for-unreal-engine) | LOD별 애니메이션 기능 토글 |

### 옮긴 것과 안 옮긴 것

![EPCharacter.png](https://github.com/user-attachments/assets/dda709d9-0e48-48f6-90d9-239314f53ff3)

| | 처리 | 이유 |
|---|---|---|
| Body | `ACharacter::GetMesh()`가 담당 | 캡슐·CMC·애님BP가 전부 `GetMesh()` 전제 |
| Face | `FaceMesh` 추가, `GetMesh()`에 부착 | - |
| Outfit | `OutfitMesh` 추가, `GetMesh()`에 부착 | - |
| Groom | BP에서 복사 붙여넣기 | 설정값이 이미 잡혀 있어 재현할 이유 없음 |
| LODSync | **안 옮김** | LOD 전환이 눈에 걸릴 만한 거리에서 싸울 게임이 아님. 필요해지면 컴포넌트만 추가하면 됨 |
| MetaHuman 컴포넌트 | **안 옮김** | 아래 LeaderPose 결정으로 얼굴 애니메이션 자체를 안 씀 |

**Body를 루트 메시로 삼은 게 첫 결정**이다.
`ACharacter`의 거의 모든 기능이 `GetMesh()`를 가리키고 있어서,
Face를 루트로 삼으면 캡슐 정렬부터 애님BP 연결까지 전부 다시 짜야 했다.

---

## LeaderPoseComponent: 그리고 그 대가

```cpp
// EPCharacter.cpp 생성자
FaceMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Face"));
FaceMesh->SetupAttachment(GetMesh());
FaceMesh->SetLeaderPoseComponent(GetMesh());

OutfitMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Outfit"));
OutfitMesh->SetupAttachment(GetMesh());
OutfitMesh->SetLeaderPoseComponent(GetMesh());
```

팔로워 메시는 자기 포즈를 계산하지 않고 **리더의 본 트랜스폼을 그대로 읽는다.**
애님BP 하나(Body)가 셋을 전부 구동한다. 얼굴·의상용 AnimInstance 평가가 통째로 사라진다.

### 인자가 세 개다

```cpp
// SkinnedMeshComponent.h:1770-1778
/**
 * @param bForceUpdate               If false, the function will be skipped if
 *                                   NewLeaderBoneComponent is the same as currently setup
 * @param bInFollowerShouldTickPose  If false, Follower components will not execute TickPose
 */
void SetLeaderPoseComponent(USkinnedMeshComponent* NewLeaderBoneComponent,
                            bool bForceUpdate = false,
                            bool bInFollowerShouldTickPose = false);
```

세 번째 인자가 *"팔로워가 자기 TickPose를 돌릴 것인가"*이다.
기본값 `false`: CPU를 아낀다는 이 글의 주장은 **정확히 이 기본값 덕분**이다.

### 대가: 표정은 못 만든다

팔로워는 **본 이름으로** 리더 본을 찾는다.

```cpp
// SkinnedMeshComponent.cpp:3228-3232  UpdateLeaderBoneMap()
const FName BoneName = FollowerRefSkeleton.GetBoneName(BoneIndex);
LeaderBoneMap[BoneIndex] = LeaderRefSkeleton.FindBoneIndex(BoneName);   // 없으면 INDEX_NONE
```

MetaHuman Face 스켈레톤의 얼굴 리그 본은 **Body 스켈레톤에 존재하지 않는다.**
전부 `INDEX_NONE`이다. 그 경우 엔진은 이렇게 처리한다.

```cpp
// SkinnedMeshComponent.cpp:2194-2212  GetBoneTransform()
// 리더에 없는 본 → 가장 가까운 '공통 조상' 본 기준의 상대(=레퍼런스 포즈) 트랜스폼
return MissingBoneInfo.RelativeTransform
     * LeaderPoseComponentInst->GetComponentSpaceTransforms()[MissingBoneInfo.CommonAncestorBoneIndex]
     * LocalToWorld;
```

**결과: 얼굴 전체가 `head`를 따라 움직이되, 표정 본들은 레퍼런스 포즈에 굳다.**
목은 붙지만 눈도 입도 절대 안 움직인다.

이건 버그가 아니라 **거래**이다.

| | LeaderPose (현재) | Face 자체 AnimInstance |
|---|---|---|
| 얼굴 애니메이션 | **불가** | 가능 |
| 매 프레임 비용 | 스킨 행렬 복사 | 얼굴 리그 본 수백 개 평가 |
| 목 연결 | 자동 | 별도 처리 |
| 래그돌 | Body 하나만 물리 → 나머지 따라옴 | 셋 다 처리 필요 |

**1인칭 추출 슈터**라서 이 거래를 받았다.
내 얼굴은 화면에 없고, 남의 얼굴은 대개 수십 미터 밖이다.
표정에 프레임을 쓸 이유가 없다.

거꾸로 말하면, **컷신이나 로비 클로즈업이 생기는 순간 이 결정을 되돌려야 한다.**
그때는 `bFollowerShouldTickPose = true`로 켜고 얼굴용 AnimBP를 붙이면 된다.
구조를 바꿀 필요는 없고, 스위치 하나이다. 그래서 지금 이 선택이 안전하다.

---

## 1인칭에서 내 머리 숨기기

카메라가 `head` 소켓에 붙어 있다. 그대로 두면 자기 얼굴 안쪽이 화면을 채운다.

```cpp
FaceMesh->bOwnerNoSee = true;
```

**그런데 이걸로 끝이 아니었다.**

`bOwnerNoSee`는 **프리미티브 컴포넌트 하나에만 적용되는 플래그**이다.
부착 계층을 타고 자식으로 내려가지 **않는다.**
Groom(머리카락·눈썹·수염)은 `FaceMesh`의 **자식**이므로 그대로 보인다.

즉 얼굴은 사라졌는데 **머리카락만 화면 앞에 떠 있는** 상태가 된다.

```cpp
// 자식까지 직접 순회해야 한다
TArray<USceneComponent*> Children;
FaceMesh->GetChildrenComponents(false, Children);
for (USceneComponent* Child : Children)
    if (auto* Prim = Cast<UPrimitiveComponent>(Child))
        Prim->bOwnerNoSee = true;
```

> 참고로 나중에 `BeginPlay`에 일괄 처리를 넣었는데, 그것도 완전하지 않다.
>
> ```cpp
> TArray<USkeletalMeshComponent*> MeshComponents;
> GetComponents<USkeletalMeshComponent>(MeshComponents);   // ← Groom은 여기 안 걸린다
> ```
>
> `UGroomComponent`는 `USkeletalMeshComponent`가 아니다.
> `UPrimitiveComponent`로 넓혀야 맞다. 아직 안 고쳤다.

---

## 사망 처리: 셀프 래그돌

```cpp
void AEPCharacter::Multicast_Die_Implementation()
{
    GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
    GetMesh()->SetCollisionProfileName(TEXT("Ragdoll"));
    GetMesh()->SetSimulatePhysics(true);
}
```

배틀그라운드식 루트 상자가 아니라 타르코프식,
**시체 자체를 뒤지는** 게임이라 캐릭터를 그대로 눕히기로 했다.

앞서 LeaderPose를 걸어둔 덕에 **Body 하나만 물리로 돌리면 Face/Outfit이 따라온다.**
이 결정이 여기서 배당을 냈다.

### 다만, 이 구현에는 구멍이 세 개 있다

**① Multicast RPC는 "그 순간 접속해 있고 릴러번트한" 클라에게만 간다.**
나중에 접속한 플레이어는 이 RPC를 못 받는다.
그 사람 화면에서는 **시체가 멀쩡히 서 있다.**
정석은 `bIsDead` 같은 복제 변수 + `OnRep`이다.
늦게 온 클라는 초기 복제로 값을 받고 같은 처리를 하니까.

**② 래그돌은 각 클라가 로컬에서 시뮬레이션한다.**
서버가 물리 결과를 보내지 않는다. **쓰러진 자세가 클라마다 다르다.**
눈요기라면 상관없지만 이 게임은 시체를 **루팅**한다.
내 화면의 시체와 서버가 아는 시체가 다르면 상호작용이 어긋난다.

**③ 시체는 캐릭터보다 오래 살아야 한다.**
리스폰이 생기면 `AEPCharacter`는 파괴되거나 재사용된다. 시체는 남아 있어야 한다.
캐릭터 자신을 시체로 쓰는 한, **시체의 수명이 캐릭터의 수명에 묶인다.**

> 🚩 이 세 가지 때문에 **이 결정은 나중에 뒤집힌다.** 아래에 적었다.

---

## 이 글 이후 바뀐 것

### 셀프 래그돌 → `AEPCorpse` 별도 액터

글을 쓸 당시 나는 *"별도 액터를 스폰하고 메타휴먼 메시를 복사하는 게
네트워크 복잡도를 올린다"*는 이유로 셀프 래그돌을 골랐다.
루팅을 붙이고 나니 위 ①②③이 전부 실제 문제로 나타났고, 결국 액터를 만들었다.

```cpp
AEPCorpse::AEPCorpse()
{
    bReplicates = true;
    CorpseMesh = CreateDefaultSubobject<USkeletalMeshComponent>("CorpseMesh");
    RootComponent = CorpseMesh;

    FaceMesh   = CreateDefaultSubobject<USkeletalMeshComponent>("FaceMesh");
    FaceMesh->SetLeaderPoseComponent(CorpseMesh);        // ← 같은 패턴을 그대로 재사용
    OutfitMesh = CreateDefaultSubobject<USkeletalMeshComponent>("OutfitMesh");
    OutfitMesh->SetLeaderPoseComponent(CorpseMesh);
}
```

그리고 "복잡하다"고 미뤘던 메시 전달을 **RPC가 아니라 복제 변수로** 했다.

```cpp
DOREPLIFETIME(AEPCorpse, CorpseMeshAsset);
DOREPLIFETIME(AEPCorpse, CorpseFaceAsset);
DOREPLIFETIME(AEPCorpse, CorpseOutfitAsset);

void AEPCorpse::OnRep_CorpseMeshAsset() { ApplyCorpseMesh(); }
```

이러면 ①이 사라진다. 늦게 접속해도 초기 복제로 에셋 참조를 받아 같은 시체를 본다.
액터가 독립적이니 ③도 사라진다.
②(로컬 물리)는 남아 있지만, 액터 위치 자체는 복제되므로 루팅 범위 판정은 서버 기준으로 맞출 수 있다.

**교훈은 "처음 판단이 틀렸다"가 아니라 "판단의 유효 범위가 있었다"이다.**
시체가 장식일 때는 셀프 래그돌이 맞았고, 시체가 **게임 오브젝트**가 되는 순간 틀려졌다.

### 그 외

| 변경 | 이유 |
|---|---|
| `FaceMesh->SetLeaderPoseComponent(GetMesh(), false, true)` | 얼굴 쪽 독자 갱신이 필요해져 팔로워 TickPose를 켬 |
| `Multicast_Die`에서 Groom 자식 명시적 숨김 | `bOwnerNoSee` 미전파 (위 참조) |
| `Multicast_Die`에 `StopMovementImmediately()` + `DisableMovement()` | 래그돌 전환 후에도 CMC가 캡슐을 계속 밀고 있었음 |
| `GetMesh()->VisibilityBasedAnimTickOption = AlwaysTickPoseAndRefreshBones` | **아래 참조** |

마지막 줄이 이 글에서 가장 멀리 가는 변경이다.

기본값은 `OnlyTickPoseWhenRendered`이다.
**데디케이티드 서버는 아무것도 렌더링하지 않는다.**
그러면 서버의 본 트랜스폼이 갱신되지 않고, 전부 레퍼런스 포즈에 머문다.

3단계에서 만든 서버 사이드 리와인드는 **매 틱 본 위치를 스냅샷으로 저장**한다.
이 옵션이 없으면 그 스냅샷이 전부 T 포즈이다.

즉 **이 글에서 만든 메시 구조가 3단계 히트 판정의 전제**이다.
→ [3-1. 본 단위 히트박스](/devlog/EP_NetPrediction-1)

---

## 다음 편
→ [2-3. 아이템 3계층 구조](/devlog/EP_Replication-3)
