---
title:  "[UE5] 익스트랙션 슈터: 쿨다운 GE는 시간을 못 맞춘다"
excerpt: "적용은 예측되는데 제거는 예측되지 않는다: 원인을 찾고, 표시를 GE에서 떼어놓기까지"

categories:
  - DevLog
tags:
  - [UE5, C++, GAS, GameplayAbilitySystem, Prediction]

toc: true
toc_sticky: true

mermaid: true

date: 2026-09-13
last_modified_at: 2026-09-13
---

📌 [GAS 기반 전투 시스템](/portfolio/EP_GAS)의 "겪은 문제" 절에서 짧게 요약했던 쿨다운 버그를
실제로 판 기록입니다. 결론부터 말하면 **완전히 고치지 못했습니다.** 표시는 고쳤고, 근본
원인의 구조는 알아냈고, 다음 방향도 정했습니다. 거기까지의 기록입니다.
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)
{: .notice--info}

## 문제

스킬 쿨다운 게이지가 이렇게 움직였다.

```
5.0 → 4.8 → 4.5 → (서버 GE 도착) → 5.0 → 4.9 → ...
```

줄어들다가 **정확히 원래 값으로** 되돌아간다. 4.7이나 4.9처럼 애매한 값이 아니라 항상
처음 값 그대로였다. 그리고 표시만 이상한 게 아니었다. 게이지가 0에 닿아 재사용 가능해
보이는데도, 다시 누르면 아무 일도 안 일어나는 경우가 있었다. 핑이 높을수록 그 창이 길었다.

## 원인 조사: 예측본과 서버본이 따로 있다

`LocalPredicted` 어빌리티는 `CommitAbility` 시점에 클라이언트가 **자기 자신의 쿨다운 GE를
예측해서 미리 만든다.** 그리고 잠시 뒤 서버가 만든 진짜 GE가 복제로 도착한다. 스택 정책이
없는 GE는 둘이 합쳐지지 않으니, 그 겹치는 구간 동안 클라에는 **같은 쿨다운 GE가 두 개**
있다.

두 GE의 시작 시각이 다르다.

| | 시작 시각을 찍은 주체 | 값 |
|---|---|---|
| 예측본 | 클라가 자기 추정 서버 시각으로 | `S0 - D` |
| 서버본 | 서버가 진짜 서버 시각으로 | `S0 + U` |

`U`는 입력이 올라가는 지연, `D`는 결과가 내려오는 지연이다. 그리고 클라이언트가 아는
서버 시각 자체가 `D`만큼 낮게 잡혀 있다.

<details markdown="1">
<summary>서버 시각 추정치 계산 (접기/펼치기)</summary>

```cpp
// GameStateBase.cpp:169
const double ServerWorldTimeDelta = ReplicatedWorldTimeSecondsDouble - World->GetTimeSeconds();
// 비행 시간을 되더해주는 항이 없다
```

</details>

서버본이 도착한 순간, GE는 자기가 얼마나 살았는지 다시 계산한다.

<details markdown="1">
<summary>경과 시간 재계산 (접기/펼치기)</summary>

```cpp
// GameplayEffect.cpp:2950
StartWorldTime = WorldTime - (ServerWorldTime - StartServerWorldTime);
```

</details>

`ServerWorldTime`이 `D`만큼 낮게 잡혀 있으니, 서버본이 실제로 살아 있던 시간(`D`)이
그대로 상쇄되어 경과 = 0이 된다. **서버본은 방금 시작한 것으로 읽힌다.** 그래서 게이지가
정확히 처음 값으로 되돌아갔다. 4.7이 아니라 5.0이었던 이유가 여기 있었다.

## 기각한 안: GE Duration을 핑만큼 깎는다

처음 생각한 해법은 "서버가 재는 핑의 절반만큼 GE Duration을 미리 깎아서, 클라와 서버가
같은 종료 시각을 보게 만든다"였다. 찾아보니 이미 있는 아이디어였다.

[GASDocumentation](https://github.com/tranek/GASDocumentation)를 뒤져보니 Epic이 정확히
이 방향("GE reconciliation")을 시도했다가, 모든 엣지 케이스를 못 막아서 접었다는 기록이
있었다. Fortnite는 아예 무기 발사에서 쿨다운 GE 자체를 버리고 자체 부기(bookkeeping)로
갔다고 적혀 있다.

**엔진을 만든 팀이 시도하고 접은 방향을, 더 적은 리소스로 다시 시도할 이유가 없었다.**
방향을 바꿨다.

## 한 것: 표시를 GE에서 떼어낸다

전체를 고칠 수는 없어도, **표시**는 GE의 시작/종료 시각을 다시 안 읽게 만들면 됐다.

<details markdown="1">
<summary>메시지로 발행하는 쿨다운 (접기/펼치기)</summary>

```cpp
// EPGA_Skill_Base.cpp:85 (GE 적용과 Broadcast가 한 함수 안에 있다)
ApplyGameplayEffectSpecToOwner(CurrentSpecHandle, CurrentActorInfo, CurrentActivationInfo, CDSpec);
BroadcastDurationMessage(CooldownChannelTag, Cooldown);   // Lyra의 GameplayMessageRouter 플러그인
```

```cpp
// EPSkillSlotWidget.cpp:113 (받은 시점이 곧 시작 시점)
CooldownStartTime = World->GetTimeSeconds();
CooldownDuration  = Message.Duration;
```

</details>

`GameplayMessageRouter`는 프로세스 로컬 pub/sub이라 **복제를 안 거친다.**
`GetServerWorldTimeSeconds()`의 편향이 낄 자리가 아예 없다. 위젯은 메시지를 받은 순간
자기 로컬 시계로 시작점을 찍고 자기 타이머로 끝낸다. 예측본이 서버본으로 스왑되는 사건
자체가 표시에 닿을 경로가 없어졌다.

<details markdown="1">
<summary>SetCooldownTag (접기/펼치기)</summary>

```cpp
// EPGA_Skill_Base.cpp:79
void UEPGA_Skill_Base::SetCooldownTag(FGameplayTag Tag)
{
    ActivationBlockedTags.AddTag(Tag);   // 재발동 차단
    CooldownChannelTag = Tag;            // 표시 채널
}
```

</details>

차단 태그와 표시 채널을 한 함수로 묶어서, 둘이 따로 놀 수 없게 했다.

### 곁가지: 예측 창을 열었다고 생각했는데 안 열려 있었다

이 작업 중에 별개의 버그를 하나 더 발견했다. `FScopedPredictionWindow`는 생성자가
두 개인데, 가드 조건이 정반대다.

<details markdown="1">
<summary>가드 조건이 반대인 두 생성자 (접기/펼치기)</summary>

```cpp
// GameplayPrediction.cpp: (ASC, FPredictionKey, bool) 버전
// "Should be called on the server": IsNetSimulating()이 false일 때만 동작 (사실상 서버 전용)

// (ASC, bool bCanGenerateNewKey) 버전
// IsNetSimulating()이 true일 때만 동작 (클라 전용)
```

</details>

`OnCastTimerComplete()`가 원래 서버 전용 생성자를 쓰고 있었다. 시전이 끝나는 시점에
거는 GE(힐, 쉴드, 쿨다운)가 **클라이언트에서는 예측 창이 한 번도 열리지 않았다는 뜻이다.**
조용한 무동작이었다. `UAbilityTask_NetworkSyncPoint::WaitNetSync`로 바꿔서
`FScopedPredictionWindow(ASC, IsPredictingClient())`가 열리는 생성자를 타게 했다.

찾던 버그는 아니었지만, 같은 코드를 들여다보다 나온 수확이었다.

## 비교 연구: Lyra는 이걸 어떻게 하나

방향을 확정하기 전에 Epic 공식 샘플(`LyraStarterGame`)을 직접 열어봤다.

**연속발사 무기엔 쿨다운 GE가 아예 없다.** `LyraGameplayAbility_RangedWeapon.h/.cpp`,
`LyraGameplayAbility.h` 어디에도 `Cooldown`이라는 단어가 한 번도 안 나온다. 연사 간격을
실제로 뭐가 막는지는 C++에서 못 찾았다. 블루프린트 자식 클래스가 있을 가능성이 높은데
텍스트로는 못 본다.

**대신 단발성 캐스트형(수류탄)은 쿨다운 GE를 쓴다.**
`Plugins/GameFeatures/ShooterCore/Content/Weapons/Grenade/GE_Grenade_Cooldown.uasset`이
있고, 에디터로 열어보니 모디파이어/실행이 0개인 순수 게이트 전용 GE였다. 태그 하나를
5초간 걸어두는 것 말고 하는 일이 없다. 다만 Duration은 `SetByCaller`가 아니라 고정
스칼라 5.0이었다. 수류탄은 무기별로 값이 달라질 이유가 없어서일 가능성이 높지만, 이건
추정이다.

정리하면: **Lyra도 지속시간이 상황마다 달라지는 무기 연사엔 쿨다운 GE를 안 쓰고, 고정된
단발 캐스트엔 쓴다.** 이 프로젝트가 가려는 방향(연사 무기부터 GE를 걷어내는 것)과 결이
같았다.

## 정직하게: 안 고쳐진 것

표시는 고쳤다. 근본 원인은 안 고쳐졌다.

| 상태 | 표시 | 재발동 |
|---|---|---|
| 원인 | GE 스왑 순간 경과가 0으로 리셋 | 서버 GE가 살아 있는 동안 `ActivationBlockedTags`가 걸림 |
| 결과 | 게이지가 되돌아감 | 클라 쿨다운이 끝나도 서버 GE 제거를 기다려야 재발동 |
| 이번에 고침 | ✅ (GE를 아예 안 봄) | ❌ |

두 증상은 원인이 다르다. **GE 적용은 예측되지만, GE 제거는 예측되지 않는다.**
`GameplayPrediction.h` 엔진 주석이 예측 대상/비대상을 직접 나열하는데, 제거는 비대상
목록에 있다. 표시는 GE를 안 보게 만들어서 우회할 수 있었지만, 재발동 차단은 태그가
실제로 GE 제거를 기다려야 풀리니 우회할 수 없었다.

### 남은 방향: GE를 버리고 타임스탬프로

```
클라: LastActivationTime (예측, 로컬)
서버: LastActivationTime (권위, 독립 계산)

CanActivateAbility() 안에서:
  GetServerWorldTimeSeconds() - LastActivationTime < Cooldown  →  거부
```

복제되지 않는 `float` 하나면 충분하다. 클라와 서버가 각자 자기 값으로 독립적으로 판단하니
"GE 제거를 기다린다"는 문제 자체가 성립하지 않는다. 무기 연사 쪽에서 이미 검증된 패턴이다.
[핑에 따라 달라지는 연사 속도](/portfolio/EP_GAS) 절에서 어빌리티 하나를 계속 살려두는
방식으로 RPC 수를 줄인 것과 같은 계열의 해법이다.

다만 **누가 판단을 내리는지부터 봐야 한다.** 클라가 그 상태를 읽고 스스로 뭔가를 막거나
허용한다면 GE를 빼도 된다. 클라가 상태만 보여주고 실제 결정은 서버가 한다면, GE(혹은
그에 준하는 복제 값)가 여전히 필요하다. 이 구분을 안 하고 무조건 빼면, 서버만 아는 상태를
클라가 영원히 모르는 상황이 생길 수 있다.

## 배운 것

**1. "쿨다운이 예측 안 된다"는 문장은 절반만 맞다.**
GE *적용*은 예측 대상이다(`ApplyGameplayEffectSpecToOwner`가
`HasAuthorityOrPredictionKey`를 본다). 안 되는 건 GE *제거*다. 이 둘을 구분 못 하면
"왜 쿨다운이 걸리긴 하는데 이상하게 움직이지"에서 계속 헤맨다.

**2. 정석이라고 알려진 방향도 실제로 시도된 적 있는지부터 찾아본다.**
GE Duration을 핑만큼 깎는 안은 그럴듯해 보였지만, 엔진을 만든 팀이 이미 해보고 접은
방향이었다. 그 사실을 모르고 시작했으면 같은 벽에 부딪히는 데 시간을 더 썼을 것이다.

**3. 표시를 고치는 것과 원인을 고치는 것은 다른 작업이다.**
메시지 버스로 표시 문제는 풀렸지만, 재발동 차단은 그대로 남아 있다. "증상이 사라졌다"를
"원인이 없어졌다"로 착각하지 않는 게 중요했다.

## 참고

- [GASDocumentation](https://github.com/tranek/GASDocumentation): GE reconciliation 시도와 Fortnite의 대안
- `Engine/Plugins/Runtime/GameplayAbilities/Public/GameplayPrediction.h`: 예측 대상/비대상 목록
- `LyraStarterGame` 소스: `Source/LyraGame/Weapons/`, `Plugins/GameFeatures/ShooterCore/`
- 엔진 소스 인용: `GameStateBase.cpp:169`, `GameplayEffect.cpp:2950`, `GameplayPrediction.cpp`
