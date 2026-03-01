---
title:  "[UE5] 추출 슈터 2-5. 멀티플레이어 복제 설계"
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
- UE5 복제 시스템 핵심 개념 (클래스 존재 범위, 프로퍼티 복제, RPC)
- **이 프로젝트에서 내린 모든 동기화 결정과 그 이유**

**이 글이 필요한 이유:**
- 앞선 포스팅(CMC, CombatComponent)에서 "왜 이 방식으로 동기화했냐"는 질문에 대한 답을 한 곳에 정리
- 이후 포스팅(HP, 사망, 피격)의 구현 이유를 미리 이해하게 함
- "UE5 Replication 정리"로 검색 유입 가능한 독립 레퍼런스

---

## 구현 내용

### 1. 전제: 서버가 유일한 진실

```
서버 = 게임 상태의 원본 (HasAuthority() == true)
클라 = 서버 상태를 복제받아 표현 + Server RPC로 요청
```

클라에서 직접 상태를 바꾸면 서버에 반영이 안 됨. 모든 상태 변경은 서버에서.

### 2. 클래스별 존재 범위

| 클래스              | 서버 존재 | 클라 존재 | 복제 범위  | 역할              |
| ---------------- | ----- | ----- | ------ | --------------- |
| GameMode         | ✅     | ❌     | 복제 안 됨 | 매치 판정, 스폰 규칙    |
| GameState        | ✅     | ✅     | 모든 클라  | 라운드 타이머 등 전역 상태 |
| PlayerController | ✅     | ✅     | 소유 클라만 | 입력 처리, HUD      |
| PlayerState      | ✅     | ✅     | 모든 클라  | 킬 수, 추출 여부      |
| Character        | ✅     | ✅     | 모든 클라  | 위치, 애니메이션       |

### 3. 프로퍼티 복제 3패턴

```cpp
// 패턴 1: 단순 복제
UPROPERTY(Replicated)
float HP;
DOREPLIFETIME(AEPCharacter, HP);

// 패턴 2: 복제 + 콜백 (클라이언트에서 반응 필요할 때)
UPROPERTY(ReplicatedUsing = OnRep_HP)
float HP;
UFUNCTION()
void OnRep_HP(float OldHP);  // 이전 값도 받을 수 있음

// 패턴 3: 조건부 복제
DOREPLIFETIME_CONDITION(AEPPlayerState, KillCount, COND_OwnerOnly);
```

**COND 선택 기준표:**

| 조건 | 의미 |
|------|------|
| `COND_None` | 조건 없음 (항상 복제, 기본값) |
| `COND_OwnerOnly` | 소유 클라이언트에게만 |
| `COND_SkipOwner` | 소유자 제외 모두에게 |
| `COND_SimulatedOnly` | Simulated Proxy에게만 |
| `COND_AutonomousOnly` | Autonomous Proxy에게만 |
| `COND_InitialOnly` | 최초 복제 시 1회만 |
| `COND_InitialOrOwner` | 최초 1회 또는 소유자에게 |

### 4. RPC 3종류와 선택 기준

```cpp
// Server RPC — 클라의 요청을 서버에 전달
UFUNCTION(Server, Reliable)
void Server_Fire(FVector_NetQuantize Origin, FVector_NetQuantizeNormal Dir);

// Client RPC — 서버가 특정 클라에게만 알림
UFUNCTION(Client, Unreliable)
void Client_PlayHitConfirmSound();  // 쏜 사람만 "챡" 소리

// NetMulticast — 서버가 모든 클라에 브로드캐스트
UFUNCTION(NetMulticast, Unreliable)
void Multicast_PlayFireEffect(FVector MuzzleLocation);
```

**Reliable vs Unreliable:**

| | Reliable | Unreliable |
|--|----------|------------|
| 보장 | 반드시 도달 | 손실 허용 |
| 비용 | 재전송 오버헤드 | 없음 |
| 사용 | 사격 요청, 사망 | 이펙트, 사운드 |

### 5. 이 프로젝트의 모든 동기화 결정

**결정 1: Sprint/ADS → CMC CompressedFlags**

❌ 처음 시도: Server RPC + UPROPERTY(Replicated)
```cpp
void Server_SetSprinting(bool b);
UPROPERTY(Replicated) bool bIsSprinting;
```
- 문제점
  1. 이동 패킷과 Sprint 패킷이 별도 전송됨
  2. 서버 재시뮬레이션 시 순서 불일치
  3. 스냅 발생

✅ 변경: CMC CompressedFlags
```
bWantsToSprint = FLAG_Custom_0 (이동 패킷에 포함)
```
1. 서버가 동일 입력으로 재시뮬레이션
2. 클라 예측과 일치


→ **이동속도에 영향을 주는 상태는 반드시 CMC로 만든다.**

---

**결정 2: Crouch → CMC 내장**

```
ACharacter::Crouch() / UnCrouch()
```
- 해당 함수 내부에서 CMC의 bWantToCrouch 변수를 변경시킨다.
- CMC를 통해 아래와 같은 처리를 한다.
  - 서버 동기화
  - 캡슐 높이 조정
  - 클라 예측
- 별도 RPC/Replicated 변수가 필요하지 않다.

---

**결정 3: HP → UPROPERTY(ReplicatedUsing), COND_OwnerOnly**

```
DOREPLIFETIME_CONDITION(AEPCharacter, HP, COND_OwnerOnly);
```

- 현재 게임의 특성을 고려하면, 상대방 HP를 알면 안된다.
  - COND_OwnerOnly: 나만 내 HP를 받는다.
  - OnRep_HP: HP가 변경되면, 콜백 함수를 호출한다.
- 사망 판정은 Multicast_Die (Reliable) 로 별도 전달한다.

---

**결정 4: EquippedWeapon → ReplicatedUsing, COND_None**

- 모든 클라가 상대방이 어떤 무기를 들고 있는지 봐야 함
  - COND_None
  - OnRep_EquippedWeapon: 클라이언트에서 Attach + LinkAnimClassLayers
  - IsNetRelevantFor과 Replication Graph를 통해, 컬링과 최적화를 진행해주어야 하지만, 이는 이후에 진행한다.

---

**결정 5: CurrentAmmo → COND_OwnerOnly**

- 탄약은 나만 알면 된다.  
→ COND_OwnerOnly

---

**결정 6: KillCount → COND_OwnerOnly**

- 타르코프에서는 킬 수가 공개 정보가 아니며, 내 킬 수는 나만 알고 있다.  
→ COND_OwnerOnly

---

**결정 7: 피격 피드백 → Multicast Unreliable × 2개**

```cpp
UFUNCTION(NetMulticast, Unreliable) void Multicast_PlayHitReact();   // 피격 애니메이션
UFUNCTION(NetMulticast, Unreliable) void Multicast_PlayPainSound();  // 통증 사운드
```

- 모든 클라이언트가 상대방 피격 반응을 봐야 한다.  
→ Multicast
- 애니메이션과 사운드를 분리한 이유
  - 각각 독립적으로 재생/중단 가능
  - 손실돼도 문제가 없다.  
  → Unreliable

---

**결정 8: 히트 확인음 "챡" → Client Unreliable**

```cpp
// EPPlayerController.h
UFUNCTION(Client, Unreliable) void Client_PlayHitConfirmSound();
```

- 쏜 사람만 들어야 한다.  
→ Client RPC
- PlayerController에 배치한다.
  - HUD/피드백 관련은 Controller 책임
  - 손실돼도 큰 문제가 없다.  
  → Unreliable

- 다음 챕터의 Lag Compensation을 위해 만들어 두었다.

---

**결정 8-2: 킬 알림 → Client Reliable**

```cpp
// EPPlayerController.h
UFUNCTION(Client, Reliable) void Client_OnKill(const FString& VictimName);
```

- 킬한 사람에게만 알려야 한다.  
→ Client RPC
- 킬 피드(킬 이름) 손실되면 피드백이 사라진다.  
→ Reliable
- 히트 확인음과 달리 게임 정보(킬 수와 연결)이므로 보장이 필요하다.

---

**결정 9: 사망 → Multicast Reliable**

- 모든 클라이언트가 사망 처리를 받아야 함 (래그돌)
- 손실되면 안 됨 (캡슐 콜리전이 그대로 남음)  
→ Reliable

- 나중에 래그돌도 SetReplicatePhysics 함수를 통해 동기화해야 한다.
  - 시체가 더 이상 움직이지 않으면, 동기화를 멈추도록 한다.
  - 시체 위치가 일정해야 루팅이 가능

---

### 6. 한 줄 원칙 정리

| 개념           | 사용 방식                 |
| ------------ | --------------------- |
| 이동속도 영향 상태   | `CMC CompressedFlags` |
| 모두가 봐야 하는 상태 | `COND_None`           |
| 나만 알면 되는 상태  | `COND_OwnerOnly`      |
| 클라 → 서버 요청   | `Server RPC`          |
| 서버 → 특정 클라   | `Client RPC`          |
| 서버 → 전체      | `Multicast RPC`       |
| 중요한 것        | `Reliable`            |
| 이펙트 / 사운드    | `Unreliable`          |

## 7. 다음 편 예고
→ 애니메이션 시스템