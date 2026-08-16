---
title:  "[UE5] 추출 슈터 2-5. 이 게임의 복제 설계 전부"
excerpt: "게임 규칙이 복제 조건을 정한다: 결정 9개와 그 근거"

categories:
  - DevLog
tags:
  - [UE5, C++, Networking]

toc: true
toc_sticky: true

mermaid: true

date: 2026-03-01
last_modified_at: 2026-08-04
---

📌 **EmploymentProj 2단계 Replication** 다섯 번째 글입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj) ·
[📚 시리즈 목차](/devlog/EP_Main) ·
[← 2-4. CombatComponent 분리](/devlog/EP_Replication-4)
{: .notice--info}

## 이 글은 무엇인가

2-1편부터 2-4편까지 만들면서 *"이건 왜 이렇게 동기화했지?"*를 매번 즉석에서 판단했다.
그 판단들을 한 곳에 모아 **기준으로 만드는 글**이다.

핵심 주장은 하나이다.

> **복제 조건은 UE 문법이 아니라 게임 규칙에서 나온다.**

먼저 그 대응표부터 보겠다. 이 표가 이 글의 요약이다.

| 게임 규칙 ([GAME.md](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)) | 복제 결정 |
|---|---|
| 적의 체력은 볼 수 없다 | `HP` → `COND_OwnerOnly` |
| 잔탄은 나만 안다 | `CurrentAmmo` → `COND_OwnerOnly` |
| 킬 수는 공개 정보가 아니다 | `KillCount` → `COND_OwnerOnly` |
| 상대가 든 무기는 보여야 한다 | `EquippedWeapon` → `COND_None` |
| 이동 속도가 바뀌면 예측이 어긋난다 | Sprint/ADS → **CMC CompressedFlags** |
| 시체를 뒤진다 | 사망 → `Multicast` **Reliable** (콜리전이 남으면 못 뒤짐) |

**타르코프식 정보 은닉이 그대로 `COND_OwnerOnly`가 된다.**
배틀로얄이었다면 킬 수는 공개였을 테고, 조건도 달랐을 거다.

---

## 전제: 서버가 유일한 진실

```
서버 = 게임 상태의 원본 (HasAuthority() == true)
클라 = 서버 상태를 복제받아 표현 + Server RPC로 요청
```

복제는 **서버 → 클라 단방향**이다. 클라가 직접 바꾼 값은 서버에 반영되지 않고,
다음 복제에 덮어써진다.

### 클래스별 존재 범위

| 클래스 | 서버 | 클라이언트 | 복제 |
|---|---|---|---|
| `GameMode` | 있음 | **없음** | 안 됨 |
| `GameState` | 있음 | 있음 | 모든 클라 |
| `PlayerController` | **접속자 수만큼 전부** | **자기 것 하나만** | 소유 클라만 |
| `PlayerState` | 있음 | 있음 (전원 것) | 모든 클라 |
| `Character` | 있음 | 있음 (전원 것) | 모든 클라 |

`PlayerController` 줄이 헷갈리기 쉽다.
**서버에는 전부 있고, 각 클라에는 자기 것만 있다.**
서버 코드에서 순회하면 전원이 나오지만 클라에서는 하나뿐이다.

### Role: 예측은 누구에게 일어나는가

같은 캐릭터 클래스라도 **어느 머신에서 보느냐**에 따라 역할이 다르다.

| | 내 캐릭터 | 남의 캐릭터 |
|---|---|---|
| 내 클라에서의 Role | `ROLE_AutonomousProxy` | `ROLE_SimulatedProxy` |
| 서버에서의 Role | `ROLE_Authority` | `ROLE_Authority` |
| 이동 | **예측** (SavedMove 저장·리플레이) | 보간 (`SmoothClientPosition`) |
| 입력 | 있음 | 없음 |

[2-1편](/devlog/EP_Replication-1)의 CMC 예측은 **AutonomousProxy에게만 일어나는 일**이다.
남의 캐릭터는 예측하지 않는다. 서버가 보내주는 위치를 부드럽게 따라갈 뿐이다.

`COND_AutonomousOnly` / `COND_SimulatedOnly`가 가리키는 게 이 구분이다.

---

## 프로퍼티 복제 3패턴

```cpp
// ① 단순 복제
UPROPERTY(Replicated)
float HP;
DOREPLIFETIME(AEPCharacter, HP);

// ② 복제 + 콜백: 클라에서 반응이 필요할 때
UPROPERTY(ReplicatedUsing = OnRep_HP)
float HP;
UFUNCTION() void OnRep_HP(float OldHP);   // 인자로 '이전 값'을 받을 수 있다

// ③ 조건부 복제
DOREPLIFETIME_CONDITION(AEPPlayerState, KillCount, COND_OwnerOnly);
```

> **중요:** 복제 조건은 헤더의 `UPROPERTY`에 없다.
> **`GetLifetimeReplicatedProps()`에 있다.**
> 헤더만 보고 "조건이 없네"라고 판단하면 틀린다.
>
> 그리고 반대 방향의 함정도 있다.
> `UPROPERTY(Replicated)`를 달아놓고 `GetLifetimeReplicatedProps()`에 등록하지 않으면
> **경고 없이 복제되지 않는다.**
> [2-4편](/devlog/EP_Replication-4)의 `AEPWeapon::MaxAmmo`가 그랬다.

### COND 목록

| 조건 | 의미 | 이 프로젝트에서 |
|---|---|---|
| `COND_None` | 항상 (기본) | `EquippedWeapon` |
| `COND_OwnerOnly` | 소유 클라에게만 | `HP`, `CurrentAmmo`, `KillCount`, `bIsExtracted` |
| `COND_SkipOwner` | 소유자 제외 전원 | (미사용) |
| `COND_SimulatedOnly` | Simulated Proxy에게만 | (미사용) |
| `COND_AutonomousOnly` | Autonomous Proxy에게만 | (미사용) |
| `COND_InitialOnly` | 최초 1회만 | 엔진 `AGameState::ElapsedTime`이 사용 |
| `COND_InitialOrOwner` | 최초 1회 또는 소유자 | (미사용) |

---

## RPC 3종

```cpp
// Server: 클라 → 서버 요청
UFUNCTION(Server, Reliable)
void Server_Fire(FVector_NetQuantize Origin, FVector_NetQuantizeNormal Dir);

// Client: 서버 → 특정 클라
UFUNCTION(Client, Unreliable)
void Client_PlayHitConfirmSound();          // 쏜 사람만 "챡"

// NetMulticast: 서버 → 전원
UFUNCTION(NetMulticast, Unreliable)
void Multicast_PlayMuzzleEffect(FVector_NetQuantize MuzzleLocation);
```

### 글로 잘 안 나오는 두 가지

**① Server RPC는 아무나 못 부른다.**
**그 액터를 소유한 클라이언트만** 호출할 수 있다.
소유권(`SetOwner`)이 없으면 호출은 조용히 버려진다.

`UEPCombatComponent`가 `AEPCharacter`에 붙어 있고
캐릭터의 Owner가 `PlayerController`이기 때문에 `Server_Fire`가 도달한다.
무기 액터에서 직접 쐈다면 소유권을 따로 설정해야 했다.
**동작하는데 왜 동작하는지 모르는 상태**가 가장 위험하다.

**② `WithValidation`**

```cpp
UFUNCTION(Server, Reliable, WithValidation)
void Server_Fire(const FVector& Origin, const FVector& Direction);

bool UEPCombatComponent::Server_Fire_Validate(const FVector& Origin, const FVector& Direction)
{
    return !Origin.ContainsNaN() && Direction.IsNormalized();
}
```

`_Validate`가 `false`면 **그 연결이 끊긴다.**
[2-4편](/devlog/EP_Replication-4)에서 지적한 `Origin` 미검증 문제의
가장 값싼 1차 방어선이 여기이다. 아직 안 넣었다.

### Reliable을 언제 쓰는가

원래 이 표를 이렇게 썼다.

| | Reliable | Unreliable |
|---|---|---|
| 사용 | 사격 요청, 사망 | 이펙트, 사운드 |

**"사격 요청"은 빼야 한다.** 기준을 다시 세우면 이렇다.

> **이 메시지를 놓치면 서버와 클라의 상태가 영구히 어긋나는가?**
> - 어긋난다 → **Reliable** (사망, 아이템 획득, 매치 상태 전이)
> - 다음 메시지가 곧 덮어쓴다 → **Unreliable** (발사, 이펙트, 위치)

자동 사격은 후자이다. 한 발을 놓쳐도 다음 발이 곧 오고,
Reliable로 재전송하면 **이후 패킷이 밀려서 나중에 뭉쳐 도착**한다.
초당 5~10발이 나가는 무기에서는 그 편이 더 나쁜다.

---

## 결정 1: Sprint/ADS → CMC CompressedFlags

**처음 방식**

```cpp
UFUNCTION(Server, Reliable) void Server_SetSprinting(bool b);
UPROPERTY(Replicated) bool bIsSprinting;
```

문제는 두 가지였다.

**① 보정 리플레이에서 상태가 사라진다.**
서버 보정이 오면 클라는 미확인 `SavedMove`들을 다시 재생한다.
그 `SavedMove`에 Sprint 상태가 없으면 `GetMaxSpeed()`가 **걷기 속도**를 반환하고,
원래 계산과 다른 결과가 나온다 → 또 어긋나고 → 또 보정. 스냅이 반복된다.

**② 그 RPC가 매 프레임 나가고 있었다.**
당시 Sprint 입력이 `ETriggerEvent::Triggered`(누르고 있는 동안 매 프레임)에 묶여 있어서,
**Reliable Server RPC가 초당 60회** 나가는 상태였다.

**변경 후**

```cpp
bWantsToSprint → FLAG_Custom_0    // 이동 패킷에 편승
```

`CompressedFlags`는 원래 항상 1바이트가 나가고 있고, 그중 4비트가 비어 있다.
빈 비트를 쓰는 것이므로,

> **초당 60회 Reliable RPC → 추가 0바이트**

그리고 서버가 같은 플래그로 재시뮬레이션하니 예측과 결과가 일치한다.

**원칙: 이동 속도에 영향을 주는 상태는 반드시 CMC로 만든다.**
→ [2-1편](/devlog/EP_Replication-1)

---

## 결정 2: Crouch는 만들지 않는다

```cpp
ACharacter::Crouch() / UnCrouch()
```

내부에서 CMC의 `bWantsToCrouch`를 바꾸고, `FLAG_WantsToCrouch`로 전송하고,
캡슐 높이를 조정하고, 클라 예측까지 한다.

**즉 결정 1에서 우리가 만든 것과 똑같은 구조를 엔진이 이미 갖고 있다.**
별도 RPC도 복제 변수도 필요 없다.

바꿔 말하면, 결정 1은 **엔진의 방식을 두 개 더 만든 것**이다.

---

## 결정 3 & 9: HP를 숨겼기 때문에 사망을 따로 알린다

이 둘은 별개의 결정이 아니라 **하나의 결정의 두 면**이다.

```cpp
DOREPLIFETIME_CONDITION(AEPCharacter, HP, COND_OwnerOnly);   // 결정 3
```

```cpp
UFUNCTION(NetMulticast, Reliable)
void Multicast_Die();                                        // 결정 9
```

적의 체력을 볼 수 없게 하려고 `COND_OwnerOnly`를 걸었다.
그 순간 **다른 클라이언트는 그 캐릭터가 죽었다는 것도 알 수 없게 된다**.
HP가 0이 되는 걸 못 보니까.

그래서 사망을 별도로 통보해야 한다. 그리고 Reliable이어야 한다.

> 놓치면 **캡슐 콜리전이 그대로 남는다.** 보이지 않는 벽이 생기고,
> 시체를 뒤질 수도 없다. 다음 메시지가 덮어쓰지 않는 종류의 상태이다.

`COND_OwnerOnly`의 대가를 정리하면:

| 못 하게 되는 것 | 대응 |
|---|---|
| 적 머리 위 체력바 | 애초에 안 만듦, 의도한 결과 |
| 남의 사망을 스스로 알기 | `Multicast_Die`로 별도 통보 |
| 팀원 체력 표시 (팀 모드가 생기면) | 팀원에게만 여는 커스텀 조건 필요 |

> 🚩 **`Multicast_Die`에는 구멍이 하나 있다.**
> Multicast RPC는 **그 순간 접속해 있고 릴러번트한** 클라에게만 간다.
> 나중에 접속한 사람은 **시체가 서 있는 걸** 본다.
> `bIsDead` 복제 변수 + `OnRep`이었다면 초기 복제로 해결됐을 문제이다.
> → [2-2편](/devlog/EP_Replication-2)에서 결국 `AEPCorpse`가 그 방식을 택한다.

래그돌 자체는 각 클라가 로컬에서 시뮬레이션하므로 자세가 조금씩 다르다.
동기화하려면 `SetReplicateMovement(true)`로 `FRepMovement`를 켜야 한다.

```cpp
// EngineTypes.h: FRepMovement
uint32 bRepPhysics : 1;    // "이 액터는 물리 시뮬레이션 중"
```
```cpp
// Actor.h:2978
/** Sync IsSimulatingPhysics() with ReplicatedMovement.bRepPhysics */
void SyncReplicatedPhysicsSimulation();
```

시체가 멈추면 복제를 꺼야 대역폭이 안 샌다. 루팅을 붙이려면 필요한 작업이다.

---

## 결정 4: `EquippedWeapon`은 모두가 봐야 한다

```cpp
DOREPLIFETIME(UEPCombatComponent, EquippedWeapon);      // COND_None
```

상대가 소총을 들었는지 권총을 들었는지는 **교전 판단의 핵심 정보**이다. 숨길 이유가 없다.

`OnRep_EquippedWeapon`에서 클라가 부착 + `LinkAnimClassLayers`를 수행한다.
복제 변수 방식이라 **나중에 접속한 사람도** 초기 복제로 `OnRep`을 받는다.
`Multicast_Die`가 못 하는 그것이다.

---

## 결정 5·6: 잔탄과 킬 수는 나만 안다

```cpp
DOREPLIFETIME_CONDITION(AEPWeapon,      CurrentAmmo,  COND_OwnerOnly);
DOREPLIFETIME_CONDITION(AEPPlayerState, KillCount,    COND_OwnerOnly);
DOREPLIFETIME_CONDITION(AEPPlayerState, bIsExtracted, COND_OwnerOnly);
```

타르코프에는 킬 피드도 스코어보드도 없다. **내 킬 수는 나만 안다.**
"저 사람 3발 남았다"도 마찬가지로 큰 정보인다.

조건을 안 걸었다면 조작된 클라이언트가 전부 읽을 수 있었다.
**보안은 클라에서 숨기는 게 아니라 애초에 안 보내는 것이다.**

---

## 결정 7·8: 피드백은 누가 받아야 하나로 갈린다

```cpp
UFUNCTION(NetMulticast, Unreliable) void Multicast_PlayHitReact();   // 피격 애니메이션
UFUNCTION(NetMulticast, Unreliable) void Multicast_PlayPainSound();  // 통증 사운드
UFUNCTION(Client, Unreliable)       void Client_PlayHitConfirmSound(); // 쏜 사람만 "챡"
```

- **맞은 쪽 반응**은 주변 모두가 봐야 한다 → Multicast
- **맞혔다는 확인음**은 쏜 사람만 들어야 한다 → Client RPC
- 셋 다 놓쳐도 게임 상태가 어긋나지 않는다 → Unreliable

애니메이션과 사운드를 나눈 이유는 **독립적으로 재생·중단**되어야 하기 때문이다.
연속 피격 시 사운드는 겹쳐도 되지만 몽타주는 재시작돼야 한다.

히트 확인음을 `PlayerController`에 둔 건 **HUD·피드백은 컨트롤러 책임**이라는 구분이다.
그리고 이건 3단계 지연 보상을 위한 준비이기도 한다.
서버가 판정한 히트를 쏜 사람에게 알리는 통로가 미리 필요했다.

### 결정 8-2: 킬 알림은 Reliable

```cpp
UFUNCTION(Client, Reliable)
void Client_OnKill(const FString& VictimName);
```

킬 피드는 놓치면 그대로 사라진다(다음 킬이 덮어쓰지 않음) → Reliable.
확인음과 달리 **게임 정보**이다.

> **다만 `FString`을 보내는 건 다시 볼 필요가 있다.**
> 문자열은 길이 가변이고 UTF-16으로 직렬화된다. 12자면 헤더 포함 30바이트쯤이다.
>
> ```cpp
> void Client_OnKill(APlayerState* Victim);    // 객체 참조(NetGUID)
> ```
>
> `APlayerState`는 이미 모든 클라에 복제돼 있으니 이름은 거기서 읽으면 된다.
> **이미 복제된 것을 또 보내지 않는다**. `MaxAmmo` 때와 같은 원칙이다.

---

## 아직 손대지 않은 축 두 개

복제 설계에는 축이 셋 있다.
**무엇을(프로퍼티) / 누구에게(조건) / 언제까지(거리·빈도).**
위의 결정들은 전부 앞의 둘이다.

### 릴러번시: 얼마나 멀리까지

```cpp
// Actor.cpp:310  기본값
SetNetCullDistanceSquared(225000000.0f);     // = 15000cm(150m)의 제곱
```

**150m 밖의 액터는 복제가 끊긴다.**
추출 슈터의 맵은 그보다 크다. 저격 교전 거리를 정하기 전에 이 값을 정해야 한다.

그리고 [2-4편](/devlog/EP_Replication-4)에서 본 대로
**Multicast RPC도 릴러번시 밖으로는 안 간다.** 총소리가 안 들린다.
FPS에서 소리가 들려야 하는 거리와 액터 복제 거리는 보통 다르다.

`IsNetRelevantFor` 오버라이드나 Replication Graph는 그 다음 이야기이다.

### 복제 빈도: 얼마나 자주

```cpp
// Actor.cpp:293-294  기본값
SetNetUpdateFrequency(100.0f);
SetMinNetUpdateFrequency(2.0f);
```

프로퍼티가 바뀌어도 **즉시 나가지 않는다.** 이 빈도에 맞춰 나간다.
FPS에서 캐릭터 복제 빈도는 곧 **적의 움직임이 얼마나 매끄러운가**이다.

이 글을 쓸 당시에는 기본값 그대로 두고 있었다.
지금은 조정했다.

```cpp
// EPCharacter.cpp
SetNetUpdateFrequency(66.f);
SetMinNetUpdateFrequency(33.f);
```

---

## 한 줄 요약표

| 상황 | 도구 |
|---|---|
| 이동 속도에 영향을 주는 상태 | `CMC CompressedFlags` (추가 0바이트) |
| 모두가 봐야 하는 상태 | `COND_None` |
| 나만 알면 되는 상태 | `COND_OwnerOnly` |
| 클라 → 서버 요청 | `Server RPC` (소유권 필요) |
| 서버 → 특정 클라 | `Client RPC` |
| 서버 → 전원 | `Multicast RPC` (릴러번시 안에서만) |
| 놓치면 상태가 영구히 어긋남 | `Reliable` |
| 다음 메시지가 곧 덮어씀 | `Unreliable` |
| 늦게 접속한 사람도 알아야 함 | **RPC 말고 복제 변수 + `OnRep`** |

마지막 줄이 이 글을 쓰면서 가장 크게 배운 것이다.
**RPC는 "그 순간 그 자리에 있던 사람"에게만 간다.**

---

## 대역폭 감각

숫자 없이 "최적화했다"고 쓰지 않기 위해 정리해둔다.

| | 크기 |
|---|---|
| `CompressedFlags` 커스텀 플래그 | **0바이트** (기존 1바이트의 빈 비트 재사용) |
| `FVector` → `FVector_NetQuantize` | 12바이트 → 약 5~6바이트 |
| `FString("PlayerName")` → `APlayerState*` | 약 30바이트 → NetGUID |
| `MaxAmmo` (정적 데이터) | **보낼 필요 없음** |

---

## 다음 편
→ [2-6. 애니메이션 시스템과 Linked Anim Layer](/devlog/EP_Replication-6)
