---
title:  "[UE5] 추출 슈터 1-4. 스폰 시스템과 DataAsset 설계"
excerpt: "UPrimaryDataAsset을 고른 진짜 이유는 메모리가 아니다"

categories:
  - DevLog
tags:
  - [UE5, C++, GameplayFramework]

toc: true
toc_sticky: true

mermaid: true

date: 2026-02-08
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 1단계 Gameplay Framework** 마지막 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 1-3. 매치 흐름과 상태 복제](/devlog/EP_Gameplay_Framework-3)
{: .notice--info}

## 개요

플레이어를 어디에 스폰할지, 그리고 아이템 데이터를 어떤 형태로 둘지 정한다.

두 주제가 한 글에 있는 이유는 둘 다 **"코드 밖에 두는 것"**에 관한 이야기이기 때문이다.
스폰 지점은 레벨에, 아이템 수치는 에셋에 둔다.

---

## 스폰 시스템

### 무엇을 고를 수 있었나

| 선택지 | 장점 | 이 프로젝트에서 |
|---|---|---|
| `APlayerStart` 배치 | 엔진 기본 경로(`ChoosePlayerStart`)에 그대로 얹힘, 레벨 디자이너가 눈으로 배치 | **채택** |
| 커스텀 스폰 볼륨 | 볼륨 내 랜덤 위치 → 지점이 정확히 겹치지 않음 | 미채택, 지금 규모에 과함 |
| 스폰 테이블 DataAsset | 맵별 스폰 세트 교체 가능 | 미채택, 맵이 하나 |

`APlayerStart`를 골랐다. 엔진이 이미 `ChoosePlayerStart` 훅을 제공하니
**우리가 답해야 할 질문이 "어디에 스폰하나"가 아니라 "어느 것을 고르나" 하나로 줄어든다.**

### 중복 없는 랜덤 배정

```cpp
AActor* AEPGameMode::ChoosePlayerStart_Implementation(AController* Player)
{
    TArray<AActor*> Available;

    for (AActor* Start : PlayerStarts)
        if (Start && !UsedPlayerStarts.Contains(Start))
            Available.Add(Start);

    if (Available.Num() > 0)
    {
        const int32 Index  = FMath::RandRange(0, Available.Num() - 1);
        AActor* Chosen     = Available[Index];
        UsedPlayerStarts.Add(Chosen);
        return Chosen;
    }

    // 전부 소진: 초기화 후 엔진 기본 동작으로
    UsedPlayerStarts.Reset();
    return Super::ChoosePlayerStart_Implementation(Player);
}
```

추출 슈터에서 스폰이 겹치면 **시작하자마자 교전**이 난다. 그래서 중복 방지가 필수이다.

### 지금 구조의 한계: 정직하게

**① 중복은 막지만 거리는 보지 않는다.**

`UsedPlayerStarts`는 *같은 지점*을 두 번 쓰는 걸 막을 뿐이다.
바로 옆에 붙어 있는 두 `PlayerStart`가 뽑히면 결국 붙어서 시작한다.
제대로 하려면 **이미 배정된 지점들과의 최소 거리**를 조건에 넣어야 한다.

```cpp
// 개선 방향
Available = PlayerStarts.FilterByPredicate([&](AActor* S){
    return !UsedPlayerStarts.Contains(S)
        && MinDistanceToUsed(S) >= MinSpawnSeparationCm;
});
```

**② 사용 이력을 회수하지 않는다.**

`UsedPlayerStarts`에 넣는 곳은 있는데, 빼는 곳은 *전부 소진됐을 때의 `Reset()`* 하나뿐이다.
플레이어가 나가도 그 지점은 계속 "사용 중"으로 남는다.
리스폰이 생기면 지점이 계속 잠식되어 결국 매번 `Reset()`이 돌고, 중복이 다시 생긴다.

회수해야 할 시점: **로그아웃 시**, **매치 재시작 시**. 아직 구현하지 않았다.

**③ `PlayerStarts` 수집 시점이 `BeginPlay`에 묶여 있다.**

```cpp
void AEPGameMode::BeginPlay()
{
    Super::BeginPlay();
    UGameplayStatics::GetAllActorsOfClass(GetWorld(), APlayerStart::StaticClass(), PlayerStarts);
}
```

그런데 엔진의 호출 순서가 이렇다.

```cpp
// GameMode.cpp: HandleMatchHasStarted()
208-215:  for each PC → RestartPlayer(PC)          // → ChoosePlayerStart
221:      GetWorldSettings()->NotifyBeginPlay();    // → AEPGameMode::BeginPlay()
```

**`ChoosePlayerStart`가 `BeginPlay`보다 먼저이다.**
[1-3편](/devlog/EP_Gameplay_Framework-3)에서 본 대로 보통은 `BeginPlay`가
`HandleMatchIsWaitingToStart`에서 먼저 발화하므로 실제로는 동작한다.
하지만 `ReadyToStartMatch()`가 진입 즉시 참이 되는 경로에서는 `PlayerStarts`가 비어 있고,
그러면 **에러 없이 조용히 엔진 기본 스폰으로 폴백**한다.

고치는 건 간단하다. 수집을 `BeginPlay`가 아니라 **처음 필요할 때** 한다.

```cpp
if (PlayerStarts.Num() == 0)
    UGameplayStatics::GetAllActorsOfClass(GetWorld(), APlayerStart::StaticClass(), PlayerStarts);
```

틱 타이밍 의존이 사라지고, 레벨 스트리밍으로 나중에 나타나는 `PlayerStart`도 다룰 수 있다.

---

## DataAsset 설계

### 왜 코드 밖에 두는가: 이 프로젝트의 이유

"데이터 주도가 좋다"는 말은 어느 프로젝트에나 붙는다. 여기서는 이유가 구체적이다.

| 사정 | 결과 |
|---|---|
| 타르코프류라 **아이템이 수백 개** | 클래스로 만들면 헤더가 수백 개. 컴파일 시간이 무너짐 |
| 밸런스 수치가 **자주 바뀜** | 데미지 1 고치려고 재컴파일하면 반복 주기가 죽음 |
| **루팅 테이블이 런타임에 조합** | 어느 아이템이 필요할지 정적으로 알 수 없음 → 하드 참조 그래프로 표현 불가 |

세 번째가 다음 절의 결론으로 이어진다.

### 이 시점의 두 클래스

```cpp
// UEPItemData : UPrimaryDataAsset
FName            ItemName;
FText            Description;
EEPItemRarity    Rarity;
int32            SellPrice;
bool             bIsQuestItem;
int32            SlotSize;

// UEPWeaponData : UPrimaryDataAsset   ← UEPItemData를 상속하지 않는다
FName            WeaponName;
EEPFireMode      FireMode;
float            Damage, FireRate, ReloadTime;
uint8            MaxAmmo;
float            BaseSpread, SpreadPerShot, MaxSpread;
TArray<FVector2D> RecoilPattern;
```

> 🚩 **여기에 문제가 하나 숨어 있다.**
> `UEPWeaponData`가 `UEPItemData`를 **상속하지 않는다.** 완전히 별개의 두 에셋이다.
>
> 무기는 아이템인데 타입 시스템에서는 아니다.
> 그래서 "인벤토리 슬롯에 무기를 넣는다" 같은 **공통 처리를 쓸 수가 없다.**
>
> 이 단절이 [2-3편](/devlog/EP_Replication-3)에서 3계층 구조로 갈아엎는 직접적인 동기가 된다.

### `UPrimaryDataAsset`을 고른 이유: 메모리 때문이 아니다

`UDataAsset` 대신 `UPrimaryDataAsset`을 골랐다.
흔히 *"UDataAsset은 메모리에 계속 남고 UPrimaryDataAsset은 필요할 때만 로드된다"*고들 하는데,
**그렇지 않다.** 엔진 헤더가 전부이다.

```cpp
// Engine/Classes/Engine/DataAsset.h
class UPrimaryDataAsset : public UDataAsset
{
    GENERATED_BODY()
public:
    ENGINE_API virtual FPrimaryAssetId GetPrimaryAssetId() const override;
    ENGINE_API virtual void PostLoad() override;

#if WITH_EDITORONLY_DATA
    ENGINE_API virtual void UpdateAssetBundleData();
    ENGINE_API virtual void PreSave(FObjectPreSaveContext ObjectSaveContext) override;
protected:
    UPROPERTY()
    FAssetBundleData AssetBundleData;      // 에디터 전용
#endif
};
```

추가된 건 **`GetPrimaryAssetId()` 하나, `PostLoad()` 하나, 에디터 전용 번들 데이터**뿐이다.
언로드 로직도 지연 로드 로직도 없다.

메모리 상주 여부를 정하는 건 클래스가 아니라 **참조 방식**이다.

| 참조 | 메모리 |
|---|---|
| `TObjectPtr<UDataAsset>` (하드) | 참조자와 함께 로드, 살아 있는 동안 유지 |
| `TObjectPtr<UPrimaryDataAsset>` (하드) | **똑같다** |
| `TSoftObjectPtr<...>` | 명시적으로 로드하기 전엔 안 올라옴 (베이스 클래스 무관) |

**진짜 이점은 "이름으로 찾을 수 있다"이다.**

`GetPrimaryAssetId()`가 있으면 AssetManager가 그 에셋을 **타입 단위로 발견**할 수 있다.
어디서도 하드 참조하지 않아도 된다.

```cpp
// 아무도 참조하지 않아도, 타입만으로 찾아서 비동기 로드
UAssetManager::Get().LoadPrimaryAssetsWithType(FPrimaryAssetType("ItemDefinition"), ...);
```

이게 이 프로젝트에 필요한 이유는 위 표의 세 번째 사정이다.
**어느 아이템이 언제 필요할지는 루팅 테이블이 굴러야 정해진다.**
정적 참조 그래프로는 표현할 수 없는 관계이다.
`FPrimaryAssetId`가 있으면 *"ItemDefinition 타입 중 ItemId가 X인 것"*을 런타임에 찾아 로드할 수 있다.

덤으로 `DefaultGame.ini`의 `PrimaryAssetTypesToScan`으로 **쿠킹·청킹 규칙을 타입 단위로** 걸 수 있다.
나중에 패치 단위를 나눌 때 여기서 나온다.

---

## 새 아이템 추가 절차

1. 콘텐츠 드로어 → 우클릭 → Miscellaneous → Data Asset
2. 클래스로 `EPItemData` 또는 `EPWeaponData` 선택
3. 값 입력

C++를 건드리지 않는다. 이게 목표였다.

---

## 이 글 이후 바뀐 것

- `UEPItemData` / `UEPWeaponData`의 단절이 **3계층 구조**로 정리됨,
  `FEPItemData`(DataTable Row) → `UEPItemDefinition`(DataAsset) → 런타임 상태.
  → [2-3편](/devlog/EP_Replication-3)
- 밸런스 수치는 DataAsset에서 **DataTable Row**로 이동 (CSV 편집 가능)
- 스폰 지점 회수와 최소 거리 조건은 아직 미구현

---

## 참고

- 엔진 소스: `Engine/Source/Runtime/Engine/Classes/Engine/DataAsset.h`
- [DataAsset과 AssetManager](https://redchiken.tistory.com/358)

---

## 다음 편
→ [2-1. CMC 확장으로 Sprint/ADS/Crouch 네트워크 동기화](/devlog/EP_Replication-1)
