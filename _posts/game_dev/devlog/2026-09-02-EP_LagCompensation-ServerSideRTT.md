---
title:  "[UE5] 익스트랙션 슈터: 발사 시각은 서버가 잰다"
excerpt: "클라이언트 신뢰에서 서버 측정으로, RTT/2에서 RTT로. 세 번 뒤집은 끝에 도달한 자리"

categories:
  - DevLog
tags:
  - [UE5, C++, LagCompensation, Networking]

toc: true
toc_sticky: true

mermaid: true

date: 2026-09-02
last_modified_at: 2026-09-02
---

📌 [3-2. 서버 사이드 리와인드: 두 함수가 다른 프레임의 시계를 보고 있었다](/devlog/EP_NetPrediction-2)의
"정직하게 ①"에서 남겨둔 숙제를 실제로 풀어본 기록입니다.  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📚 시리즈 목차](/devlog/EP_Main)
{: .notice--info}

## 문제

[지난 글](/devlog/EP_NetPrediction-2)은 "정직하게 ①"에서 이렇게 적어뒀다.

> 조작된 클라이언트는 최근 0.7초 중 가장 유리한 순간을 골라 보낼 수 있다.
> 정석은 클라이언트 값 대신 서버가 잰 왕복 시간을 쓰는 것이다.

그런데 실제로 코드를 다시 열어보니 문제가 그보다 심각했다. GAS로 전환하면서 발사 경로가
바뀌었는데, 그 자리에 남아 있던 변수 이름이 거짓말을 하고 있었다.

<details markdown="1">
<summary>이름과 값이 어긋난 코드 (접기/펼치기)</summary>

```cpp
// GA_Item_PrimaryUse.cpp, 서버 브랜치
const float ClientTime = GS ? GS->GetServerWorldTimeSeconds() : Char->GetWorld()->GetTimeSeconds();
...
Combat->HandleServerFire(Origin, Char->GetControlRotation().Vector(), ClientTime);
```

</details>

이름은 `ClientTime`인데 값은 **서버가 그 자리에서 읽은 자기 시계**다. 클라이언트 발사 시각을
서버로 나르던 경로(`Server_Fire` RPC)가 GAS 전환 때 사라지면서, 서버 브랜치가 자기 시계로
그 빈자리를 채우고 있었다. 결과: `ServerNow - ClientFireTime ≈ 0`. 지연 보상이 이름만 남고
사실상 꺼져 있었다.

고칠 방향은 두 가지였다.

1. 끊긴 경로를 되살려서 클라이언트가 다시 발사 시각을 보내게 한다
2. 애초에 클라이언트 시각을 안 쓰고 서버가 직접 계산한다

**결론부터 말하면 1번을 먼저 구현했다가 버리고 2번으로 갔다.** 왜 그랬는지가 이 글의 전부다.

---

## 시도 1: 클라이언트 값 + 서버 검증

처음엔 "정석대로 (2)"가 아니라 절충안을 골랐다. 클라이언트가 자기 화면 기준 발사 시각을
보내고, 서버가 그 사수의 RTT 기대치로 검증하는 방식이다.

```
ExpectedDelay  = GetPingInMilliseconds() * 0.001 * 0.5      // 서버가 잰 값, 조작 불가
ClaimedDelay   = ServerNow - ClientFireTime                  // 클라가 주장하는 값
ValidatedDelay = Clamp(ClaimedDelay, ExpectedDelay ± Tolerance)
```

클라이언트가 정직하면 `ClaimedDelay ≈ ExpectedDelay`일 거라고 가정했다. **이 가정이 틀렸다.**

### 스케일이 애초에 안 맞았다

`ClientFireTime`은 클라이언트의 `GetServerWorldTimeSeconds()`로 만들어진다. 이 함수를
직접 열어보면:

<details markdown="1">
<summary>GetServerWorldTimeSeconds (접기/펼치기)</summary>

```cpp
// GameStateBase.cpp
double AGameStateBase::GetServerWorldTimeSeconds() const
{
    ...
    return World->GetTimeSeconds() + ServerWorldTimeSecondsDelta;
}
```

</details>

클라이언트의 `ServerWorldTimeSecondsDelta`는 서버→클라 편도 지연(`D`)만큼 편향돼 있다.
왕복(`U+D`) 보정이 아니라 **편도 보정**이다. 대입해서 유도하면:

```
ClientFireTime ≈ Sv(발사) - D
ServerNow      ≈ Sv(발사) + U
ClaimedDelay   = ServerNow - ClientFireTime ≈ U + D = RTT 전체
```

`ExpectedDelay`는 의도적으로 RTT/2를 낸 값인데, `ClaimedDelay`는 구조적으로 RTT 전체가
나온다. 둘이 애초에 2배 차이가 나게 설계돼 있었다. `Tolerance`가 이 절반을 매번 흡수해야
하는 상황이니, 검증이 아니라 상시 클램프로 변질됐다.

### 그런데도 나쁜 네트워크에서 계속 빗나갔다

스케일 버그를 고치고, 로그 중복 계산 버그를 고치고, ini에 남아 있던 실험값
(`FireTimeToleranceSeconds=0.7`)까지 정리했는데도, PktLag로 나쁜 네트워크를 흉내 낸
테스트에서는 계속 빗맞았다. **검증 클램프가 거의 개입 안 하는 상황에서도 틀렸다.** 그러면
`Tolerance` 문제가 아니라 재료 자체(`ClientFireTime`)가 문제라는 뜻이다.

원인은 같은 함수였다. `GetServerWorldTimeSeconds()`의 클라-서버 편향은 클라이언트의
캘리브레이션 패킷이 정기적으로, 제때 도착한다는 전제 위에 있다. 패킷 유실이나 지터가 심한
네트워크에서는 이 전제 자체가 흔들리고, `ClientFireTime`은 정직한 클라이언트에서도
예측 불가능하게 흔들린다.

**여기서 방향을 접었다.** 클라이언트 시각을 신뢰하는 한, 그 신뢰의 품질은 클라이언트의
네트워크 상태에 종속된다. 정확도가 가장 필요한 순간(나쁜 네트워크)에 정확도가 가장
떨어지는 구조다.

---

## 시도 2: 서버가 자기 핑만으로 계산한다

`Epic Games`의 공식 UnrealTournament 소스를 열어봤다. `AUTCharacter::GetRewindLocation`과
`AUTPlayerController::GetPredictionTime`이 하는 일은 이랬다.

<details markdown="1">
<summary>UT의 GetPredictionTime (접기/펼치기)</summary>

```cpp
// UTPlayerController.cpp
float AUTPlayerController::GetPredictionTime()
{
    return (PlayerState && (GetNetMode() != NM_Standalone))
        ? (0.0005f * FMath::Clamp(PlayerState->ExactPing - PredictionFudgeFactor, 0.f, MaxPredictionPing))
        : 0.f;
}
```

</details>

클라이언트는 발사 시각을 아예 보고하지 않는다. 서버가 이미 알고 있는 그 사수의 핑만으로
"얼마나 되돌릴지"를 혼자 결정한다. `ClientFireTime`이라는 개념 자체가 없으니, 클라-서버
시계 편향 문제가 애초에 들어올 자리가 없다.

<details markdown="1">
<summary>ComputeRewindTime (접기/펼치기)</summary>

```cpp
float UEPServerSideRewindComponent::ComputeRewindTime(
    const AEPCharacter* Shooter, float ServerNow) const
{
    const UEPCombatDeveloperSettings* CombatSettings = GetDefault<UEPCombatDeveloperSettings>();

    float PredictionDelay = 0.f;
    if (const APlayerState* PS = Shooter ? Shooter->GetPlayerState() : nullptr)
    {
        PredictionDelay = FMath::Clamp(
            PS->GetPingInMilliseconds() * 0.001f - CombatSettings->PredictionFudgeSeconds,
            0.f, CombatSettings->MaxRewindSeconds);
    }

    return ServerNow - PredictionDelay;
}
```

</details>

`Server_ConfirmFire` RPC는 그대로 두되, 이제 `Origin`/`Direction`만 싣는다. 시각을 실어
보낼 이유가 없다.

**보안 측면도 이득이다.** 하이브리드안은 "클라가 거짓 시각을 보낼 수 있다"는 위협을
`Tolerance`로 막아야 했다. 이번 설계는 클라가 조작할 수 있는 값 자체가 없으니 그 위협이
성립하지 않는다.

---

## 그런데 숫자가 안 맞았다

UT를 그대로 따라 `RTT/2`로 시작했다. 200/200(들어오고 나가는 지연 각각 200ms) 환경에서
실제로 쏴봤다.

<details markdown="1">
<summary>RTT/2 실측 로그 (접기/펼치기)</summary>

```
[SSR] ServerNow=29.103 PredictionDelay=0.196, Candidates=1
[SERVER_REWIND_POS] RewindTime=28.595 ...
```

</details>

**표적이 화면에 보이는 위치보다 캐릭터 너비만큼 앞을 쏴야 맞았다.** 리와인드가 부족했다.

다시 유도해봤다.

```
사수가 절대시각 A에 발사
사수 화면이 보여준 표적 상태 = A - D(다운링크) - InterpDelay 시점의 참값
발사 명령은 U(업링크)를 거쳐 ServerNow = A + U에 서버 도착

RewindTime = A - D - InterpDelay
           = (ServerNow - U) - D - InterpDelay
           = ServerNow - (U + D) - InterpDelay
           = ServerNow - RTT - InterpDelay
```

`U`(발사 명령의 사수→서버)와 `D`(표적 위치가 사수 화면에 뜨는 서버→사수)는 서로 다른
구간이고, 둘 다 되돌려야 한다. `RTT/2`는 이 중 한쪽만 보상하고 있었다. UT가 왜 RTT/2만
쓰는지는 끝까지 확실히 못 알아냈다. 의도적으로 사수에게 완전 보상을 안 주는 밸런스
선택이었을 가능성이 있다. 참고 사례로는 유효해도, 이 프로젝트의 목표("사수가 화면에서
본 대로 맞는다")에는 안 맞았다.

`*0.5`를 지우고 `RTT` 전체 + `InterpDelay`(표적 CMC의 스무딩 지연, 0.1초)로 다시 쐈다.

<details markdown="1">
<summary>RTT 전체 실측 로그 (접기/펼치기)</summary>

```
[SSR] ServerNow=29.103 PredictionDelay=0.408, Candidates=1
```

</details>

**이번엔 반대로 캐릭터 너비만큼 뒤를 쏴야 맞았다.** 과보정이었다.

두 오차가 크기는 같고 부호만 반대였다. 정답은 정확히 중간이라는 뜻이다.

| 시도 | `TotalDelay` | 결과 |
|---|---|---|
| RTT/2 + InterpDelay | 0.296초 | 캐릭터 너비만큼 **부족** |
| RTT 전체 + InterpDelay | 0.508초 | 캐릭터 너비만큼 **과함** |
| 중간값 | 0.402초 | (계산상 정답) |
| RTT 전체 단독 (InterpDelay 없이) | 0.408초 | 중간값과 0.006초 차이 |

`InterpDelay`를 아예 빼고 `RTT` 하나만 썼더니 계산상 정답과 노이즈 수준 차이로 맞아떨어졌다.
실제로 다시 쏴보니 남은 오차는 0.03~0.05초어치 전진 수준, 핑 롤링 평균의 지연 같은 잡음
범위였다. 여기서 확정했다.

```
PredictionDelay = Clamp(RTT - PredictionFudgeSeconds, 0, MaxRewindSeconds)
RewindTime      = ServerNow - PredictionDelay
```

---

## 이론과 실측이 어긋난 이유

`InterpDelay`를 이론적으로 유도했을 때 근거로 삼은 건 Valve의 Source 엔진 공식 렉 보상
문서였다.

```
Command Execution Time = Current Server Time - Packet Latency - Client View Interpolation
```

Source는 `Client View Interpolation`(기본 `cl_interp` 100ms)을 **명시적으로** 더한다.
우리가 유도한 `InterpDelay`와 같은 항이다. 그런데 정작 UT의 실제 출시 코드
(`AUTCharacter::GetRewindLocation`)에는 이 항이 없다. `PredictionTime` 하나만 쓴다.

**Source는 이론상 넣고, UT는 실전에서 뺐다.** 업계도 이 부분에서 갈린다. 우리 실측 결과가
근거 없는 우연이 아니라, UT라는 실제 선례와 일치하는 선택이었다는 뜻이다. 다만 왜 이론과
UT의 실제 선택이 어긋나는지 명확한 원인까지는 못 찾았다. 이건 "정직하게" 절에 남겨둔다.

---

## 업계가 더 쓰는 쪽

찾아본 김에 정리해둔다.

| 게임/엔진 | 발사 시각 결정 | 근거 |
|---|---|---|
| Source (CS 계열) | **서버 측정**. `Packet Latency`를 서버가 커맨드 도착 간격으로 직접 잰다 | Valve 공식 문서 |
| Unreal Tournament | **서버 측정**. `ExactPing`으로 `PredictionTime` 계산, 보간 지연 항 없음 | 엔진 소스 직독 |
| 오버워치 | 서버가 사수의 지연을 측정해 임계값(250ms) 이내면 전부 보상, 넘으면 클램프 | GDC 2017 발표 |
| 발로란트 | **클라이언트가 보고**. "클라와 서버가 움직임 타임라인이 어떻게 동기화되는지 안다"는 전제 위에서, 클라가 보낸 시각을 쓰되 최대 되돌림 시간을 제한 | Riot 공식 기술 블로그 |

발로란트만 클라이언트 값을 신뢰한다. 차이는 "움직임 타임라인이 어떻게 동기화되는지 안다"는
문장에 있다. 자체 네트워크 인프라(Riot Direct)와 정교한 클라-서버 시계 동기화를 갖춘
전제 위에서만 성립하는 방식이다. 우리 프로젝트가 쓰던 `GetServerWorldTimeSeconds()`는
편도만 보정하는 구조적 편향이 증명됐으니, 그 신뢰를 뒷받침할 정교함이 없었다. 그래서
서버 측정 쪽(Source/UT 계열)이 이 프로젝트에는 맞는 선택이었다.

**"클라 신뢰가 나쁘다"가 아니라, "그 신뢰를 얼마나 정교한 시계 동기화로 뒷받침하는가"의
문제였다.**

---

## 정직하게: 남아 있는 것들

### ① `InterpDelay`를 왜 빼도 되는지 완전히 설명은 못 했다

UT라는 실제 선례가 있다는 것과, 그 선례가 **왜** 맞는지는 다른 문제다. Source의 이론은
여전히 유효해 보이는데 우리 실측과는 어긋난다. UE5의 `NetworkSimulatedSmoothLocationTime`
스무딩 메커니즘이 Source의 `cl_interp`와 실제로 다르게 동작하는 부분이 있는지는 더
파봐야 안다.

### ② RTT가 대칭이라는 가정 위에 있다

`U`와 `D`를 따로 측정할 방법이 없어서 `RTT = U+D`만 알고 개별 값은 모른다. 극단적으로
비대칭인 회선(일부 모바일, 위성)에서는 오차가 커질 수 있다. 8인 규모 프로젝트에서는
지금 당장의 걱정거리는 아니다.

### ③ RPC 직접 호출은 이 수정의 범위 밖이다

`Server_ConfirmFire`는 소유 커넥션이면 누구나 호출 가능한 평범한 RPC다. 클라이언트가
보낼 수 있는 "시각" 자체가 없어졌으니 예전에 걱정했던 "유리한 순간을 골라 보낸다"는
공격 표면은 없어졌지만, 어빌리티의 쿨다운과 탄약 체크를 우회하는 반복 호출은 여전히 별개의
노출이다.

---

## 배운 것

**1. "정석"이라고 적어둔 문장도 검증 없이는 정석이 아니다.**
지난 글에서 "서버가 잰 왕복 시간을 쓰는 게 정석"이라고 적어뒀지만, 정작 그걸 구현하면서
RTT/2로 시작했다가 실측으로 RTT 전체로 고쳤다. 참고 자료(UT)를 그대로 베끼는 것과,
그걸 내 프로젝트의 실측으로 검증하는 것은 다른 일이다.

**2. 두 방향의 오차가 대칭이면, 답은 계산으로 나온다.**
"부족하다"와 "과하다"를 각각 한 번씩 겪고 나서야 정답이 그 중간이라는 걸 알았다. 감으로
세 번째 값을 추측하는 대신, 두 지점을 선형보간했더니 실측과 노이즈 수준으로 맞아떨어졌다.

**3. 업계 표준은 하나가 아니다.**
Source, UT, 오버워치, 발로란트 넷 다 방식이 조금씩 다르다. "이 게임은 이렇게 한다"는
참고는 되지만, 그 게임이 그 방식을 쓸 수 있는 전제(네트워크 인프라, 시계 동기화 정교함)가
내 프로젝트에도 있는지부터 확인해야 한다.

---

## 참고

- [Latency Compensating Methods in Client/Server In-Game Protocol Design (Valve)](https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization)
- [Peeking into VALORANT's Netcode (Riot Games)](https://www.riotgames.com/en/news/peeking-valorants-netcode)
- 엔진/소스 인용: `GameStateBase.cpp`, UE 5.7 (`CharacterMovementComponent.cpp`),
  UnrealTournament 소스(`UTPlayerController.cpp:281-285`, `UTCharacter.cpp:414-435`)
