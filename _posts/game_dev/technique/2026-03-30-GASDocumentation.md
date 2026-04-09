---
title:  "[UE5] GASDocumentation 한글 번역"
excerpt: "GASDocumentation"

categories:
  - Techniques
tags:
  - [UE5, GAS]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-03-30
last_modified_at: 2026-03-30
---

🚨 이 문서는 **tranek**님의 **GASDocumentation**을 번역한 것이며,  
저작권은 전적으로 **tranek**님에게 있음을 밝힙니다.  
[👾 깃허브](https://github.com/tranek/GASDocumentation)
{: .notice--warning}

# GASDocumentation
Unreal Engine 5의 GameplayAbilitySystem 플러그인(GAS)에 대한 저의 이해를 간단한 멀티플레이어 샘플 프로젝트와 함께 담았습니다. 이것은 공식 문서가 아니며, 이 프로젝트나 저 자신은 Epic Games와 제휴하지 않습니다. 이 정보의 정확성에 대해 어떠한 보증도 하지 않습니다.

이 문서의 목표는 GAS의 주요 개념과 클래스를 설명하고, 제가 경험한 바에 따른 추가적인 설명을 제공하는 것입니다. 커뮤니티 사용자들 사이에는 GAS에 대한 많은 '부족 지식'이 있으며, 저는 제가 아는 모든 것을 여기에 공유하고자 합니다.

샘플 프로젝트와 문서는 **Unreal Engine 5.3**(UE5)에 맞춰 최신입니다. 이전 버전의 Unreal Engine을 위한 문서 브랜치도 있지만, 더 이상 지원되지 않으며 버그나 오래된 정보가 포함될 수 있습니다. 엔진 버전에 맞는 브랜치를 사용해 주세요.

[GASShooter](https://github.com/tranek/GASShooter)는 멀티플레이어 FPS/TPS를 위한 GAS의 고급 기술을 시연하는 자매 샘플 프로젝트입니다.

최고의 문서는 항상 플러그인 소스 코드일 것입니다.

<a name="table-of-contents"></a>
## 목차

> 1. [GameplayAbilitySystem 플러그인 소개](#intro)
> 1. [샘플 프로젝트](#sp)
> 1. [GAS를 사용하여 프로젝트 설정하기](#setup)
> 1. [개념](#concepts)  
>    4.1 [Ability System Component](#concepts-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.1 [리플리케이션 모드](#concepts-asc-rm)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.2 [설정 및 초기화](#concepts-asc-setup)  
>    4.2 [Gameplay Tags](#concepts-gt)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.1 [Gameplay Tags 변경에 반응하기](#concepts-gt-change)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.2 [플러그인 .ini 파일에서 Gameplay Tags 로드하기](#concepts-gt-loadfromplugin)  
>    4.3 [Attributes](#concepts-a)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.1 [Attribute 정의](#concepts-a-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.2 [BaseValue vs CurrentValue](#concepts-a-value)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.3 [Meta Attributes](#concepts-a-meta)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.4 [Attribute 변경에 반응하기](#concepts-a-changes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.5 [Derived Attributes](#concepts-a-derived)  
>    4.4 [Attribute Set](#concepts-as)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.1 [Attribute Set 정의](#concepts-as-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2 [Attribute Set 설계](#concepts-as-design)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.1 [개별 Attribute를 가진 하위 컴포넌트](#concepts-as-design-subcomponents)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.2 [런타임에 AttributeSet 추가 및 제거하기](#concepts-as-design-addremoveruntime)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3 [아이템 Attribute (무기 탄약)](#concepts-as-design-itemattributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.1 [아이템에 일반 Float 값 사용](#concepts-as-design-itemattributes-plainfloats)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.2 [아이템에 `AttributeSet` 사용](#concepts-as-design-itemattributes-attributeset)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.3 [아이템에 `ASC` 사용](#concepts-as-design-itemattributes-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.3 [Attribute 정의하기](#concepts-as-attributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.4 [Attribute 초기화하기](#concepts-as-init)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.5 [PreAttributeChange()](#concepts-as-preattributechange)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.6 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.7 [OnAttributeAggregatorCreated()](#concepts-as-onattributeaggregatorcreated)  
>    4.5 [Gameplay Effects](#concepts-ge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.1 [Gameplay Effect 정의](#concepts-ge-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.2 [Gameplay Effect 적용하기](#concepts-ge-applying)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.3 [Gameplay Effect 제거하기](#concepts-ga-removing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4 [Gameplay Effect Modifiers](#concepts-ge-mods)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.1 [Multiply 및 Divide Modifiers](#concepts-ge-mods-multiplydivide)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.2 [Modifiers에 Gameplay Tags 사용](#concepts-ge-mods-gameplaytags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.5 [Gameplay Effect 스태킹](#concepts-ge-stacking)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.6 [Granted Abilities](#concepts-ge-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.7 [Gameplay Effect Tags](#concepts-ge-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.8 [면역](#concepts-ge-immunity)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9 [Gameplay Effect Spec](#concepts-ge-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9.1 [SetByCallers](#concepts-ge-spec-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.10 [Gameplay Effect Context](#concepts-ge-context)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.11 [Modifier Magnitude Calculation](#concepts-ge-mmc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12 [Gameplay Effect Execution Calculation](#concepts-ge-ec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1 [Execution Calculation으로 데이터 보내기](#concepts-ge-ec-senddata)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.1 [SetByCaller](#concepts-ge-ec-senddata-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.2 [백킹 데이터 Attribute Calculation Modifier](#concepts-ge-ec-senddata-backingdataattribute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.3 [백킹 데이터 임시 변수 Calculation Modifier](#concepts-ge-ec-senddata-backingdatatempvariable)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.4 [Gameplay Effect Context](#concepts-ge-ec-senddata-effectcontext)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.13 [Custom Application Requirement](#concepts-ge-car)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.14 [Cost Gameplay Effect](#concepts-ge-cost)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15 [Cooldown Gameplay Effect](#concepts-ge-cooldown)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.1 [Cooldown Gameplay Effect의 남은 시간 가져오기](#concepts-ge-cooldown-tr)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.2 [Cooldown 시작 및 종료 감지](#concepts-ge-cooldown-listen)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.3 [Cooldown 예측](#concepts-ge-cooldown-prediction)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.16 [활성화된 Gameplay Effect 지속 시간 변경](#concepts-ge-duration)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.17 [런타임에 동적 Gameplay Effect 생성하기](#concepts-ge-dynamic)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.18 [Gameplay Effect Containers](#concepts-ge-containers)  
>    4.6 [Gameplay Abilities](#concepts-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1 [Gameplay Ability 정의](#concepts-ga-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.1 [리플리케이션 정책](#concepts-ga-definition-reppolicy)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.2 [서버가 원격 Ability 취소를 존중함](#concepts-ga-definition-remotecancel)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.3 [입력 직접 리플리케이트](#concepts-ga-definition-repinputdirectly)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2 [입력을 ASC에 바인딩하기](#concepts-ga-input)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2.1 [Ability 활성화 없이 입력에 바인딩하기](#concepts-ga-input-noactivate)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.3 [Ability 부여하기](#concepts-ga-granting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4 [Ability 활성화하기](#concepts-ga-activating)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.1 [패시브 Ability](#concepts-ga-activating-passive)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.2 [활성화 실패 태그](#concepts-ga-activating-failedtags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.5 [Ability 취소하기](#concepts-ga-cancelabilities)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.6 [활성화된 Ability 가져오기](#concepts-ga-definition-activeability)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.7 [인스턴싱 정책](#concepts-ga-instancing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.8 [네트워크 실행 정책](#concepts-ga-net)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.9 [Ability Tags](#concepts-ga-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.10 [Gameplay Ability Spec](#concepts-ga-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.11 [Ability로 데이터 전달하기](#concepts-ga-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.12 [Ability 비용 및 Cooldown](#concepts-ga-commit)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.13 [Ability 레벨 올리기](#concepts-ga-leveling)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.14 [Ability Sets](#concepts-ga-sets)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.15 [Ability Batching](#concepts-ga-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.16 [네트워크 보안 정책](#concepts-ga-netsecuritypolicy)  
>    4.7 [Ability Tasks](#concepts-at)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.1 [Ability Task 정의](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.2 [커스텀 Ability Tasks](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.3 [Ability Task 사용하기](#concepts-at-using)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.4 [Root Motion Source Ability Tasks](#concepts-at-rms)  
>    4.8 [Gameplay Cues](#concepts-gc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.1 [Gameplay Cue 정의](#concepts-gc-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.2 [Gameplay Cue 트리거하기](#concepts-gc-trigger)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.3 [로컬 Gameplay Cues](#concepts-gc-local)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.4 [Gameplay Cue Parameters](#concepts-gc-parameters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.5 [Gameplay Cue Manager](#concepts-gc-manager)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.6 [Gameplay Cue 발동 방지](#concepts-gc-prevention)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7 [Gameplay Cue Batching](#concepts-gc-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.1 [수동 RPC](#concepts-gc-batching-manualrpc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.2 [하나의 GE에 여러 GC 사용](#concepts-gc-batching-gcsonge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.8 [Gameplay Cue Events](#concepts-gc-events)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.9 [Gameplay Cue 신뢰성](#concepts-gc-reliability)  
>    4.9 [Ability System Globals](#concepts-asg)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.9.1 [InitGlobalData()](#concepts-asg-initglobaldata)  
>    4.10 [예측](#concepts-p)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.1 [예측 키](#concepts-p-key)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.2 [Ability에서 새로운 예측 창 만들기](#concepts-p-windows)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.3 [예측 스폰 Actors](#concepts-p-spawn)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.4 [GAS 예측의 미래](#concepts-p-future)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.5 [네트워크 예측 플러그인](#concepts-p-npp)  
>    4.11 [타겟팅](#concepts-targeting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.1 [Target Data](#concepts-targeting-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.2 [Target Actors](#concepts-targeting-actors)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.3 [Target Data 필터](#concepts-target-data-filters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.4 [Gameplay Ability World Reticles](#concepts-targeting-reticles)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.5 [Gameplay Effect Containers 타겟팅](#concepts-targeting-containers)  
> 1. [일반적으로 구현되는 Ability 및 Effect](#cae)  
>    5.1 [스턴](#cae-stun)  
>    5.2 [스프린트](#cae-sprint)  
>    5.3 [정조준](#cae-ads)  
>    5.4 [생명력 흡수](#cae-ls)  
>    5.5 [클라이언트와 서버에서 무작위 숫자 생성](#cae-random)  
>    5.6 [치명타](#cae-crit)  
>    5.7 [스태킹되지 않는 Gameplay Effect (가장 큰Magnitude만 대상에 영향을 줌)](#cae-nonstackingge)  
>    5.8 [게임 일시정지 중 Target Data 생성](#cae-paused)  
>    5.9 [원 버튼 상호작용 시스템](#cae-onebuttoninteractionsystem)  
> 1. [GAS 디버깅](#debugging)  
>    6.1 [showdebug abilitysystem](#debugging-sd)  
>    6.2 [Gameplay Debugger](#debugging-gd)  
>    6.3 [GAS 로깅](#debugging-log)  
> 1. [최적화](#optimizations)  
>    7.1 [Ability Batching](#optimizations-abilitybatching)  
>    7.2 [Gameplay Cue Batching](#optimizations-gameplaycuebatching)  
>    7.3 [AbilitySystemComponent 리플리케이션 모드](#optimizations-ascreplicationmode)  
>    7.4 [Attribute 프록시 리플리케이션](#optimizations-attributeproxyreplication)  
>    7.5 [ASC 지연 로딩](#optimizations-asclazyloading)  
> 1. [편의성 개선 제안](#qol)  
>    8.1 [Gameplay Effect Containers](#qol-gameplayeffectcontainers)  
>    8.2 [ASC Delegate에 바인딩하기 위한 Blueprint AsyncTasks](#qol-asynctasksascdelegates)  
> 1. [문제 해결](#troubleshooting)  
>    9.1 [`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#troubleshooting-notlocal)  
>    9.2 [`ScriptStructCache` 오류](#troubleshooting-scriptstructcache)  
>    9.3 [애니메이션 몽타주가 클라이언트에 리플리케이트되지 않음](#troubleshooting-replicatinganimmontages)  
>    9.4 [Blueprint Actor 복제 시 AttributeSet이 nullptr로 설정됨](#troubleshooting-duplicatingblueprintactors)  
>    9.5 [unresolved external symbol UEPushModelPrivate::MarkPropertyDirty(int,int)](#troubleshooting-unresolvedexternalsymbolmarkpropertydirty)  
>    9.6 [Enum 이름이 이제 경로 이름으로 표현됨](#troubleshooting-enumnamesarenowpathnames)  
> 1. [일반적인 GAS 약어](#acronyms)
> 1. [다른 자료](#resources)  
>    11.1 [Epic Game의 Dave Ratti와의 Q&A](#resources-daveratti)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.1 [커뮤니티 질문 1](#resources-daveratti-community1)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.2 [커뮤니티 질문 2](#resources-daveratti-community2)  
> 1. [GAS 변경 로그](#changelog)  
>    * [5.3](#changelog-5.3)  
>    * [5.2](#changelog-5.2)  
>    * [5.1](#changelog-5.1)  
>    * [5.0](#changelog-5.0)  
>    * [4.27](#changelog-4.27)  
>    * [4.26](#changelog-4.26)  
>    * [4.25.1](#changelog-4.25.1)  
>    * [4.25](#changelog-4.25)  
>    * [4.24](#changelog-4.24)
         
<a name="intro"></a>
## 1. GameplayAbilitySystem 플러그인 소개
[공식 문서](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)에서 발췌:
>Gameplay Ability System은 RPG나 MOBA 타이틀에서 볼 수 있는 유형의 능력과 속성을 구축하기 위한 매우 유연한 프레임워크입니다. 게임 내 캐릭터가 사용할 액션 또는 패시브 능력을 구축하고, 이러한 액션의 결과로 다양한 속성을 쌓거나 소모하는 상태 효과를 구현하고, 이러한 액션의 사용을 조절하기 위한 "쿨다운" 타이머 또는 자원 비용을 구현하며, 능력의 레벨과 각 레벨에서의 효과를 변경하고, 파티클 또는 사운드 효과를 활성화하는 등 다양한 작업을 수행할 수 있습니다. 간단히 말해, 이 시스템은 점프와 같은 간단한 능력부터 현대 RPG 또는 MOBA 타이틀에서 가장 좋아하는 캐릭터의 능력 세트만큼 복잡한 인게임 능력을 설계, 구현 및 효율적으로 네트워크 처리하는 데 도움을 줄 수 있습니다.

GameplayAbilitySystem 플러그인은 Epic Games에서 개발하며 Unreal Engine에 포함되어 있습니다. 이는 Paragon 및 Fortnite와 같은 AAA 상업 게임에서 전투 테스트를 거쳤습니다.

이 플러그인은 싱글 플레이어 및 멀티 플레이어 게임에서 다음과 같은 기능에 대한 즉시 사용 가능한 솔루션을 제공합니다:
*   선택적 비용 및 쿨다운을 포함하는 레벨 기반 캐릭터 능력 또는 스킬 구현 ([GameplayAbilities](#concepts-ga))
*   액터에 속한 숫자 `Attributes` 조작 ([Attributes](#concepts-a))
*   액터에 상태 효과 적용 ([GameplayEffects](#concepts-ge))
*   액터에 `GameplayTags` 적용 ([GameplayTags](#concepts-gt))
*   시각 또는 음향 효과 생성 ([GameplayCues](#concepts-gc))
*   위에 언급된 모든 것의 리플리케이션

멀티플레이어 게임에서 GAS는 다음 항목에 대한 [클라이언트 측 예측](#concepts-p)을 지원합니다:
*   Ability 활성화
*   애니메이션 몽타주 재생
*   `Attributes` 변경
*   `GameplayTags` 적용
*   `GameplayCues` 생성
*   `CharacterMovementComponent`에 연결된 `RootMotionSource` 기능을 통한 이동.

**GAS는 C++로 설정해야 하지만**, `GameplayAbilities` 및 `GameplayEffects`는 디자이너가 Blueprint에서 생성할 수 있습니다.

GAS의 현재 문제점:
*   `GameplayEffect` 지연 시간 조정 (`Ability` 쿨다운을 예측할 수 없어, 지연 시간이 높은 플레이어가 지연 시간이 낮은 플레이어에 비해 쿨다운이 짧은 `Ability`의 발사 속도가 낮아지는 결과를 초래합니다).
*   `GameplayEffects` 제거를 예측할 수 없습니다. 그러나 역 효과를 가진 `GameplayEffects` 추가를 예측하여 사실상 제거할 수 있습니다. 이는 항상 적절하거나 실행 가능한 것은 아니며 여전히 문제로 남아 있습니다.
*   상용구 템플릿, 멀티플레이어 예제 및 문서 부족. 이 문서가 도움이 되기를 바랍니다!

**[⬆ 맨 위로](#table-of-contents)**

<a name="sp"></a>
## 2. 샘플 프로젝트
이 문서에는 GameplayAbilitySystem 플러그인을 처음 사용하지만 Unreal Engine에는 익숙한 사용자를 위한 멀티플레이어 3인칭 슈터 샘플 프로젝트가 포함되어 있습니다. 사용자는 C++, Blueprint, UMG, 리플리케이션 및 UE의 기타 중급 주제를 알고 있을 것으로 예상됩니다. 이 프로젝트는 플레이어/AI 제어 영웅에 대한 `PlayerState` 클래스의 `AbilitySystemComponent`(`ASC`)와 AI 제어 미니언에 대한 `Character` 클래스의 `ASC`를 사용하여 기본적인 3인칭 슈터 멀티플레이어 준비 프로젝트를 설정하는 방법을 보여주는 예제를 제공합니다.

목표는 GAS 기본 사항을 보여주고 잘 주석 처리된 코드로 일반적으로 요청되는 일부 `Ability`를 시연하면서 이 프로젝트를 간단하게 유지하는 것입니다. 초보자에게 초점을 맞추고 있기 때문에, 이 프로젝트는 [투사체 예측](#concepts-p-spawn)과 같은 고급 주제는 보여주지 않습니다.

시연된 개념:
*   `PlayerState` vs `Character`에 `ASC`
*   리플리케이트된 `Attributes`
*   리플리케이트된 애니메이션 몽타주
*   `GameplayTags`
*   `GameplayAbilities` 내부 및 외부에서 `GameplayEffects` 적용 및 제거
*   방어력으로 완화된 피해를 적용하여 캐릭터의 체력 변경
*   `GameplayEffectExecutionCalculations`
*   스턴 효과
*   사망 및 리스폰
*   서버의 `Ability`에서 액터(투사체) 생성
*   조준 시야와 스프린트로 로컬 플레이어 속도 예측 변경
*   스프린트를 위해 꾸준히 스태미나 소모
*   `Ability` 시전에 마나 사용
*   패시브 `Ability`
*   `GameplayEffects` 스태킹
*   액터 타겟팅
*   Blueprint에서 생성된 `GameplayAbilities`
*   C++에서 생성된 `GameplayAbilities`
*   액터별 인스턴스화된 `GameplayAbilities`
*   비인스턴스화된 `GameplayAbilities` (점프)
*   정적 `GameplayCues` (FireGun 투사체 충격 파티클 효과)
*   액터 `GameplayCues` (스프린트 및 스턴 파티클 효과)

영웅 클래스는 다음 `Ability`를 가집니다:

| Ability | 입력 바인딩 | 예측됨 | C++ / Blueprint | 설명 |
|---|---|---|---|---|
| 점프 | 스페이스 바 | 예 | C++ | 영웅을 점프시킵니다. |
| 총 | 마우스 왼쪽 버튼 | 아니요 | C++ | 영웅의 총에서 투사체를 발사합니다. 애니메이션은 예측되지만 투사체는 예측되지 않습니다. |
| 정조준 | 마우스 오른쪽 버튼 | 예 | Blueprint | 버튼을 누르고 있는 동안 영웅은 더 느리게 걷고 카메라가 확대되어 총으로 더 정확하게 사격할 수 있습니다. |
| 스프린트 | Left Shift | 예 | Blueprint | 버튼을 누르고 있는 동안 영웅은 더 빨리 달리며 스태미나를 소모합니다. |
| 전방 대시 | Q | 예 | Blueprint | 영웅이 스태미나를 소모하며 앞으로 대시합니다. |
| 패시브 방어력 스택 | 패시브 | 아니요 | Blueprint | 4초마다 영웅은 최대 4스택까지 방어력 스택을 얻습니다. 피해를 받으면 방어력 스택 하나가 제거됩니다. |
| 유성 | R | 아니요 | Blueprint | 플레이어가 적에게 유성을 떨어뜨려 피해를 주고 기절시키는 위치를 조준합니다. 타겟팅은 예측되지만 유성 스폰은 예측되지 않습니다. |

`GameplayAbilities`가 C++ 또는 Blueprint로 생성되는지는 중요하지 않습니다. 여기서는 각 언어로 구현하는 방법을 예시하기 위해 두 가지를 혼합하여 사용했습니다.

미니언은 사전 정의된 `GameplayAbilities`를 가지고 있지 않습니다. 빨간 미니언은 체력 재생이 더 높고 파란 미니언은 시작 체력이 더 높습니다.

`GameplayAbility` 이름 지정의 경우, `_BP` 접미사를 사용하여 `GameplayAbility`의 로직이 Blueprint로 생성되었음을 나타냈습니다. 접미사가 없으면 로직이 C++로 생성되었음을 의미합니다.

**Blueprint 애셋 이름 접두사**

| 접두사 | 애셋 유형 |
|---|---|
| GA_ | GameplayAbility |
| GC_ | GameplayCue |
| GE_ | GameplayEffect |

**[⬆ 맨 위로](#table-of-contents)**

<a name="setup"></a>
## 3. GAS를 사용하여 프로젝트 설정하기
GAS를 사용하여 프로젝트를 설정하는 기본 단계:
1.  에디터에서 GameplayAbilitySystem 플러그인 활성화
1.  `YourProjectName.Build.cs` 파일을 편집하여 `PrivateDependencyModuleNames`에 `"GameplayAbilities", "GameplayTags", "GameplayTasks"`를 추가
1.  Visual Studio 프로젝트 파일 새로고침/재생성
1.  4.24부터 5.2까지 [`TargetData`](#concepts-targeting-data)를 사용하려면 `UAbilitySystemGlobals::Get().InitGlobalData()`를 호출해야 합니다. 샘플 프로젝트는 `UAssetManager::StartInitialLoading()`에서 이 작업을 수행합니다. 이는 5.3부터 자동으로 호출됩니다. 자세한 내용은 [`InitGlobalData()`](#concepts-asg-initglobaldata)를 참조하십시오.

이것이 GAS를 활성화하기 위해 해야 할 전부입니다. 이제 `Character` 또는 `PlayerState`에 [`ASC`](#concepts-asc)와 [`AttributeSet`](#concepts-as)을 추가하고 [`GameplayAbilities`](#concepts-ga)와 [`GameplayEffects`](#concepts-ge)를 만들기 시작하세요!

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts"></a>
## 4. GAS 개념

#### 섹션

> 4.1 [Ability System Component](#concepts-asc)  
> 4.2 [Gameplay Tags](#concepts-gt)  
> 4.3 [Attributes](#concepts-a)  
> 4.4 [Attribute Set](#concepts-as)  
> 4.5 [Gameplay Effects](#concepts-ge)  
> 4.6 [Gameplay Abilities](#concepts-ga)  
> 4.7 [Ability Tasks](#concepts-at)  
> 4.8 [Gameplay Cues](#concepts-gc)  
> 4.9 [Ability System Globals](#concepts-asg)  
> 4.10 [예측](#concepts-p)

<a name="concepts-asc"></a>
### 4.1 Ability System Component
`AbilitySystemComponent`(`ASC`)는 GAS의 핵심입니다. 이는 시스템과의 모든 상호 작용을 처리하는 `UActorComponent`([`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html))입니다. [`GameplayAbilities`](#concepts-ga)를 사용하거나, [`Attributes`](#concepts-a)를 가지거나, [`GameplayEffects`](#concepts-ge)를 받으려는 모든 `Actor`는 `ASC`를 하나씩 부착해야 합니다. 이 모든 객체는 `ASC` 내부에 존재하며 `ASC`에 의해 관리 및 복제됩니다 (단, `Attribute`는 [`AttributeSet`](#concepts-as)에 의해 복제됩니다). 개발자는 이를 서브클래싱할 것으로 예상되지만 필수는 아닙니다.

`ASC`가 부착된 `Actor`는 `ASC`의 `OwnerActor`라고 불립니다. `ASC`의 물리적 표현 `Actor`는 `AvatarActor`라고 불립니다. `OwnerActor`와 `AvatarActor`는 MOBA 게임의 간단한 AI 미니언의 경우와 같이 동일한 `Actor`일 수 있습니다. 또한 `OwnerActor`가 `PlayerState`이고 `AvatarActor`가 영웅의 `Character` 클래스인 MOBA 게임의 플레이어 제어 영웅의 경우와 같이 다른 `Actor`일 수도 있습니다. 대부분의 `Actor`는 자신에게 `ASC`를 가지고 있을 것입니다. 만약 `Actor`가 리스폰되고 리스폰 간에 `Attribute` 또는 `GameplayEffect`의 지속성이 필요하다면 (MOBA의 영웅처럼), `ASC`의 이상적인 위치는 `PlayerState`입니다.

**참고:** `ASC`가 `PlayerState`에 있는 경우, `PlayerState`의 `NetUpdateFrequency`를 늘려야 합니다. 기본적으로 `PlayerState`에서는 매우 낮은 값으로 설정되어 있으며, `Attribute` 및 `GameplayTags`와 같은 변경 사항이 클라이언트에서 발생하는 데 지연 또는 인지된 지연을 유발할 수 있습니다. [`Adaptive Network Update Frequency`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)를 활성화해야 합니다. Fortnite는 이를 사용합니다.

`OwnerActor`와 `AvatarActor`가 다른 `Actor`인 경우, 둘 다 `IAbilitySystemInterface`를 구현해야 합니다. 이 인터페이스는 `UAbilitySystemComponent* GetAbilitySystemComponent() const`라는 하나의 함수를 오버라이드해야 하며, 이는 해당 `ASC`에 대한 포인터를 반환합니다. `ASC`는 이 인터페이스 함수를 찾아 시스템 내부에서 서로 상호 작용합니다.

`ASC`는 현재 활성화된 `GameplayEffects`를 `FActiveGameplayEffectsContainer ActiveGameplayEffects`에 보유합니다.

`ASC`는 부여된 `Gameplay Abilities`를 `FGameplayAbilitySpecContainer ActivatableAbilities`에 보유합니다. `ActivatableAbilities.Items`를 반복할 계획이라면, 루프 위에 `ABILITYLIST_SCOPE_LOCK();`를 추가하여 목록이 변경되지 않도록 잠그십시오 (`Ability` 제거로 인해). 스코프 내의 모든 `ABILITYLIST_SCOPE_LOCK();`는 `AbilityScopeLockCount`를 증가시키고, 스코프를 벗어나면 감소시킵니다. `ABILITYLIST_SCOPE_LOCK();` 스코프 내에서 `Ability`를 제거하려고 시도하지 마십시오 (능력 제거 함수는 목록이 잠겨 있으면 `Ability` 제거를 방지하기 위해 내부적으로 `AbilityScopeLockCount`를 확인합니다).

<a name="concepts-asc-rm"></a>
### 4.1.1 리플리케이션 모드
`ASC`는 `GameplayEffects`, `GameplayTags`, `GameplayCues`를 리플리케이션하기 위한 세 가지 다른 리플리케이션 모드 - `Full`, `Mixed`, `Minimal` -를 정의합니다. `Attribute`는 해당 `AttributeSet`에 의해 리플리케이션됩니다.

| 리플리케이션 모드 | 사용 시점 | 설명 |
|---|---|---|
| `Full` | 싱글 플레이어 | 모든 `GameplayEffect`가 모든 클라이언트에 리플리케이션됩니다. |
| `Mixed` | 멀티플레이어, 플레이어 제어 `Actors` | `GameplayEffects`는 소유 클라이언트에만 리플리케이션됩니다. `GameplayTags`와 `GameplayCues`만 모두에게 리플리케이션됩니다. |
| `Minimal` | 멀티플레이어, AI 제어 `Actors` | `GameplayEffects`는 아무에게도 리플리케이션되지 않습니다. `GameplayTags`와 `GameplayCues`만 모두에게 리플리케이션됩니다. |

**참고:** `Mixed` 리플리케이션 모드는 `OwnerActor`의 `Owner`가 `Controller`일 것으로 예상합니다. `PlayerState`의 `Owner`는 기본적으로 `Controller`이지만, `Character`의 `Owner`는 그렇지 않습니다. `PlayerState`가 아닌 `OwnerActor`와 함께 `Mixed` 리플리케이션 모드를 사용하는 경우, 유효한 `Controller`를 사용하여 `OwnerActor`에 대해 `SetOwner()`를 호출해야 합니다.

4.24부터 `PossessedBy()`는 이제 `Pawn`의 소유자를 새로운 `Controller`로 설정합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-asc-setup"></a>
### 4.1.2 설정 및 초기화
`ASC`는 일반적으로 `OwnerActor`의 생성자에서 생성되고 명시적으로 복제되도록 표시됩니다. **이것은 C++로 해야 합니다**.

```c++
AGDPlayerState::AGDPlayerState()
{
	// Ability System Component를 생성하고 명시적으로 복제되도록 설정
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

`ASC`는 서버와 클라이언트 모두에서 `OwnerActor`와 `AvatarActor`로 초기화되어야 합니다. `Pawn`의 `Controller`가 설정된 후 (빙의 후) 초기화해야 합니다. 싱글 플레이어 게임은 서버 경로만 신경 쓰면 됩니다.

`ASC`가 `Pawn`에 있는 플레이어 제어 캐릭터의 경우, 일반적으로 `Pawn`의 `PossessedBy()` 함수에서 서버를 초기화하고, `PlayerController`의 `AcknowledgePossession()` 함수에서 클라이언트를 초기화합니다.

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC MixedMode 리플리케이션은 ASC Owner의 Owner가 Controller여야 함을 요구합니다.
	SetOwner(NewController);
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

`ASC`가 `PlayerState`에 있는 플레이어 제어 캐릭터의 경우, 일반적으로 `Pawn`의 `PossessedBy()` 함수에서 서버를 초기화하고, `Pawn`의 `OnRep_PlayerState()` 함수에서 클라이언트를 초기화합니다. 이렇게 하면 `PlayerState`가 클라이언트에 존재하는지 확인할 수 있습니다.

```c++
// 서버 전용
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 서버에서 ASC 설정. 클라이언트는 OnRep_PlayerState()에서 이 작업을 수행합니다.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI는 PlayerController가 없을 것이므로, 여기서 다시 초기화하여 확실히 합니다. PlayerController가 있는 영웅의 경우 두 번 초기화해도 해롭지 않습니다.
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```c++
// 클라이언트 전용
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 클라이언트를 위한 ASC 설정. 서버는 PossessedBy에서 이 작업을 수행합니다.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// 클라이언트를 위한 ASC Actor Info 초기화. 서버는 새로운 Actor를 빙의할 때 ASC를 초기화합니다.
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!` 오류 메시지가 표시되면 클라이언트에서 `ASC`를 초기화하지 않은 것입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gt"></a>
### 4.2 Gameplay Tags
[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html)는 `GameplayTagManager`에 등록되는 `Parent.Child.Grandchild...` 형식의 계층적 이름입니다. 이 태그는 객체의 상태를 분류하고 설명하는 데 매우 유용합니다. 예를 들어, 캐릭터가 기절 상태인 경우, 기절 지속 시간 동안 `State.Debuff.Stun` `GameplayTag`를 부여할 수 있습니다.

과거에 부울이나 Enum으로 처리했던 것들을 `GameplayTags`로 대체하고, 객체가 특정 `GameplayTags`를 가지고 있는지 여부에 따라 부울 논리를 수행하게 될 것입니다.

객체에 태그를 부여할 때는 일반적으로 `ASC`가 있는 경우 `ASC`에 추가하여 GAS가 태그와 상호 작용할 수 있도록 합니다. `UAbilitySystemComponent`는 `IGameplayTagAssetInterface`를 구현하여 소유한 `GameplayTags`에 접근하는 함수를 제공합니다.

여러 `GameplayTags`는 `FGameplayTagContainer`에 저장될 수 있습니다. `GameplayTagContainer`는 몇 가지 효율적인 마법을 추가하므로 `TArray<FGameplayTag>`보다 `GameplayTagContainer`를 사용하는 것이 좋습니다. 태그는 표준 `FNames`이지만, 프로젝트 설정에서 `Fast Replication`이 활성화된 경우 `FGameplayTagContainers`에 효율적으로 패킹되어 복제될 수 있습니다. `Fast Replication`은 서버와 클라이언트가 동일한 `GameplayTags` 목록을 가지고 있어야 합니다. 이는 일반적으로 문제가 되지 않으므로 이 옵션을 활성화해야 합니다. `GameplayTagContainers`는 반복을 위해 `TArray<FGameplayTag>`를 반환할 수도 있습니다.

`FGameplayTagCountContainer`에 저장된 `GameplayTags`는 해당 `GameplayTag`의 인스턴스 수를 저장하는 `TagMap`을 가집니다. `FGameplayTagCountContainer`는 여전히 `GameplayTag`를 포함할 수 있지만 `TagMapCount`는 0입니다. `ASC`가 여전히 `GameplayTag`를 가지고 있는 경우 디버깅 중에 이를 발견할 수 있습니다. `HasTag()` 또는 `HasMatchingTag()` 또는 유사한 함수는 `TagMapCount`를 확인하고 `GameplayTag`가 없거나 `TagMapCount`가 0이면 false를 반환합니다.

`GameplayTags`는 `DefaultGameplayTags.ini`에 미리 정의되어야 합니다. Unreal Engine Editor는 프로젝트 설정에서 `GameplayTags`를 수동으로 편집할 필요 없이 개발자가 `GameplayTags`를 관리할 수 있는 인터페이스를 제공합니다. `GameplayTag` 에디터는 `GameplayTags`를 생성, 이름 변경, 참조 검색 및 삭제할 수 있습니다.

![GameplayTag Editor in Project Settings](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

`GameplayTag` 참조를 검색하면 에디터에 익숙한 `Reference Viewer` 그래프가 나타나 `GameplayTag`를 참조하는 모든 애셋을 보여줍니다. 그러나 이것은 `GameplayTag`를 참조하는 C++ 클래스는 표시하지 않습니다.

`GameplayTags`의 이름을 변경하면 리디렉션이 생성되어 원래 `GameplayTag`를 참조하는 애셋이 새 `GameplayTag`로 리디렉션될 수 있습니다. 가능하면 대신 새 `GameplayTag`를 생성하고, 모든 참조를 새 `GameplayTag`로 수동으로 업데이트한 다음, 이전 `GameplayTag`를 삭제하여 리디렉션을 생성하지 않는 것이 좋습니다.

`Fast Replication` 외에도 `GameplayTag` 에디터에는 일반적으로 복제되는 `GameplayTags`를 채워 추가로 최적화하는 옵션이 있습니다.

`GameplayTags`는 `GameplayEffect`에서 추가된 경우 복제됩니다. `ASC`는 복제되지 않으며 수동으로 관리해야 하는 `LooseGameplayTags`를 추가할 수 있도록 허용합니다. 샘플 프로젝트는 `State.Dead`에 `LooseGameplayTag`를 사용하여 소유 클라이언트가 체력이 0으로 떨어질 때 즉시 반응할 수 있도록 합니다. 리스폰 시 `TagMapCount`는 수동으로 0으로 재설정됩니다. `LooseGameplayTags`로 작업할 때만 `TagMapCount`를 수동으로 조정하십시오. `TagMapCount`를 수동으로 조정하는 것보다 `UAbilitySystemComponent::AddLooseGameplayTag()` 및 `UAbilitySystemComponent::RemoveLooseGameplayTag()` 함수를 사용하는 것이 좋습니다.

C++에서 `GameplayTag`에 대한 참조 얻기:
```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

부모 또는 자식 `GameplayTags`를 얻는 것과 같은 고급 `GameplayTag` 조작을 위해서는 `GameplayTagManager`에서 제공하는 함수들을 살펴보십시오. `GameplayTagManager`에 접근하려면 `GameplayTagManager.h`를 포함하고 `UGameplayTagManager::Get().FunctionName`으로 호출하십시오. `GameplayTagManager`는 `GameplayTags`를 지속적인 문자열 조작 및 비교보다 빠른 처리를 위해 관계형 노드(부모, 자식 등)로 실제로 저장합니다.

`GameplayTags` 및 `GameplayTagContainers`는 `Meta = (Categories = "GameplayCue")` `UPROPERTY` 지정자를 가질 수 있으며, 이는 Blueprint에서 `GameplayTag`가 `GameplayCue`의 부모 태그를 가진 `GameplayTags`만 표시하도록 필터링합니다. 이는 `GameplayTag` 또는 `GameplayTagContainer` 변수가 `GameplayCues`에만 사용되어야 한다는 것을 알 때 유용합니다.

대안으로, `FGameplayTag`를 캡슐화하고 Blueprint에서 `GameplayTag`를 `GameplayCue`의 부모 태그를 가진 태그만 표시하도록 자동으로 필터링하는 `FGameplayCueTag`라는 별도의 구조체가 있습니다.

함수에서 `GameplayTag` 매개변수를 필터링하려면 `UFUNCTION` 지정자 `Meta = (GameplayTagFilter = "GameplayCue")`를 사용하십시오. 함수 내의 `GameplayTagContainer` 매개변수는 필터링할 수 없습니다. 이를 허용하도록 엔진을 편집하고 싶다면 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp`의 `SGameplayTagGraphPin::ParseDefaultValueData()`가 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);`를 호출하고 `SGameplayTagGraphPin::GetListContent()`에서 `FilterString`을 `SGameplayTagWidget`에 전달하는 방식을 살펴보십시오. `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp`에 있는 이 함수들의 `GameplayTagContainer` 버전은 메타 필드 속성을 확인하지 않고 필터를 전달합니다.

샘플 프로젝트는 `GameplayTags`를 광범위하게 사용합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gt-change"></a>
### 4.2.1 Gameplay Tags 변경에 반응하기
`ASC`는 `GameplayTags`가 추가되거나 제거될 때를 위한 델리게이트를 제공합니다. 이는 `GameplayTag`가 추가/제거될 때만 또는 `GameplayTag`의 `TagMapCount`의 변경에 대해 발동하도록 지정할 수 있는 `EGameplayTagEventType`를 인수로 받습니다.

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

콜백 함수는 `GameplayTag`와 새로운 `TagCount`에 대한 매개변수를 가집니다.
```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gt-loadfromplugin"></a>
### 4.2.2 플러그인 .ini 파일에서 Gameplay Tags 로드하기
`GameplayTags`가 포함된 자체 .ini 파일을 가진 플러그인을 생성하는 경우, 해당 플러그인의 `StartupModule()` 함수에서 해당 플러그인의 `GameplayTag` .ini 디렉토리를 로드할 수 있습니다.

예를 들어, Unreal Engine에 포함된 CommonConversation 플러그인은 다음과 같이 수행합니다:

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());
	
	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

이렇게 하면 엔진이 시작될 때 플러그인이 활성화된 경우 `Plugins\CommonConversation\Config\Tags` 디렉토리를 찾아 `GameplayTags`가 포함된 .ini 파일을 프로젝트에 로드합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-a"></a>
### 4.3 Attributes

<a name="concepts-a-definition"></a>
#### 4.3.1 Attribute 정의
`Attributes`는 [`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html) 구조체로 정의된 float 값입니다. 이 값들은 캐릭터가 가진 체력의 양부터 캐릭터의 레벨, 물약의 충전 횟수까지 모든 것을 나타낼 수 있습니다. `Actor`에 속한 게임 플레이 관련 숫자 값이라면 `Attribute`를 사용하는 것을 고려해야 합니다. `Attribute`는 일반적으로 [`GameplayEffects`](#concepts-ge)에 의해서만 수정되어야 `ASC`가 변경 사항을 [예측](#concepts-p)할 수 있습니다.

`Attribute`는 [`AttributeSet`](#concepts-as)에 의해 정의되고, 유지되며, 변경 사항이 관리됩니다. `AttributeSet`은 복제용으로 표시된 `Attribute`를 복제하는 역할을 합니다. `Attribute`를 정의하는 방법에 대한 내용은 [`AttributeSets`](#concepts-as) 섹션을 참조하십시오.

**팁:** 에디터의 `Attributes` 목록에 `Attribute`가 표시되지 않도록 하려면 `Meta = (HideInDetailsView)` `property specifier`를 사용할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-a-value"></a>
#### 4.3.2 BaseValue vs CurrentValue
`Attribute`는 `BaseValue`와 `CurrentValue` 두 가지 값으로 구성됩니다. `BaseValue`는 `Attribute`의 영구적인 값이며, `CurrentValue`는 `BaseValue`에 `GameplayEffects`로 인한 임시 수정 사항이 추가된 값입니다. 예를 들어, `Character`가 이동 속도 `Attribute`를 가지고 있고 `BaseValue`가 초당 600단위라고 가정해봅시다. 아직 이동 속도를 수정하는 `GameplayEffects`가 없으므로 `CurrentValue`도 초당 600단위입니다. 만약 그녀가 일시적으로 초당 50단위의 이동 속도 버프를 받으면 `BaseValue`는 600단위로 동일하게 유지되는 반면, `CurrentValue`는 이제 600 + 50으로 총 650단위가 됩니다. 이동 속도 버프가 만료되면 `CurrentValue`는 `BaseValue`인 600단위로 되돌아갑니다.

GAS 초보자들은 종종 `BaseValue`를 `Attribute`의 최대값으로 오해하고 그렇게 다루려고 합니다. 이것은 잘못된 접근 방식입니다. 변경될 수 있거나 `Ability`나 UI에서 참조되는 `Attribute`의 최대값은 별도의 `Attribute`로 처리해야 합니다. 하드코딩된 최대 및 최소값의 경우, `FAttributeMetaData`를 가진 `DataTable`을 정의하여 최대 및 최소값을 설정하는 방법이 있지만, Epic의 구조체 위 주석에서는 "진행 중인 작업"이라고 부릅니다. 자세한 내용은 `AttributeSet.h`를 참조하십시오. 혼동을 방지하기 위해, `Ability`나 UI에서 참조될 수 있는 최대값은 별도의 `Attribute`로 만들고, `Attribute` 클램핑에만 사용되는 하드코딩된 최대 및 최소값은 `AttributeSet`에 하드코딩된 float 값으로 정의하는 것을 권장합니다. `Attribute`의 클램핑은 `CurrentValue` 변경에 대한 [PreAttributeChange()](#concepts-as-preattributechange)와 `GameplayEffects`로 인한 `BaseValue` 변경에 대한 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)에서 논의됩니다.

`BaseValue`에 대한 영구적인 변경은 `Instant` `GameplayEffects`에서 발생하며, `Duration` 및 `Infinite` `GameplayEffects`는 `CurrentValue`를 변경합니다. 주기적인 `GameplayEffects`는 `Instant` `GameplayEffects`처럼 처리되며 `BaseValue`를 변경합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-a-meta"></a>
#### 4.3.3 Meta Attributes
일부 `Attributes`는 `Attributes`와 상호 작용하기 위한 임시 값의 자리 표시자로 취급됩니다. 이를 `Meta Attributes`라고 합니다. 예를 들어, 우리는 일반적으로 피해를 `Meta Attribute`로 정의합니다. `GameplayEffect`가 직접 체력 `Attribute`를 변경하는 대신, 피해라는 `Meta Attribute`를 자리 표시자로 사용합니다. 이런 방식으로 피해 값은 [`GameplayEffectExecutionCalculation`](#concepts-ge-ec)에서 버프 및 디버프로 수정될 수 있으며, `AttributeSet`에서 추가로 조작될 수 있습니다. 예를 들어, 현재 보호막 `Attribute`에서 피해를 뺀 다음, 남은 값을 체력 `Attribute`에서 최종적으로 빼는 식입니다. 피해 `Meta Attribute`는 `GameplayEffects` 간에 지속성이 없으며, 모든 `GameplayEffect`에 의해 덮어쓰여집니다. `Meta Attributes`는 일반적으로 복제되지 않습니다.

`Meta Attributes`는 피해 및 치유와 같은 것들에 대해 "얼마나 피해를 입혔는가?"와 "이 피해로 무엇을 할 것인가?" 사이에 좋은 논리적 분리를 제공합니다. 이러한 논리적 분리는 우리의 `Gameplay Effects`와 `Execution Calculations`가 대상이 피해를 어떻게 처리하는지 알 필요가 없음을 의미합니다. 피해 예시를 계속하면, `Gameplay Effect`가 피해량을 결정하고 `AttributeSet`이 해당 피해로 무엇을 할지 결정합니다. 모든 캐릭터가 동일한 `Attributes`를 가지고 있지 않을 수 있으며, 특히 서브클래스화된 `AttributeSets`를 사용하는 경우 더욱 그렇습니다. 기본 `AttributeSet` 클래스는 체력 `Attribute`만 가질 수 있지만, 서브클래스화된 `AttributeSet`은 보호막 `Attribute`를 추가할 수 있습니다. 보호막 `Attribute`를 가진 서브클래스화된 `AttributeSet`은 기본 `AttributeSet` 클래스와 다르게 받은 피해를 분배할 것입니다.

`Meta Attributes`는 좋은 디자인 패턴이지만 필수는 아닙니다. 모든 피해 인스턴스에 하나의 `Execution Calculation`만 사용하고 모든 캐릭터가 공유하는 하나의 `Attribute Set` 클래스만 사용하는 경우, `Execution Calculation` 내부에서 체력, 보호막 등에 피해 분배를 수행하고 해당 `Attributes`를 직접 수정해도 괜찮을 수 있습니다. 유연성만 희생할 뿐이지만, 그것이 당신에게 괜찮을 수도 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-a-changes"></a>
#### 4.3.4 Attribute 변경에 반응하기
UI 또는 다른 게임 플레이를 업데이트하기 위해 `Attribute`가 변경될 때를 감지하려면 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`를 사용하십시오. 이 함수는 `Attribute`가 변경될 때마다 자동으로 호출되는 델리게이트를 반환하며, `NewValue`, `OldValue`, `FGameplayEffectModCallbackData` 매개변수를 가진 `FOnAttributeChangeData`를 제공합니다. **참고:** `FGameplayEffectModCallbackData`는 서버에서만 설정됩니다.

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

샘플 프로젝트는 `GDPlayerState`의 `Attribute` 값 변경 델리게이트에 바인딩하여 HUD를 업데이트하고 체력이 0이 되었을 때 플레이어의 죽음에 반응합니다.

`ASyncTask`로 래핑하는 사용자 지정 Blueprint 노드가 샘플 프로젝트에 포함되어 있습니다. 이는 `UI_HUD` UMG 위젯에서 체력, 마나, 스태미나 값을 업데이트하는 데 사용됩니다. 이 `AsyncTask`는 UMG 위젯의 `Destruct` 이벤트에서 호출하는 `EndTask()`가 수동으로 호출될 때까지 영원히 지속됩니다. `AsyncTaskAttributeChanged.h/cpp`를 참조하십시오.

![Listen for Attribute Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-a-derived"></a>
#### 4.3.5 Derived Attributes
하나 이상의 다른 `Attribute`에서 값의 일부 또는 전부를 파생하는 `Attribute`를 만들려면 하나 이상의 `Attribute 기반` 또는 [`MMC`](#concepts-ge-mmc) [`Modifier`](#concepts-ge-mods)를 가진 `Infinite` `GameplayEffect`를 사용하십시오. `Derived Attribute`는 의존하는 `Attribute`가 업데이트될 때 자동으로 업데이트됩니다.

`Derived Attribute`에 대한 모든 `Modifier`의 최종 공식은 `Modifier Aggregator`에 대한 공식과 동일합니다. 특정 순서로 계산이 이루어져야 하는 경우, 모든 것을 `MMC` 내에서 수행하십시오.

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**참고:** PIE에서 여러 클라이언트로 플레이하는 경우, 에디터 환경설정에서 `Run Under One Process`를 비활성화해야 합니다. 그렇지 않으면 첫 번째 클라이언트 외의 클라이언트에서는 독립 `Attribute`가 업데이트될 때 `Derived Attribute`가 업데이트되지 않습니다.

이 예시에서는 `Infinite` `GameplayEffect`가 `TestAttrB`와 `TestAttrC` `Attribute`에서 `TestAttrA`의 값을 `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)` 공식으로 파생합니다. `TestAttrA`는 `Attribute` 중 하나라도 값이 업데이트될 때마다 자동으로 값을 재계산합니다.

![Derived Attribute Example](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as"></a>
### 4.4 Attribute Set

<a name="concepts-as-definition"></a>
#### 4.4.1 Attribute Set 정의
`AttributeSet`은 `Attributes`의 변경 사항을 정의하고, 보유하며, 관리합니다. 개발자는 [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html)에서 서브클래스를 만들어야 합니다. `OwnerActor`의 생성자에서 `AttributeSet`을 생성하면 해당 `ASC`에 자동으로 등록됩니다. **이것은 C++로 해야 합니다**.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-design"></a>
#### 4.4.2 Attribute Set 설계
`ASC`는 하나 또는 여러 개의 `AttributeSet`을 가질 수 있습니다. `AttributeSet`은 메모리 오버헤드가 무시할 만하므로, 몇 개의 `AttributeSet`을 사용할지는 개발자의 조직적인 결정에 달려 있습니다.

게임의 모든 `Actor`가 공유하는 하나의 큰 모놀리식 `AttributeSet`을 가지고 필요한 경우에만 `Attribute`를 사용하고 사용하지 않는 `Attribute`는 무시하는 것이 허용됩니다.

대안으로, 필요한 경우 `Actor`에 선택적으로 추가하는 `Attribute` 그룹을 나타내는 여러 `AttributeSet`을 가질 수도 있습니다. 예를 들어, 체력 관련 `Attribute`에 대한 `AttributeSet`, 마나 관련 `Attribute`에 대한 `AttributeSet` 등을 가질 수 있습니다. MOBA 게임에서 영웅은 마나가 필요할 수 있지만 미니언은 아닐 수 있습니다. 따라서 영웅은 마나 `AttributeSet`을 얻고 미니언은 얻지 못합니다.

또한, `AttributeSet`은 `Actor`가 가지는 `Attribute`를 선택적으로 선택하는 또 다른 수단으로 서브클래스화될 수 있습니다. `Attribute`는 내부적으로 `AttributeSetClassName.AttributeName`으로 참조됩니다. `AttributeSet`을 서브클래스화하면, 부모 클래스의 모든 `Attribute`는 여전히 부모 클래스의 이름을 접두사로 가집니다.

하나 이상의 `AttributeSet`을 가질 수 있지만, `ASC`에 동일한 클래스의 `AttributeSet`을 두 개 이상 가져서는 안 됩니다. 동일한 클래스의 `AttributeSet`을 두 개 이상 가지는 경우, 어떤 `AttributeSet`을 사용해야 할지 알 수 없으며 그냥 하나를 선택할 것입니다.

<a name="concepts-as-design-subcomponents"></a>
##### 4.4.2.1 개별 Attribute를 가진 하위 컴포넌트
개별적으로 피해를 입힐 수 있는 갑옷 조각과 같이 `Pawn`에 여러 개의 피해를 입힐 수 있는 컴포넌트가 있는 시나리오에서는, `Pawn`이 가질 수 있는 최대 피해 가능 컴포넌트 수를 안다면 하나의 `AttributeSet`에 그만큼의 체력 `Attribute`를 만드는 것을 권장합니다 - DamageableCompHealth0, DamageableCompHealth1 등은 해당 피해 가능 컴포넌트의 논리적 '슬롯'을 나타냅니다. 피해 가능 컴포넌트 클래스 인스턴스에 `GameplayAbilities` 또는 [`Executions`](#concepts-ge-ec)가 읽을 수 있는 슬롯 번호 `Attribute`를 할당하여 어떤 `Attribute`에 피해를 적용할지 알 수 있도록 합니다. 최대 피해 가능 컴포넌트 수보다 적거나 0개의 피해 가능 컴포넌트를 가진 `Pawn`도 괜찮습니다. `AttributeSet`에 `Attribute`가 있다고 해서 반드시 사용해야 하는 것은 아닙니다. 사용되지 않는 `Attribute`는 미미한 메모리만 차지합니다.

만약 하위 컴포넌트 각각에 많은 `Attribute`가 필요하고, 잠재적으로 무한한 수의 하위 컴포넌트가 존재하며, 하위 컴포넌트가 분리되어 다른 플레이어에 의해 사용될 수 있거나 (예: 무기), 또는 다른 이유로 이 접근 방식이 맞지 않는다면, `Attribute` 사용을 중단하고 대신 컴포넌트에 일반적인 float 값을 저장하는 것을 권장합니다. [아이템 Attribute](#concepts-as-design-itemattributes)를 참조하십시오.

<a name="concepts-as-design-addremoveruntime"></a>
##### 4.4.2.2 런타임에 AttributeSet 추가 및 제거하기
`AttributeSet`은 런타임에 `ASC`에서 추가 및 제거될 수 있습니다. 그러나 `AttributeSet`을 제거하는 것은 위험할 수 있습니다. 예를 들어, 서버보다 클라이언트에서 `AttributeSet`이 먼저 제거되고 `Attribute` 값 변경이 클라이언트에 복제되면, `Attribute`는 해당 `AttributeSet`을 찾지 못하고 게임이 충돌할 것입니다.

인벤토리에 무기를 추가할 때:
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

인벤토리에서 무기를 제거할 때:
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```
<a name="concepts-as-design-itemattributes"></a>
##### 4.4.2.3 아이템 Attribute (무기 탄약)
`Attribute`를 사용하는 장비 가능한 아이템(무기 탄약, 방어구 내구도 등)을 구현하는 방법은 몇 가지가 있습니다. 이 모든 접근 방식은 아이템에 직접 값을 저장합니다. 이는 아이템이 수명 동안 둘 이상의 플레이어에게 장착될 수 있는 경우에 필요합니다.

> 1.  아이템에 일반 float 값 사용 (**권장**)
> 1.  아이템에 별도의 `AttributeSet` 사용
> 1.  아이템에 별도의 `ASC` 사용

<a name="concepts-as-design-itemattributes-plainfloats"></a>
###### 4.4.2.3.1 아이템에 일반 Float 값 사용
`Attribute` 대신 아이템 클래스 인스턴스에 일반 float 값을 저장합니다. Fortnite 및 [GASShooter](https://github.com/tranek/GASShooter)는 총 탄약을 이런 방식으로 처리합니다. 총의 경우, 최대 클립 크기, 클립 내 현재 탄약, 예비 탄약 등을 총 인스턴스에 직접 복제된 float 값(`COND_OwnerOnly`)으로 저장합니다. 무기가 예비 탄약을 공유하는 경우, 예비 탄약을 공유 탄약 `AttributeSet`의 캐릭터로 이동해야 합니다 (재장전 `Ability`는 `Cost GE`를 사용하여 예비 탄약에서 총의 float 클립 탄약으로 가져올 수 있습니다). 현재 클립 탄약에 `Attribute`를 사용하지 않으므로, `UGameplayAbility`의 일부 함수를 오버라이드하여 총의 float 값에 대한 비용을 확인하고 적용해야 합니다. `Ability`를 부여할 때 총을 [`GameplayAbilitySpec`](https://github.com/tranek/GASDocumentation#concepts-ga-spec)의 `SourceObject`로 만들면 `Ability` 내부에서 `Ability`를 부여한 총에 접근할 수 있습니다.

자동 사격 중 총이 탄약량을 다시 복제하여 로컬 탄약량을 덮어쓰는 것을 방지하려면, `PreReplication()`에서 플레이어가 `IsFiring` `GameplayTag`를 가지고 있는 동안에는 복제를 비활성화하십시오. 여기서는 본질적으로 자체적인 로컬 예측을 수행하는 것입니다.

```c++
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
	Super::PreReplication(ChangedPropertyTracker);

	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```

장점:
1.  `AttributeSet` 사용의 한계를 피할 수 있습니다 (아래 참조).

제한 사항:
1.  기존 `GameplayEffect` 워크플로를 사용할 수 없습니다 (탄약 사용을 위한 `Cost GE` 등).
1.  총의 float 값에 대한 탄약 비용을 확인하고 적용하기 위해 `UGameplayAbility`의 주요 함수를 오버라이드하는 작업이 필요합니다.

<a name="concepts-as-design-itemattributes-attributeset"></a>
###### 4.4.2.3.2 아이템에 `AttributeSet` 사용
플레이어의 인벤토리에 추가될 때 [플레이어의 `ASC`에 추가되는](#concepts-as-design-addremoveruntime) 아이템의 별도 `AttributeSet`을 사용하는 것은 가능하지만, 몇 가지 주요 제약 사항이 있습니다. 저는 [GASShooter](https://github.com/tranek/GASShooter) 초기 버전에서 무기 탄약에 대해 이 방법으로 구현했습니다. 무기는 최대 클립 크기, 클립 내 현재 탄약, 예비 탄약 등과 같은 `Attribute`를 무기 클래스에 있는 `AttributeSet`에 저장합니다. 무기가 예비 탄약을 공유하는 경우, 예비 탄약은 공유 탄약 `AttributeSet`의 캐릭터로 이동합니다. 무기가 서버에서 플레이어의 인벤토리에 추가되면, 무기는 `AttributeSet`을 플레이어의 `ASC::SpawnedAttributes`에 추가합니다. 그런 다음 서버는 이를 클라이언트에 복제합니다. 무기가 인벤토리에서 제거되면 `AttributeSet`을 `ASC::SpawnedAttributes`에서 제거합니다.

`AttributeSet`이 `OwnerActor`가 아닌 다른 것(예: 무기)에 있는 경우, 초기에는 `AttributeSet`에서 일부 컴파일 오류가 발생할 것입니다. 해결책은 생성자 대신 `BeginPlay()`에서 `AttributeSet`을 생성하고, 무기에 `IAbilitySystemInterface`를 구현하는 것입니다 (무기를 플레이어 인벤토리에 추가할 때 `ASC`에 대한 포인터를 설정).

```c++
void AGSWeapon::BeginPlay()
{
	if (!AttributeSet)
	{
		AttributeSet = NewObject<UGSWeaponAttributeSet>(this);
	}
	//...
}
```

이 [GASShooter의 이전 버전](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735)을 확인하여 실제 사용 사례를 볼 수 있습니다.

장점:
1.  기존 `GameplayAbility` 및 `GameplayEffect` 워크플로를 사용할 수 있습니다 (탄약 사용을 위한 `Cost GE` 등).
1.  매우 적은 수의 아이템에 대해 설정하기 간단합니다.

제한 사항:
1.  무기 유형마다 새로운 `AttributeSet` 클래스를 만들어야 합니다. `ASC`는 `Attribute`의 변경 사항이 `ASC`의 `SpawnedAttributes` 배열에서 해당 `AttributeSet` 클래스의 첫 번째 인스턴스를 찾기 때문에 클래스당 하나의 `AttributeSet` 인스턴스만 기능적으로 가질 수 있습니다. 동일한 `AttributeSet` 클래스의 추가 인스턴스는 무시됩니다.
1.  이전 이유인 `AttributeSet` 클래스당 하나의 `AttributeSet` 인스턴스 때문에 플레이어의 인벤토리에는 각 유형의 무기가 하나만 있을 수 있습니다.
1.  `AttributeSet`을 제거하는 것은 위험합니다. GASShooter에서 플레이어가 로켓으로 자살하면, 플레이어는 즉시 인벤토리에서 로켓 발사기를 제거합니다 (`ASC`에서 `AttributeSet`도 포함). 서버가 로켓 발사기의 탄약 `Attribute`가 변경되었다고 복제할 때, `AttributeSet`은 클라이언트의 `ASC`에 더 이상 존재하지 않아 게임이 충돌했습니다.

<a name="concepts-as-design-itemattributes-asc"></a>
###### 4.4.2.3.3 아이템에 `ASC` 사용
각 아이템에 전체 `AbilitySystemComponent`를 넣는 것은 극단적인 접근 방식입니다. 저는 개인적으로 이렇게 해본 적이 없으며, 실제 사용되는 것을 본 적도 없습니다. 작동시키려면 많은 엔지니어링 작업이 필요할 것입니다.

> 동일한 소유자(owner)를 가지지만 다른 아바타(avatar)를 가진 여러 AbilitySystemComponent(예: 폰과 무기/아이템/투사체에 PlayerState를 소유자로 설정)를 사용하는 것이 가능한가요?
> 
> 제가 보기에 첫 번째 문제는 소유자 액터에 IGameplayTagAssetInterface와 IAbilitySystemInterface를 구현하는 것입니다. 전자는 가능할 수 있습니다: 모든 ASC에서 태그를 집계하기만 하면 됩니다 (하지만 조심하세요 - HasAllMatchingGameplayTags는 교차 ASC 집계를 통해서만 충족될 수 있습니다. 단순히 각 ASC에 해당 호출을 전달하고 결과를 OR 하는 것으로는 충분하지 않을 것입니다). 하지만 후자는 훨씬 더 까다롭습니다: 어떤 ASC가 권한 있는 ASC인가요? 누군가 GE를 적용하고 싶다면 - 어떤 ASC가 GE를 받아야 할까요? 아마 이 문제를 해결할 수 있겠지만, 이 문제의 측면이 가장 어려울 것입니다: 소유자 아래에 여러 ASC가 있을 것이기 때문입니다.
> 
> 폰과 무기에 별도의 ASC가 자체적으로는 말이 될 수 있습니다. 예를 들어, 무기를 설명하는 태그와 소유 폰을 설명하는 태그를 구분하는 것입니다. 아마도 무기에 부여된 태그가 소유자에게도 "적용"되고 다른 것은 적용되지 않는 것이 말이 될 수도 있습니다 (예: attribute와 GE는 독립적이지만 소유자는 위에서 설명한 대로 소유 태그를 집계할 것입니다). 이것은 작동할 수 있다고 확신합니다. 하지만 동일한 소유자를 가진 여러 ASC는 불안정할 수 있습니다.

*Epic의 Dave Ratti가 [커뮤니티 질문 #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)에 대한 답변*

장점:
1.  기존 `GameplayAbility` 및 `GameplayEffect` 워크플로를 사용할 수 있습니다 (탄약 사용을 위한 `Cost GE` 등).
1.  `AttributeSet` 클래스를 재사용할 수 있습니다 (각 무기 `ASC`에 하나씩).

제한 사항:
1.  알 수 없는 엔지니어링 비용
1.  과연 가능한가?

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-attributes"></a>
#### 4.4.3 Attribute 정의하기
**`Attribute`는 `AttributeSet`의 헤더 파일에서 C++로만 정의할 수 있습니다**. 모든 `AttributeSet` 헤더 파일 상단에 다음 매크로 블록을 추가하는 것이 좋습니다. 그러면 `Attribute`에 대한 getter 및 setter 함수가 자동으로 생성됩니다.

```c++
// AttributeSet.h의 매크로 사용
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

복제된 체력 `Attribute`는 다음과 같이 정의됩니다:

```c++
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

헤더에 `OnRep` 함수도 정의합니다:
```c++
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

`AttributeSet`의 .cpp 파일은 예측 시스템에서 사용하는 `GAMEPLAYATTRIBUTE_REPNOTIFY` 매크로를 사용하여 `OnRep` 함수를 채워야 합니다:
```c++
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}
```

마지막으로, `Attribute`는 `GetLifetimeReplicatedProps`에 추가되어야 합니다:
```c++
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

`REPNOTIFY_Always`는 `OnRep` 함수가 로컬 값이 서버에서 복제되는 값과 이미 같더라도 (예측으로 인해) 트리거되도록 지시합니다. 기본적으로 로컬 값이 서버에서 복제되는 값과 같으면 `OnRep` 함수를 트리거하지 않습니다.

`Meta Attribute`와 같이 `Attribute`가 복제되지 않는 경우, `OnRep` 및 `GetLifetimeReplicatedProps` 단계는 건너뛸 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-init"></a>
#### 4.4.4 Attribute 초기화하기
`Attribute`를 초기화하는 방법( `BaseValue`와 `CurrentValue`를 초기 값으로 설정하는 방법)에는 여러 가지가 있습니다. Epic은 `Instant` `GameplayEffect`를 사용하는 것을 권장합니다. 이는 샘플 프로젝트에서도 사용되는 방법입니다.

`Attribute`를 초기화하기 위한 `Instant` `GameplayEffect`를 만드는 방법은 샘플 프로젝트의 `GE_HeroAttributes` Blueprint를 참조하십시오. 이 `GameplayEffect`의 적용은 C++에서 이루어집니다.

`Attribute`를 정의할 때 `ATTRIBUTE_ACCESSORS` 매크로를 사용했다면, 각 `Attribute`에 대해 `AttributeSet`에 초기화 함수가 자동으로 생성되며, C++에서 자유롭게 호출할 수 있습니다.

```c++
// InitHealth(float InitialValue)는 `ATTRIBUTE_ACCESSORS` 매크로로 정의된 'Health' Attribute에 대해 자동으로 생성된 함수입니다.
AttributeSet->InitHealth(100.0f);
```

`Attribute`를 초기화하는 더 많은 방법은 `AttributeSet.h`를 참조하십시오.

**참고:** 4.24 이전에는 `FAttributeSetInitterDiscreteLevels`가 `FGameplayAttributeData`와 함께 작동하지 않았습니다. 이는 `Attribute`가 원시 float였을 때 생성되었으며, `FGameplayAttributeData`가 `Plain Old Data`(`POD`)가 아니라는 불평을 할 것입니다. 이는 4.24에서 수정되었습니다. https://issues.unrealengine.com/issue/UE-76557.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-preattributechange"></a>
#### 4.4.5 PreAttributeChange()
`PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)`는 `AttributeSet`의 주요 함수 중 하나로, 변경이 발생하기 전에 `Attribute`의 `CurrentValue` 변경에 반응합니다. 이는 참조 매개변수 `NewValue`를 통해 들어오는 `CurrentValue` 변경을 클램핑하기에 이상적인 장소입니다.

예를 들어, 이동 속도 `Modifier`를 클램핑하기 위해 샘플 프로젝트는 다음과 같이 합니다:
```c++
if (Attribute == GetMoveSpeedAttribute())
{
	// 초당 150단위 미만으로 늦출 수 없으며, 초당 1000단위 이상으로 올릴 수 없습니다.
	NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}
```
`GetMoveSpeedAttribute()` 함수는 `AttributeSet.h`에 추가한 매크로 블록에 의해 생성됩니다 ([Attribute 정의하기](#concepts-as-attributes)).

이것은 `Attribute` setter( `AttributeSet.h`의 매크로 블록으로 정의됨 ([Attribute 정의하기](#concepts-as-attributes)))를 사용하든, [`GameplayEffects`](#concepts-ge)를 사용하든 `Attribute`의 모든 변경 사항에서 트리거됩니다.

**참고:** 여기에서 발생하는 모든 클램핑은 `ASC`의 `Modifier`를 영구적으로 변경하지 않습니다. 이는 `Modifier`를 쿼리하여 반환되는 값만 변경합니다. 즉, [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 및 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)와 같이 모든 `Modifier`에서 `CurrentValue`를 다시 계산하는 모든 것은 다시 클램핑을 구현해야 합니다.

**참고:** Epic의 `PreAttributeChange()`에 대한 주석은 게임 플레이 이벤트에 사용하지 말고 주로 클램핑에 사용하라고 말합니다. `Attribute` 변경에 대한 게임 플레이 이벤트에 권장되는 장소는 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)` ([Attribute 변경에 반응하기](#concepts-a-changes))입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-postgameplayeffectexecute"></a>
#### 4.4.6 PostGameplayEffectExecute()
`PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)`는 `Instant` [`GameplayEffect`](#concepts-ge)로 인한 `Attribute`의 `BaseValue` 변경 후에만 트리거됩니다. 이것은 `GameplayEffect`로 인해 `Attribute`가 변경될 때 더 많은 `Attribute` 조작을 수행하기에 유효한 장소입니다.

예를 들어, 샘플 프로젝트에서는 여기서 최종 피해 `Meta Attribute`를 체력 `Attribute`에서 뺍니다. 보호막 `Attribute`가 있다면, 피해를 먼저 보호막에서 뺀 다음 남은 값을 체력에서 뺄 것입니다. 샘플 프로젝트는 또한 이 위치를 사용하여 피격 반응 애니메이션을 적용하고, 떠다니는 피해 숫자를 표시하며, 살인자에게 경험치와 골드 현상금을 할당합니다. 설계상 피해 `Meta Attribute`는 항상 `Instant` `GameplayEffect`를 통해 전달되며 `Attribute` setter를 통해서는 전달되지 않습니다.

마나, 스태미나와 같이 `Instant` `GameplayEffects`로만 `BaseValue`가 변경되는 다른 `Attribute`도 여기서 최대 값 `Attribute`에 클램핑될 수 있습니다.

**참고:** `PostGameplayEffectExecute()`가 호출될 때, `Attribute` 변경은 이미 발생했지만 아직 클라이언트에 복제되지 않았으므로, 여기서 값을 클램핑해도 클라이언트에 두 번의 네트워크 업데이트가 발생하지 않습니다. 클라이언트는 클램핑 후에만 업데이트를 받게 됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-as-onattributeaggregatorcreated"></a>
#### 4.4.7 OnAttributeAggregatorCreated()
`OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)`는 이 세트의 `Attribute`에 대해 `Aggregator`가 생성될 때 트리거됩니다. 이는 [`FAggregatorEvaluateMetaData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData/index.html)의 사용자 정의 설정을 허용합니다. `AggregatorEvaluateMetaData`는 `Aggregator`가 적용된 모든 [`Modifiers`](#concepts-ge-mods)를 기반으로 `Attribute`의 `CurrentValue`를 평가할 때 사용됩니다. 기본적으로 `AggregatorEvaluateMetaData`는 `Aggregator`가 `MostNegativeMod_AllPositiveMods`의 예시와 같이 어떤 `Modifiers`가 자격을 갖추는지 결정하는 데만 사용됩니다. 이 예시는 모든 긍정적인 `Modifiers`를 허용하지만 부정적인 `Modifiers`는 가장 부정적인 하나로 제한합니다. Paragon에서는 이를 사용하여 플레이어에게 동시에 적용되는 느려짐 효과의 수에 관계없이 가장 부정적인 이동 속도 느려짐 효과만 플레이어에게 적용하고 모든 긍정적인 이동 속도 버프는 적용했습니다. 자격을 갖추지 못한 `Modifiers`는 여전히 `ASC`에 존재하지만, 최종 `CurrentValue`로 집계되지 않습니다. 조건이 변경되면 나중에 자격을 갖출 수 있습니다. 예를 들어, 가장 부정적인 `Modifier`가 만료되면 다음으로 가장 부정적인 `Modifier`(존재하는 경우)가 자격을 갖춥니다.

가장 부정적인 `Modifier`와 모든 긍정적인 `Modifier`만 허용하는 예시에서 `AggregatorEvaluateMetaData`를 사용하려면:

```c++
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```c++
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
	{
		return;
	}

	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```

자격 부여를 위한 사용자 정의 `AggregatorEvaluateMetaData`는 정적 변수로 `FAggregatorEvaluateMetaDataLibrary`에 추가되어야 합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge"></a>
### 4.5 Gameplay Effects

<a name="concepts-ge-definition"></a>
#### 4.5.1 Gameplay Effect 정의
[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html)(`GE`)는 `Ability`가 자신과 다른 대상의 [`Attributes`](#concepts-a)와 [`GameplayTags`](#concepts-gt)를 변경하는 수단입니다. 이들은 즉각적인 `Attribute` 변경(예: 피해 또는 치유)을 유발하거나 이동 속도 증가 또는 기절과 같은 장기적인 상태 버프/디버프를 적용할 수 있습니다. `UGameplayEffect` 클래스는 단일 `GameplayEffect`를 정의하는 **데이터 전용** 클래스로 의도되었습니다. `GameplayEffects`에 추가적인 로직을 추가해서는 안 됩니다. 일반적으로 디자이너는 `UGameplayEffect`의 많은 Blueprint 자식 클래스를 생성할 것입니다.

`GameplayEffects`는 [`Modifiers`](#concepts-ge-mods)와 [`Executions` (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec)를 통해 `Attribute`를 변경합니다.

`GameplayEffects`에는 `Instant`, `Duration`, `Infinite`의 세 가지 지속 시간 유형이 있습니다.

또한, `GameplayEffects`는 [`GameplayCues`](#concepts-gc)를 추가/실행할 수 있습니다. `Instant` `GameplayEffect`는 `GameplayCue` `GameplayTags`에 대해 `Execute`를 호출하는 반면, `Duration` 또는 `Infinite` `GameplayEffect`는 `GameplayCue` `GameplayTags`에 대해 `Add` 및 `Remove`를 호출합니다.

| 지속 시간 유형 | GameplayCue 이벤트 | 사용 시점 |
|---|---|---|
| `Instant` | Execute | `Attribute`의 `BaseValue`에 즉각적인 영구적인 변경을 위해. `GameplayTags`는 잠시라도 적용되지 않습니다. |
| `Duration` | Add & Remove | `Attribute`의 `CurrentValue`에 일시적인 변경을 위해, 그리고 `GameplayEffect`가 만료되거나 수동으로 제거될 때 제거될 `GameplayTags`를 적용하기 위해. 지속 시간은 `UGameplayEffect` 클래스/Blueprint에서 지정됩니다. |
| `Infinite` | Add & Remove | `Attribute`의 `CurrentValue`에 일시적인 변경을 위해, 그리고 `GameplayEffect`가 제거될 때 제거될 `GameplayTags`를 적용하기 위해. 이들은 자체적으로 만료되지 않으며 `Ability` 또는 `ASC`에 의해 수동으로 제거되어야 합니다. |

`Duration` 및 `Infinite` `GameplayEffects`에는 `Period`에 정의된 대로 `X`초마다 `Modifier` 및 `Execution`을 적용하는 `주기적 효과(Periodic Effects)`를 적용하는 옵션이 있습니다. `주기적 효과`는 `Attribute`의 `BaseValue` 변경 및 `GameplayCues` `실행(Executing)`에 관해서는 `Instant` `GameplayEffects`처럼 처리됩니다. 이는 시간이 지남에 따라 피해(DOT) 유형 효과에 유용합니다. **참고:** `주기적 효과`는 [예측](#concepts-p)할 수 없습니다.

`Duration` 및 `Infinite` `GameplayEffects`는 적용 후 `Ongoing Tag Requirements`가 충족되지 않거나 충족될 때 일시적으로 켜고 끌 수 있습니다 ([Gameplay Effect Tags](#concepts-ge-tags)). `GameplayEffect`를 끄면 `Modifier`의 효과와 적용된 `GameplayTags`는 제거되지만 `GameplayEffect` 자체는 제거되지 않습니다. `GameplayEffect`를 다시 켜면 `Modifier`와 `GameplayTags`가 다시 적용됩니다.

`Duration` 또는 `Infinite` `GameplayEffect`의 `Modifier`를 수동으로 재계산해야 하는 경우 (예: `Attribute`에서 오지 않는 데이터를 사용하는 `MMC`가 있는 경우), `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`을 사용하여 이미 가지고 있는 레벨 (`UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`)을 호출할 수 있습니다. 백킹 `Attribute`를 기반으로 하는 `Modifier`는 해당 백킹 `Attribute`가 업데이트될 때 자동으로 업데이트됩니다. `Modifier`를 업데이트하는 `SetActiveGameplayEffectLevel()`의 핵심 함수는 다음과 같습니다:

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// 프라이빗 함수이므로, 이미 설정된 레벨로 설정할 필요 없이 이 세 함수를 호출할 것입니다.
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects`는 일반적으로 인스턴스화되지 않습니다. `Ability` 또는 `ASC`가 `GameplayEffect`를 적용하려고 할 때, `GameplayEffect`의 `ClassDefaultObject`에서 [`GameplayEffectSpec`](#concepts-ge-spec)을 생성합니다. 성공적으로 적용된 `GameplayEffectSpec`은 `FActiveGameplayEffect`라는 새로운 구조체에 추가되며, `ASC`는 이를 `ActiveGameplayEffects`라는 특수 컨테이너 구조체에서 추적합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-applying"></a>
#### 4.5.2 Gameplay Effect 적용하기
`GameplayEffects`는 [`GameplayAbilities`](#concepts-ga)의 함수와 `ASC`의 함수를 통해 여러 가지 방법으로 적용될 수 있으며, 일반적으로 `ApplyGameplayEffectTo`의 형태를 취합니다. 다른 함수들은 본질적으로 `Target`에 `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()`를 호출하는 편의 함수입니다.

예를 들어, 투사체에서 `GameplayAbility` 외부에서 `GameplayEffects`를 적용하려면, `Target`의 `ASC`를 가져와서 `ApplyGameplayEffectToSelf` 함수 중 하나를 사용해야 합니다.

`ASC`에 `Duration` 또는 `Infinite` `GameplayEffects`가 적용될 때, 해당 델리게이트에 바인딩하여 감지할 수 있습니다:
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
콜백 함수:
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

서버는 복제 모드와 관계없이 항상 이 함수를 호출합니다. 자율 프록시는 `Full` 및 `Mixed` 복제 모드에서 복제된 `GameplayEffects`에 대해서만 이 함수를 호출합니다. 시뮬레이트된 프록시는 `Full` [복제 모드](#concepts-asc-rm)에서만 이 함수를 호출합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-removing"></a>
#### 4.5.3 Gameplay Effect 제거하기
`GameplayEffects`는 [`GameplayAbilities`](#concepts-ga)의 함수와 `ASC`의 함수를 통해 여러 가지 방법으로 제거될 수 있으며, 일반적으로 `RemoveActiveGameplayEffect`의 형태를 취합니다. 다른 함수들은 본질적으로 `Target`에 `FActiveGameplayEffectsContainer::RemoveActiveEffects()`를 호출하는 편의 함수입니다.

`GameplayAbility` 외부에서 `GameplayEffects`를 제거하려면, `Target`의 `ASC`를 가져와서 `RemoveActiveGameplayEffect` 함수 중 하나를 사용해야 합니다.

`ASC`에서 `Duration` 또는 `Infinite` `GameplayEffects`가 제거될 때, 해당 델리게이트에 바인딩하여 감지할 수 있습니다:
```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```
콜백 함수:
```c++
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```

서버는 복제 모드와 관계없이 항상 이 함수를 호출합니다. 자율 프록시는 `Full` 및 `Mixed` 복제 모드에서 복제된 `GameplayEffects`에 대해서만 이 함수를 호출합니다. 시뮬레이트된 프록시는 `Full` [복제 모드](#concepts-asc-rm)에서만 이 함수를 호출합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-mods"></a>
#### 4.5.4 Gameplay Effect Modifiers
`Modifier`는 `Attribute`를 변경하며, `Attribute`를 [예측적으로](#concepts-p) 변경하는 유일한 방법입니다. `GameplayEffect`는 0개 또는 여러 개의 `Modifier`를 가질 수 있습니다. 각 `Modifier`는 지정된 연산을 통해 하나의 `Attribute`만 변경하는 역할을 합니다.

| 연산 | 설명 |
|---|---|
| `Add` | `Modifier`에 지정된 `Attribute`에 결과를 추가합니다. 빼기는 음수 값을 사용하십시오. |
| `Multiply` | `Modifier`에 지정된 `Attribute`에 결과를 곱합니다. |
| `Divide` | `Modifier`에 지정된 `Attribute`에 결과를 나눕니다. |
| `Override` | `Modifier`에 지정된 `Attribute`를 결과로 덮어씁니다. |

`Attribute`의 `CurrentValue`는 모든 `Modifier`가 `BaseValue`에 추가된 총 결과입니다. `Modifier`가 어떻게 집계되는지에 대한 공식은 `GameplayEffectAggregator.cpp`의 `FAggregatorModChannel::EvaluateWithBase`에 다음과 같이 정의되어 있습니다:
```c++
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

모든 `Override` `Modifier`는 최종 값을 덮어쓰며, 마지막으로 적용된 `Modifier`가 우선순위를 가집니다.

**참고:** 백분율 기반 변경의 경우, `Add` 연산 후에 발생하도록 `Multiply` 연산을 사용해야 합니다.

**참고:** [예측](#concepts-p)은 백분율 변경에 문제가 있습니다.

`Modifier`에는 Scalable Float, Attribute Based, Custom Calculation Class, Set By Caller의 네 가지 유형이 있습니다. 이들은 모두 float 값을 생성하며, 이 값은 연산에 따라 `Modifier`의 지정된 `Attribute`를 변경하는 데 사용됩니다.

| `Modifier` 유형 | 설명 |
|---|---|
| `Scalable Float` | `FScalableFloats`는 변수가 행이고 레벨이 열인 데이터 테이블을 가리킬 수 있는 구조체입니다. Scalable Floats는 `Ability`의 현재 레벨(또는 [`GameplayEffectSpec`](#concepts-ge-spec)에서 덮어쓰여진 경우 다른 레벨)에서 지정된 테이블 행의 값을 자동으로 읽습니다. 이 값은 계수로 추가로 조작될 수 있습니다. 데이터 테이블/행이 지정되지 않은 경우, 값은 1로 처리되므로 계수를 사용하여 모든 레벨에서 단일 값을 하드 코딩할 수 있습니다. ![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png) |
| `Attribute 기반` | `Attribute 기반` `Modifier`는 `Source`(`GameplayEffectSpec`을 생성한 주체) 또는 `Target`(`GameplayEffectSpec`을 받은 주체)에 있는 백킹 `Attribute`의 `CurrentValue` 또는 `BaseValue`를 가져와 계수와 계수 전후 추가로 추가로 수정합니다. `스냅샷팅(Snapshotting)`은 `GameplayEffectSpec`이 생성될 때 백킹 `Attribute`가 캡처되는 것을 의미하며, 스냅샷팅 없음은 `GameplayEffectSpec`이 적용될 때 `Attribute`가 캡처되는 것을 의미합니다. |
| `Custom Calculation Class` | `Custom Calculation Class`는 복잡한 `Modifier`에 가장 큰 유연성을 제공합니다. 이 `Modifier`는 [`ModifierMagnitudeCalculation`](#concepts-ge-mmc) 클래스를 가져와 결과 float 값을 계수와 계수 전후 추가로 추가로 조작할 수 있습니다. |
| `Set By Caller` | `SetByCaller` `Modifier`는 `GameplayEffect` 외부에서 런타임에 `Ability` 또는 `GameplayEffectSpec`을 만든 주체에 의해 `GameplayEffectSpec`에 설정되는 값입니다. 예를 들어, 플레이어가 `Ability`를 충전하기 위해 버튼을 누른 시간에 따라 피해를 설정하려면 `SetByCaller`를 사용합니다. `SetByCaller`는 본질적으로 `GameplayEffectSpec`에 있는 `TMap<FGameplayTag, float>`입니다. `Modifier`는 `Aggregator`에게 제공된 `GameplayTag`와 관련된 `SetByCaller` 값을 찾도록 지시하는 것입니다. `Modifier`가 사용하는 `SetByCaller`는 개념의 `GameplayTag` 버전만 사용할 수 있습니다. `FName` 버전은 여기에서 비활성화됩니다. `Modifier`가 `SetByCaller`로 설정되었지만, 올바른 `GameplayTag`를 가진 `SetByCaller`가 `GameplayEffectSpec`에 존재하지 않으면 런타임 오류가 발생하고 0이 반환됩니다. 이는 `Divide` 연산의 경우 문제를 일으킬 수 있습니다. `SetByCaller` 사용 방법에 대한 자세한 내용은 [`SetByCallers`](#concepts-ge-spec-setbycaller)를 참조하십시오. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-mods-multiplydivide"></a>
##### 4.5.4.1 Multiply 및 Divide Modifiers
기본적으로 모든 `Multiply` 및 `Divide` `Modifier`는 `Attribute`의 `BaseValue`에 곱하거나 나누기 전에 함께 더해집니다.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Additive = SumMods(Mods[EGameplayModOp::Additive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
	float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	float Division = SumMods(Mods[EGameplayModOp::Division], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
	...
	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
	...
}
```

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}
```
*`GameplayEffectAggregator.cpp`에서 발췌*

`Multiply`와 `Divide` `Modifier`는 모두 이 공식에서 `1`의 `Bias` 값을 가집니다 (`Addition`은 `0`의 `Bias`를 가집니다). 따라서 다음과 같이 보일 것입니다:

```
1 + (Mod1.Magnitude - 1) + (Mod2.Magnitude - 1) + ...
```

이 공식은 몇 가지 예상치 못한 결과를 초래합니다. 첫째, 이 공식은 모든 `Modifier`를 `BaseValue`에 곱하거나 나누기 전에 함께 더합니다. 대부분의 사람들은 곱하거나 나누기를 기대할 것입니다. 예를 들어, `1.5`의 `Multiply` `Modifier`가 두 개 있다면, 대부분의 사람들은 `BaseValue`가 `1.5 x 1.5 = 2.25`로 곱해질 것으로 예상할 것입니다. 대신, 이것은 `1.5`를 함께 더하여 `BaseValue`를 `2`로 곱합니다 (`50% 증가 + 또 다른 50% 증가 = 100% 증가`). 이는 `GameplayPrediction.h`의 `500` 기본 속도에 `10%` 속도 버프를 적용하면 `550`이 되고, 또 다른 `10%` 속도 버프를 추가하면 `600`이 되는 예시를 위한 것입니다.

둘째, 이 공식에는 Paragon을 염두에 두고 설계되었기 때문에 어떤 값을 사용할 수 있는지에 대한 문서화되지 않은 규칙이 있습니다.

`Multiply` 및 `Divide` 곱셈 덧셈 공식에 대한 규칙:
*   `(1 미만 값이 하나 이하) AND (1 이상 2 미만 값이 임의 개수)`
*   `OR (2 이상 값이 하나)`

공식의 `Bias`는 기본적으로 `[1, 2)` 범위의 숫자에서 정수 자릿수를 뺍니다. 첫 번째 `Modifier`의 `Bias`는 시작 `Sum` 값 (루프 전에 `Bias`로 설정됨)에서 빼지기 때문에, 어떤 값도 자체적으로 작동하며, 하나의 값 `< 1`이 `[1, 2)` 범위의 숫자와 함께 작동하는 이유입니다.

`Multiply`의 몇 가지 예:  
곱셈자: `0.5`  
`1 + (0.5 - 1) = 0.5`, 정확

곱셈자: `0.5, 0.5`  
`1 + (0.5 - 1) + (0.5 - 1) = 0`, 예상 값 `1`과 다름. `1` 미만의 여러 값이 곱셈자를 더하는 데는 의미가 없습니다. Paragon은 [`Multiply` `Modifier`에 대해 가장 큰 음수 값만 허용하도록](#cae-nonstackingge) 설계되었으므로 `BaseValue`에 곱해지는 `1` 미만의 값은 최대 하나만 있을 것입니다.

곱셈자: `1.1, 0.5`  
`1 + (0.5 - 1) + (1.1 - 1) = 0.6`, 정확

곱셈자: `5, 5`  
`1 + (5 - 1) + (5 - 1) = 9`, 예상 값 `10`과 다름. 항상 `Modifier의 합 - Modifier 수 + 1`이 될 것입니다.

많은 게임에서는 `Multiply` 및 `Divide` `Modifier`가 `BaseValue`에 적용되기 전에 함께 곱해지거나 나눠지기를 원할 것입니다. 이를 달성하려면 `FAggregatorModChannel::EvaluateWithBase()`의 **엔진 코드를 변경**해야 합니다.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```

```c++
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-mods-gameplaytags"></a>
##### 4.5.4.2 Modifiers에 Gameplay Tags 사용

각 [Modifier](#concepts-ge-mods)에 대해 `SourceTags` 및 `TargetTags`를 설정할 수 있습니다. 이는 `GameplayEffect`의 [`Application Tag requirements`](#concepts-ge-tags)와 동일하게 작동합니다. 따라서 태그는 효과가 적용될 때만 고려됩니다. 즉, 주기적, 무한 효과를 가질 때, 효과의 첫 적용 시에만 고려되며 각 주기적 실행 시에는 *고려되지 않습니다*.

`Attribute Based` Modifier도 `SourceTagFilter` 및 `TargetTagFilter`를 설정할 수 있습니다. `Attribute Based` Modifier의 원본인 `Attribute`의 Magnitude를 결정할 때, 이러한 필터는 해당 `Attribute`에 대한 특정 Modifier를 제외하는 데 사용됩니다. 필터의 모든 태그를 Source 또는 Target이 가지지 않은 Modifier는 제외됩니다.

이는 자세히 말하면 다음과 같습니다: `Source ASC`의 태그와 `Target ASC`의 태그는 `GameplayEffects`에 의해 캡처됩니다. `Source ASC` 태그는 `GameplayEffectSpec`이 생성될 때 캡처되고, `Target ASC` 태그는 효과가 실행될 때 캡처됩니다. 무한 또는 지속 효과의 Modifier가 적용될 자격이 있는지 (즉, Aggregator가 자격이 있는지) 결정할 때, 이 필터가 설정되면 캡처된 태그가 필터와 비교됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-stacking"></a>
#### 4.5.5 Stacking Gameplay Effects
`GameplayEffects`는 기본적으로 이전에 존재하는 `GameplayEffectSpec` 인스턴스에 대해 알거나 신경 쓰지 않는 새로운 `GameplayEffectSpec` 인스턴스를 적용합니다. `GameplayEffects`는 스태킹되도록 설정할 수 있으며, 이 경우 새로운 `GameplayEffectSpec` 인스턴스가 추가되는 대신 현재 존재하는 `GameplayEffectSpec`의 스택 수가 변경됩니다. 스태킹은 `Duration` 및 `Infinite` `GameplayEffects`에 대해서만 작동합니다.

스태킹에는 Source별 집계와 Target별 집계의 두 가지 유형이 있습니다.

| 스태킹 유형 | 설명 |
|---|---|
| Source별 집계 | Target에 Source `ASC`별로 별도의 스택 인스턴스가 존재합니다. 각 Source는 X만큼의 스택을 적용할 수 있습니다. |
| Target별 집계 | Source와 관계없이 Target에 하나의 스택 인스턴스만 존재합니다. 각 Source는 공유 스택 한도까지 스택을 적용할 수 있습니다. |

스택은 만료, 지속 시간 갱신 및 기간 재설정을 위한 정책도 가집니다. `GameplayEffect` Blueprint에는 유용한 호버 툴팁이 있습니다.

샘플 프로젝트에는 `GameplayEffect` 스택 변경을 감지하는 사용자 지정 Blueprint 노드가 포함되어 있습니다. HUD UMG 위젯은 이를 사용하여 플레이어가 가진 패시브 방어력 스택 양을 업데이트합니다. 이 `AsyncTask`는 UMG 위젯의 `Destruct` 이벤트에서 호출하는 `EndTask()`가 수동으로 호출될 때까지 영원히 지속됩니다. `AsyncTaskEffectStackChanged.h/cpp`를 참조하십시오.

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-ga"></a>
#### 4.5.6 Granted Abilities
`GameplayEffects`는 새로운 [`GameplayAbilities`](#concepts-ga)를 `ASC`에 부여할 수 있습니다. `Duration` 및 `Infinite` `GameplayEffects`만 `Ability`를 부여할 수 있습니다.

이것의 일반적인 사용 사례는 다른 플레이어가 밀려나거나 당겨지는 것과 같은 행동을 강제하려고 할 때입니다. 자동으로 활성화되는 `Ability` (부여될 때 자동으로 `Ability`를 활성화하는 방법은 [패시브 Ability](#concepts-ga-activating-passive) 참조)를 부여하는 `GameplayEffect`를 적용하여 원하는 행동을 수행하게 합니다.

디자이너는 `GameplayEffect`가 어떤 `Ability`를 부여하는지, 어떤 레벨에서 부여하는지, 어떤 [입력에 바인딩할지](#concepts-ga-input), 그리고 부여된 `Ability`의 제거 정책을 선택할 수 있습니다.

| 제거 정책 | 설명 |
|---|---|
| Ability 즉시 취소 | `Ability`를 부여한 `GameplayEffect`가 대상에서 제거될 때 부여된 `Ability`가 즉시 취소되고 제거됩니다. |
| 종료 시 Ability 제거 | 부여된 `Ability`가 완료되도록 허용된 다음 대상에서 제거됩니다. |
| 아무것도 하지 않음 | 부여된 `Ability`는 대상에서 부여 `GameplayEffect`가 제거되는 것에 영향을 받지 않습니다. 대상은 나중에 수동으로 제거될 때까지 `Ability`를 영구적으로 가집니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-tags"></a>
#### 4.5.7 Gameplay Effect Tags
`GameplayEffects`는 여러 [`GameplayTagContainers`](#concepts-gt)를 가집니다. 디자이너는 각 범주에 대한 `Added` 및 `Removed` `GameplayTagContainers`를 편집하고, 그 결과는 컴파일 시 `Combined` `GameplayTagContainer`에 나타납니다. `Added` 태그는 이 `GameplayEffect`가 추가하는 새로운 태그로, 그 부모가 이전에 가지고 있지 않았던 태그입니다. `Removed` 태그는 부모 클래스가 가지고 있지만 이 서브클래스에는 없는 태그입니다.

| 범주 | 설명 |
|---|---|
| Gameplay Effect Asset Tags | `GameplayEffect`가 가진 태그입니다. 이 태그 자체로는 어떤 기능도 수행하지 않으며, `GameplayEffect`를 설명하는 용도로만 사용됩니다. |
| Granted Tags | `GameplayEffect`에 존재하지만, `GameplayEffect`가 적용된 `ASC`에도 부여되는 태그입니다. `GameplayEffect`가 제거되면 `ASC`에서도 제거됩니다. `Duration` 및 `Infinite` `GameplayEffects`에만 적용됩니다. |
| Ongoing Tag Requirements | 일단 적용되면, 이 태그들은 `GameplayEffect`의 켜짐/꺼짐 상태를 결정합니다. `GameplayEffect`는 꺼진 상태에서도 적용될 수 있습니다. `Ongoing Tag Requirements`를 충족하지 못해 `GameplayEffect`가 꺼졌지만, 나중에 요구 사항이 충족되면 `GameplayEffect`가 다시 켜지고 `Modifier`를 다시 적용합니다. `Duration` 및 `Infinite` `GameplayEffects`에만 적용됩니다. |
| Application Tag Requirements | `GameplayEffect`가 대상에 적용될 수 있는지 여부를 결정하는 대상의 태그입니다. 이 요구 사항이 충족되지 않으면 `GameplayEffect`는 적용되지 않습니다. |
| Remove Gameplay Effects with Tags | 대상에 적용된 `GameplayEffect` 중 이 `GameplayEffect`의 `Asset Tags` 또는 `Granted Tags`에 이 태그 중 하나라도 포함된 `GameplayEffect`는 이 `GameplayEffect`가 성공적으로 적용될 때 대상에서 제거됩니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-immunity"></a>
#### 4.5.8 면역
`GameplayEffects`는 [`GameplayTags`](#concepts-gt)를 기반으로 면역을 부여하여 다른 `GameplayEffects`의 적용을 효과적으로 차단할 수 있습니다. 면역은 `Application Tag Requirements`와 같은 다른 수단을 통해서도 효과적으로 달성할 수 있지만, 이 시스템을 사용하면 면역으로 인해 `GameplayEffect`가 차단될 때 `UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate` 델리게이트를 제공합니다.

`GrantedApplicationImmunityTags`는 `Source ASC` (있는 경우 Source `Ability`의 `AbilityTags`에서 가져온 태그 포함)에 지정된 태그 중 하나라도 있는지 확인합니다. 이는 특정 캐릭터 또는 소스로부터의 모든 `GameplayEffects`에 대한 면역을 해당 태그를 기반으로 제공하는 방법입니다.

`Granted Application Immunity Query`는 들어오는 `GameplayEffectSpec`이 적용을 차단하거나 허용하기 위한 쿼리 중 하나와 일치하는지 확인합니다.

쿼리에는 `GameplayEffect` Blueprint에 유용한 호버 툴팁이 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-spec"></a>
#### 4.5.9 Gameplay Effect Spec
[`GameplayEffectSpec`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html)(`GESpec`)은 `GameplayEffects`의 인스턴스화로 생각할 수 있습니다. 이들은 `GameplayEffect` 클래스, 생성 레벨, 그리고 누가 생성했는지에 대한 참조를 가지고 있습니다. 이들은 런타임에 `GameplayEffect`와 달리 디자이너가 런타임 이전에 생성해야 하는 `GameplayEffect`와 달리 자유롭게 생성 및 수정할 수 있습니다. `GameplayEffect`를 적용할 때, `GameplayEffect`에서 `GameplayEffectSpec`이 생성되며, 실제로 대상에 적용되는 것은 이 `GameplayEffectSpec`입니다.

`GameplayEffectSpec`은 `UAbilitySystemComponent::MakeOutgoingSpec()`을 사용하여 `GameplayEffect`에서 생성되며, 이는 `BlueprintCallable`입니다. `GameplayEffectSpec`은 즉시 적용될 필요가 없습니다. `GameplayEffectSpec`을 `Ability`에서 생성된 투사체에 전달하여 투사체가 나중에 맞춘 대상에 적용하도록 하는 것이 일반적입니다. `GameplayEffectSpec`이 성공적으로 적용되면 `FActiveGameplayEffect`라는 새로운 구조체를 반환합니다.

주목할 만한 `GameplayEffectSpec` 내용:
*   이 `GameplayEffect`가 생성된 `GameplayEffect` 클래스.
*   이 `GameplayEffectSpec`의 레벨. 일반적으로 `GameplayEffectSpec`을 생성한 `Ability`의 레벨과 같지만 다를 수 있습니다.
*   `GameplayEffectSpec`의 지속 시간. 기본적으로 `GameplayEffect`의 지속 시간이지만 다를 수 있습니다.
*   주기적 효과를 위한 `GameplayEffectSpec`의 주기. 기본적으로 `GameplayEffect`의 주기이지만 다를 수 있습니다.
*   이 `GameplayEffectSpec`의 현재 스택 수. 스택 한도는 `GameplayEffect`에 있습니다.
*   [`GameplayEffectContextHandle`](#concepts-ge-context)는 이 `GameplayEffectSpec`을 누가 생성했는지 알려줍니다.
*   스냅샷팅으로 인해 `GameplayEffectSpec` 생성 시 캡처된 `Attribute`.
*   `GameplayEffect`가 부여하는 `GameplayTags` 외에 `GameplayEffectSpec`이 대상에 부여하는 `DynamicGrantedTags`.
*   `GameplayEffect`가 가진 `AssetTags` 외에 `GameplayEffectSpec`이 가진 `DynamicAssetTags`.
*   `SetByCaller` `TMap`.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-spec-setbycaller"></a>
##### 4.5.9.1 SetByCallers
`SetByCaller`는 `GameplayEffectSpec`이 `GameplayTag` 또는 `FName`과 연결된 float 값을 가지고 다닐 수 있도록 합니다. 이들은 `GameplayEffectSpec`의 각 `TMap`(`TMap<FGameplayTag, float>` 및 `TMap<FName, float>`)에 저장됩니다. 이들은 `GameplayEffect`의 `Modifier`로 사용되거나 float 값을 전달하는 일반적인 수단으로 사용될 수 있습니다. `Ability` 내에서 생성된 숫자 데이터를 `SetByCaller`를 통해 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 또는 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)로 전달하는 것이 일반적입니다.

| `SetByCaller` 사용 | 참고 사항 |
|---|---|
| `Modifiers` | `GameplayEffect` 클래스에 미리 정의되어야 합니다. `GameplayTag` 버전만 사용할 수 있습니다. `GameplayEffect` 클래스에 정의되었지만 `GameplayEffectSpec`에 해당 태그와 float 값 쌍이 없는 경우, `GameplayEffectSpec` 적용 시 런타임 오류가 발생하고 0이 반환됩니다. 이는 `Divide` 연산의 경우 잠재적인 문제입니다. [`Modifiers`](#concepts-ge-mods)를 참조하십시오. |
| 다른 곳 | 어디에도 미리 정의될 필요가 없습니다. `GameplayEffectSpec`에 존재하지 않는 `SetByCaller`를 읽으면 선택적 경고와 함께 개발자가 정의한 기본값을 반환할 수 있습니다. |

Blueprint에서 `SetByCaller` 값을 할당하려면 필요한 버전(`GameplayTag` 또는 `FName`)에 해당하는 Blueprint 노드를 사용하십시오:

![Assigning SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/setbycaller.png)

Blueprint에서 `SetByCaller` 값을 읽으려면 Blueprint 라이브러리에 사용자 지정 노드를 만들어야 합니다.

C++에서 `SetByCaller` 값을 할당하려면 필요한 버전의 함수(`GameplayTag` 또는 `FName`)를 사용하십시오:

```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);
```
```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);
```

C++에서 `SetByCaller` 값을 읽으려면 필요한 버전의 함수(`GameplayTag` 또는 `FName`)를 사용하십시오:

```c++
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```
```c++
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```

`FName` 버전보다 `GameplayTag` 버전을 사용하는 것을 권장합니다. 이는 Blueprint에서 철자 오류를 방지할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-context"></a>
#### 4.5.10 Gameplay Effect Context
[`GameplayEffectContext`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html) 구조체는 `GameplayEffectSpec`의 발동자 및 [`TargetData`](#concepts-targeting-data)에 대한 정보를 담고 있습니다. 또한 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) / [`GameplayEffectExecutionCalculations`](#concepts-ge-ec), [`AttributeSets`](#concepts-as), [`GameplayCues`](#concepts-gc)와 같은 장소 간에 임의의 데이터를 전달하기 위한 좋은 구조체입니다.

`GameplayEffectContext`를 서브클래스화하려면:

1.  `FGameplayEffectContext`를 서브클래스화
1.  `FGameplayEffectContext::GetScriptStruct()` 오버라이드
1.  `FGameplayEffectContext::Duplicate()` 오버라이드
1.  새 데이터가 복제되어야 하는 경우 `FGameplayEffectContext::NetSerialize()` 오버라이드
1.  부모 구조체 `FGameplayEffectContext`와 같이 서브클래스에 `TStructOpsTypeTraits` 구현
1.  [`AbilitySystemGlobals`](#concepts-asg) 클래스에서 `AllocGameplayEffectContext()`를 오버라이드하여 서브클래스의 새 객체를 반환

[GASShooter](https://github.com/tranek/GASShooter)는 서브클래스화된 `GameplayEffectContext`를 사용하여 `TargetData`를 추가하며, 특히 산탄총은 둘 이상의 적을 맞출 수 있으므로 `GameplayCues`에서 접근할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-mmc"></a>
#### 4.5.11 Modifier Magnitude Calculation
[`ModifierMagnitudeCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html)(`ModMagCalc` 또는 `MMC`)는 `GameplayEffects`에서 [`Modifier`](#concepts-ge-mods)로 사용되는 강력한 클래스입니다. 이들은 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec)와 유사하게 작동하지만 덜 강력하며, 가장 중요하게 [예측](#concepts-p)할 수 있습니다. 이들의 유일한 목적은 `CalculateBaseMagnitude_Implementation()`에서 float 값을 반환하는 것입니다. Blueprint 및 C++에서 이 함수를 서브클래스화하고 오버라이드할 수 있습니다.

`MMC`는 `Instant`, `Duration`, `Infinite`, `Periodic` 등 모든 지속 시간의 `GameplayEffects`에서 사용될 수 있습니다.

`MMC`의 강점은 `GameplayEffect`의 `Source` 또는 `Target`에 있는 여러 `Attribute`의 값을 [`GameplayTags` 및 `SetByCallers`](#concepts-ge-spec-setbycaller)를 읽기 위해 `GameplayEffectSpec`에 완전히 접근하여 캡처하는 기능에 있습니다. `Attribute`는 스냅샷될 수도 있고 그렇지 않을 수도 있습니다. 스냅샷된 `Attribute`는 `GameplayEffectSpec`이 생성될 때 캡처되는 반면, 스냅샷되지 않은 `Attribute`는 `GameplayEffectSpec`이 적용될 때 캡처되며 `Infinite` 및 `Duration` `GameplayEffects`의 경우 `Attribute`가 변경될 때 자동으로 업데이트됩니다. `Attribute`를 캡처하면 `ASC`의 기존 `Modifier`에서 `CurrentValue`를 재계산합니다. 이 재계산은 `AbilitySet`의 [`PreAttributeChange()`](#concepts-as-preattributechange)를 실행하지 않으므로, 여기에서 다시 클램핑을 수행해야 합니다.

| 스냅샷 | Source 또는 Target | `GameplayEffectSpec` 생성 시 캡처됨 | `Infinite` 또는 `Duration` `GE`의 `Attribute` 변경 시 자동 업데이트 |
|---|---|---|---|
| 예 | Source | 생성 | 아니요 |
| 예 | Target | 적용 | 아니요 |
| 아니요 | Source | 적용 | 예 |
| 아니요 | Target | 적용 | 예 |

`MMC`의 결과 float 값은 `GameplayEffect`의 `Modifier`에서 계수와 계수 전후 추가로 추가로 수정될 수 있습니다.

`Target`의 마나 `Attribute`를 캡처하여 독 효과로 인해 감소시키는 `MMC` 예시입니다. 여기서 감소량은 `Target`이 가진 마나량과 `Target`이 가질 수 있는 태그에 따라 달라집니다:
```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{

	//ManaDef는 헤더 FGameplayEffectAttributeCaptureDefinition ManaDef에 정의됨
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	//MaxManaDef는 헤더 FGameplayEffectAttributeCaptureDefinition MaxManaDef에 정의됨
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// 버프가 사용될 수 있는지를 결정하는 Source와 Target의 태그를 수집
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // 0으로 나누는 것을 방지

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// 대상이 마나의 절반 이상을 가지고 있다면 효과를 두 배로
		Reduction *= 2;
	}
	
	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// 대상이 PoisonMana에 약하다면 효과를 두 배로
		Reduction *= 2;
	}
	
	return Reduction;
}
```

`MMC`의 생성자에서 `FGameplayEffectAttributeCaptureDefinition`을 `RelevantAttributesToCapture`에 추가하지 않고 `Attribute`를 캡처하려고 하면, 캡처 중에 Spec 누락 오류가 발생할 것입니다. `Attribute`를 캡처할 필요가 없다면 `RelevantAttributesToCapture`에 아무것도 추가할 필요가 없습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-ec"></a>
#### 4.5.12 Gameplay Effect Execution Calculation
[`GameplayEffectExecutionCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectExecutionCalculat-/index.html)(`ExecutionCalculation`, `Execution`(플러그인 소스 코드에서 이 용어를 자주 볼 수 있습니다) 또는 `ExecCalc`)는 `GameplayEffects`가 `ASC`를 변경하는 가장 강력한 방법입니다. [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)와 마찬가지로, 이들은 `Attribute`를 캡처하고 선택적으로 스냅샷할 수 있습니다. `MMC`와 달리, 이들은 하나 이상의 `Attribute`를 변경할 수 있으며 본질적으로 프로그래머가 원하는 모든 것을 할 수 있습니다. 이 힘과 유연성의 단점은 이들이 [예측](#concepts-p)될 수 없으며 C++로 구현되어야 한다는 것입니다.

`ExecutionCalculations`는 `Instant` 및 `Periodic` `GameplayEffects`에만 사용될 수 있습니다. 'Execute'라는 단어가 포함된 모든 것은 일반적으로 이 두 가지 유형의 `GameplayEffects`를 나타냅니다.

스냅샷팅은 `GameplayEffectSpec`이 생성될 때 `Attribute`를 캡처하는 반면, 스냅샷팅하지 않으면 `GameplayEffectSpec`이 적용될 때 `Attribute`를 캡처합니다. `Attribute`를 캡처하면 `ASC`의 기존 `Modifier`에서 `CurrentValue`를 재계산합니다. 이 재계산은 `AbilitySet`의 [`PreAttributeChange()`](#concepts-as-preattributechange)를 실행하지 않으므로, 여기에서 다시 클램핑을 수행해야 합니다.

| 스냅샷 | Source 또는 Target | `GameplayEffectSpec` 생성 시 캡처됨 |
|---|---|---|
| 예 | Source | 생성 |
| 예 | Target | 적용 |
| 아니요 | Source | 적용 |
| 아니요 | Target | 적용 |

`Attribute` 캡처를 설정하려면 Epic의 ActionRPG 샘플 프로젝트에서 설정한 패턴을 따르십시오. `Attribute`를 캡처하는 방법을 정의하고 보유하는 구조체를 정의하고, 구조체의 생성자에서 하나의 사본을 만드십시오. 모든 `ExecCalc`마다 이러한 구조체가 있을 것입니다. **참고:** 각 구조체는 동일한 네임스페이스를 공유하므로 고유한 이름이 필요합니다. 구조체에 동일한 이름을 사용하면 `Attribute` 캡처 시 잘못된 동작이 발생합니다 (대부분 잘못된 `Attribute`의 값을 캡처함).

`Local Predicted`, `Server Only`, `Server Initiated` [`GameplayAbilities`](#concepts-ga)의 경우, `ExecCalc`는 서버에서만 호출됩니다.

`Source`와 `Target`의 여러 `Attribute`에서 읽는 복잡한 공식을 기반으로 받은 피해를 계산하는 것이 `ExecCalc`의 가장 일반적인 예입니다. 포함된 샘플 프로젝트에는 `GameplayEffectSpec`의 [`SetByCaller`](#concepts-ge-spec-setbycaller)에서 피해 값을 읽고 `Target`에서 캡처된 방어력 `Attribute`를 기반으로 해당 값을 완화하는 간단한 피해 계산 `ExecCalc`가 있습니다. `GDDamageExecCalculation.cpp/.h`를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-ec-senddata"></a>
##### 4.5.12.1 Execution Calculation으로 데이터 보내기
`ExecutionCalculation`으로 데이터를 보내는 방법은 `Attribute` 캡처 외에도 몇 가지가 있습니다.

<a name="concepts-ge-ec-senddata-setbycaller"></a>
###### 4.5.12.1.1 SetByCaller
[`GameplayEffectSpec`](#concepts-ge-spec-setbycaller)에 설정된 모든 [`SetByCaller`](#concepts-ge-spec-setbycaller)는 `ExecutionCalculation`에서 직접 읽을 수 있습니다.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

<a name="concepts-ge-ec-senddata-backingdataattribute"></a>
###### 4.5.12.1.2 백킹 데이터 Attribute Calculation Modifier
`GameplayEffect`에 값을 하드 코딩하고 싶다면, 캡처된 `Attribute` 중 하나를 백킹 데이터로 사용하는 `CalculationModifier`를 사용하여 전달할 수 있습니다.

이 스크린샷 예시에서는 캡처된 Damage `Attribute`에 50을 추가하고 있습니다. 하드코딩된 값만 가져오도록 `Override`로 설정할 수도 있습니다.

![Backing Data Attribute Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdataattribute.png)

`ExecutionCalculation`은 `Attribute`를 캡처할 때 이 값을 읽어옵니다.

```c++
float Damage = 0.0f;
// ExecutionCalculation에 CalculationModifier로 설정된 선택적 피해 값을 피해 GE에서 캡처 시도
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-backingdatatempvariable"></a>
###### 4.5.12.1.3 백킹 데이터 임시 변수 Calculation Modifier
`GameplayEffect`에 값을 하드 코딩하고 싶다면, `Temporary Variable` 또는 C++에서 `Transient Aggregator`라고 불리는 것을 사용하는 `CalculationModifier`를 통해 전달할 수 있습니다. `Temporary Variable`은 `GameplayTag`와 연결됩니다.

이 스크린샷 예시에서는 `Data.Damage` `GameplayTag`를 사용하여 `Temporary Variable`에 50을 추가하고 있습니다.

![Backing Data Temporary Variable Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdatatempvariable.png)

`ExecutionCalculation`의 생성자에 백킹 `Temporary Variable`을 추가합니다:

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

`ExecutionCalculation`은 `Attribute` 캡처 함수와 유사한 특별한 캡처 함수를 사용하여 이 값을 읽어옵니다.

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-effectcontext"></a>
###### 4.5.12.1.4 Gameplay Effect Context
`GameplayEffectSpec`에 커스텀 [`GameplayEffectContext`](#concepts-ge-context)를 통해 `ExecutionCalculation`으로 데이터를 보낼 수 있습니다.

`ExecutionCalculation`에서 `FGameplayEffectCustomExecutionParameters`에서 `EffectContext`에 접근할 수 있습니다.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```

`GameplayEffectSpec` 또는 `EffectContext`의 무언가를 변경해야 하는 경우:

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```

`ExecutionCalculation`에서 `GameplayEffectSpec`을 수정할 때는 주의하십시오. `GetOwningSpecForPreExecuteMod()`에 대한 주석을 참조하십시오.

```c++
/** const가 아닌 접근. 특히 Attribute 캡처 후 Spec을 수정할 때 주의하십시오. */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-car"></a>
#### 4.5.13 Custom Application Requirement
[`CustomApplicationRequirement`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectCustomApplication-/index.html)(`CAR`) 클래스는 `GameplayEffect`에 대한 간단한 `GameplayTag` 검사와 달리, 디자이너에게 `GameplayEffect`가 적용될 수 있는지 여부에 대한 고급 제어 기능을 제공합니다. 이들은 `CanApplyGameplayEffect()`를 오버라이드하여 Blueprint로 구현할 수 있으며, `CanApplyGameplayEffect_Implementation()`를 오버라이드하여 C++로 구현할 수 있습니다.

`CAR` 사용 시점의 예시:
*   `Target`은 특정 양의 `Attribute`를 가져야 합니다.
*   `Target`은 특정 수의 `GameplayEffect` 스택을 가져야 합니다.

`CAR`는 이 `GameplayEffect`의 인스턴스가 이미 `Target`에 있는지 확인하고 새 인스턴스를 적용하는 대신 기존 인스턴스의 [지속 시간을 변경하는](#concepts-ge-duration) 것과 같은 더 고급 작업을 수행할 수도 있습니다 (`CanApplyGameplayEffect()`에 대해 false 반환).

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-cost"></a>
#### 4.5.14 Cost Gameplay Effect
[`GameplayAbilities`](#concepts-ga)에는 `Ability`의 비용으로 사용하도록 특별히 설계된 선택적 `GameplayEffect`가 있습니다. 비용은 `ASC`가 `GameplayAbility`를 활성화할 수 있도록 가져야 하는 `Attribute`의 양입니다. `GA`가 `Cost GE`를 감당할 수 없다면 활성화할 수 없습니다. 이 `Cost GE`는 `Instant` `GameplayEffect`여야 하며, `Attribute`에서 빼는 하나 이상의 `Modifier`를 포함해야 합니다. 기본적으로 `Cost GE`는 예측되도록 의도되었으며, [`ExecutionCalculations`](#concepts-ge-ec)를 사용하지 않는 것이 좋습니다. `MMC`는 복잡한 비용 계산에 완벽하게 허용되며 권장됩니다.

처음에는 비용이 있는 `GA`마다 고유한 `Cost GE`를 가질 가능성이 높습니다. 더 고급 기술은 여러 `GA`에 하나의 `Cost GE`를 재사용하고, `Cost GE`에서 생성된 `GameplayEffectSpec`을 `GA`별 데이터(비용 값은 `GA`에 정의됨)로 수정하는 것입니다. **이는 `Instanced` `Ability`에만 해당됩니다**.

`Cost GE` 재사용을 위한 두 가지 기술:

1.  **`MMC` 사용.** 이것이 가장 쉬운 방법입니다. `GameplayEffectSpec`에서 가져올 수 있는 `GameplayAbility` 인스턴스에서 비용 값을 읽는 [`MMC`](#concepts-ge-mmc)를 만드십시오.

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

이 예시에서 비용 값은 제가 추가한 `GameplayAbility` 자식 클래스의 `FScalableFloat`입니다.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

![Cost GE With MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/costmmc.png)

2.  **`UGameplayAbility::GetCostGameplayEffect()` 오버라이드.** 이 함수를 오버라이드하고 [`런타임에 GameplayEffect를 생성`](#concepts-ge-dynamic)하여 `GameplayAbility`의 비용 값을 읽으십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-cooldown"></a>
#### 4.5.15 Cooldown Gameplay Effect
[`GameplayAbilities`](#concepts-ga)에는 `Ability`의 쿨다운으로 사용하도록 특별히 설계된 선택적 `GameplayEffect`가 있습니다. 쿨다운은 `Ability`가 활성화된 후 다시 활성화될 수 있는 기간을 결정합니다. `GA`가 아직 쿨다운 중이면 활성화할 수 없습니다. 이 `Cooldown GE`는 `Modifier`가 없고 `GameplayEffect`의 `GrantedTags`에 `GameplayAbility`마다 또는 `Ability` 슬롯마다(게임에 쿨다운을 공유하는 교체 가능한 `Ability`가 있는 경우) 고유한 `GameplayTag`(`"Cooldown Tag"`)를 가진 `Duration` `GameplayEffect`여야 합니다. `GA`는 실제로 `Cooldown GE`의 존재 대신 `Cooldown Tag`의 존재를 확인합니다. 기본적으로 `Cooldown GE`는 예측되도록 의도되었으며, [`ExecutionCalculations`](#concepts-ge-ec)를 사용하지 않는 것이 좋습니다. `MMC`는 복잡한 쿨다운 계산에 완벽하게 허용되며 권장됩니다.

처음에는 쿨다운이 있는 `GA`마다 고유한 `Cooldown GE`를 가질 가능성이 높습니다. 더 고급 기술은 여러 `GA`에 하나의 `Cooldown GE`를 재사용하고, `Cooldown GE`에서 생성된 `GameplayEffectSpec`을 `GA`별 데이터(쿨다운 지속 시간과 `Cooldown Tag`는 `GA`에 정의됨)로 수정하는 것입니다. **이는 `Instanced` `Ability`에만 해당됩니다**.

`Cooldown GE` 재사용을 위한 두 가지 기술:

1.  **[`SetByCaller`](#concepts-ge-spec-setbycaller) 사용.** 이것이 가장 쉬운 방법입니다. 공유 `Cooldown GE`의 지속 시간을 `GameplayTag`와 함께 `SetByCaller`로 설정합니다. `GameplayAbility` 서브클래스에서 지속 시간 (`FScalableFloat`)에 대한 float, 고유한 `Cooldown Tag`에 대한 `FGameplayTagContainer`, 그리고 `Cooldown Tag`와 `Cooldown GE`의 태그의 합집합을 반환 포인터로 사용할 임시 `FGameplayTagContainer`를 정의합니다.

```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// GetCooldownTags()에서 포인터를 반환할 임시 컨테이너.
// 이것은 CooldownTags와 Cooldown GE의 Cooldown 태그의 합집합이 될 것입니다.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

그런 다음 `UGameplayAbility::GetCooldownTags()`를 오버라이드하여 `Cooldown Tags`와 기존 `Cooldown GE` 태그의 합집합을 반환하도록 합니다.

```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags는 CDO의 TempCooldownTags에 쓰므로, Ability Cooldown 태그가 변경될 경우 (다른 슬롯으로 이동) 지웁니다.
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

마지막으로, `UGameplayAbility::ApplyCooldown()`를 오버라이드하여 `Cooldown Tags`를 삽입하고 `SetByCaller`를 쿨다운 `GameplayEffectSpec`에 추가합니다.

```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

이 그림에서 쿨다운의 지속 시간 `Modifier`는 `Data.Cooldown`이라는 `Data Tag`를 가진 `SetByCaller`로 설정됩니다. `Data.Cooldown`은 위 코드에서 `OurSetByCallerTag`가 될 것입니다.

![Cooldown GE with SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownsbc.png)

2.  **[`MMC`](#concepts-ge-mmc) 사용.** 위와 동일한 설정이지만, `Cooldown GE`의 지속 시간을 `SetByCaller`로 설정하고 `ApplyCooldown`에서도 그렇게 하는 대신, 지속 시간을 `Custom Calculation Class`로 설정하고 새로 만들 `MMC`를 가리키게 합니다.

```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// GetCooldownTags()에서 포인터를 반환할 임시 컨테이너.
// 이것은 CooldownTags와 Cooldown GE의 Cooldown 태그의 합집합이 될 것입니다.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

그런 다음 `UGameplayAbility::GetCooldownTags()`를 오버라이드하여 `Cooldown Tags`와 기존 `Cooldown GE` 태그의 합집합을 반환하도록 합니다.

```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags는 CDO의 TempCooldownTags에 쓰므로, Ability Cooldown 태그가 변경될 경우 (다른 슬롯으로 이동) 지웁니다.
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

마지막으로, `UGameplayAbility::ApplyCooldown()`를 오버라이드하여 `Cooldown Tags`를 쿨다운 `GameplayEffectSpec`에 주입합니다.

```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

![Cooldown GE with MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownmmc.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-cooldown-tr"></a>
##### 4.5.15.1 Cooldown Gameplay Effect의 남은 시간 가져오기
```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
		TArray< TPair<float, float> > DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```

**참고:** 클라이언트에서 쿨다운의 남은 시간을 쿼리하려면 복제된 `GameplayEffects`를 수신할 수 있어야 합니다. 이는 `ASC`의 [복제 모드](#concepts-asc-rm)에 따라 달라집니다.

<a name="concepts-ge-cooldown-listen"></a>
##### 4.5.15.2 Cooldown 시작 및 종료 감지
쿨다운이 시작될 때를 감지하려면 `AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf`에 바인딩하여 `Cooldown GE`가 적용될 때 반응하거나, `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`에 바인딩하여 `Cooldown Tag`가 추가될 때 반응할 수 있습니다. `Cooldown GE`가 추가될 때를 감지하는 것을 권장합니다. 왜냐하면 `Cooldown GE`를 적용한 `GameplayEffectSpec`에도 접근할 수 있기 때문입니다. 이를 통해 `Cooldown GE`가 로컬에서 예측된 것인지 서버의 교정된 것인지 판단할 수 있습니다.

쿨다운이 끝날 때를 감지하려면 `AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()`에 바인딩하여 `Cooldown GE`가 제거될 때 반응하거나, `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`에 바인딩하여 `Cooldown Tag`가 제거될 때 반응할 수 있습니다. `Cooldown Tag`가 제거될 때를 감지하는 것을 권장합니다. 왜냐하면 서버의 교정된 `Cooldown GE`가 들어오면 로컬에서 예측된 `Cooldown GE`를 제거하여 `OnAnyGameplayEffectRemovedDelegate()`가 발동되더라도 여전히 쿨다운 중일 수 있기 때문입니다. 예측된 `Cooldown GE`가 제거되고 서버의 교정된 `Cooldown GE`가 적용되는 동안에는 `Cooldown Tag`가 변경되지 않습니다.

**참고:** 클라이언트에서 `GameplayEffect`가 추가되거나 제거될 때를 감지하려면 복제된 `GameplayEffects`를 수신할 수 있어야 합니다. 이는 `ASC`의 [복제 모드](#concepts-asc-rm)에 따라 달라집니다.

샘플 프로젝트에는 쿨다운 시작 및 종료를 감지하는 사용자 지정 Blueprint 노드가 포함되어 있습니다. HUD UMG 위젯은 이를 사용하여 유성 `Ability`의 쿨다운 남은 시간을 업데이트합니다. 이 `AsyncTask`는 UMG 위젯의 `Destruct` 이벤트에서 호출하는 `EndTask()`가 수동으로 호출될 때까지 영원히 지속됩니다. [`AsyncTaskCooldownChanged.h/cpp`](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp)를 참조하십시오.

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

<a name="concepts-ge-cooldown-prediction"></a>
##### 4.5.15.3 Cooldown 예측
현재 쿨다운은 실제로 예측할 수 없습니다. 로컬에서 예측된 `Cooldown GE`가 적용될 때 UI 쿨다운 타이머를 시작할 수 있지만, `GameplayAbility`의 실제 쿨다운은 서버 쿨다운의 남은 시간과 연결되어 있습니다. 플레이어의 지연 시간에 따라 로컬에서 예측된 쿨다운이 만료될 수 있지만, `GameplayAbility`는 서버에서 여전히 쿨다운 중일 수 있으며, 이는 서버 쿨다운이 만료될 때까지 `GameplayAbility`의 즉각적인 재활성화를 방지합니다.

샘플 프로젝트는 로컬에서 예측된 쿨다운이 시작될 때 유성 `Ability`의 UI 아이콘을 회색으로 표시한 다음, 서버의 교정된 `Cooldown GE`가 들어오면 쿨다운 타이머를 시작하여 이 문제를 해결합니다.

이로 인한 게임 플레이 결과는 지연 시간이 높은 플레이어가 지연 시간이 낮은 플레이어보다 짧은 쿨다운 `Ability`의 발사 속도가 낮아 불리하다는 것입니다. Fortnite는 쿨다운 `GameplayEffects`를 사용하지 않는 사용자 지정 장부 기록을 통해 이를 피합니다.

진정한 예측된 쿨다운을 허용하는 것 (로컬 쿨다운이 만료되었을 때 플레이어가 `GameplayAbility`를 활성화할 수 있지만 서버는 여전히 쿨다운 중)은 Epic이 [미래의 GAS 반복](#concepts-p-future)에서 언젠가 구현하고 싶은 기능입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-duration"></a>
#### 4.5.16 활성화된 Gameplay Effect 지속 시간 변경
`Cooldown GE` 또는 모든 `Duration` `GameplayEffect`의 남은 시간을 변경하려면 `GameplayEffectSpec`의 `Duration`을 변경하고, `StartServerWorldTime`을 업데이트하고, `CachedStartServerWorldTime`을 업데이트하고, `StartWorldTime`을 업데이트하고, `CheckDuration()`으로 지속 시간 확인을 다시 실행해야 합니다. 서버에서 이 작업을 수행하고 `FActiveGameplayEffect`를 더럽혀졌다고 표시하면 클라이언트에 변경 사항이 복제됩니다.
**참고:** 이 작업에는 `const_cast`가 포함되며 Epic의 의도된 지속 시간 변경 방법이 아닐 수 있지만, 지금까지는 잘 작동하는 것 같습니다.

```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-dynamic"></a>
#### 4.5.17 런타임에 동적 Gameplay Effect 생성하기
런타임에 동적 `GameplayEffects`를 생성하는 것은 고급 주제입니다. 이 작업을 너무 자주 할 필요는 없을 것입니다.

`Instant` `GameplayEffects`만 C++에서 처음부터 런타임에 생성할 수 있습니다. `Duration` 및 `Infinite` `GameplayEffects`는 런타임에 동적으로 생성할 수 없습니다. 왜냐하면 이들이 복제될 때 존재하지 않는 `GameplayEffect` 클래스 정의를 찾기 때문입니다. 이 기능을 달성하려면 대신 에디터에서 일반적으로 하는 것처럼 archetype `GameplayEffect` 클래스를 만들어야 합니다. 그런 다음 런타임에 필요한 내용으로 `GameplayEffectSpec` 인스턴스를 사용자 정의하십시오.

런타임에 생성된 `Instant` `GameplayEffects`는 [로컬에서 예측된](#concepts-p) `GameplayAbility` 내에서도 호출될 수 있습니다. 그러나 동적 생성에 부작용이 있을지는 아직 알려져 있지 않습니다.

##### 예시

샘플 프로젝트는 캐릭터가 마지막 타격을 받을 때 `AttributeSet`에서 킬러에게 골드와 경험치 포인트를 보내기 위해 하나를 생성합니다.

```c++
// 현상금을 주기 위한 동적 즉시 Gameplay Effect 생성
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```

두 번째 예는 로컬 예측 `GameplayAbility` 내에서 생성된 런타임 `GameplayEffect`를 보여줍니다. 자신의 책임하에 사용하십시오 (코드의 주석 참조)!

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// 런타임에 GE 생성.
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // 런타임 GE에는 Instant만 작동합니다.

		// 간단한 Scalable Float Modifier를 추가합니다. MyAttribute를 42로 덮어씁니다.
		// 실제 응용 프로그램에서는 TriggerEventData를 통해 전달된 정보를 사용합니다.
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// GE 적용.

		// GE 클래스 기본 객체에서 GESpec를 생성하는 ASC의 동작을 피하기 위해 여기에서 GESpec를 생성합니다.
		// 여기에 동적 GE가 있으므로, 이는 기본 GameplayEffect 클래스로 GESpec를 생성하게 되어 Modifier를 잃게 됩니다. 주의: 여기에서 수행되는 이 "꼼수"가 단점이 있을지는 알려져 있지 않습니다!
		// Spec은 GE가 Spec의 UPROPERTY이므로 GarbageCollector에 의해 GE 객체가 수집되는 것을 방지합니다.
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // 핸들 내의 shared ptr에 의해 수명이 관리되므로 "new"
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ge-containers"></a>
#### 4.5.18 Gameplay Effect Containers
Epic의 [Action RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)는 `FGameplayEffectContainer`라는 구조체를 구현합니다. 이것들은 바닐라 GAS에는 없지만 `GameplayEffects`와 [`TargetData`](#concepts-targeting-data)를 포함하는 데 매우 유용합니다. `GameplayEffects`에서 `GameplayEffectSpecs`를 생성하고 `GameplayEffectContext`에서 기본값을 설정하는 것과 같은 일부 작업을 자동화합니다. `GameplayAbility`에서 `GameplayEffectContainer`를 만들고 생성된 투사체에 전달하는 것은 매우 쉽고 간단합니다. 저는 포함된 샘플 프로젝트에서 `GameplayEffectContainers`를 구현하지 않기로 결정하여 바닐라 GAS에서 이들을 어떻게 작업하는지 보여주었지만, 이들을 살펴보고 프로젝트에 추가하는 것을 강력히 권장합니다.

`GameplayEffectContainers` 내부의 `GESpecs`에 접근하여 `SetByCaller`를 추가하는 등의 작업을 수행하려면 `FGameplayEffectContainer`를 분해하고 `GESpecs` 배열에서 `GESpec` 참조에 인덱스로 접근해야 합니다. 이렇게 하려면 접근하려는 `GESpec`의 인덱스를 미리 알고 있어야 합니다.

![SetByCaller with a GameplayEffectContainer](https://github.tranek/GASDocumentation/raw/master/Images/gecontainersetbycaller.png)

`GameplayEffectContainers`에는 [타겟팅](#concepts-targeting-containers)을 위한 선택적이고 효율적인 수단도 포함되어 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga"></a>
### 4.6 Gameplay Abilities

<a name="concepts-ga-definition"></a>
#### 4.6.1 Gameplay Ability 정의
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html)(`GA`)는 `Actor`가 게임에서 수행할 수 있는 모든 행동 또는 스킬입니다. 예를 들어, 스프린트와 총을 쏘는 것처럼 한 번에 둘 이상의 `GameplayAbility`가 활성화될 수 있습니다. 이들은 Blueprint 또는 C++로 만들 수 있습니다.

`GameplayAbilities`의 예시:
*   점프하기
*   스프린트하기
*   총 쏘기
*   X초마다 수동으로 공격 막기
*   물약 사용하기
*   문 열기
*   자원 수집하기
*   건물 건설하기

`GameplayAbilities`로 구현해서는 안 되는 것:
*   기본 이동 입력
*   UI와의 일부 상호 작용 - 상점에서 아이템을 구매하기 위해 `GameplayAbility`를 사용하지 마십시오.

이들은 규칙이 아니라 제 권장 사항입니다. 당신의 디자인과 구현은 다를 수 있습니다.

`GameplayAbility`는 `Attribute` 변경량을 수정하거나 `GameplayAbility`의 기능을 변경하기 위한 레벨 기능을 기본적으로 제공합니다.

`GameplayAbility`는 [`Net Execution Policy`](#concepts-ga-net)에 따라 소유 클라이언트 및/또는 서버에서 실행되지만 시뮬레이트된 프록시에서는 실행되지 않습니다. `Net Execution Policy`는 `GameplayAbility`가 로컬에서 [예측](#concepts-p)될지 여부를 결정합니다. 이들은 [선택적 비용 및 쿨다운 `GameplayEffects`](#concepts-ga-commit)에 대한 기본 동작을 포함합니다. `GameplayAbility`는 이벤트 대기, `Attribute` 변경 대기, 플레이어가 대상을 선택할 때까지 대기, 또는 `Root Motion Source`로 `Character` 이동과 같이 시간이 지남에 따라 발생하는 작업에 [`AbilityTasks`](#concepts-at)를 사용합니다. **시뮬레이트된 클라이언트는 `GameplayAbility`를 실행하지 않습니다**. 대신, 서버가 `Ability`를 실행할 때, 시뮬레이트된 프록시에서 시각적으로 재생되어야 하는 모든 것 (애니메이션 몽타주 등)은 `AbilityTasks` 또는 음향 및 파티클과 같은 미적인 것들을 위한 [`GameplayCues`](#concepts-gc)를 통해 복제되거나 RPC될 것입니다.

모든 `GameplayAbility`는 게임 플레이 로직으로 `ActivateAbility()` 함수를 오버라이드합니다. `GameplayAbility`가 완료되거나 취소될 때 실행되는 `EndAbility()`에 추가 로직을 추가할 수 있습니다.

간단한 `GameplayAbility`의 순서도:
![Simple GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)


더 복잡한 `GameplayAbility`의 순서도:
![Complex GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

복잡한 능력은 서로 상호 작용하는 (활성화, 취소 등) 여러 `GameplayAbility`를 사용하여 구현할 수 있습니다.

<a name="concepts-ga-definition-reppolicy"></a>
##### 4.6.1.1 리플리케이션 정책
이 옵션을 사용하지 마십시오. 이름이 오해의 소지가 있으며 필요하지 않습니다. [`GameplayAbilitySpecs`](#concepts-ga-spec)는 기본적으로 서버에서 소유 클라이언트로 복제됩니다. 위에서 언급했듯이, **`GameplayAbility`는 시뮬레이트된 프록시에서 실행되지 않습니다**. 이들은 시뮬레이트된 프록시에 시각적 변경 사항을 복제하거나 RPC하기 위해 `AbilityTasks` 및 `GameplayCues`를 사용합니다. Epic의 Dave Ratti는 [미래에 이 옵션을 제거](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)하고 싶다고 밝혔습니다.

<a name="concepts-ga-definition-remotecancel"></a>
##### 4.6.1.2 서버가 원격 Ability 취소를 존중함
이 옵션은 종종 문제의 원인이 됩니다. 클라이언트의 `GameplayAbility`가 취소 또는 자연스러운 완료로 인해 종료되면, 서버 버전도 완료 여부와 관계없이 강제로 종료됩니다. 후자의 문제가 특히 지연 시간이 높은 플레이어가 사용하는 로컬 예측 `GameplayAbility`의 경우 중요합니다. 일반적으로 이 옵션을 비활성화하는 것이 좋습니다.

<a name="concepts-ga-definition-repinputdirectly"></a>
##### 4.6.1.3 입력 직접 리플리케이트
이 옵션을 설정하면 입력 누름 및 해제 이벤트가 항상 서버로 복제됩니다. Epic은 이 기능을 사용하지 않고, [입력이 `ASC`에 바인딩된 경우](#concepts-ga-input) 기존 입력 관련 [`AbilityTasks`](#concepts-at)에 내장된 `Generic Replicated Events`에 의존할 것을 권장합니다.

Epic의 주석:
```c++
/** 직접 입력 상태 복제. bReplicateInputDirectly가 Ability에서 true이고 일반적으로 사용하기에 좋지 않은 경우 (대신 Generic Replicated Events를 사용하는 것이 좋음) 호출됩니다. */
UAbilitySystemComponent::ServerSetInputPressed()
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-input"></a>
#### 4.6.2 입력을 ASC에 바인딩하기
`ASC`는 입력 액션을 직접 바인딩하고 `GameplayAbility`를 부여할 때 해당 입력을 `GameplayAbility`에 할당할 수 있도록 합니다. `GameplayAbility`에 할당된 입력 액션은 `GameplayTag` 요구 사항이 충족되면 눌렀을 때 자동으로 `GameplayAbility`를 활성화합니다. 할당된 입력 액션은 입력에 반응하는 내장 `AbilityTasks`를 사용하는 데 필요합니다.

`GameplayAbility`를 활성화하는 데 할당된 입력 액션 외에도 `ASC`는 일반 `Confirm` 및 `Cancel` 입력을 받습니다. 이 특수 입력은 [`Target Actors`](#concepts-targeting-actors)를 확인하거나 취소하는 것과 같은 작업을 위해 `AbilityTasks`에서 사용됩니다.

`ASC`에 입력을 바인딩하려면 먼저 입력 액션 이름을 바이트로 변환하는 enum을 만들어야 합니다. enum 이름은 프로젝트 설정에서 입력 액션에 사용된 이름과 정확히 일치해야 합니다. `DisplayName`은 중요하지 않습니다.

샘플 프로젝트에서 발췌:
```c++
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 없음
	None			UMETA(DisplayName = "None"),
	// 1 확인
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 취소
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 마우스 왼쪽 버튼
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 마우스 오른쪽 버튼
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 스프린트
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 점프
	Jump			UMETA(DisplayName = "Jump")
};
```

`ASC`가 `Character`에 있다면, `SetupPlayerInputComponent()`에서 `ASC`에 바인딩하는 함수를 포함합니다:
```c++
// AbilitySystemComponent에 바인딩
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

`ASC`가 `PlayerState`에 있는 경우, `SetupPlayerInputComponent()` 내부에 `PlayerState`가 아직 클라이언트에 복제되지 않았을 수 있는 잠재적인 경쟁 조건이 존재합니다. 따라서 `SetupPlayerInputComponent()` 및 `OnRep_PlayerState()`에서 입력에 바인딩을 시도하는 것을 권장합니다. `OnRep_PlayerState()`만으로는 충분하지 않을 수 있습니다. `PlayerState`가 `PlayerController`가 클라이언트에게 `ClientRestart()`를 호출하도록 지시하기 전에 복제될 때 `Actor`의 `InputComponent`가 null일 수 있기 때문입니다. 샘플 프로젝트는 프로세스를 게이트하는 부울 변수를 사용하여 두 위치 모두에서 바인딩을 시도하여 입력을 한 번만 바인딩하도록 시연합니다.

**참고:** 샘플 프로젝트에서 enum의 `Confirm`과 `Cancel`은 프로젝트 설정의 입력 액션 이름(`ConfirmTarget` 및 `CancelTarget`)과 일치하지 않지만, `BindAbilityActivationToInputComponent()`에서 그들 사이의 매핑을 제공합니다. 이들은 매핑을 제공하므로 일치할 필요가 없다는 점에서 특별하지만, 일치할 수도 있습니다. enum의 다른 모든 입력은 프로젝트 설정의 입력 액션 이름과 일치해야 합니다.

하나의 입력에 의해서만 활성화되는 `GameplayAbility` (예: MOBA의 "슬롯"에 항상 존재하는 `Ability`)의 경우, `UGameplayAbility` 서브클래스에 변수를 추가하여 입력을 정의할 수 있도록 하는 것을 선호합니다. 그런 다음 `Ability`를 부여할 때 `ClassDefaultObject`에서 이를 읽을 수 있습니다.

<a name="concepts-ga-input-noactivate"></a>
##### 4.6.2.1 Ability 활성화 없이 입력에 바인딩하기
입력이 눌러졌을 때 `GameplayAbility`가 자동으로 활성화되는 것을 원치 않지만, `AbilityTasks`와 함께 사용하기 위해 입력에 바인딩하고 싶다면, `UGameplayAbility` 서브클래스에 `bActivateOnInput`이라는 새로운 bool 변수를 추가하고 기본값을 `true`로 설정한 다음, `UAbilitySystemComponent::AbilityLocalInputPressed()`를 오버라이드할 수 있습니다.

```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// 이 InputID가 GenericConfirm/Cancel로 오버로드되어 있고 GenericConfim/Cancel 콜백이 바인딩되어 있으면 입력을 소비
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// InputPressed 이벤트를 호출합니다. 이것은 여기서 복제되지 않습니다. 누군가 듣고 있다면 InputPressed 이벤트를 서버로 복제할 수 있습니다.
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// Ability가 활성화되지 않았으므로, 활성화 시도
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-granting"></a>
#### 4.6.3 Ability 부여하기
`GameplayAbility`를 `ASC`에 부여하면 `ASC`의 `ActivatableAbilities` 목록에 추가되어 `GameplayTag` 요구 사항을 충족하면 언제든지 `GameplayAbility`를 활성화할 수 있습니다.

서버에서 `GameplayAbility`를 부여하면 [`GameplayAbilitySpec`](#concepts-ga-spec)이 소유 클라이언트로 자동으로 복제됩니다. 다른 클라이언트/시뮬레이트된 프록시는 `GameplayAbilitySpec`을 수신하지 않습니다.

샘플 프로젝트는 `Character` 클래스에 `TArray<TSubclassOf<UGDGameplayAbility>>`를 저장하며, 게임 시작 시 이를 읽어 `Ability`를 부여합니다:
```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// Ability 부여, 단 서버에서만
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(StartupAbility, GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), this));
	}

	AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

이러한 `GameplayAbility`를 부여할 때, `UGameplayAbility` 클래스, `Ability` 레벨, 바인딩된 입력, 그리고 이 `GameplayAbility`를 `ASC`에 부여한 `SourceObject`를 사용하여 `GameplayAbilitySpec`을 생성합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-activating"></a>
#### 4.6.4 Ability 활성화하기
`GameplayAbility`에 입력 액션이 할당되면, 입력이 눌러지고 `GameplayTag` 요구 사항이 충족되면 자동으로 활성화됩니다. 이것이 항상 `GameplayAbility`를 활성화하는 바람직한 방법은 아닐 수 있습니다. `ASC`는 `GameplayAbility`를 활성화하는 네 가지 다른 방법( `GameplayTag`별, `GameplayAbility` 클래스별, `GameplayAbilitySpec` 핸들별, 이벤트별)을 제공합니다. 이벤트로 `GameplayAbility`를 활성화하면 [이벤트와 함께 데이터 페이로드를 전달](#concepts-ga-data)할 수 있습니다.

```c++
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);

UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);

bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);

bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);

FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);
```
이벤트로 `GameplayAbility`를 활성화하려면 `GameplayAbility`에 `Triggers`가 설정되어 있어야 합니다. `GameplayTag`를 할당하고 `GameplayEvent`에 대한 옵션을 선택하십시오. 이벤트를 보내려면 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)` 함수를 사용하십시오. 이벤트로 `GameplayAbility`를 활성화하면 페이로드와 함께 데이터를 전달할 수 있습니다.

`GameplayAbility` `Triggers`는 `GameplayTag`가 추가되거나 제거될 때 `GameplayAbility`를 활성화할 수도 있습니다.

**참고:** Blueprint에서 이벤트로 `GameplayAbility`를 활성화할 때 `ActivateAbilityFromEvent` 노드를 사용해야 합니다.

**참고:** 패시브 `Ability`처럼 항상 실행되는 `GameplayAbility`가 아니라면 `GameplayAbility`가 종료되어야 할 때 `EndAbility()`를 호출하는 것을 잊지 마십시오.

**로컬 예측된** `GameplayAbility`의 활성화 시퀀스:
1.  **소유 클라이언트**는 `TryActivateAbility()`를 호출합니다.
1.  `InternalTryActivateAbility()`를 호출합니다.
1.  `CanActivateAbility()`를 호출하고 `GameplayTag` 요구 사항이 충족되었는지, `ASC`가 비용을 감당할 수 있는지, `GameplayAbility`가 쿨다운 중이 아닌지, 다른 인스턴스가 현재 활성화되어 있지 않은지 여부를 반환합니다.
1.  `CallServerTryActivateAbility()`를 호출하고 생성된 `Prediction Key`를 전달합니다.
1.  `CallActivateAbility()`를 호출합니다.
1.  `PreActivate()`를 호출합니다. Epic은 이를 "상용구 초기화 작업"이라고 부릅니다.
1.  `ActivateAbility()`를 호출하여 `Ability`를 최종적으로 활성화합니다.

**서버**는 `CallServerTryActivateAbility()`를 수신합니다.
1.  `ServerTryActivateAbility()`를 호출합니다.
1.  `InternalServerTryActivateAbility()`를 호출합니다.
1.  `InternalTryActivateAbility()`를 호출합니다.
1.  `CanActivateAbility()`를 호출하고 `GameplayTag` 요구 사항이 충족되었는지, `ASC`가 비용을 감당할 수 있는지, `GameplayAbility`가 쿨다운 중이 아닌지, 다른 인스턴스가 현재 활성화되어 있지 않은지 여부를 반환합니다.
1.  성공하면 `ClientActivateAbilitySucceed()`를 호출하여 활성화가 서버에 의해 확인되었음을 `ActivationInfo`에 업데이트하고 `OnConfirmDelegate` 델리게이트를 브로드캐스팅하도록 지시합니다. 이는 입력 확인과 다릅니다.
1.  `CallActivateAbility()`를 호출합니다.
1.  `PreActivate()`를 호출합니다. Epic은 이를 "상용구 초기화 작업"이라고 부릅니다.
1.  `ActivateAbility()`를 호출하여 `Ability`를 최종적으로 활성화합니다.

서버가 언제든지 활성화에 실패하면 `ClientActivateAbilityFailed()`를 호출하여 클라이언트의 `GameplayAbility`를 즉시 종료하고 예측된 모든 변경 사항을 되돌립니다.

<a name="concepts-ga-activating-passive"></a>
##### 4.6.4.1 패시브 Ability
자동으로 활성화되고 계속 실행되는 패시브 `GameplayAbility`를 구현하려면 `UGameplayAbility::OnAvatarSet()`을 오버라이드하고 `TryActivateAbility()`를 호출하십시오. `OnAvatarSet()`은 `GameplayAbility`가 부여되고 `AvatarActor`가 설정될 때 자동으로 호출됩니다.

사용자 지정 `UGameplayAbility` 클래스에 `GameplayAbility`가 부여될 때 활성화되어야 하는지 여부를 지정하는 `bool` 변수를 추가하는 것을 권장합니다. 샘플 프로젝트는 패시브 방어력 스태킹 `Ability`에 이 작업을 수행합니다.

패시브 `GameplayAbility`는 일반적으로 `Server Only`의 [`Net Execution Policy`](#concepts-ga-net)를 가질 것입니다.

```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

Epic은 이 함수를 패시브 `Ability`를 시작하고 `BeginPlay` 유형의 작업을 수행하기에 올바른 장소라고 설명합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-activating-failedtags"></a>
##### 4.6.4.2 활성화 실패 태그

Ability는 Ability 활성화가 실패한 이유를 알려주는 기본 로직을 가지고 있습니다. 이를 활성화하려면 기본 실패 사례에 해당하는 GameplayTag를 설정해야 합니다.

프로젝트에 다음 태그(또는 사용자 지정 명명 규칙)를 추가하십시오:
```
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

그런 다음 [`GASDocumentation\Config\DefaultGame.ini`](https://github.com/tranek/GASDocumentation/blob/master/Config/DefaultGame.ini#L8-L13)에 추가하십시오:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

이제 `Ability` 활성화가 실패할 때마다 해당 `GameplayTag`가 출력 로그 메시지에 포함되거나 `showdebug AbilitySystem` HUD에 표시됩니다.
```
LogAbilitySystem: Display: InternalServerTryActivateAbility. Default__GA_FireGun_C의 클라이언트 활성화를 거부합니다. InternalTryActivateAbility 실패: Activation.Fail.BlockedByTags
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. 예측 키 :109 Ability: Default__GA_FireGun_C
```

![Activation Failed Tags Displayed in showdebug AbilitySystem](https://github.com/tranek/GASDocumentation/raw/master/Images/activationfailedtags.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-cancelabilities"></a>
#### 4.6.5 Ability 취소하기
`GameplayAbility` 내부에서 취소하려면 `CancelAbility()`를 호출하십시오. 그러면 `EndAbility()`가 호출되고 `WasCancelled` 매개변수가 true로 설정됩니다.

외부에서 `GameplayAbility`를 취소하려면 `ASC`가 몇 가지 함수를 제공합니다:

```c++
/** 지정된 Ability CDO를 취소합니다. */
void CancelAbility(UGameplayAbility* Ability);	

/** 전달된 Spec 핸들로 표시된 Ability를 취소합니다. 핸들이 다시 활성화된 Ability 중에서 발견되지 않으면 아무것도 발생하지 않습니다. */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** 지정된 태그를 가진 모든 Ability를 취소합니다. Ignore 인스턴스는 취소하지 않습니다. */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** 태그에 관계없이 모든 Ability를 취소합니다. Ignore 인스턴스는 취소하지 않습니다. */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** 모든 Ability를 취소하고 남아 있는 모든 인스턴스화된 Ability를 파괴합니다. */
virtual void DestroyActiveState();
```

**참고:** `CancelAllAbilities`는 `Non-Instanced` `GameplayAbility`를 가지고 있는 경우 제대로 작동하지 않는 것으로 보입니다. `Non-Instanced` `GameplayAbility`에 도달하면 포기하는 것 같습니다. `CancelAbilities`는 `Non-Instanced` `GameplayAbility`를 더 잘 처리할 수 있으며, 샘플 프로젝트에서 사용되는 것이 바로 그것입니다 (점프는 비인스턴스 `GameplayAbility`입니다). 사용 결과는 다를 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-definition-activeability"></a>
#### 4.6.6 활성화된 Ability 가져오기
초보자들은 종종 "활성화된 Ability를 어떻게 얻을 수 있나요?"라고 묻습니다. 아마도 변수를 설정하거나 취소하기 위해서일 것입니다. 두 개 이상의 `GameplayAbility`가 동시에 활성화될 수 있으므로 하나의 "활성화된 Ability"는 없습니다. 대신, `ASC`의 `ActivatableAbilities` 목록( `ASC`가 소유하는 부여된 `GameplayAbility`)을 검색하여 찾고 있는 [`Asset` 또는 `Granted` `GameplayTag`](#concepts-ga-tags)와 일치하는 것을 찾아야 합니다.

`UAbilitySystemComponent::GetActivatableAbilities()`는 반복할 수 있는 `TArray<FGameplayAbilitySpec>`을 반환합니다.

`ASC`는 `GameplayAbilitySpec` 목록을 수동으로 반복하는 대신 검색에 도움이 되는 `GameplayTagContainer`를 매개변수로 받는 또 다른 도우미 함수를 가집니다. `bOnlyAbilitiesThatSatisfyTagRequirements` 매개변수는 `GameplayTag` 요구 사항을 충족하고 지금 활성화될 수 있는 `GameplayAbilitySpec`만 반환합니다. 예를 들어, 무기를 사용하는 기본 공격 `GameplayAbility`와 맨손으로 공격하는 기본 공격 `GameplayAbility` 두 가지가 있을 수 있으며, 무기가 장착되었는지 여부에 따라 올바른 `Ability`가 활성화되어 `GameplayTag` 요구 사항을 설정합니다. 자세한 내용은 이 함수에 대한 Epic의 주석을 참조하십시오.
```c++
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

찾고 있는 `FGameplayAbilitySpec`을 얻으면, `IsActive()`를 호출할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-instancing"></a>
#### 4.6.7 인스턴싱 정책
`GameplayAbility`의 `Instancing Policy`는 활성화될 때 `GameplayAbility`가 인스턴스화되는지 여부와 방법을 결정합니다.

| `인스턴싱 정책` | 설명 | 사용 시점 예시 |
|---|---|---|
| 액터별 인스턴스 | 각 `ASC`는 `GameplayAbility`의 인스턴스를 하나만 가지며 활성화 간에 재사용됩니다. | 이것이 아마도 가장 많이 사용하게 될 `Instancing Policy`일 것입니다. 어떤 `Ability`에도 사용할 수 있으며 활성화 간의 지속성을 제공합니다. 디자이너는 활성화 간에 필요한 변수를 수동으로 재설정해야 합니다. |
| 실행별 인스턴스 | `GameplayAbility`가 활성화될 때마다 `GameplayAbility`의 새 인스턴스가 생성됩니다. | 이러한 `GameplayAbility`의 장점은 활성화할 때마다 변수가 재설정된다는 것입니다. 활성화할 때마다 새로운 `GameplayAbility`를 생성하므로 `Instanced Per Actor`보다 성능이 떨어집니다. 샘플 프로젝트에서는 이를 사용하지 않습니다. |
| 비인스턴스 | `GameplayAbility`는 `ClassDefaultObject`에서 작동합니다. 인스턴스가 생성되지 않습니다. | 이 세 가지 중 가장 좋은 성능을 제공하지만, 할 수 있는 일에 가장 제한적입니다. `Non-Instanced` `GameplayAbility`는 상태를 저장할 수 없으며, 이는 동적 변수나 `AbilityTask` 델리게이트에 바인딩할 수 없음을 의미합니다. 미니언의 MOBA 또는 RTS 기본 공격과 같이 자주 사용되는 간단한 `Ability`에 가장 적합합니다. 샘플 프로젝트의 점프 `GameplayAbility`는 `Non-Instanced`입니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-net"></a>
#### 4.6.8 네트워크 실행 정책
`GameplayAbility`의 `Net Execution Policy`는 `GameplayAbility`를 누가 실행하고 어떤 순서로 실행할지 결정합니다.

| `네트워크 실행 정책` | 설명 |
|---|---|
| `로컬 전용` | `GameplayAbility`는 소유 클라이언트에서만 실행됩니다. 이는 로컬에서만 미적인 변경을 하는 `Ability`에 유용할 수 있습니다. 싱글 플레이어 게임은 `Server Only`를 사용해야 합니다. |
| `로컬 예측` | `Local Predicted` `GameplayAbility`는 소유 클라이언트에서 먼저 활성화된 다음 서버에서 활성화됩니다. 서버 버전은 클라이언트가 잘못 예측한 모든 것을 수정합니다. [예측](#concepts-p)을 참조하십시오. |
| `서버 전용` | `GameplayAbility`는 서버에서만 실행됩니다. 패시브 `GameplayAbility`는 일반적으로 `Server Only`입니다. 싱글 플레이어 게임은 이를 사용해야 합니다. |
| `서버 시작` | `Server Initiated` `GameplayAbility`는 서버에서 먼저 활성화된 다음 소유 클라이언트에서 활성화됩니다. 저는 개인적으로 거의 사용해본 적이 없습니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-tags"></a>
#### 4.6.9 Ability Tags
`GameplayAbility`는 내장 논리가 있는 `GameplayTagContainer`를 가집니다. 이 `GameplayTags`는 복제되지 않습니다.

| `GameplayTag Container` | 설명 |
|---|---|
| `Ability Tags` | `GameplayAbility`가 소유하는 `GameplayTags`입니다. 이들은 `GameplayAbility`를 설명하는 `GameplayTags`일 뿐입니다. |
| `Cancel Abilities with Tag` | 이 `GameplayAbility`가 활성화될 때 `Ability Tags`에 이 `GameplayTags`를 가진 다른 `GameplayAbility`는 취소됩니다. |
| `Block Abilities with Tag` | 이 `GameplayAbility`가 활성화되어 있는 동안 `Ability Tags`에 이 `GameplayTags`를 가진 다른 `GameplayAbility`는 활성화가 차단됩니다. |
| `Activation Owned Tags` | 이 `GameplayAbility`가 활성화되어 있는 동안 이 `GameplayTags`는 `GameplayAbility`의 소유자에게 부여됩니다. 이것들은 복제되지 않는다는 것을 기억하십시오. |
| `Activation Required Tags` | 이 `GameplayAbility`는 소유자가 이 `GameplayTags`를 **모두** 가지고 있을 때만 활성화될 수 있습니다. |
| `Activation Blocked Tags` | 소유자가 이 `GameplayTags` 중 **하나라도** 가지고 있다면 이 `GameplayAbility`는 활성화될 수 없습니다. |
| `Source Required Tags` | 이 `GameplayAbility`는 `Source`가 이 `GameplayTags`를 **모두** 가지고 있을 때만 활성화될 수 있습니다. `Source` `GameplayTags`는 `GameplayAbility`가 이벤트에 의해 트리거될 때만 설정됩니다. |
| `Source Blocked Tags` | `Source`가 이 `GameplayTags` 중 **하나라도** 가지고 있다면 이 `GameplayAbility`는 활성화될 수 없습니다. `Source` `GameplayTags`는 `GameplayAbility`가 이벤트에 의해 트리거될 때만 설정됩니다. |
| `Target Required Tags` | 이 `GameplayAbility`는 `Target`이 이 `GameplayTags`를 **모두** 가지고 있을 때만 활성화될 수 있습니다. `Target` `GameplayTags`는 `GameplayAbility`가 이벤트에 의해 트리거될 때만 설정됩니다. |
| `Target Blocked Tags` | `Target`이 이 `GameplayTags` 중 **하나라도** 가지고 있다면 이 `GameplayAbility`는 활성화될 수 없습니다. `Target` `GameplayTags`는 `GameplayAbility`가 이벤트에 의해 트리거될 때만 설정됩니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-spec"></a>
#### 4.6.10 Gameplay Ability Spec
`GameplayAbilitySpec`은 `GameplayAbility`가 부여된 후 `ASC`에 존재하며, 활성화 가능한 `GameplayAbility`( `GameplayAbility` 클래스, 레벨, 입력 바인딩, 그리고 `GameplayAbility` 클래스와 별도로 유지되어야 하는 런타임 상태)를 정의합니다.

서버에서 `GameplayAbility`가 부여되면, 서버는 `GameplayAbilitySpec`을 소유 클라이언트로 복제하여 클라이언트가 이를 활성화할 수 있도록 합니다.

`GameplayAbilitySpec`을 활성화하면 `Instancing Policy`에 따라 `GameplayAbility` 인스턴스가 생성됩니다 (또는 `Non-Instanced` `GameplayAbility`의 경우 생성되지 않습니다).

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-data"></a>
#### 4.6.11 Ability로 데이터 전달하기
`GameplayAbility`의 일반적인 패러다임은 `활성화->데이터 생성->적용->종료`입니다. 때로는 기존 데이터에 대해 작업을 수행해야 합니다. GAS는 외부 데이터를 `GameplayAbility`에 가져오는 몇 가지 옵션을 제공합니다:

| 방법 | 설명 |
|---|---|
| 이벤트로 `GameplayAbility` 활성화 | 데이터 페이로드를 포함하는 이벤트로 `GameplayAbility`를 활성화합니다. 이벤트의 페이로드는 로컬 예측 `GameplayAbility`를 위해 클라이언트에서 서버로 복제됩니다. 기존 변수에 맞지 않는 임의의 데이터에는 두 개의 `Optional Object` 또는 [`TargetData`](#concepts-targeting-data) 변수를 사용하십시오. 이 방법의 단점은 입력 바인딩으로 `Ability`를 활성화하는 것을 방지한다는 것입니다. 이벤트로 `GameplayAbility`를 활성화하려면 `GameplayAbility`에 `Triggers`가 설정되어 있어야 합니다. `GameplayTag`를 할당하고 `GameplayEvent`에 대한 옵션을 선택하십시오. 이벤트를 보내려면 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)` 함수를 사용하십시오. |
| `WaitGameplayEvent` `AbilityTask` 사용 | `WaitGameplayEvent` `AbilityTask`를 사용하여 `GameplayAbility`가 활성화된 후 페이로드 데이터를 가진 이벤트를 수신하도록 지시합니다. 이벤트 페이로드 및 전송 프로세스는 이벤트로 `GameplayAbility`를 활성화하는 것과 동일합니다. 이 방법의 단점은 이벤트가 `AbilityTask`에 의해 복제되지 않으며 `Local Only` 및 `Server Only` `GameplayAbility`에만 사용해야 한다는 것입니다. 이벤트 페이로드를 복제하는 자체 `AbilityTask`를 작성할 수도 있습니다. |
| `TargetData` 사용 | 사용자 지정 `TargetData` 구조체는 클라이언트와 서버 간에 임의의 데이터를 [GameplayAbility에서 전달](#concepts-ga-data)하는 좋은 방법입니다. |
| `OwnerActor` 또는 `AvatarActor`에 데이터 저장 | `OwnerActor`, `AvatarActor` 또는 참조할 수 있는 다른 개체에 저장된 복제된 변수를 사용합니다. 이 방법은 가장 유연하며 입력 바인딩으로 활성화된 `GameplayAbility`와 함께 작동합니다. 그러나 사용 시점에 복제를 통해 데이터가 동기화될 것이라는 보장은 없습니다. 미리 이를 확인해야 합니다. 즉, 복제된 변수를 설정한 다음 즉시 `GameplayAbility`를 활성화하면 잠재적인 패킷 손실로 인해 수신자에게 발생하는 순서가 보장되지 않습니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-commit"></a>
#### 4.6.12 Ability 비용 및 Cooldown
`GameplayAbility`는 선택적 비용 및 쿨다운 기능을 제공합니다. 비용은 `ASC`가 `GameplayAbility`를 활성화하기 위해 가져야 하는 미리 정의된 `Attribute` 양으로, `Instant` `GameplayEffect`([`Cost GE`](#concepts-ge-cost))로 구현됩니다. 쿨다운은 `GameplayAbility`의 재활성화를 만료될 때까지 방지하는 타이머로, `Duration` `GameplayEffect`([`Cooldown GE`](#concepts-ge-cooldown))로 구현됩니다.

`GameplayAbility`가 `UGameplayAbility::Activate()`를 호출하기 전에 `UGameplayAbility::CanActivateAbility()`를 호출합니다. 이 함수는 소유 `ASC`가 비용을 감당할 수 있는지(`UGameplayAbility::CheckCost()`) 확인하고, `GameplayAbility`가 쿨다운 중이 아닌지(`UGameplayAbility::CheckCooldown()`) 확인합니다.

`GameplayAbility`가 `Activate()`를 호출한 후에는 `UGameplayAbility::CommitAbility()`를 사용하여 언제든지 비용과 쿨다운을 커밋할 수 있습니다. `CommitAbility()`는 `UGameplayAbility::CommitCost()`와 `UGameplayAbility::CommitCooldown()`을 호출합니다. 디자이너는 동시에 커밋되지 않아야 하는 경우 `CommitCost()` 또는 `CommitCooldown()`을 별도로 호출하도록 선택할 수 있습니다. 비용과 쿨다운을 커밋하는 것은 `CheckCost()`와 `CheckCooldown()`을 한 번 더 호출하며, `GameplayAbility`가 이와 관련하여 실패할 수 있는 마지막 기회입니다. `GameplayAbility`가 활성화된 후 소유 `ASC`의 `Attribute`가 변경되어 커밋 시 비용을 충족하지 못할 수 있습니다. 비용과 쿨다운을 커밋하는 것은 커밋 시 [예측 키](#concepts-p-key)가 유효하면 [로컬에서 예측](#concepts-p)될 수 있습니다.

구현 세부 정보는 [`CostGE`](#concepts-ge-cost) 및 [`CooldownGE`](#concepts-ge-cooldown)를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-leveling"></a>
#### 4.6.13 Ability 레벨 올리기
Ability 레벨을 올리는 일반적인 두 가지 방법:

| 레벨업 방법 | 설명 |
|---|---|
| 새 레벨에서 부여 취소 및 다시 부여 | `ASC`에서 `GameplayAbility`를 부여 취소(제거)하고 서버에서 다음 레벨로 다시 부여합니다. 이 경우 활성화되어 있던 `GameplayAbility`는 종료됩니다. |
| `GameplayAbilitySpec`의 레벨 증가 | 서버에서 `GameplayAbilitySpec`을 찾아 레벨을 증가시키고, 소유 클라이언트에 복제되도록 더티 플래그를 지정합니다. 이 방법은 활성화되어 있던 `GameplayAbility`를 레벨업 시 종료시키지 않습니다. |

두 방법의 주요 차이점은 레벨업 시 활성화된 `GameplayAbility`를 취소할지 여부입니다. `GameplayAbility`에 따라 두 가지 방법을 모두 사용할 가능성이 높습니다. `UGameplayAbility` 서브클래스에 어떤 방법을 사용할지 지정하는 `bool` 변수를 추가하는 것을 권장합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-sets"></a>
#### 4.6.14 Ability Sets
`GameplayAbilitySets`는 `UDataAsset` 클래스의 편의 기능으로, `Ability`를 부여하기 위한 논리를 가진 캐릭터의 입력 바인딩과 시작 `GameplayAbility` 목록을 저장합니다. 서브클래스에는 추가 로직이나 속성이 포함될 수도 있습니다. Paragon은 영웅마다 모든 부여된 `GameplayAbility`를 포함하는 `GameplayAbilitySet`을 가지고 있었습니다.

저는 이 클래스가 지금까지 본 바로는 불필요하다고 생각합니다. 샘플 프로젝트는 `GDCharacterBase` 및 해당 서브클래스 내에서 `GameplayAbilitySet`의 모든 기능을 처리합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-batching"></a>
#### 4.6.15 Ability Batching
활성화하고, 선택적으로 `TargetData`를 서버로 보내고, 한 프레임 내에 모두 종료되는 [`GameplayAbilities`](#concepts-ga)는 [두세 개의 RPC를 하나의 RPC로 압축하도록 일괄 처리](#concepts-ga-batching)될 수 있습니다. 이러한 유형의 `Ability`는 일반적으로 히트스캔 총에 사용됩니다. 히트스캔 총은 활성화하고, 라인 트레이스를 수행하고, [`TargetData`](#concepts-targeting-data)를 서버로 보내고, `Ability`를 모두 한 프레임 내에 하나의 원자적인 그룹으로 종료합니다. [GASShooter](https://github.com/tranek/GASShooter) 샘플 프로젝트는 히트스캔 총에 이 기술을 시연합니다.

반자동 총은 가장 좋은 시나리오이며, `CallServerTryActivateAbility()`, `ServerSetReplicatedTargetData()` (총알 히트 결과), `ServerEndAbility()`를 세 개의 RPC 대신 하나의 RPC로 일괄 처리합니다.

완전 자동/버스트 총은 `CallServerTryActivateAbility()`와 첫 번째 총알에 대한 `ServerSetReplicatedTargetData()`를 두 개의 RPC 대신 하나의 RPC로 일괄 처리합니다. 각 후속 총알은 자체 `ServerSetReplicatedTargetData()` RPC입니다. 마지막으로, 총이 발사를 멈추면 `ServerEndAbility()`가 별도의 RPC로 전송됩니다. 이는 최악의 시나리오로, 첫 번째 총알에서 두 개의 RPC 대신 하나만 절약됩니다. 이 시나리오는 [`Gameplay Event`](#concepts-ga-data)를 통해 `Ability`를 활성화하는 것으로도 구현될 수 있었으며, 이는 `EventPayload`와 함께 총알의 `TargetData`를 클라이언트에서 서버로 보냅니다. 후자 접근 방식의 단점은 `TargetData`가 `Ability` 외부에서 생성되어야 하는 반면, 일괄 처리 접근 방식은 `Ability` 내부에서 `TargetData`를 생성한다는 것입니다.

`Ability Batching`은 [`ASC`](#concepts-asc)에서 기본적으로 비활성화되어 있습니다. `Ability Batching`을 활성화하려면 `ShouldDoServerAbilityRPCBatch()`를 오버라이드하여 true를 반환하도록 합니다:

```c++
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }
```

이제 `Ability Batching`이 활성화되었으므로, 일괄 처리할 `Ability`를 활성화하기 전에 `FScopedServerAbilityRPCBatcher` 구조체를 미리 생성해야 합니다. 이 특수 구조체는 범위 내에서 뒤따르는 모든 `Ability`를 일괄 처리하려고 시도합니다. `FScopedServerAbilityRPCBatcher`가 범위를 벗어나면 활성화된 `Ability`는 일괄 처리하려고 시도하지 않습니다. `FScopedServerAbilityRPCBatcher`는 일괄 처리될 수 있는 각 함수의 특수 코드를 통해 RPC 전송 호출을 가로채고 대신 메시지를 일괄 구조체로 패킹하여 작동합니다. `FScopedServerAbilityRPCBatcher`가 범위를 벗어나면, `UAbilitySystemComponent::EndServerAbilityRPCBatch()`에서 이 일괄 구조체를 서버로 자동으로 RPC합니다. 서버는 `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)`에서 일괄 RPC를 수신합니다. `BatchInfo` 매개변수는 `Ability`가 종료되어야 하는지, 활성화 시점에 입력이 눌러졌는지에 대한 플래그와 `TargetData`가 포함되었는지 여부를 포함합니다. 이는 일괄 처리가 제대로 작동하는지 확인하기 위해 중단점을 설정하기 좋은 함수입니다. 또는 cvar `AbilitySystem.ServerRPCBatching.Log 1`을 사용하여 특수 `Ability` 일괄 처리 로깅을 활성화할 수 있습니다.

이 메커니즘은 C++로만 수행할 수 있으며 `FGameplayAbilitySpecHandle`로만 `Ability`를 활성화할 수 있습니다.

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately)
{
	bool AbilityActivated = false;
	if (InAbilityHandle.IsValid())
	{
		FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle);
		AbilityActivated = TryActivateAbility(InAbilityHandle, true);

		if (EndAbilityImmediately)
		{
			FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
			if (AbilitySpec)
			{
				UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
				GSAbility->ExternalEndAbility();
			}
		}

		return AbilityActivated;
	}

	return AbilityActivated;
}
```

GASShooter는 반자동 및 완전 자동 총에 동일한 일괄 처리된 `GameplayAbility`를 재사용하며, 이들은 `EndAbility()`를 직접 호출하지 않습니다 (현재 발사 모드에 따라 플레이어 입력과 일괄 처리된 `Ability` 호출을 관리하는 로컬 전용 `Ability`에 의해 `Ability` 외부에서 처리됩니다). 모든 RPC는 `FScopedServerAbilityRPCBatcher`의 범위 내에서 발생해야 하므로, `EndAbilityImmediately` 매개변수를 제공하여 제어/관리 로컬 전용 `Ability`가 이 `Ability`가 `EndAbility()` 호출을 일괄 처리해야 하는지 (반자동) 또는 `EndAbility()` 호출을 일괄 처리하지 않고 (완전 자동) `EndAbility()` 호출이 나중에 자체 RPC에서 발생할지 지정할 수 있습니다.

GASShooter는 일괄 처리된 `Ability`를 트리거하는 데 사용되는 로컬 전용 `Ability`가 일괄 처리 `Ability`를 활성화할 수 있도록 Blueprint 노드를 노출합니다.

![Activate Batched Ability](https://github.com/tranek/GASDocumentation/raw/master/Images/batchabilityactivate.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-ga-netsecuritypolicy"></a>
#### 4.6.16 네트워크 보안 정책
`GameplayAbility`의 `NetSecurityPolicy`는 `Ability`가 네트워크에서 어디에서 실행되어야 하는지 결정합니다. 이는 클라이언트가 제한된 `Ability`를 실행하려는 시도로부터 보호를 제공합니다.

| `NetSecurityPolicy` | 설명 |
|---|---|
| `ClientOrServer` | 보안 요구 사항 없음. 클라이언트 또는 서버가 이 `Ability`의 실행 및 종료를 자유롭게 트리거할 수 있습니다. |
| `ServerOnlyExecution` | 클라이언트가 이 `Ability`의 실행을 요청하는 경우 서버는 이를 무시합니다. 클라이언트는 여전히 서버에 이 `Ability`의 취소 또는 종료를 요청할 수 있습니다. |
| `ServerOnlyTermination` | 클라이언트가 이 `Ability`의 취소 또는 종료를 요청하는 경우 서버는 이를 무시합니다. 클라이언트는 여전히 `Ability` 실행을 요청할 수 있습니다. |
| `ServerOnly` | 서버는 이 `Ability`의 실행 및 종료를 모두 제어합니다. 클라이언트가 어떤 요청을 하더라도 무시됩니다. |

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-at"></a>
### 4.7 어빌리티 태스크 (Ability Tasks)

<a name="concepts-at-definition"></a>
### 4.7.1 어빌리티 태스크 (Ability Task) 정의
`GameplayAbilities`는 한 프레임에서만 실행됩니다. 이것만으로는 많은 유연성을 제공하지 못합니다. 시간이 지남에 따라 발생하는 행동을 수행하거나 나중에 특정 시점에 발생한 델리게이트에 응답하려면 `AbilityTasks`라고 하는 지연 동작을 사용합니다.

GAS는 기본적으로 많은 `AbilityTasks`를 제공합니다:
* `RootMotionSource`를 사용하여 캐릭터를 이동시키는 태스크
* 애니메이션 몽타주를 재생하는 태스크
* `Attribute` 변경에 응답하는 태스크
* `GameplayEffect` 변경에 응답하는 태스크
* 플레이어 입력에 응답하는 태스크
* 기타 등등

`UAbilityTask` 생성자는 게임 전반에 걸쳐 동시에 실행될 수 있는 `AbilityTasks`의 최대치를 1000개로 하드코딩하여 적용합니다. RTS 게임처럼 월드에 수백 명의 캐릭터가 동시에 존재할 수 있는 게임을 위해 `GameplayAbilities`를 설계할 때 이 점을 염두에 두십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-at-definition"></a>
### 4.7.2 커스텀 어빌리티 태스크 (Custom Ability Tasks)
종종 여러분은 직접 커스텀 `AbilityTasks`를 생성할 것입니다 (C++로). 샘플 프로젝트에는 두 가지 커스텀 `AbilityTasks`가 포함되어 있습니다:
1.  `PlayMontageAndWaitForEvent`는 기본 `PlayMontageAndWait`와 `WaitGameplayEvent` `AbilityTasks`를 조합한 것입니다. 이를 통해 애니메이션 몽타주가 `AnimNotifies`로부터 게임플레이 이벤트를 시작한 `GameplayAbility`로 다시 보낼 수 있습니다. 이를 사용하여 애니메이션 몽타주 중 특정 시간에 동작을 트리거하십시오.
2.  `WaitReceiveDamage`는 `OwnerActor`가 피해를 입는 것을 감지합니다. 패시브 방어력 스택 `GameplayAbility`는 영웅이 피해를 입을 때 방어력 스택을 하나 제거합니다.

`AbilityTasks`는 다음으로 구성됩니다:
* `AbilityTask`의 새 인스턴스를 생성하는 정적 함수
* `AbilityTask`가 목적을 완료할 때 브로드캐스트되는 델리게이트
* 주요 작업을 시작하고 외부 델리게이트에 바인딩하는 `Activate()` 함수
* 바인딩된 외부 델리게이트를 포함한 정리를 위한 `OnDestroy()` 함수
* 바인딩된 모든 외부 델리게이트를 위한 콜백 함수
* 멤버 변수 및 모든 내부 헬퍼 함수

**참고:** `AbilityTasks`는 하나의 출력 델리게이트 타입만 선언할 수 있습니다. 사용 여부와 관계없이 모든 출력 델리게이트는 이 타입이어야 합니다. 사용하지 않는 델리게이트 매개변수에는 기본값을 전달하십시오.

`AbilityTasks`는 소유하는 `GameplayAbility`를 실행하는 클라이언트 또는 서버에서만 실행됩니다. 그러나 `AbilityTask` 생성자에서 `bSimulatedTask = true;`를 설정하고, `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`를 오버라이드하고, 멤버 변수를 복제하도록 설정하면 `AbilityTasks`를 시뮬레이션된 클라이언트에서 실행하도록 설정할 수 있습니다. 이것은 모든 이동 변경 사항을 복제하는 대신 전체 이동 `AbilityTask`를 시뮬레이션하려는 이동 `AbilityTasks`와 같은 드문 상황에서만 유용합니다. 모든 `RootMotionSource` `AbilityTasks`는 이 작업을 수행합니다. 예시로 `AbilityTask_MoveToLocation.h/.cpp`를 참조하십시오.

`AbilityTask` 생성자에서 `bTickingTask = true;`를 설정하고 `virtual void TickTask(float DeltaTime);`를 오버라이드하면 `AbilityTasks`가 `Tick`을 수행할 수 있습니다. 이는 프레임 간에 값을 부드럽게 보간해야 할 때 유용합니다. 예시로 `AbilityTask_MoveToLocation.h/.cpp`를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-at-using"></a>
### 4.7.3 어빌리티 태스크 (Ability Task) 사용
C++에서 `AbilityTask`를 생성하고 활성화하는 방법 (From `GDGA_FireGun.cpp`):
```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

블루프린트에서는 `AbilityTask`를 위해 생성한 블루프린트 노드를 사용하기만 하면 됩니다. `ReadyForActivation()`을 호출할 필요가 없습니다. 이는 `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp`에 의해 자동으로 호출됩니다. `K2Node_LatentGameplayTaskCall`은 또한 `AbilityTask` 클래스에 `BeginSpawningActor()` 및 `FinishSpawningActor()`가 존재하는 경우 자동으로 호출합니다 (`AbilityTask_WaitTargetData` 참조). 다시 말해, `K2Node_LatentGameplayTaskCall`은 블루프린트에서만 자동 마법을 수행합니다. C++에서는 `ReadyForActivation()`, `BeginSpawningActor()`, `FinishSpawningActor()`를 수동으로 호출해야 합니다.

![Blueprint WaitTargetData AbilityTask](https://github.com/tranek/GASDocumentation/raw/master/Images/abilitytask.png)

`AbilityTask`를 수동으로 취소하려면 블루프린트 (`Async Task Proxy`라고 함) 또는 C++에서 `AbilityTask` 객체에 `EndTask()`를 호출하기만 하면 됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-at-rms"></a>
### 4.7.4 루트 모션 소스 (Root Motion Source) 어빌리티 태스크 (Ability Tasks)
GAS는 `CharacterMovementComponent`에 연결된 `Root Motion Sources`를 사용하여 넉백, 복잡한 점프, 끌어당기기, 돌진 등과 같은 동작을 위해 시간이 지남에 따라 `Characters`를 이동시키는 `AbilityTasks`를 제공합니다.

**참고:** `RootMotionSource` `AbilityTasks` 예측은 엔진 버전 4.19 및 4.25 이상에서 작동합니다. 예측은 엔진 버전 4.20-4.24에서 버그가 있지만, `AbilityTasks`는 약간의 네트워크 보정을 통해 멀티플레이어에서 기능을 수행하며 싱글 플레이어에서는 완벽하게 작동합니다. 4.25의 [예측 수정](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c)을 커스텀 4.20-4.24 엔진으로 체리 피킹할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc"></a>
### 4.8 게임플레이 큐 (Gameplay Cues)

<a name="concepts-gc-definition"></a>
#### 4.8.1 게임플레이 큐 (Gameplay Cue) 정의
`GameplayCues` (`GC`)는 사운드 효과, 파티클 효과, 카메라 흔들림 등 게임플레이와 관련 없는 것들을 실행합니다. `GameplayCues`는 일반적으로 복제되며 (명시적으로 로컬에서 `Executed`, `Added` 또는 `Removed`되지 않는 한) 예측됩니다.

우리는 **`GameplayCue.`라는 필수 부모 이름을 가진** 해당 `GameplayTag`와 이벤트 타입 (`Execute`, `Add`, 또는 `Remove`)을 `ASC`를 통해 `GameplayCueManager`로 전송하여 `GameplayCues`를 트리거합니다. `IGameplayCueInterface`를 구현하는 `GameplayCueNotify` 객체 및 기타 `Actors`는 `GameplayCue`의 `GameplayTag` (`GameplayCueTag`)를 기반으로 이러한 이벤트에 구독할 수 있습니다.

**참고:** 다시 말하지만, `GameplayCue` `GameplayTags`는 부모 `GameplayTag`인 `GameplayCue`로 시작해야 합니다. 예를 들어, 유효한 `GameplayCue` `GameplayTag`는 `GameplayCue.A.B.C`일 수 있습니다.

`GameplayCueNotifies`에는 `Static`과 `Actor` 두 가지 클래스가 있습니다. 이들은 다른 이벤트에 응답하며, 다른 타입의 `GameplayEffects`가 이들을 트리거할 수 있습니다. 해당 이벤트를 논리로 오버라이드하십시오.

| `GameplayCue` 클래스 | 이벤트 | `GameplayEffect` 타입 | 설명 |
|---|---|---|---|
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute` | `Instant` 또는 `Periodic` | Static `GameplayCueNotifies`는 `ClassDefaultObject` (즉, 인스턴스가 없음)에서 작동하며, 히트 임팩트와 같은 일회성 효과에 적합합니다. |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html) | `Add` 또는 `Remove` | `Duration` 또는 `Infinite` | Actor `GameplayCueNotifies`는 `Added`될 때 새로운 인스턴스를 생성합니다. 이들은 인스턴스화되므로 `Removed`될 때까지 시간이 지남에 따라 동작을 수행할 수 있습니다. 이들은 백업 `Duration` 또는 `Infinite` `GameplayEffect`가 제거되거나 수동으로 제거를 호출할 때 제거될 루핑 사운드 및 파티클 효과에 적합합니다. 또한 동일한 효과의 여러 애플리케이션이 사운드나 파티클을 한 번만 시작하도록 동시에 `Added`될 수 있는 수를 관리하는 옵션도 제공합니다. |

`GameplayCueNotifies`는 기술적으로 어떤 이벤트에도 응답할 수 있지만, 일반적으로 이렇게 사용합니다.

**참고:** `GameplayCueNotify_Actor`를 사용할 때는 `Auto Destroy on Remove`를 확인하십시오. 그렇지 않으면 해당 `GameplayCueTag`에 대한 후속 `Add` 호출이 작동하지 않습니다.

`Full`이 아닌 `ASC` [복제 모드](#concepts-asc-rm)를 사용할 경우, `Add` 및 `Remove` `GC` 이벤트는 서버 플레이어 (리스닝 서버)에서 두 번 발생합니다. 한 번은 `GE`를 적용하기 위해, 다시 한 번은 "Minimal" `NetMultiCast`를 통해 클라이언트에게 전달됩니다. 그러나 `WhileActive` 이벤트는 여전히 한 번만 발생합니다. 모든 이벤트는 클라이언트에서만 한 번 발생합니다.

샘플 프로젝트에는 기절 및 질주 효과를 위한 `GameplayCueNotify_Actor`가 포함되어 있습니다. 또한 FireGun의 발사체 충돌을 위한 `GameplayCueNotify_Static`도 있습니다. 이 `GCs`는 `GE`를 통해 복제하는 대신 [로컬에서 트리거](#concepts-gc-local)하여 더 최적화될 수 있습니다. 저는 샘플 프로젝트에서 초보자적인 사용법을 보여주는 방법을 선택했습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-trigger"></a>
#### 4.8.2 게임플레이 큐 (Gameplay Cue) 트리거링

`GameplayEffect` 내부에서 성공적으로 적용될 때 (태그 또는 면역에 의해 차단되지 않음), 트리거되어야 하는 모든 `GameplayCues`의 `GameplayTags`를 채웁니다.

![GameplayCue Triggered from a GameplayEffect](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility`는 `GameplayCues`를 `Execute`, `Add`, 또는 `Remove`하기 위한 블루프린트 노드를 제공합니다.

![GameplayCue Triggered from a GameplayAbility](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

C++에서는 `ASC`에서 직접 함수를 호출할 수 있습니다 (또는 `ASC` 서브클래스에서 블루프린트로 노출):

```c++
/** GameplayCues는 단독으로도 발생할 수 있습니다. 이들은 히트 결과 등을 전달하기 위한 선택적 이펙트 컨텍스트를 가집니다. */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 지속적인 게임플레이 큐를 추가합니다. */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 지속적인 게임플레이 큐를 제거합니다. */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** GameplayEffect의 일부가 아닌, 자체적으로 추가된 모든 GameplayCue를 제거합니다. */
void RemoveAllGameplayCues();
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-local"></a>
#### 4.8.3 로컬 게임플레이 큐 (Local Gameplay Cues)
`GameplayAbilities` 및 `ASC`에서 `GameplayCues`를 발동하는 노출된 함수는 기본적으로 복제됩니다. 각 `GameplayCue` 이벤트는 멀티캐스트 RPC입니다. 이로 인해 많은 RPC가 발생할 수 있습니다. GAS는 또한 네트워크 업데이트당 동일한 `GameplayCue` RPC를 최대 두 개로 제한합니다. 우리는 로컬 `GameplayCues`를 사용하여 이를 피할 수 있습니다. 로컬 `GameplayCues`는 개별 클라이언트에서만 `Execute`, `Add`, 또는 `Remove`를 수행합니다.

로컬 `GameplayCues`를 사용할 수 있는 시나리오:
* 발사체 충돌
* 근접 충돌
* 애니메이션 몽타주에서 발동되는 `GameplayCues`

`ASC` 서브클래스에 추가해야 하는 로컬 `GameplayCue` 함수:

```c++
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);
```

```c++
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);
}

void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);
}

void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);
}
```

`GameplayCue`가 로컬로 `Added`되었다면, 로컬로 `Removed`되어야 합니다. 복제를 통해 `Added`되었다면, 복제를 통해 `Removed`되어야 합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-parameters"></a>
#### 4.8.4 게임플레이 큐 파라미터 (Gameplay Cue Parameters)
`GameplayCues`는 매개변수로 `GameplayCue`에 대한 추가 정보를 포함하는 `FGameplayCueParameters` 구조체를 받습니다. `GameplayAbility` 또는 `ASC`의 함수에서 `GameplayCue`를 수동으로 트리거하는 경우, `GameplayCue`에 전달되는 `GameplayCueParameters` 구조체를 수동으로 채워야 합니다. `GameplayCue`가 `GameplayEffect`에 의해 트리거되는 경우, `GameplayCueParameters` 구조체의 다음 변수들은 자동으로 채워집니다:

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude (`GameplayEffect`에 `GameplayCue` 태그 컨테이너 위의 드롭다운에서 선택된 `Attribute`에 대한 Magnitude가 있고, 해당 `Attribute`에 영향을 미치는 해당 `Modifier`가 있는 경우)

`GameplayCueParameters` 구조체의 `SourceObject` 변수는 `GameplayCue`를 수동으로 트리거할 때 임의의 데이터를 `GameplayCue`에 전달하기에 잠재적으로 좋은 장소입니다.

**참고:** `Instigator`와 같은 매개변수 구조체의 일부 변수는 이미 `EffectContext`에 존재할 수 있습니다. `EffectContext`는 또한 월드에 `GameplayCue`를 생성할 위치에 대한 `FHitResult`를 포함할 수 있습니다. `EffectContext`를 서브클래싱하는 것은 `GameplayCues`, 특히 `GameplayEffect`에 의해 트리거되는 `GameplayCues`에 더 많은 데이터를 전달하는 좋은 방법일 수 있습니다.

`GameplayCueParameters` 구조체를 채우는 [`UAbilitySystemGlobals`](#concepts-asg)의 3가지 함수에 대한 자세한 내용은 해당 섹션을 참조하십시오. 이 함수들은 가상이므로 더 많은 정보를 자동으로 채우도록 오버라이드할 수 있습니다.

```c++
/** GameplayCue 매개변수를 초기화합니다. */
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-manager"></a>
#### 4.8.5 게임플레이 큐 매니저 (Gameplay Cue Manager)
기본적으로 `GameplayCueManager`는 게임 디렉토리 전체를 스캔하여 `GameplayCueNotifies`를 찾고 플레이 시 메모리에 로드합니다. `DefaultGame.ini`에서 `GameplayCueManager`가 스캔하는 경로를 설정하여 변경할 수 있습니다.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

우리는 `GameplayCueManager`가 모든 `GameplayCueNotifies`를 스캔하고 찾기를 원하지만, 플레이 시 모든 것을 비동기적으로 로드하는 것을 원하지 않습니다. 이렇게 하면 모든 `GameplayCueNotify`와 그들이 참조하는 모든 사운드 및 파티클이 레벨에서 사용되든 안 되든 메모리에 로드될 것입니다. Paragon과 같은 대규모 게임에서는 이것이 수백 메가바이트의 불필요한 에셋을 메모리에 로드하여 시작 시 랙이나 게임 멈춤을 유발할 수 있습니다.

시작 시 모든 `GameplayCue`를 비동기적으로 로드하는 대안은 게임 내에서 `GameplayCues`가 트리거될 때만 비동기적으로 로드하는 것입니다. 이는 불필요한 메모리 사용과 잠재적인 게임 하드 멈춤을 완화하며, 플레이 중 특정 `GameplayCue`가 처음 트리거될 때 잠재적인 지연이 발생할 수 있습니다. SSD의 경우 이 잠재적인 지연은 존재하지 않습니다. 저는 HDD에서는 테스트하지 않았습니다. UE 에디터에서 이 옵션을 사용하는 경우, 에디터가 파티클 시스템을 컴파일해야 하는 경우 `GameplayCues`의 첫 로드 시 약간의 끊김이나 멈춤이 있을 수 있습니다. 빌드에서는 파티클 시스템이 이미 컴파일되어 있으므로 이 문제는 발생하지 않습니다.

먼저 `UGameplayCueManager`를 서브클래싱하고 `AbilitySystemGlobals` 클래스에게 `DefaultGame.ini`에서 우리의 `UGameplayCueManager` 서브클래스를 사용하도록 지시해야 합니다.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```

우리의 `UGameplayCueManager` 서브클래스에서 `ShouldAsyncLoadRuntimeObjectLibraries()`를 오버라이드합니다.

```c++
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-prevention"></a>
#### 4.8.6 게임플레이 큐 (Gameplay Cue) 발동 방지
때로는 `GameplayCues`가 발동되지 않기를 원할 때가 있습니다. 예를 들어, 공격을 막는 경우 피해 `GameplayEffect`에 부착된 히트 임팩트를 재생하지 않거나 대신 커스텀 임팩트를 재생하고 싶을 수 있습니다. 이는 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 내부에서 `OutExecutionOutput.MarkGameplayCuesHandledManually()`를 호출한 다음, `Target` 또는 `Source's` `ASC`에 수동으로 `GameplayCue` 이벤트를 전송함으로써 수행할 수 있습니다.

특정 `ASC`에서 `GameplayCues`가 전혀 발동되지 않기를 원하는 경우, `AbilitySystemComponent->bSuppressGameplayCues = true;`를 설정할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-batching"></a>
#### 4.8.7 게임플레이 큐 배치 (Gameplay Cue Batching)
트리거되는 각 `GameplayCue`는 비신뢰성 NetMulticast RPC입니다. 여러 `GCs`를 동시에 발동하는 상황에서, 이들을 하나의 RPC로 통합하거나 더 적은 데이터를 전송하여 대역폭을 절약하는 몇 가지 최적화 방법이 있습니다.

<a name="concepts-gc-batching-manualrpc"></a>
##### 4.8.7.1 수동 RPC (Manual RPC)
산탄총이 8발의 펠릿을 발사한다고 가정해 봅시다. 이것은 8번의 트레이스와 충돌 `GameplayCues`입니다. [GASShooter](https://github.com/tranek/GASShooter)는 모든 트레이스 정보를 [`EffectContext`](#concepts-ge-ec)에 [`TargetData`](#concepts-targeting-data)로 저장하여 이들을 하나의 RPC로 결합하는 느슨한 접근 방식을 취합니다. 이 방법은 RPC 수를 8개에서 1개로 줄이지만, 여전히 하나의 RPC를 통해 많은 데이터 (~500바이트)를 네트워크로 전송합니다. 더 최적화된 접근 방식은 히트 위치를 효율적으로 인코딩하는 커스텀 구조체를 가진 RPC를 보내거나, 수신 측에서 충돌 위치를 다시 생성/근사화하기 위해 무작위 시드 번호를 부여하는 것입니다. 클라이언트는 이 커스텀 구조체를 언팩하여 [로컬로 실행되는 `GameplayCues`](#concepts-gc-local)로 다시 변환합니다.

작동 방식:
1.  `FScopedGameplayCueSendContext`를 선언합니다. 이는 스코프를 벗어날 때까지 `UGameplayCueManager::FlushPendingCues()`를 억제합니다. 즉, 모든 `GameplayCues`는 `FScopedGameplayCueSendContext`가 스코프를 벗어날 때까지 대기열에 쌓이게 됩니다.
2.  `UGameplayCueManager::FlushPendingCues()`를 오버라이드하여 특정 커스텀 `GameplayTag`를 기반으로 함께 배치될 수 있는 `GameplayCues`를 커스텀 구조체로 병합하고 클라이언트에 RPC로 보냅니다.
3.  클라이언트는 커스텀 구조체를 수신하고 이를 로컬로 실행되는 `GameplayCues`로 언팩합니다.

이 방법은 `GameplayCueParameters`가 제공하는 것과 맞지 않는 `GameplayCues`에 대한 특정 매개변수가 필요하고, 피해 숫자, 치명타 표시기, 깨진 방패 표시기, 치명타 적중 표시기 등과 같이 `EffectContext`에 추가하고 싶지 않을 때도 사용할 수 있습니다.

https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

<a name="concepts-gc-batching-gcsonge"></a>
##### 4.8.7.2 하나의 GE에 여러 GC (Multiple GCs on one GE)
`GameplayEffect`의 모든 `GameplayCues`는 이미 하나의 RPC로 전송됩니다. 기본적으로 `UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()`는 `ASC`의 `Replication Mode`와 관계없이 전체 `GameplayEffectSpec` (그러나 `FGameplayEffectSpecForRPC`로 변환됨)을 비신뢰성 NetMulticast로 전송합니다. 이것은 `GameplayEffectSpec`에 포함된 내용에 따라 잠재적으로 많은 대역폭을 소비할 수 있습니다. `cvar AbilitySystem.AlwaysConvertGESpecToGCParams 1`을 설정하여 이를 최적화할 수 있습니다. 이렇게 하면 `GameplayEffectSpecs`를 `FGameplayCueParameter` 구조체로 변환하고 전체 `FGameplayEffectSpecForRPC` 대신 해당 구조체를 RPC로 보냅니다. 이것은 잠재적으로 대역폭을 절약하지만, `GESpec`가 `GameplayCueParameters`로 변환되는 방식과 `GCs`가 알아야 할 정보에 따라 정보가 줄어들 수도 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-events"></a>
#### 4.8.8 게임플레이 큐 이벤트 (Gameplay Cue Events)
`GameplayCues`는 특정 `EGameplayCueEvents`에 응답합니다:

| `EGameplayCueEvent` | 설명 |
|---|---|
| `OnActive` | `GameplayCue`가 활성화될 때 (추가될 때) 호출됩니다. |
| `WhileActive` | `GameplayCue`가 활성화되어 있을 때 호출됩니다. 심지어 방금 적용되지 않았더라도 (진행 중인 참여 등). 이것은 `Tick`이 아닙니다! `GameplayCueNotify_Actor`가 추가되거나 관련성이 생길 때 `OnActive`와 마찬가지로 한 번 호출됩니다. `Tick()`이 필요하면 `GameplayCueNotify_Actor`의 `Tick()`을 사용하십시오. 결국 `AActor`입니다. |
| `Removed` | `GameplayCue`가 제거될 때 호출됩니다. 이 이벤트에 응답하는 블루프린트 `GameplayCue` 함수는 `OnRemove`입니다. |
| `Executed` | `GameplayCue`가 실행될 때 호출됩니다: 즉시 효과 또는 주기적인 `Tick()`입니다. 이 이벤트에 응답하는 블루프린트 `GameplayCue` 함수는 `OnExecute`입니다. |

`GameplayCue` 시작 시 발생하지만 늦게 참여하는 플레이어가 놓쳐도 괜찮은 것들은 `GameplayCue`의 `OnActive`에 넣으십시오. 늦게 참여하는 플레이어가 보기를 원하는 `GameplayCue`의 지속적인 효과는 `WhileActive`에 넣으십시오. 예를 들어, MOBA의 타워 구조물 폭발에 대한 `GameplayCue`가 있다면, 초기 폭발 파티클 시스템과 폭발 사운드는 `OnActive`에 넣고, 잔존하는 지속적인 화재 파티클 또는 사운드는 `WhileActive`에 넣을 수 있습니다. 이 시나리오에서는 늦게 참여하는 플레이어가 `OnActive`의 초기 폭발을 다시 재생하는 것은 의미가 없지만, 폭발 후 지면에 남아있는 지속적인 루핑 화재 효과를 `WhileActive`에서 보기를 원할 것입니다. `OnRemove`는 `OnActive`와 `WhileActive`에서 추가된 모든 것을 정리해야 합니다. `WhileActive`는 `Actor`가 `GameplayCueNotify_Actor`의 관련성 범위에 들어갈 때마다 호출됩니다. `OnRemove`는 `Actor`가 `GameplayCueNotify_Actor`의 관련성 범위를 벗어날 때마다 호출됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-gc-reliability"></a>
#### 4.8.9 게임플레이 큐 신뢰성 (Gameplay Cue Reliability)

`GameplayCues`는 일반적으로 신뢰할 수 없는 것으로 간주되어 게임플레이에 직접적인 영향을 미치는 어떤 것에도 부적합합니다.

**실행된 `GameplayCues`:** 이 `GameplayCues`는 비신뢰성 멀티캐스트를 통해 적용되며 항상 비신뢰성입니다.

**`GameplayEffects`에서 적용된 `GameplayCues`:**
* Autonomous proxy는 `OnActive`, `WhileActive`, `OnRemove`를 신뢰성 있게 수신합니다.
`FActiveGameplayEffectsContainer::NetDeltaSerialize()`는 `UAbilitySystemComponent::HandleDeferredGameplayCues()`를 호출하여 `OnActive`와 `WhileActive`를 호출합니다. `FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()`는 `OnRemoved`를 호출합니다.
* Simulated proxies는 `WhileActive`와 `OnRemove`를 신뢰성 있게 수신합니다.
`UAbilitySystemComponent::MinimalReplicationGameplayCues`의 복제는 `WhileActive`와 `OnRemove`를 호출합니다. `OnActive` 이벤트는 비신뢰성 멀티캐스트에 의해 호출됩니다.

**`GameplayEffect` 없이 적용된 `GameplayCues`:**
* Autonomous proxy는 `OnRemove`를 신뢰성 있게 수신합니다.
`OnActive`와 `WhileActive` 이벤트는 비신뢰성 멀티캐스트에 의해 호출됩니다.
* Simulated proxies는 `WhileActive`와 `OnRemove`를 신뢰성 있게 수신합니다.
`UAbilitySystemComponent::MinimalReplicationGameplayCues`의 복제는 `WhileActive`와 `OnRemove`를 호출합니다. `OnActive` 이벤트는 비신뢰성 멀티캐스트에 의해 호출됩니다.

`GameplayCue`에서 '신뢰성'이 필요한 경우, `GameplayEffect`에서 적용하고 `WhileActive`를 사용하여 FX를 추가하고 `OnRemove`를 사용하여 FX를 제거하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-asg"></a>
### 4.9 어빌리티 시스템 글로벌 (Ability System Globals)
[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) 클래스는 GAS에 대한 전역 정보를 보유합니다. 대부분의 변수는 `DefaultGame.ini`에서 설정할 수 있습니다. 일반적으로 이 클래스와 직접 상호작용할 필요는 없지만, 그 존재를 알아두는 것이 좋습니다. [`GameplayCueManager`](#concepts-gc-manager) 또는 [`GameplayEffectContext`](#concepts-ge-context)와 같은 것을 서브클래싱해야 하는 경우, `AbilitySystemGlobals`를 통해 이 작업을 수행해야 합니다.

`AbilitySystemGlobals`를 서브클래싱하려면 `DefaultGame.ini`에 클래스 이름을 설정하십시오:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

<a name="concepts-asg-initglobaldata"></a>
#### 4.9.1 InitGlobalData()
UE 4.24와 5.2 사이에서는 [`TargetData`](#concepts-targeting-data)를 사용하기 위해 `UAbilitySystemGlobals::Get().InitGlobalData()`를 호출해야 합니다. 그렇지 않으면 `ScriptStructCache`와 관련된 오류가 발생하고 클라이언트가 서버에서 연결 해제될 것입니다. 이 함수는 프로젝트에서 한 번만 호출하면 됩니다. Fortnite는 `UAssetManager::StartInitialLoading()`에서 호출했고 Paragon은 `UEngine::Init()`에서 호출했습니다. 샘플 프로젝트에서 볼 수 있듯이 `UAssetManager::StartInitialLoading()`에 넣는 것이 좋은 위치라고 생각합니다. `TargetData`와 관련된 문제를 피하기 위해 프로젝트에 복사해야 하는 상용구 코드라고 생각합니다. 5.3부터는 자동으로 호출됩니다.

`AbilitySystemGlobals`의 `GlobalAttributeSetDefaultsTableNames`를 사용할 때 충돌이 발생하는 경우, Fortnite처럼 `AssetManager` 또는 `GameInstance`에서 `UAbilitySystemGlobals::Get().InitGlobalData()`를 나중에 호출해야 할 수도 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p"></a>
### 4.10 예측 (Prediction)
GAS는 클라이언트 측 예측을 기본적으로 지원합니다. 그러나 모든 것을 예측하는 것은 아닙니다. GAS의 클라이언트 측 예측은 클라이언트가 서버의 허락을 기다리지 않고 `GameplayAbility`를 활성화하고 `GameplayEffects`를 적용할 수 있다는 것을 의미합니다. 클라이언트는 서버가 이를 허락할 것이라고 "예측"하고 `GameplayEffects`를 적용할 대상도 예측할 수 있습니다. 서버는 클라이언트가 활성화한 후 네트워크 지연 시간만큼 후에 `GameplayAbility`를 실행하고, 클라이언트의 예측이 맞았는지 틀렸는지를 알려줍니다. 클라이언트의 예측 중 틀린 것이 있다면, 클라이언트는 "잘못된 예측"으로 인한 변경 사항을 "되돌려" 서버와 일치시킬 것입니다.

GAS 관련 예측에 대한 최종적인 출처는 플러그인 소스 코드의 `GameplayPrediction.h`입니다.

에픽의 사고방식은 "문제가 없을 만큼"만 예측하는 것입니다. 예를 들어, Paragon과 Fortnite는 피해를 예측하지 않습니다. 아마도 그들은 어차피 예측할 수 없는 [`ExecutionCalculations`](#concepts-ge-ec)를 피해 계산에 사용할 것입니다. 그렇다고 해서 피해와 같은 특정 것을 예측할 수 없다는 의미는 아닙니다. 만약 여러분이 그것을 시도해서 잘 작동한다면 그것은 훌륭한 일입니다.

> ...우리는 "모든 것을 완벽하게 자동으로 예측"하는 해결책에 전적으로 동의하지 않습니다. 우리는 여전히 플레이어 예측은 최소한으로 유지하는 것이 가장 좋다고 생각합니다 (즉, 문제가 없을 만큼 최소한의 것만 예측).

*에픽의 Dave Ratti의 새로운 [네트워크 예측 플러그인](#concepts-p-npp)에 대한 코멘트*

**예측되는 것:**
> * 어빌리티 활성화
> * 트리거된 이벤트
> * GameplayEffect 적용:
>    * 어트리뷰트 수정 (예외: 실행은 현재 예측되지 않으며, 어트리뷰트 수정자만 예측됨)
>    * GameplayTag 수정
> * Gameplay Cue 이벤트 (예측 가능한 GameplayEffect 내에서 발생하는 것과 자체적으로 발생하는 것 모두)
> * 몽타주
> * 이동 (UE의 UCharacterMovement에 내장됨)

**예측되지 않는 것:**
> * GameplayEffect 제거
> * GameplayEffect 주기적 효과 (도트 틱)

*From `GameplayPrediction.h`*

`GameplayEffect` 적용은 예측할 수 있지만, `GameplayEffect` 제거는 예측할 수 없습니다. 이 제한을 우회하는 한 가지 방법은 `GameplayEffect`를 제거하려고 할 때 역효과를 예측하는 것입니다. 예를 들어, 이동 속도 40% 감소를 예측한다고 가정해 봅시다. 이동 속도 40% 증가를 적용하여 예측적으로 제거할 수 있습니다. 그런 다음 두 `GameplayEffects`를 동시에 제거합니다. 이것이 모든 시나리오에 적합한 것은 아니며, `GameplayEffect` 제거 예측에 대한 지원은 여전히 필요합니다. 에픽의 Dave Ratti는 이를 [향후 GAS 버전에 추가](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)할 의사를 표명했습니다.

`GameplayEffects` 제거를 예측할 수 없기 때문에 `GameplayAbility` 재사용 대기시간(쿨다운)을 완전히 예측할 수 없으며, 이에 대한 역 `GameplayEffect` 해결책도 없습니다. 서버의 복제된 `Cooldown GE`는 클라이언트에 존재하며, 이를 우회하려는 모든 시도(예: `Minimal` 복제 모드 사용)는 서버에 의해 거부될 것입니다. 이는 대기 시간이 긴 클라이언트가 서버에 쿨다운을 알리고 서버의 `Cooldown GE` 제거를 받는 데 더 오래 걸린다는 것을 의미합니다. 즉, 대기 시간이 긴 플레이어는 대기 시간이 짧은 플레이어보다 발사 속도가 낮아 불리해집니다. Fortnite는 `Cooldown GEs` 대신 커스텀 장부 관리를 사용하여 이 문제를 피합니다.

피해 예측과 관련하여, 저는 개인적으로 GAS를 시작할 때 대부분의 사람들이 가장 먼저 시도하는 것 중 하나임에도 불구하고 추천하지 않습니다. 특히 죽음을 예측하는 것은 추천하지 않습니다. 피해를 예측할 수는 있지만, 그렇게 하는 것은 까다롭습니다. 피해 적용을 잘못 예측하면 플레이어는 적의 체력이 다시 솟아오르는 것을 보게 될 것입니다. 죽음을 예측하려고 하면 특히 어색하고 답답할 수 있습니다. 캐릭터의 죽음을 잘못 예측하여 래그돌링을 시작했지만, 서버가 이를 수정하면 래그돌링을 멈추고 다시 공격해 오는 상황을 상상해 보세요.

**참고:** `Attribute`를 변경하는 `Instant` `GameplayEffects` (`Cost GEs`와 같은)는 자신에게 원활하게 예측될 수 있지만, 다른 캐릭터에 대한 `Instant` `Attribute` 변경을 예측하면 해당 `Attribute`에 짧은 이상 현상 또는 "깜박임"이 나타납니다. 예측된 `Instant` `GameplayEffects`는 실제로는 잘못 예측되었을 경우 롤백될 수 있도록 `Infinite` `GameplayEffects`처럼 취급됩니다. 서버의 `GameplayEffect`가 적용될 때, 일시적으로 동일한 `GameplayEffect`가 두 개 존재하여 `Modifier`가 잠시 동안 두 번 적용되거나 전혀 적용되지 않을 수 있습니다. 결국 스스로 수정되지만 때로는 깜박임이 플레이어에게 눈에 뜁니다.

GAS의 예측 구현이 해결하려는 문제:
> 1. "이걸 할 수 있을까?" 예측을 위한 기본 프로토콜.
> 2. "되돌리기" 예측이 실패했을 때 부작용을 되돌리는 방법.
> 3. "다시 하기" 로컬에서 예측했지만 서버로부터도 복제되는 부작용을 다시 재생하는 것을 피하는 방법.
> 4. "완전성" 모든 부작용을 정말로 예측했는지 확인하는 방법.
> 5. "의존성" 종속 예측 및 예측된 이벤트 체인을 관리하는 방법.
> 6. "오버라이드" 서버가 복제/소유하는 상태를 예측적으로 오버라이드하는 방법.

*From `GameplayPrediction.h`*

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p-key"></a>
#### 4.10.1 예측 키 (Prediction Key)
GAS의 예측은 `Prediction Key`라는 개념을 기반으로 작동하며, 이는 클라이언트가 `GameplayAbility`를 활성화할 때 생성하는 정수 식별자입니다.

* 클라이언트는 `GameplayAbility`를 활성화할 때 예측 키를 생성합니다. 이것이 `Activation Prediction Key`입니다.
* 클라이언트는 이 예측 키를 `CallServerTryActivateAbility()`와 함께 서버로 보냅니다.
* 클라이언트는 이 예측 키가 유효한 동안 적용하는 모든 `GameplayEffects`에 이 예측 키를 추가합니다.
* 클라이언트의 예측 키는 스코프를 벗어납니다. 동일한 `GameplayAbility` 내에서 추가로 예측되는 효과는 새로운 [스코프 예측 창](#concepts-p-windows)이 필요합니다.

* 서버는 클라이언트로부터 예측 키를 받습니다.
* 서버는 적용하는 모든 `GameplayEffects`에 이 예측 키를 추가합니다.
* 서버는 예측 키를 클라이언트로 다시 복제합니다.

* 클라이언트는 서버로부터 예측 키를 사용하여 적용된 복제된 `GameplayEffects`를 받습니다. 복제된 `GameplayEffects` 중 클라이언트가 동일한 예측 키로 적용한 `GameplayEffects`와 일치하는 것이 있다면, 이들은 올바르게 예측된 것입니다. 클라이언트가 예측된 `GameplayEffect`를 제거할 때까지 대상에 일시적으로 동일한 `GameplayEffect` 두 복사본이 존재합니다.
* 클라이언트는 서버로부터 예측 키를 다시 받습니다. 이것이 `Replicated Prediction Key`입니다. 이 예측 키는 이제 만료된 것으로 표시됩니다.
* 클라이언트는 이제 만료된 복제된 예측 키로 생성된 **모든** `GameplayEffects`를 제거합니다. 서버에 의해 복제된 `GameplayEffects`는 유지됩니다. 클라이언트가 추가했지만 서버로부터 일치하는 복제 버전을 받지 못한 `GameplayEffects`는 잘못 예측된 것입니다.

예측 키는 `GameplayAbilities`에서 활성화 예측 키로 시작하는 명령어의 원자적인 그룹화 "창" 동안 유효함이 보장됩니다. 이것은 한 프레임 동안만 유효하다고 생각할 수 있습니다. 지연 동작 `AbilityTasks`의 콜백은 `AbilityTask`에 새로운 [스코프 예측 창](#concepts-p-windows)을 생성하는 내장 동기화 지점이 없는 한 더 이상 유효한 예측 키를 가지지 못할 것입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p-windows"></a>
#### 4.10.2 어빌리티에서 새로운 예측 창 (Prediction Window) 생성
`AbilityTasks`의 콜백에서 더 많은 액션을 예측하려면 새로운 스코프 예측 키(Scoped Prediction Key)를 가진 새로운 스코프 예측 창(Scoped Prediction Window)을 생성해야 합니다. 이는 클라이언트와 서버 간의 동기화 지점(Synch Point)이라고도 합니다. 입력과 관련된 모든 `AbilityTasks`와 같은 일부 `AbilityTasks`는 새로운 스코프 예측 창을 생성하는 내장 기능을 가지고 있습니다. 이는 `AbilityTasks`의 콜백 내 원자적 코드가 유효한 스코프 예측 키를 사용할 수 있다는 의미입니다. `WaitDelay` 태스크와 같은 다른 태스크는 콜백을 위한 새로운 스코프 예측 창을 생성하는 내장 코드가 없습니다. `WaitDelay`와 같이 스코프 예측 창을 생성하는 내장 코드가 없는 `AbilityTask` 이후에 액션을 예측해야 하는 경우, `WaitNetSync` `AbilityTask`와 `OnlyServerWait` 옵션을 사용하여 수동으로 수행해야 합니다. 클라이언트가 `OnlyServerWait`를 가진 `WaitNetSync`에 도달하면, `GameplayAbility`의 활성화 예측 키를 기반으로 새로운 스코프 예측 키를 생성하고, 이를 서버에 RPC로 보내며, 적용하는 모든 새로운 `GameplayEffects`에 추가합니다. 서버가 `OnlyServerWait`를 가진 `WaitNetSync`에 도달하면, 클라이언트로부터 새로운 스코프 예측 키를 받을 때까지 기다린 후 계속합니다. 이 스코프 예측 키는 활성화 예측 키와 동일한 작업을 수행합니다. `GameplayEffects`에 적용되고 클라이언트로 다시 복제되어 만료된 것으로 표시됩니다. 스코프 예측 키는 스코프를 벗어날 때까지 유효합니다. 즉, 스코프 예측 창이 닫힙니다. 따라서 다시 말하지만, 원자적 작업(지연 작업이 아닌)만 새로운 스코프 예측 키를 사용할 수 있습니다.

필요한 만큼 많은 스코프 예측 창을 생성할 수 있습니다.

자신만의 커스텀 `AbilityTasks`에 동기화 지점 기능을 추가하고 싶다면, 입력 관련 태스크들이 본질적으로 `WaitNetSync` `AbilityTask` 코드를 어떻게 주입하는지 살펴보십시오.

**참고:** `WaitNetSync`를 사용할 때, 이는 서버의 `GameplayAbility`가 클라이언트로부터 응답을 받을 때까지 실행을 계속하는 것을 차단합니다. 이는 게임을 해킹하고 의도적으로 새로운 스코프 예측 키 전송을 지연시키는 악의적인 사용자들에 의해 악용될 수 있습니다. 에픽은 `WaitNetSync`를 매우 드물게 사용하지만, 만약 이것이 걱정된다면 클라이언트 없이 자동으로 계속 진행되는 지연 기능이 있는 `AbilityTask`의 새 버전을 구축할 것을 권장합니다.

샘플 프로젝트는 질주 `GameplayAbility`에서 `WaitNetSync`를 사용하여 스태미나 소모를 적용할 때마다 새로운 스코프 예측 창을 생성하여 이를 예측할 수 있도록 합니다. 이상적으로는 비용과 쿨다운을 적용할 때 유효한 예측 키를 원합니다.

소유하는 클라이언트에서 예측된 `GameplayEffect`가 두 번 재생되는 경우, 예측 키가 만료된 것이며 "재실행(redo)" 문제를 겪고 있는 것입니다. 일반적으로 `GameplayEffect`를 적용하기 직전에 `OnlyServerWait`를 가진 `WaitNetSync` `AbilityTask`를 배치하여 새로운 스코프 예측 키를 생성함으로써 이 문제를 해결할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p-spawn"></a>
#### 4.10.3 액터 (Actor) 예측적으로 생성
클라이언트에서 `Actors`를 예측적으로 생성하는 것은 고급 주제입니다. GAS는 기본적으로 이를 처리하는 기능을 제공하지 않습니다 (`SpawnActor` `AbilityTask`는 서버에서만 `Actor`를 생성합니다). 핵심 개념은 클라이언트와 서버 모두에서 복제된 `Actor`를 생성하는 것입니다.

`Actor`가 단순한 시각적 요소이거나 게임플레이 목적에 영향을 주지 않는다면, 간단한 해결책은 `Actor's`의 `IsNetRelevantFor()` 함수를 오버라이드하여 서버가 소유 클라이언트에 복제하는 것을 제한하는 것입니다. 소유 클라이언트는 로컬에서 생성된 버전을 가질 것이고, 서버와 다른 클라이언트는 서버의 복제된 버전을 가질 것입니다.
```c++
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const
{
	return !IsOwnedBy(ViewTarget);
}
```

생성된 `Actor`가 피해를 예측해야 하는 발사체처럼 게임플레이에 영향을 미치는 경우, 이 문서의 범위를 벗어나는 고급 논리가 필요합니다. 에픽 게임즈의 GitHub에서 UnrealTournament가 발사체를 예측적으로 생성하는 방식을 살펴보십시오. 그들은 소유 클라이언트에서만 생성되는 더미 발사체를 가지고 있으며, 이는 서버의 복제된 발사체와 동기화됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p-future"></a>
#### 4.10.4 GAS 예측의 미래
`GameplayPrediction.h`는 장차 `GameplayEffect` 제거 및 주기적인 `GameplayEffects` 예측 기능을 추가할 수 있다고 명시하고 있습니다.

에픽의 Dave Ratti는 쿨다운 예측 시 대기 시간이 긴 플레이어가 대기 시간이 짧은 플레이어에 비해 불리해지는 `latency reconciliation` 문제 해결에 [관심을 표명했습니다](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

에픽의 새로운 [`Network Prediction` 플러그인](#concepts-p-npp)은 이전에 `CharacterMovementComponent`가 그랬던 것처럼 GAS와 완전히 상호 운용 가능할 것으로 예상됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-p-npp"></a>
#### 4.10.5 네트워크 예측 플러그인 (Network Prediction Plugin)
에픽은 최근 `CharacterMovementComponent`를 새로운 `Network Prediction` 플러그인으로 대체하는 이니셔티브를 시작했습니다. 이 플러그인은 아직 초기 단계에 있지만, 언리얼 엔진 GitHub에서 매우 이른 접근이 가능합니다. 어떤 미래 엔진 버전에서 실험적인 베타를 선보일지는 아직 알 수 없습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-targeting"></a>
### 4.11 타겟팅 (Targeting)

<a name="concepts-targeting-data"></a>
#### 4.11.1 타겟 데이터 (Target Data)
[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html)는 네트워크를 통해 전달하기 위한 일반적인 타겟팅 데이터 구조체입니다. `TargetData`는 일반적으로 `AActor`/`UObject` 참조, `FHitResults`, 기타 일반적인 위치/방향/원점 정보를 포함합니다. 그러나 [클라이언트와 서버 간에 `GameplayAbilities`에서 데이터를 전달](#concepts-ga-data)하는 간단한 수단으로 원하는 모든 것을 내부에 넣도록 서브클래싱할 수 있습니다. 기본 구조체 `FGameplayAbilityTargetData`는 직접 사용하기 위한 것이 아니라 서브클래싱을 위한 것입니다. `GAS`는 `GameplayAbilityTargetTypes.h`에 위치한 몇 가지 서브클래싱된 `FGameplayAbilityTargetData` 구조체를 기본적으로 제공합니다.

`TargetData`는 일반적으로 [`Target Actors`](#concepts-targeting-actors)에 의해 생성되거나 **수동으로 생성**되며, [`EffectContext`](#concepts-ge-context)를 통해 [`AbilityTasks`](#concepts-at) 및 [`GameplayEffects`](#concepts-ge)에 의해 소비됩니다. `EffectContext` 내부에 존재하므로, [`Executions`](#concepts-ge-ec), [`MMCs`](#concepts-ge-mmc), [`GameplayCues`](#concepts-gc), 그리고 [`AttributeSet`](#concepts-as)의 백엔드 함수들이 `TargetData`에 접근할 수 있습니다.

우리는 일반적으로 `FGameplayAbilityTargetData`를 직접 전달하지 않고, 내부적으로 `FGameplayAbilityTargetData` 포인터의 TArray를 가진 [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html)을 사용합니다. 이 중간 구조체는 `TargetData`의 다형성을 지원합니다.

`FGameplayAbilityTargetData`에서 상속받는 예시:
```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // FGameplayAbilityTargetData의 모든 자식 구조체에 필요합니다.
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// FGameplayAbilityTargetData의 모든 자식 구조체에 필요합니다.
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // 엔진은 FName & FPredictionKey에 대한 NetSerialize를 이미 정의했습니다. Epic에게 감사합니다!
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
}

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // FGameplayAbilityTargetDataHandle 네트워크 직렬화가 작동하려면 이것이 필수입니다.
	};
};
```
핸들에 타겟 데이터를 추가하는 방법:
```c++
UFUNCTION(BlueprintPure)
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)
{
	// 타겟 데이터 타입을 생성합니다. 
	// 핸들은 핸들이 소멸될 때 이 데이터를 자동으로 정리하고 삭제합니다. 
	// 이 데이터를 핸들에 추가하지 않으면 메모리 관리 및 메모리 누수와 관련되므로 주의해야 합니다. 따라서 항상 프레임의 어느 시점에서든 핸들에 추가하는 것이 안전합니다!
	FGameplayAbilityTargetData_CustomData* MyCustomData = new FGameplayAbilityTargetData_CustomData();
	// 입력된 이름과 우리가 원하는 다른 변경 사항을 사용하도록 구조체 정보를 설정합니다.
	MyCustomData->CoolName = CustomName;
	
	// 블루프린트 사용을 위한 핸들 래퍼를 만듭니다.
	FGameplayAbilityTargetDataHandle Handle;
	// 타겟 데이터를 핸들에 추가합니다.
	Handle.Add(MyCustomData);
	// 블루프린트로 핸들을 출력합니다.
	return Handle
}
```

값을 가져오려면 타입 안전성 검사가 필요합니다. 핸들의 타겟 데이터에서 값을 가져오는 유일한 방법은 일반 C/C++ 캐스팅을 사용하는 것인데, 이는 타입 안전성이 없어서 객체 슬라이싱 및 충돌을 유발할 수 있기 때문입니다. 타입 검사를 하는 여러 가지 방법이 있습니다 (원하는 대로). 두 가지 일반적인 방법은 다음과 같습니다:
- 게임플레이 태그(Gameplay Tag(s)): 특정 코드 아키텍처의 기능이 발생할 때마다 기본 부모 타입으로 캐스팅하여 게임플레이 태그를 가져온 다음, 상속된 클래스로 캐스팅하기 위해 해당 태그와 비교하는 서브클래스 계층 구조를 사용할 수 있습니다.
- 스크립트 구조체 및 정적 구조체(Script Struct & Static Structs): 대신 직접적인 클래스 비교를 수행할 수 있습니다 (이는 많은 IF 문이나 템플릿 함수 생성을 포함할 수 있습니다). 아래는 이 방법을 사용하는 예시이지만, 기본적으로 모든 `FGameplayAbilityTargetData`에서 스크립트 구조체를 가져와서 찾고 있는 타입인지 비교할 수 있습니다 (이는 `USTRUCT`이고 모든 상속된 클래스가 `GetScriptStruct`에 구조체 타입을 지정해야 한다는 장점입니다). 아래는 타입 검사를 위해 이러한 함수를 사용하는 예시입니다:
```c++
UFUNCTION(BlueprintPure)
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)
{   
    // NOTE: 이 '::Get(int32 Index)' 함수에는 두 가지 버전이 있습니다. 
    // 1) 'const FGameplayAbilityTargetData*'를 반환하는 const 버전은 타겟 데이터 값을 읽는 데 좋습니다. 
    // 2) 'FGameplayAbilityTargetData*'를 반환하는 non-const 버전은 타겟 데이터 값을 수정하는 데 좋습니다.
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // 이것은 인덱스의 유효성을 자동으로 검사합니다. 
    
    // 사용할 것이 있는지 유효성 검사를 합니다. null 데이터는 캐스팅할 것이 없다는 의미입니다.
    if(Data == nullptr)
    {
       	return NAME_None;
    }
    // 이것은 기본적으로 타입 검사 단계입니다. static_cast는 타입 안전성이 없으므로 이 검사를 수행합니다.
    // 이 검사를 수행하지 않으면 구조체가 객체 슬라이싱되어 해당 타입인지 확인할 방법이 없습니다.
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())
    {
        // 여기에서 캐스팅을 수행합니다. 이미 올바른 타입임을 알고 있기 때문입니다.
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);    
        return CustomData->CoolName;
    }
    return NAME_None;
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-targeting-actors"></a>
#### 4.11.2 타겟 액터 (Target Actors)
`GameplayAbilities`는 `WaitTargetData` `AbilityTask`를 사용하여 [`TargetActors`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html)를 생성하여 월드에서 타겟팅 정보를 시각화하고 캡처합니다. `TargetActors`는 선택적으로 [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles)를 사용하여 현재 타겟을 표시할 수 있습니다. 확인 시, 타겟팅 정보는 [`TargetData`](#concepts-targeting-data)로 반환되며, 이는 [`GameplayEffects`](#concepts-ge)에 전달될 수 있습니다.

`TargetActors`는 `AActor`를 기반으로 하므로, 스태틱 메시 또는 데칼과 같이 타겟팅하는 **위치**와 **방법**을 나타내는 모든 종류의 시각적 구성 요소를 가질 수 있습니다. 스태틱 메시는 캐릭터가 만들 객체의 배치 위치를 시각화하는 데 사용될 수 있습니다. 데칼은 지면에 효과 영역을 표시하는 데 사용될 수 있습니다. 샘플 프로젝트는 [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html)를 지면에 데칼과 함께 사용하여 메테오 어빌리티의 피해 효과 영역을 나타냅니다. 또한 아무것도 표시할 필요가 없습니다. 예를 들어, [GASShooter](https://github.com/tranek/GASShooter)에서 사용되는 것처럼 대상을 향해 즉시 선을 트레이스하는 히트스캔 총에 대해서는 아무것도 표시하는 것이 의미가 없을 것입니다.

이들은 기본 트레이스 또는 충돌 오버랩을 사용하여 타겟팅 정보를 캡처하고, `TargetActor` 구현에 따라 결과를 `FHitResults` 또는 `AActor` 배열로 `TargetData`에 변환합니다. `WaitTargetData` `AbilityTask`는 `TEnumAsByte<EGameplayTargetingConfirmation::Type> ConfirmationType` 매개변수를 통해 타겟이 언제 확인되는지 결정합니다. `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant`를 사용하지 않을 때, `TargetActor`는 일반적으로 `Tick()`에서 트레이스/오버랩을 수행하고 구현에 따라 `FHitResult`로 위치를 업데이트합니다. 이것이 `Tick()`에서 트레이스/오버랩을 수행하지만, 복제되지 않고 일반적으로 한 번에 하나 이상의 `TargetActor`가 실행되지 않으므로 일반적으로 문제가 되지 않습니다. 다만 `Tick()`을 사용하며 GASShooter의 로켓 런처 보조 어빌리티와 같이 일부 복잡한 `TargetActors`는 `Tick()`에서 많은 작업을 수행할 수 있다는 점을 알아두십시오. `Tick()`에서 트레이싱하는 것은 클라이언트에 매우 반응적이지만, 성능 저하가 너무 크다면 `TargetActor`의 틱 속도를 낮추는 것을 고려할 수 있습니다. `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant`의 경우, `TargetActor`는 즉시 생성되고 `TargetData`를 생성한 후 파괴됩니다. `Tick()`은 호출되지 않습니다.

| `EGameplayTargetingConfirmation::Type` | 타겟이 확인되는 시점 |
|---|---|
| `Instant` | 타겟팅은 특별한 로직이나 사용자 입력 없이 즉시 발생하며, 언제 '발사'할지 결정하지 않습니다. |
| `UserConfirmed` | 타겟팅은 [어빌리티가 `Confirm` 입력에 바인딩](#concepts-ga-input)되거나 `UAbilitySystemComponent::TargetConfirm()`을 호출하여 사용자가 타겟팅을 확인할 때 발생합니다. `TargetActor`는 바인딩된 `Cancel` 입력 또는 `UAbilitySystemComponent::TargetCancel()` 호출에도 응답하여 타겟팅을 취소합니다. |
| `Custom` | `GameplayTargeting Ability`는 `UGameplayAbility::ConfirmTaskByInstanceName()`을 호출하여 타겟팅 데이터가 준비되었을 때를 결정할 책임이 있습니다. `TargetActor`는 또한 `UGameplayAbility::CancelTaskByInstanceName()`에 응답하여 타겟팅을 취소합니다. |
| `CustomMulti` | `GameplayTargeting Ability`는 `UGameplayAbility::ConfirmTaskByInstanceName()`을 호출하여 타겟팅 데이터가 준비되었을 때를 결정할 책임이 있습니다. `TargetActor`는 또한 `UGameplayAbility::CancelTaskByInstanceName()`에 응답하여 타겟팅을 취소합니다. 데이터 생성 시 `AbilityTask`를 종료해서는 안 됩니다. |

모든 `EGameplayTargetingConfirmation::Type`이 모든 `TargetActor`에서 지원되는 것은 아닙니다. 예를 들어, `AGameplayAbilityTargetActor_GroundTrace`는 `Instant` 확인을 지원하지 않습니다.

`WaitTargetData` `AbilityTask`는 `AGameplayAbilityTargetActor` 클래스를 매개변수로 받아 `AbilityTask`가 활성화될 때마다 인스턴스를 생성하고 `AbilityTask`가 종료될 때 `TargetActor`를 파괴합니다. `WaitTargetDataUsingActor` `AbilityTask`는 이미 생성된 `TargetActor`를 받지만, `AbilityTask`가 종료될 때 여전히 파괴합니다. 이 두 `AbilityTasks`는 사용할 때마다 `TargetActor`를 생성하거나 새로 생성된 `TargetActor`를 필요로 한다는 점에서 비효율적입니다. 이는 프로토타이핑에는 좋지만, 자동 소총과 같이 지속적으로 `TargetData`를 생성하는 경우와 같이 프로덕션에서는 이를 최적화하는 것을 고려할 수 있습니다. GASShooter는 [`AGameplayAbilityTargetActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h)의 커스텀 서브클래스와 처음부터 작성된 새로운 [`WaitTargetDataWithReusableActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h) `AbilityTask`를 가지고 있어 `TargetActor`를 파괴하지 않고 재사용할 수 있도록 합니다.

`TargetActors`는 기본적으로 복제되지 않지만, 로컬 플레이어가 어디를 타겟팅하고 있는지 다른 플레이어에게 보여주는 것이 게임에서 의미가 있다면 복제될 수 있습니다. 이들은 `WaitTargetData` `AbilityTask`의 RPC를 통해 서버와 통신하는 기본 기능을 포함합니다. `TargetActor`의 `ShouldProduceTargetDataOnServer` 속성이 `false`로 설정되면, 클라이언트는 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()`에서 `CallServerSetReplicatedTargetData()`를 통해 확인 시 `TargetData`를 서버로 RPC합니다. `ShouldProduceTargetDataOnServer`가 `true`이면, 클라이언트는 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()`에서 서버로 일반 확인 이벤트 `EAbilityGenericReplicatedEvent::GenericConfirm` RPC를 보내고, 서버는 RPC를 수신하면 데이터 생성을 위해 트레이스 또는 오버랩 검사를 수행합니다. 클라이언트가 타겟팅을 취소하면, `UAbilityTask_WaitTargetData::OnTargetDataCancelledCallback`에서 서버로 일반 취소 이벤트 `EAbilityGenericReplicatedEvent::GenericCancel` RPC를 보냅니다. 보시다시피, `TargetActor`와 `WaitTargetData` `AbilityTask` 모두에 많은 델리게이트가 있습니다. `TargetActor`는 입력을 받아 `TargetData` 준비, 확인 또는 취소 델리게이트를 생성하고 브로드캐스트합니다. `WaitTargetData`는 `TargetActor`의 `TargetData` 준비, 확인 및 취소 델리게이트를 수신하고 해당 정보를 `GameplayAbility` 및 서버로 전달합니다. `TargetData`를 서버로 보낼 경우, 부정 행위를 방지하기 위해 `TargetData`가 합리적인지 확인하는 유효성 검사를 서버에서 수행하는 것이 좋습니다. `TargetData`를 서버에서 직접 생성하면 이 문제가 완전히 해결되지만, 소유 클라이언트에 대한 잘못된 예측으로 이어질 수 있습니다.

사용하는 `AGameplayAbilityTargetActor`의 특정 서브클래스에 따라 `WaitTargetData` `AbilityTask` 노드에 다른 `ExposeOnSpawn` 매개변수가 노출됩니다. 일반적인 매개변수는 다음과 같습니다:

| 일반적인 `TargetActor` 매개변수 | 정의 |
|---|---|
| Debug | `true`인 경우, `TargetActor`가 비배포 빌드에서 트레이스를 수행할 때마다 디버그 트레이싱/오버랩 정보를 그립니다. `Instant`가 아닌 `TargetActors`는 `Tick()`에서 트레이스를 수행하므로 이러한 디버그 드로잉 호출도 `Tick()`에서 발생합니다. |
| Filter | [선택 사항] 트레이스/오버랩 발생 시 타겟에서 `Actors`를 필터링(제거)하기 위한 특수 구조체입니다. 일반적인 사용 사례는 플레이어의 `Pawn`을 필터링하거나, 특정 클래스의 타겟만 요구하는 것입니다. 더 고급 사용 사례는 [타겟 데이터 필터](#concepts-target-data-filters)를 참조하십시오. |
| Reticle Class | [선택 사항] `TargetActor`가 생성할 `AGameplayAbilityWorldReticle`의 서브클래스입니다. |
| Reticle Parameters | [선택 사항] 레티클을 구성합니다. [레티클](#concepts-targeting-reticles)을 참조하십시오. |
| Start Location | 트레이싱이 시작되어야 하는 위치를 위한 특수 구조체입니다. 일반적으로 플레이어의 시점, 무기 총구 또는 `Pawn`의 위치가 됩니다. |

기본 `TargetActor` 클래스를 사용하면, `Actors`는 트레이스/오버랩 내에 직접 있을 때만 유효한 타겟입니다. 트레이스/오버랩을 벗어나면 (이동하거나 시선을 돌리면) 더 이상 유효하지 않습니다. `TargetActor`가 마지막 유효한 타겟을 기억하기를 원한다면, 이 기능을 커스텀 `TargetActor` 클래스에 추가해야 합니다. 저는 이를 지속적인 타겟이라고 부르는데, 이는 `TargetActor`가 확인 또는 취소를 받거나, `TargetActor`가 트레이스/오버랩에서 새로운 유효한 타겟을 찾거나, 타겟이 더 이상 유효하지 않을 때 (파괴될 때)까지 지속되기 때문입니다. GASShooter는 로켓 런처의 보조 어빌리티 호밍 로켓 타겟팅을 위해 지속적인 타겟을 사용합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-target-data-filters"></a>
#### 4.11.3 타겟 데이터 필터 (Target Data Filters)
`Make GameplayTargetDataFilter`와 `Make Filter Handle` 노드를 모두 사용하여 플레이어의 `Pawn`을 필터링하거나 특정 클래스만 선택할 수 있습니다. 더 고급 필터링이 필요한 경우, `FGameplayTargetDataFilter`를 서브클래싱하고 `FilterPassesForActor` 함수를 오버라이드할 수 있습니다.
```c++
USTRUCT(BlueprintType)
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter
{
	GENERATED_BODY()

	/** 액터가 필터를 통과하고 타겟팅될 경우 true를 반환합니다. */
	virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};
```

그러나 이 방법은 `FGameplayTargetDataFilterHandle`이 필요하므로 `Wait Target Data` 노드에 직접 작동하지 않습니다. 서브클래스를 받기 위해 새로운 커스텀 `Make Filter Handle`을 만들어야 합니다:
```c++
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, AActor* FilterActor)
{
	FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);
	NewFilter->InitializeFilterContext(FilterActor);

	FGameplayTargetDataFilterHandle FilterHandle;
	FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter);
	return FilterHandle;
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-targeting-reticles"></a>
#### 4.11.4 게임플레이 어빌리티 월드 레티클 (Gameplay Ability World Reticles)
[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html) (`Reticles`)은 `Instant` 확인이 아닌 [`TargetActors`](#concepts-targeting-actors)로 타겟팅할 때 **누구**를 타겟팅하고 있는지 시각화합니다. `TargetActors`는 모든 `Reticles`의 생성 및 파괴 수명 주기를 담당합니다. `Reticles`는 `AActors`이므로 표현을 위해 모든 종류의 시각적 구성 요소를 사용할 수 있습니다. [GASShooter](https://github.com/tranek/GASShooter)에서 볼 수 있는 일반적인 구현은 `WidgetComponent`를 사용하여 UMG 위젯을 화면 공간(항상 플레이어 카메라를 향함)에 표시하는 것입니다. `Reticles`는 자신이 어떤 `AActor`에 있는지 알지 못하지만, 커스텀 `TargetActor`에 해당 기능을 서브클래싱할 수 있습니다. `TargetActors`는 일반적으로 `Tick()`마다 `Reticle`의 위치를 타겟 위치로 업데이트합니다.

GASShooter는 로켓 런처의 보조 어빌리티 호밍 로켓의 고정 타겟을 표시하기 위해 `Reticles`를 사용합니다. 적 위의 빨간색 표시기가 `Reticle`입니다. 유사한 흰색 이미지는 로켓 런처의 십자선입니다.
![Reticles in GASShooter](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles`에는 디자이너를 위한 몇 가지 `BlueprintImplementableEvents`가 함께 제공됩니다 (블루프린트에서 개발하도록 의도됨):

```c++
/** bIsTargetValid 값이 변경될 때마다 호출됩니다. */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnValidTargetChanged(bool bNewValue);

/** bIsTargetAnActor 값이 변경될 때마다 호출됩니다. */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnTargetingAnActor(bool bNewValue);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnParametersInitialized();

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamFloat(FName ParamName, float value);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamVector(FName ParamName, FVector value);
```

`Reticles`는 `TargetActor`가 제공하는 [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html)를 사용하여 설정을 선택적으로 사용할 수 있습니다. 기본 구조체는 `FVector AOEScale`이라는 하나의 변수만 제공합니다. 이 구조체를 기술적으로 서브클래싱할 수 있지만, `TargetActor`는 기본 구조체만 허용합니다. 기본 `TargetActors`에서 이를 서브클래싱하는 것을 허용하지 않는 것은 다소 근시안적인 것으로 보입니다. 그러나 자신만의 커스텀 `TargetActor`를 만들 경우, 자신만의 커스텀 레티클 매개변수 구조체를 제공하고 이를 생성할 때 `AGameplayAbilityWorldReticles`의 서브클래스에 수동으로 전달할 수 있습니다.

`Reticles`는 기본적으로 복제되지 않지만, 로컬 플레이어가 누구를 타겟팅하고 있는지 다른 플레이어에게 보여주는 것이 게임에서 의미가 있다면 복제될 수 있습니다.

기본 `TargetActors`를 사용하는 경우 `Reticles`는 현재 유효한 타겟에만 표시됩니다. 예를 들어, 타겟을 트레이스하기 위해 `AGameplayAbilityTargetActor_SingleLineTrace`를 사용하는 경우, 적이 트레이스 경로에 직접 있을 때만 `Reticle`이 나타납니다. 시선을 돌리면 적은 더 이상 유효한 타겟이 아니며 `Reticle`은 사라집니다. `Reticle`이 마지막 유효한 타겟에 머물기를 원한다면, 마지막 유효한 타겟을 기억하고 `Reticle`을 유지하도록 `TargetActor`를 커스터마이징해야 합니다. 저는 이를 지속적인 타겟이라고 부르는데, 이는 `TargetActor`가 확인 또는 취소를 받거나, `TargetActor`가 트레이스/오버랩에서 새로운 유효한 타겟을 찾거나, 타겟이 더 이상 유효하지 않을 때 (파괴될 때)까지 지속되기 때문입니다. GASShooter는 로켓 런처의 보조 어빌리티 호밍 로켓 타겟팅을 위해 지속적인 타겟을 사용합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="concepts-targeting-containers"></a>
#### 4.11.5 게임플레이 이펙트 컨테이너 타겟팅 (Gameplay Effect Containers Targeting)
[`GameplayEffectContainers`](#concepts-ge-containers)는 [`TargetData`](#concepts-targeting-data)를 효율적으로 생성하는 선택적 수단을 제공합니다. 이 타겟팅은 클라이언트와 서버에서 `EffectContainer`가 적용될 때 즉시 발생합니다. `TargetActors`보다 더 효율적인 이유는 타겟팅 객체의 CDO에서 실행되기 때문입니다(`Actors`를 생성하고 파괴할 필요 없음). 하지만 플레이어 입력이 없고, 확인이 필요 없이 즉시 발생하며, 취소할 수 없고, 클라이언트에서 서버로 데이터를 보낼 수 없습니다(둘 다 데이터를 생성함). 즉시 트레이스와 충돌 오버랩에 잘 작동합니다. 에픽의 [Action RPG Sample Project](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)는 컨테이너를 사용한 두 가지 타겟팅 유형의 예를 포함합니다. 어빌리티 소유자를 타겟팅하고 이벤트에서 `TargetData`를 가져오는 것입니다. 또한 플레이어로부터 특정 오프셋(자식 블루프린트 클래스에서 설정)에서 즉시 구체 트레이스를 수행하는 블루프린트도 구현합니다. C++ 또는 블루프린트에서 `URPGTargetType`을 서브클래싱하여 자신만의 타겟팅 유형을 만들 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae"></a>
## 5. 일반적으로 구현되는 어빌리티 및 효과

<a name="cae-stun"></a>
### 5.1 기절 (Stun)
일반적으로 기절의 경우, 캐릭터의 활성 `GameplayAbilities`를 모두 취소하고, 새로운 `GameplayAbility` 활성화를 방지하며, 기절 지속 시간 동안 이동을 방지하려고 합니다. 샘플 프로젝트의 메테오 `GameplayAbility`는 적중한 대상에게 기절을 적용합니다.

대상 캐릭터의 활성 `GameplayAbilities`를 취소하기 위해, 기절 [`GameplayTag`가 추가](#concepts-gt-change)될 때 `AbilitySystemComponent->CancelAbilities()`를 호출합니다.

기절 상태에서 새로운 `GameplayAbilities`가 활성화되는 것을 방지하기 위해, `GameplayAbilities`에는 [`Activation Blocked Tags` `GameplayTagContainer`](#concepts-ga-tags)에 기절 `GameplayTag`가 부여됩니다.

기절 상태에서 이동을 방지하기 위해, `CharacterMovementComponent's`의 `GetMaxSpeed()` 함수를 오버라이드하여 소유자가 기절 `GameplayTag`를 가지고 있을 때 0을 반환하도록 합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-sprint"></a>
### 5.2 질주 (Sprint)
샘플 프로젝트는 질주하는 방법의 예를 제공합니다. `Left Shift`를 누르고 있는 동안 더 빠르게 달리는 것입니다.

더 빠른 이동은 `CharacterMovementComponent`에 의해 예측적으로 처리되며, 네트워크를 통해 서버로 플래그를 보냅니다. 자세한 내용은 `GDCharacterMovementComponent.h/cpp`를 참조하십시오.

`GA`는 `Left Shift` 입력에 응답하고, `CMC`에게 질주를 시작하고 멈추도록 지시하며, `Left Shift`가 눌러져 있는 동안 예측적으로 스태미나를 충전합니다. 자세한 내용은 `GA_Sprint_BP`를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-ads"></a>
### 5.3 조준 (Aim Down Sights)
샘플 프로젝트는 질주와 똑같은 방식으로 처리하지만, 이동 속도를 증가시키는 대신 감소시킵니다.

이동 속도를 예측적으로 감소시키는 방법에 대한 자세한 내용은 `GDCharacterMovementComponent.h/cpp`를 참조하십시오.

입력 처리 방법에 대한 자세한 내용은 `GA_AimDownSight_BP`를 참조하십시오. 조준 시 스태미나 소모는 없습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-ls"></a>
### 5.4 생명력 흡수 (Lifesteal)
저는 피해 [`ExecutionCalculation`](#concepts-ge-ec) 내에서 생명력 흡수를 처리합니다. `GameplayEffect`에는 `Effect.CanLifesteal`과 같은 `GameplayTag`가 있을 것입니다. `ExecutionCalculation`은 `GameplayEffectSpec`에 `Effect.CanLifesteal` `GameplayTag`가 있는지 확인합니다. `GameplayTag`가 존재하면, `ExecutionCalculation`은 수정자로 부여할 체력량과 함께 [동적 `Instant` `GameplayEffect`](#concepts-ge-dynamic)를 생성하고 `Source's` `ASC`에 다시 적용합니다.

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();

	SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-random"></a>
### 5.5 클라이언트 및 서버에서 난수 생성
때로는 총알 반동이나 분산과 같은 것을 위해 `GameplayAbility` 내에서 "난수"를 생성해야 할 때가 있습니다. 클라이언트와 서버 모두 동일한 난수를 생성하기를 원할 것입니다. 이를 위해 `GameplayAbility` 활성화 시 `random seed`를 동일하게 설정해야 합니다. 클라이언트가 활성화를 잘못 예측하여 난수 시퀀스가 서버와 동기화되지 않는 경우를 대비하여 `GameplayAbility`를 활성화할 때마다 `random seed`를 설정해야 합니다.

| 시드 설정 방법 | 설명 |
|---|---|
| 활성화 예측 키 사용 | `GameplayAbility` 활성화 예측 키는 `Activation()`에서 클라이언트와 서버 모두에 동기화되어 사용 가능한 int16으로 보장됩니다. 이를 클라이언트와 서버 모두에서 `random seed`로 설정할 수 있습니다. 이 방법의 단점은 게임이 시작될 때마다 예측 키가 항상 0으로 시작하고 키 생성 간에 사용할 값을 일관되게 증가시킨다는 것입니다. 이는 각 매치에서 정확히 동일한 난수 시퀀스를 가질 것임을 의미합니다. 이는 요구 사항에 따라 충분히 무작위적일 수도 있고 아닐 수도 있습니다. |
| `GameplayAbility` 활성화 시 이벤트 페이로드를 통해 시드 전송 | 이벤트를 통해 `GameplayAbility`를 활성화하고 클라이언트에서 서버로 복제된 이벤트 페이로드를 통해 무작위로 생성된 시드를 보냅니다. 이는 더 많은 무작위성을 허용하지만, 클라이언트는 게임을 쉽게 해킹하여 매번 동일한 시드 값만 보낼 수 있습니다. 또한 이벤트를 통해 `GameplayAbilities`를 활성화하면 입력 바인딩을 통해 활성화되는 것을 방지합니다. |

무작위 편차가 작다면 대부분의 플레이어는 매 게임마다 시퀀스가 동일하다는 것을 눈치채지 못할 것이며, 활성화 예측 키를 `random seed`로 사용하는 것이 작동할 것입니다. 해커 방지가 필요한 더 복잡한 작업을 수행하는 경우, 서버가 예측 키를 생성하거나 이벤트 페이로드를 통해 보낼 `random seed`를 생성할 수 있는 `Server Initiated` `GameplayAbility`가 더 잘 작동할 수 있습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-crit"></a>
### 5.6 치명타 (Critical Hits)
저는 피해 [`ExecutionCalculation`](#concepts-ge-ec) 내부에서 치명타를 처리합니다. `GameplayEffect`에는 `Effect.CanCrit`와 같은 `GameplayTag`가 있을 것입니다. `ExecutionCalculation`은 `GameplayEffectSpec`에 `Effect.CanCrit` `GameplayTag`가 있는지 확인합니다. `GameplayTag`가 존재하면, `ExecutionCalculation`은 치명타 확률(소스에서 캡처된 `Attribute`)에 해당하는 난수를 생성하고, 성공하면 치명타 피해(역시 소스에서 캡처된 `Attribute`)를 추가합니다. 저는 피해를 예측하지 않으므로 `ExecutionCalculation`이 서버에서만 실행되기 때문에 클라이언트와 서버의 난수 생성기를 동기화하는 것에 대해 걱정할 필요가 없습니다. 만약 `MMC`를 사용하여 피해 계산을 예측적으로 수행하려고 시도했다면, `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance`에서 `random seed`에 대한 참조를 가져와야 했을 것입니다.

[GASShooter](https://github.com/tranek/GASShooter)가 헤드샷을 처리하는 방식을 참조하십시오. 이는 확률에 대한 난수에 의존하지 않고 `FHitResult` 뼈 이름을 확인한다는 점을 제외하고는 동일한 개념입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-nonstackingge"></a>
### 5.7 중첩되지 않는 게임플레이 이펙트 (Gameplay Effects)이지만 가장 큰 크기만 대상에 영향을 미칩니다
Paragon의 둔화 효과는 중첩되지 않았습니다. 각 둔화 인스턴스는 정상적으로 적용되고 수명 주기를 추적했지만, 가장 큰 크기의 둔화 효과만 실제로 `Character`에 영향을 미쳤습니다. GAS는 `AggregatorEvaluateMetaData`를 통해 이러한 시나리오를 기본적으로 제공합니다. 자세한 내용 및 구현은 [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated)를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-paused"></a>
### 5.8 게임 일시 중지 중 타겟 데이터 (Target Data) 생성
플레이어의 `WaitTargetData` `AbilityTask`에서 [`TargetData`](#concepts-targeting-data) 생성을 기다리는 동안 게임을 일시 중지해야 하는 경우, 일시 중지 대신 `slomo 0`을 사용할 것을 권장합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="cae-onebuttoninteractionsystem"></a>
### 5.9 원 버튼 상호작용 시스템 (One Button Interaction System)
[GASShooter](https://github.com/tranek/GASShooter)는 플레이어가 'E'를 누르거나 길게 눌러 플레이어 부활, 무기 상자 열기, 미닫이문 열고 닫기와 같은 상호작용 가능한 객체와 상호작용할 수 있는 원 버튼 상호작용 시스템을 구현합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="debugging"></a>
## 6. GAS 디버깅
GAS 관련 문제를 디버깅할 때 종종 다음을 알고 싶을 것입니다:
> * "내 어트리뷰트 값은 무엇인가?"
> * "어떤 게임플레이 태그를 가지고 있는가?"
> * "현재 어떤 게임플레이 이펙트를 가지고 있는가?"
> * "어떤 어빌리티를 부여받았고, 어떤 어빌리티가 실행 중이며, 어떤 어빌리티가 활성화가 차단되어 있는가?"

GAS는 이러한 질문에 런타임에 답하기 위한 두 가지 기술을 제공합니다. [`showdebug abilitysystem`](#debugging-sd)과 [`GameplayDebugger`](#debugging-gd)의 훅입니다.

**팁:** 언리얼 엔진은 C++ 코드를 최적화하는 경향이 있어 일부 함수를 디버깅하기 어렵게 만듭니다. 코드를 깊이 추적할 때 드물게 이러한 상황에 직면할 것입니다. Visual Studio 솔루션 구성을 `DebugGame Editor`로 설정해도 코드 추적이나 변수 검사가 여전히 불가능하다면, 최적화된 함수를 `UE_DISABLE_OPTIMIZATION` 및 `UE_ENABLE_OPTIMIZATION` 매크로 또는 CoreMiscDefines.h에 정의된 배포 버전으로 래핑하여 모든 최적화를 비활성화할 수 있습니다. 이는 소스에서 플러그인을 다시 빌드하지 않는 한 플러그인 코드에 사용할 수 없습니다. 인라인 함수의 경우 작동 여부는 해당 함수가 무엇을 하는지, 어디에 있는지에 따라 달라질 수 있습니다. 디버깅이 끝나면 매크로를 제거해야 합니다!

```c++
UE_DISABLE_OPTIMIZATION
void MyClass::MyFunction(int32 MyIntParameter)
{
	// My code
}
UE_ENABLE_OPTIMIZATION
```

**[⬆ 맨 위로](#table-of-contents)**

<a name="debugging-sd"></a>
### 6.1 showdebug abilitysystem
게임 내 콘솔에 `showdebug abilitysystem`을 입력합니다. 이 기능은 세 가지 "페이지"로 나뉩니다. 세 페이지 모두 현재 가지고 있는 `GameplayTags`를 표시합니다. 콘솔에 `AbilitySystem.Debug.NextCategory`를 입력하여 페이지를 전환할 수 있습니다.

첫 번째 페이지는 모든 `Attributes`의 `CurrentValue`를 보여줍니다:
![First Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

두 번째 페이지는 현재 당신에게 적용된 모든 `Duration` 및 `Infinite` `GameplayEffects`, 스택 수, 부여하는 `GameplayTags`, 그리고 부여하는 `Modifiers`를 보여줍니다.
![Second Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

세 번째 페이지는 부여받은 모든 `GameplayAbilities`, 현재 실행 중인지, 활성화가 차단되었는지, 그리고 현재 실행 중인 `AbilityTasks`의 상태를 보여줍니다.
![Third Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

대상을 전환하려면 (액터 주변의 녹색 직사각형 프리즘으로 표시됨) `PageUp` 키 또는 `NextDebugTarget` 콘솔 명령을 사용하여 다음 대상으로 이동하고, `PageDown` 키 또는 `PreviousDebugTarget` 콘솔 명령을 사용하여 이전 대상으로 이동합니다.

**참고:** 어빌리티 시스템 정보가 현재 선택된 디버그 액터에 따라 업데이트되도록 하려면, `DefaultGame.ini`에서 `AbilitySystemGlobals`에 `bUseDebugTargetFromHud=true`를 다음과 같이 설정해야 합니다:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**참고:** `showdebug abilitysystem`이 작동하려면 GameMode에서 실제 HUD 클래스가 선택되어야 합니다. 그렇지 않으면 명령을 찾을 수 없으며 "Unknown Command"가 반환됩니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="debugging-gd"></a>
### 6.2 게임플레이 디버거 (Gameplay Debugger)
GAS는 게임플레이 디버거에 기능을 추가합니다. 어포스트로피(`'`) 키로 게임플레이 디버거에 접근합니다. 숫자 키패드의 3번을 눌러 어빌리티(Abilities) 카테고리를 활성화합니다. 가지고 있는 플러그인에 따라 카테고리가 다를 수 있습니다. 노트북처럼 숫자 키패드가 없는 키보드를 사용하는 경우, 프로젝트 설정에서 키 바인딩을 변경할 수 있습니다.

**다른** `Characters`에 대한 `GameplayTags`, `GameplayEffects`, `GameplayAbilities`를 확인하고 싶을 때 게임플레이 디버거를 사용하십시오. 아쉽게도 대상의 `Attributes`의 `CurrentValue`는 표시되지 않습니다. 화면 중앙에 있는 `Character`를 대상으로 합니다. 에디터의 월드 아웃라이너에서 대상을 선택하거나 다른 `Character`를 보고 다시 어포스트로피(`'`)를 눌러 대상을 변경할 수 있습니다. 현재 검사 중인 `Character`는 그 위에 가장 큰 빨간색 원이 표시됩니다.

![Gameplay Debugger](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="debugging-log"></a>
### 6.3 GAS 로깅
GAS 소스 코드에는 다양한 상세도 수준으로 생성되는 많은 로깅 문이 포함되어 있습니다. 이것들은 대부분 `ABILITY_LOG()` 문으로 나타날 것입니다. 기본 상세도 수준은 `Display`입니다. 기본적으로 콘솔에는 더 높은 수준은 표시되지 않습니다.

로그 카테고리의 상세도 수준을 변경하려면 콘솔에 다음을 입력하십시오:

```
log [category] [verbosity]
```

예를 들어, `ABILITY_LOG()` 문을 켜려면 콘솔에 다음을 입력합니다:
```
log LogAbilitySystem VeryVerbose
```

기본값으로 되돌리려면 다음을 입력합니다:
```
log LogAbilitySystem Display
```

모든 로그 카테고리를 표시하려면 다음을 입력합니다:
```
log list
```

주요 GAS 관련 로깅 카테고리:

| 로깅 카테고리 | 기본 상세도 수준 |
|---|---|
| LogAbilitySystem | Display |
| LogAbilitySystemComponent | Log |
| LogGameplayCueDetails | Log |
| LogGameplayCueTranslator | Display |
| LogGameplayEffectDetails | Log |
| LogGameplayEffects | Display |
| LogGameplayTags | Log |
| LogGameplayTasks | Log |
| VLogAbilitySystem | Display |

자세한 내용은 [로깅 위키](https://unrealcommunity.wiki/logging-lgpidy6i)를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="optimizations"></a>
## 7. 최적화

<a name="optimizations-abilitybatching"></a>
### 7.1 어빌리티 배치 (Ability Batching)
한 프레임 내에서 활성화되고, 선택적으로 `TargetData`를 서버로 보내고, 종료되는 [`GameplayAbilities`](#concepts-ga)는 [두세 개의 RPC를 하나의 RPC로 압축하기 위해 배치](#concepts-ga-batching)될 수 있습니다. 이러한 유형의 어빌리티는 주로 히트스캔 총에 사용됩니다.

<a name="optimizations-gameplaycuebatching"></a>
### 7.2 게임플레이 큐 배치 (Gameplay Cue Batching)
여러 [`GameplayCues`](#concepts-gc)를 동시에 보내는 경우, [이들을 하나의 RPC로 배치](#concepts-gc-batching)하는 것을 고려하십시오. 목표는 RPC(`GameplayCues`는 비신뢰성 NetMulticast입니다) 수를 줄이고 가능한 한 적은 데이터를 보내는 것입니다.

<a name="optimizations-ascreplicationmode"></a>
### 7.3 어빌리티 시스템 컴포넌트 복제 모드 (AbilitySystemComponent Replication Mode)
기본적으로 [`ASC`](#concepts-asc)는 [`Full Replication Mode`](#concepts-asc-rm)에 있습니다. 이는 모든 [`GameplayEffects`](#concepts-ge)를 모든 클라이언트에 복제합니다 (싱글 플레이어 게임에는 문제가 없습니다). 멀티플레이어 게임에서는 플레이어 소유 `ASCs`를 `Mixed Replication Mode`로 설정하고 AI 제어 캐릭터를 `Minimal Replication Mode`로 설정하십시오. 이렇게 하면 플레이어 캐릭터에 적용된 `GEs`는 해당 캐릭터의 소유자에게만 복제되고, AI 제어 캐릭터에 적용된 `GEs`는 클라이언트에 전혀 복제되지 않습니다. `Replication Mode`와 관계없이 [`GameplayTags`](#concepts-gt)는 여전히 복제되고 [`GameplayCues`](#concepts-gc)는 모든 클라이언트에 대해 비신뢰성 NetMulticast로 작동합니다. 이렇게 하면 모든 클라이언트가 `GEs`를 볼 필요가 없을 때 `GEs` 복제로 인한 네트워크 데이터를 줄일 수 있습니다.

<a name="optimizations-attributeproxyreplication"></a>
### 7.4 어트리뷰트 프록시 복제 (Attribute Proxy Replication)
Fortnite Battle Royale (FNBR)과 같이 많은 플레이어가 있는 대규모 게임에서는 항상 관련 있는 `PlayerStates`에 많은 [`ASCs`](#concepts-asc)가 존재하며 많은 [`Attributes`](#concepts-a)를 복제합니다. 이 병목 현상을 최적화하기 위해 Fortnite는 **시뮬레이션된 플레이어 제어 프록시**에서 `PlayerState::ReplicateSubobjects()`에서 `ASC`와 그 [`AttributeSets`](#concepts-as)의 복제를 완전히 비활성화합니다. Autonomous proxies와 AI 제어 `Pawns`는 [`Replication Mode`](#concepts-asc-rm)에 따라 여전히 완전히 복제됩니다. 항상 관련 있는 `PlayerStates`의 `ASC`에서 `Attributes`를 복제하는 대신, FNBR은 플레이어의 `Pawn`에 복제된 프록시 구조체를 사용합니다. 서버의 `ASC`에서 `Attributes`가 변경되면 프록시 구조체에서도 변경됩니다. 클라이언트는 프록시 구조체에서 복제된 `Attributes`를 수신하고 변경 사항을 로컬 `ASC`에 다시 푸시합니다. 이를 통해 `Attribute` 복제가 `Pawn`의 관련성 및 `NetUpdateFrequency`를 사용할 수 있습니다. 이 프록시 구조체는 또한 비트마스크로 소수의 화이트리스트에 등록된 `GameplayTags`를 복제합니다. 이 최적화는 네트워크를 통한 데이터 양을 줄이고 폰 관련성(pawn relevancy)의 이점을 활용할 수 있도록 합니다. AI 제어 `Pawns`는 `Pawn`에 `ASC`를 가지고 있으며 이미 관련성을 사용하므로 이 최적화는 필요하지 않습니다.

> (Replication Graph 등) 이후 수행된 다른 서버 측 최적화와 함께 여전히 필요한지는 확실하지 않으며, 가장 유지보수하기 쉬운 패턴은 아닙니다.

*에픽의 Dave Ratti의 [커뮤니티 질문 #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)에 대한 답변*

<a name="optimizations-asclazyloading"></a>
### 7.5 ASC 지연 로딩 (ASC Lazy Loading)
Fortnite Battle Royale (FNBR) 월드에는 많은 피해를 입을 수 있는 `AActors` (나무, 건물 등)가 있으며, 각각 [`ASC`](#concepts-asc)를 가지고 있습니다. 이는 메모리 비용을 증가시킬 수 있습니다. FNBR은 `ASC`가 필요할 때만 (플레이어에게 처음 피해를 입을 때) `ASC`를 지연 로딩하여 이를 최적화합니다. 이는 일부 `AActors`가 매치에서 전혀 피해를 입지 않을 수 있으므로 전체 메모리 사용량을 줄입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="qol"></a>
## 8. 편의성 제안 (Quality of Life Suggestions)

<a name="qol-gameplayeffectcontainers"></a>
### 8.1 게임플레이 이펙트 컨테이너 (Gameplay Effect Containers)
[GameplayEffectContainers](#concepts-ge-containers)는 [`GameplayEffectSpecs`](#concepts-ge-spec), [`TargetData`](#concepts-targeting-data), [간단한 타겟팅](#concepts-targeting-containers), 그리고 관련 기능을 사용하기 쉬운 구조체로 결합합니다. 이는 `GameplayEffectSpecs`를 어빌리티에서 생성된 발사체로 전송하여 나중에 충돌 시 적용하는 데 좋습니다.

<a name="qol-asynctasksascdelegates"></a>
### 8.2 ASC 델리게이트에 바인딩하기 위한 블루프린트 비동기 태스크 (Blueprint AsyncTasks to Bind to ASC Delegates)
디자이너 친화적인 반복 시간을 늘리기 위해, 특히 UI용 UMG 위젯을 설계할 때, UMG 블루프린트 그래프에서 직접 `ASC`의 일반적인 변경 델리게이트에 바인딩하는 블루프린트 AsyncTasks (C++로)를 생성하십시오. 유일한 단점은 위젯이 파괴될 때처럼 수동으로 파괴해야 한다는 것입니다. 그렇지 않으면 메모리에 영원히 남아있을 것입니다. 샘플 프로젝트에는 세 가지 블루프린트 AsyncTasks가 포함되어 있습니다.

`Attribute` 변경 사항 감지:

![Listen for Attributes Changes BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

재사용 대기시간 변경 감지:

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

`GE` 스택 변경 감지:

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting"></a>
## 9. 문제 해결 (Troubleshooting)

<a name="troubleshooting-notlocal"></a>
### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
클라이언트에서 [`ASC`를 초기화](#concepts-asc-setup)해야 합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting-scriptstructcache"></a>
### 9.2 `ScriptStructCache` 오류
[`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)를 호출해야 합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting-replicatinganimmontages"></a>
### 9.3 애니메이션 몽타주가 클라이언트에 복제되지 않음
[GameplayAbilities](#concepts-ga)에서 `PlayMontage` 노드 대신 `PlayMontageAndWait` 블루프린트 노드를 사용하고 있는지 확인하십시오. 이 [AbilityTask](#concepts-at)는 `ASC`를 통해 몽타주를 자동으로 복제하지만, `PlayMontage` 노드는 그렇지 않습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting-duplicatingblueprintactors"></a>
### 9.4 블루프린트 액터 복제 시 AttributeSets가 nullptr로 설정됨
언리얼 엔진에 [버그](https://issues.unrealengine.com/issue/UE-81109)가 있어 기존 블루프린트 액터 클래스에서 복제된 블루프린트 액터 클래스의 `AttributeSet` 포인터를 `nullptr`로 설정합니다. 이를 해결하는 몇 가지 방법이 있습니다. 저는 클래스에 맞춤형 `AttributeSet` 포인터를 만들지 않고(헤더에 포인터 없음, 생성자에서 `CreateDefaultSubobject` 호출 안 함), 대신 `PostInitializeComponents()`에서 `AttributeSets`를 `ASC`에 직접 추가하는 방식으로 성공했습니다(샘플 프로젝트에는 표시되지 않음). 복제된 `AttributeSets`는 여전히 `ASC`의 `SpawnedAttributes` 배열에 존재할 것입니다. 다음과 같이 보일 것입니다:

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... 가지고 있을 수 있는 다른 모든 AttributeSets
	}
}
```

이 시나리오에서는 [`AttributeSet`에 매크로로 만든 함수를 호출](#concepts-as-attributes)하는 대신 `ASC`의 함수를 사용하여 `AttributeSet`의 값을 읽고 설정할 것입니다.

```c++
/** 어트리뷰트의 현재 (최종) 값을 반환합니다. */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** 어트리뷰트의 기본 값을 설정합니다. 기존 활성 수정자는 지워지지 않으며 새 기본 값에 따라 작동합니다. */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

따라서 `GetHealth()`는 다음과 같이 보일 것입니다:

```c++
float AGDPlayerState::GetHealth() const
{
	if (AbilitySystemComponent)
	{
		return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
	}

	return 0.0f;
}
```

체력 `Attribute`를 설정(초기화)하는 방법은 다음과 같습니다:

```c++
const float NewHealth = 100.0f;
if (AbilitySystemComponent)
{
	AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}
```

다시 한 번 강조하지만, `ASC`는 `AttributeSet` 클래스당 최대 하나의 `AttributeSet` 객체만 예상합니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting-unresolvedexternalsymbolmarkpropertydirty"></a>
### 9.5 해결되지 않은 외부 심볼 UEPushModelPrivate::MarkPropertyDirty(int,int)

다음과 같은 컴파일러 오류가 발생하는 경우:

```
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)
```

이는 `FFastArraySerializer`에서 `MarkItemDirty()`를 호출하려고 할 때 발생합니다. 저는 쿨다운 지속 시간을 업데이트할 때와 같이 `ActiveGameplayEffect`를 업데이트할 때 이러한 상황에 직면했습니다.

```c++
ActiveGameplayEffects.MarkItemDirty(*AGE);
```

무슨 일이 일어나고 있냐면, `WITH_PUSH_MODEL`이 여러 곳에서 정의되고 있다는 것입니다. `PushModelMacros.h`는 이를 0으로 정의하고 있지만, 여러 곳에서는 1로 정의되어 있습니다. `PushModel.h`는 이를 1로 보고 있지만, `PushModel.cpp`는 이를 0으로 보고 있습니다.

해결책은 프로젝트의 `Build.cs`에 `NetCore`를 `PublicDependencyModuleNames`에 추가하는 것입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="troubleshooting-enumnamesarenowpathnames"></a>
### 9.6 열거형 이름이 이제 경로 이름으로 표시됩니다 (Enum names are now represented by path name)

다음과 같은 컴파일러 경고가 발생하는 경우:

```
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.
```

UE 5.1에서는 `BindAbilityActivationToInputComponent()`의 생성자에서 `FString`을 사용하는 것이 더 이상 사용되지 않습니다. 대신 `FTopLevelAssetPath`를 전달해야 합니다.

이전의 사용되지 않는 방법:
```c++
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), FString("EGDAbilityInputID"), static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

새로운 방법:
```c++
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

자세한 내용은 `Engine\Source\Runtime\CoreUObject\Public\UObject\TopLevelAssetPath.h`를 참조하십시오.

**[⬆ 맨 위로](#table-of-contents)**

<a name="acronyms"></a>
## 10. 일반적인 GAS 약어 (Common GAS Acronyms)

| 이름 | 약어 |
|---|---|
| AbilitySystemComponent | ASC |
| AbilityTask | AT |
| [Epic의 액션 RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) | ARPG, ARPG Sample |
| CharacterMovementComponent | CMC |
| GameplayAbility | GA |
| GameplayAbilitySystem | GAS |
| GameplayCue | GC |
| GameplayEffect | GE |
| GameplayEffectExecutionCalculation | ExecCalc, Execution |
| GameplayTag | Tag, GT |
| ModifierMagnitudeCalculation | ModMagCalc, MMC |

**[⬆ 맨 위로](#table-of-contents)**

<a name="resources"></a>
## 11. 기타 자료 (Other Resources)
* [공식 문서](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 소스 코드!
   * 특히 `GameplayPrediction.h`
* [에픽의 Lyra 샘플 프로젝트](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [에픽의 액션 RPG 샘플 프로젝트](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/)에는 GAS 전용 텍스트 채널이 있습니다. `#gameplay-ability-system`
   * 고정된 메시지를 확인하세요
* [Dan 'Pan'의 GitHub 자료 저장소](https://github.com/Pantong51/GASContent)
* [SabreDartStudios의 유튜브 비디오](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

<a name="resources-daveratti"></a>
### 11.1 에픽 게임즈의 Dave Ratti와의 Q&A

<a name="resources-daveratti-community1"></a>
#### 11.1.1 커뮤니티 질문 1 (Community Questions 1)
[Dave Ratti가 Unreal Slackers Discord 서버의 GAS 관련 커뮤니티 질문에 답변](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89):

1. `GameplayAbilities` 외부에서 또는 그와 무관하게 필요할 때 스코프 예측 창을 어떻게 만들 수 있을까요? 예를 들어, 발사 후 잊어버리는 발사체가 적에게 명중했을 때 피해 `GameplayEffect`를 로컬로 예측하는 방법은 무엇인가요?

> PredictionKey 시스템은 이런 용도로 설계되지 않았습니다. 근본적으로 이 시스템은 클라이언트가 예측 동작을 시작하고, 서버에 키와 함께 이를 알리고, 클라이언트와 서버 모두 동일한 작업을 실행하며 예측 부작용을 주어진 예측 키와 연관시키는 방식으로 작동합니다. 예를 들어, "나는 어빌리티를 예측적으로 활성화하고 있습니다" 또는 "나는 타겟 데이터를 생성했으며 WaitTargetData 태스크 이후의 어빌리티 그래프 부분을 예측적으로 실행할 것입니다".
>
> 이 패턴을 사용하면 PredictionKey가 서버에서 "반사되어" `UAbilitySystemComponent::ReplicatedPredictionKeyMap` (복제된 속성)을 통해 클라이언트로 돌아옵니다. 키가 서버에서 다시 복제되면 클라이언트는 로컬 예측 부작용(GameplayCues, GameplayEffects)을 모두 취소할 수 있습니다. 복제된 버전은 *거기에 있을 것이며*, 만약 그렇지 않다면 그것은 잘못된 예측이었습니다. 여기에서 예측 부작용을 언제 되돌릴지 정확히 아는 것이 중요합니다. 너무 빠르면 간극이 생기고, 너무 늦으면 "두 배"가 될 것입니다. (이는 루핑 GameplayCue 또는 지속 시간 기반 Gameplay Effect와 같은 상태 기반 예측을 의미합니다. "버스트" GameplayCues 및 즉시 Gameplay Effects는 결코 "취소"되거나 롤백되지 않습니다. 예측 키가 연결되어 있다면 클라이언트에서 단순히 건너뛰어집니다.)
>
> 이 점을 더욱 명확히 하자면, 예측 동작은 서버가 자체적으로 수행하지 않고, 클라이언트가 지시했을 때만 수행한다는 것이 중요합니다. 따라서 "필요할 때 키를 생성하고 서버에 알려 무언가를 실행하도록 하는" 일반적인 방법은 "무언가"가 클라이언트의 지시를 받았을 때만 서버가 수행하는 것이 아닌 한 작동하지 않습니다.
>
> 원래 질문으로 돌아가서: 발사 후 잊어버리는 발사체와 같은 것. Paragon과 Fortnite 모두 GameplayCues를 사용하는 발사체 액터 클래스를 가지고 있습니다. 그러나 우리는 Prediction Key 시스템을 사용하여 이들을 수행하지 않습니다. 대신 우리는 Non-Replicated GameplayCues 개념을 가지고 있습니다. 로컬에서만 발동되고 서버에 의해 완전히 건너뛰어지는 GameplayCues입니다. 본질적으로 이들은 모두 `UGameplayCueManager::HandleGameplayCue`에 대한 직접 호출입니다. 이들은 `UAbilitySystemComponent`를 통해 라우팅되지 않으므로 예측 키 검사/조기 반환이 이루어지지 않습니다.
>
> 복제되지 않는 GameplayCues의 단점은 복제되지 않는다는 것입니다. 따라서 이러한 함수를 호출하는 코드 경로가 모두에게 실행되도록 하는 것은 발사체 클래스/블루프린트의 책임입니다. 우리는 큐 시작(BeginPlay에서 호출됨), 폭발, 벽/캐릭터 충돌 등에 대한 큐를 가지고 있습니다.
>
> 이러한 유형의 이벤트는 이미 클라이언트 측에서 생성되므로 복제되지 않는 게임플레이 큐를 호출하는 것은 큰 문제가 아니었습니다. 복잡한 블루프린트는 까다로울 수 있으며, 작성자가 무엇이 어디에서 실행되는지 이해하도록 하는 것이 중요합니다.

2. 로컬에서 예측되는 `GameplayAbility` 내에서 스코프 예측 창을 만들기 위해 `OnlyServerWait`와 함께 `WaitNetSync` `AbilityTask`를 사용할 때, 서버가 예측 키를 가진 RPC를 기다리는 동안 플레이어가 패킷 전송을 지연시켜 `GameplayAbility` 타이밍을 제어함으로써 잠재적으로 부정 행위를 할 수 있을까요? 이것이 Paragon이나 Fortnite에서 문제가 된 적이 있었나요? 만약 그렇다면, 에픽은 이를 해결하기 위해 무엇을 했나요?

> 네, 이것은 타당한 우려입니다. 서버에서 실행되는 모든 어빌리티 블루프린트가 클라이언트 "신호"를 기다리고 있다면, 랙 스위치 유형의 익스플로잇에 잠재적으로 취약합니다.
>
> Paragon은 `UAbilityTask_WaitTargetData`와 유사한 커스텀 타겟팅 태스크를 가지고 있었습니다. 이 태스크에서 우리는 즉시 타겟팅 모드에 대해 클라이언트를 기다리는 타임아웃 또는 "최대 지연"을 가졌습니다. 타겟팅 모드가 사용자 확인(버튼 누르기)을 기다리고 있었다면, 사용자가 시간을 들일 수 있었기 때문에 무시되었습니다. 그러나 즉시 타겟팅을 확인하는 어빌리티의 경우, 우리는 A) 서버 측에서 타겟 데이터를 생성하거나 B) 어빌리티를 취소하기 전에 일정 시간만 기다렸습니다.
>
> 우리는 `WaitNetSync`에 대해서는 그러한 메커니즘을 사용한 적이 없으며, 매우 드물게 사용했습니다.
>
> Fortnite는 이런 것을 사용하지 않는다고 생각합니다. Fortnite의 무기 어빌리티는 특별히 단일 Fortnite 특정 RPC로 배치됩니다. 어빌리티를 활성화하고, 타겟 데이터를 제공하고, 어빌리티를 종료하는 하나의 RPC입니다. 따라서 배틀 로얄에서는 무기 어빌리티가 본질적으로 이것에 취약하지 않습니다.
>
> 제 생각에 이것은 아마도 시스템 전체적으로 해결될 수 있는 문제이지만, 우리가 가까운 시일 내에 직접 변경할 것으로는 보이지 않습니다. 당신이 언급한 경우를 위해 `WaitNetSync`에 최대 지연을 포함하도록 수정하는 것은 아마도 합리적인 작업일 것이지만, 다시 말하지만, 가까운 미래에 우리 측에서 이 작업을 수행할 가능성은 낮습니다.

3. Paragon과 Fortnite는 어떤 `EGameplayEffectReplicationMode`를 사용했으며, 에픽은 각 모드를 언제 사용해야 하는지에 대해 어떤 권장 사항을 가지고 있나요?

> 두 게임 모두 플레이어 제어 캐릭터에는 본질적으로 혼합 모드(Mixed mode)를 사용하고, AI 제어 캐릭터(AI 미니언, 정글 크립, AI 허스크 등)에는 최소 모드(Minimal)를 사용합니다. 저는 멀티플레이어 게임에서 시스템을 사용하는 대부분의 사람들에게 이 방법을 추천합니다. 이 설정을 프로젝트 초기에 할수록 좋습니다.
>
> Fortnite는 최적화를 위해 몇 단계를 더 나아갑니다. 시뮬레이션된 프록시의 경우 `UAbilitySystemComponent`를 전혀 복제하지 않습니다. 소유 Fortnite 플레이어 상태 클래스의 `::ReplicateSubobjects()` 내부에서 컴포넌트 및 어트리뷰트 서브객체는 건너뛰어집니다. 우리는 어빌리티 시스템 컴포넌트에서 최소한의 복제된 데이터를 폰 자체의 구조체로 푸시합니다(기본적으로 어트리뷰트 값의 하위 집합과 비트마스크로 복제되는 태그의 화이트리스트 하위 집합). 우리는 이것을 "프록시"라고 부릅니다. 수신 측에서는 폰에 복제된 프록시 데이터를 가져와 플레이어 상태의 어빌리티 시스템 컴포넌트에 다시 푸시합니다. 따라서 FNBR의 각 플레이어에 대해 ASC가 있지만, 직접 복제되지 않습니다. 대신 폰의 최소 프록시 구조체를 통해 데이터를 복제하고 수신 측에서 ASC로 다시 라우팅합니다. 이것은 A) 더 적은 데이터 집합 B) 폰 관련성을 활용한다는 점에서 이점이 있습니다.
>
> (Replication Graph 등) 이후 수행된 다른 서버 측 최적화와 함께 여전히 필요한지는 확실하지 않으며, 가장 유지보수하기 쉬운 패턴은 아닙니다.

4. `GameplayPrediction.h`에 따라 `GameplayEffects` 제거를 예측할 수 없는데, `GameplayEffects` 제거 시 지연의 영향을 완화하기 위한 전략이 있나요? 예를 들어, 이동 속도 감소를 제거할 때 현재는 서버가 `GameplayEffect` 제거를 복제할 때까지 기다려야 하므로 플레이어 캐릭터 위치가 튀는 현상이 발생합니다.

> 이것은 어려운 문제이며 좋은 답변이 없습니다. 우리는 일반적으로 허용 오차와 스무딩을 통해 이러한 문제를 피했습니다. 어빌리티 시스템과 캐릭터 이동 시스템 간의 정밀한 동기화가 좋지 않은 상태이며 우리가 해결하고자 하는 문제라는 점에 전적으로 동의합니다.
>
> 저는 `GEs`의 예측 제거를 허용하는 방안을 고려했지만, 계속 진행해야 했기 때문에 모든 예외 케이스를 해결할 수 없었습니다. 그러나 캐릭터 이동 시스템은 어빌리티 시스템과 가능한 이동 속도 수정자 등에 대해 아무것도 모르는 내부 저장된 이동 버퍼를 여전히 가지고 있기 때문에 이것이 모든 것을 해결하지는 못합니다. `GEs` 제거를 예측할 수 없는 경우에도 수정 피드백 루프에 빠질 가능성이 여전히 있습니다.
>
> 정말 절망적인 상황이라고 생각한다면, 이동 속도 `GEs`를 억제하는 `GE`를 예측적으로 추가할 수 있습니다. 저는 직접 해본 적은 없지만 이전에 이론화해본 적이 있습니다. 특정 종류의 문제를 해결하는 데 도움이 될 수 있습니다.

5. Paragon과 Fortnite에서는 `AbilitySystemComponent`가 `PlayerState`에 존재하고, Action RPG Sample에서는 `Character`에 존재한다는 것을 알고 있습니다. `AbilitySystemComponent`가 어디에 존재해야 하는지, 그리고 그 `Owner`는 무엇이어야 하는지에 대한 에픽의 내부 규칙, 가이드라인 또는 권장 사항이 있나요?

> 일반적으로 리스폰할 필요가 없는 것은 Owner와 Avatar 액터가 동일해야 한다고 말할 수 있습니다. AI 적, 건물, 월드 소품 등이 이에 해당합니다.
>
> 리스폰하는 모든 것은 Owner와 Avatar가 달라야 합니다. 그래야 리스폰 후 어빌리티 시스템 컴포넌트를 저장/재생성/복원할 필요가 없습니다. PlayerState는 모든 클라이언트에 복제되므로(PlayerController는 그렇지 않음) 논리적인 선택입니다. 단점은 PlayerStates가 항상 관련성이 있으므로 100인 플레이어 게임에서 문제가 발생할 수 있다는 것입니다(질문 #3에서 FN이 한 작업에 대한 참고 사항 참조).

6. 동일한 소유자(Owner)를 가지지만 다른 아바타(Avatar)를 가지는 여러 `AbilitySystemComponents`를 (예: `PlayerState`로 설정된 `Owner`를 가진 폰 및 무기/아이템/발사체에) 가지는 것이 가능한가요?

> 제가 보기에 첫 번째 문제는 소유 액터에 IGameplayTagAssetInterface와 IAbilitySystemInterface를 구현하는 것입니다. 전자는 가능할 수도 있습니다: 모든 ASC에서 태그를 단순히 집계하면 됩니다 (하지만 조심하세요. HasAllMatchingGameplayTags는 교차 ASC 집계를 통해서만 충족될 수 있습니다. 각 ASC에 호출을 전달하고 결과를 OR하는 것만으로는 충분하지 않습니다). 하지만 후자는 훨씬 더 까다롭습니다: 어떤 ASC가 권위 있는 ASC인가요? 누군가 GE를 적용하고 싶다면 어떤 ASC가 받아야 할까요? 아마 이런 문제를 해결할 수 있겠지만, 이 문제의 측면이 가장 어려울 것입니다: 여러 ASC를 소유한 Owner가 있을 것입니다.
>
> 폰과 무기에 별도의 ASC를 가지는 것은 자체적으로 의미가 있을 수 있습니다. 예를 들어, 무기를 설명하는 태그와 소유한 폰을 설명하는 태그를 구분하는 것입니다. 아마도 무기에 부여된 태그가 Owner에게도 "적용"되고 다른 것에는 적용되지 않는 것이 합리적일 수 있습니다 (예: 어트리뷰트와 GE는 독립적이지만 Owner는 위에서 설명한 대로 소유 태그를 집계할 것입니다). 이것은 확실히 작동할 수 있습니다. 하지만 동일한 Owner를 가진 여러 ASC를 가지는 것은 위험할 수 있습니다.

7. 서버가 소유 클라이언트에서 로컬로 예측된 어빌리티의 쿨다운 지속 시간을 덮어쓰는 것을 막는 방법이 있나요? 높은 지연 시간 시나리오에서, 이는 소유 클라이언트가 로컬 쿨다운이 만료되었을 때 어빌리티를 다시 활성화하려고 "시도"하게 하지만, 서버에서는 여전히 쿨다운 중입니다. 소유 클라이언트의 활성화 요청이 네트워크를 통해 서버에 도달할 때쯤이면, 서버는 쿨다운이 끝났을 수도 있고, 서버가 남아 있는 밀리초 동안 활성화 요청을 대기열에 넣을 수 있을지도 모릅니다. 그렇지 않으면 현재로서는 지연 시간이 긴 클라이언트가 지연 시간이 짧은 클라이언트보다 어빌리티를 다시 활성화하는 데 더 긴 지연 시간을 가집니다. 이는 쿨다운이 1초 미만인 기본 공격과 같이 쿨다운이 매우 짧은 어빌리티에서 가장 분명합니다. 로컬로 예측된 어빌리티의 쿨다운 지속 시간을 서버가 덮어쓰는 것을 막는 방법이 없다면, 어빌리티 재활성화 시 높은 지연 시간의 영향을 완화하기 위한 에픽의 전략은 무엇인가요? 다른 예시 기반 방식으로 설명하자면, 에픽은 Paragon의 기본 공격 및 기타 어빌리티를 어떻게 설계하여 높은 지연 시간 플레이어라도 낮은 지연 시간 플레이어와 동일한 속도로 로컬 예측으로 공격하거나 활성화할 수 있었나요?

> 짧게 말하면, 이를 막을 방법은 없으며 Paragon은 분명히 이 문제를 겪었습니다. 지연 시간이 긴 연결은 기본 공격 시 ROF(발사율)가 낮았습니다.
>
> 저는 GE 지속 시간을 계산할 때 지연 시간을 고려하는 "GE 조정"을 추가하여 이 문제를 해결하려고 시도했습니다. 본질적으로 서버가 총 GE 시간의 일부를 소모하도록 허용하여 GE 클라이언트 측의 유효 시간이 어떤 양의 지연 시간에도 100% 일관되도록 하는 것입니다 (물론 변동은 여전히 문제를 일으킬 수 있습니다). 그러나 저는 이것을 출시할 수 있는 상태로 만들지 못했고 프로젝트가 빠르게 진행되어 이 문제를 완전히 해결하지 못했습니다.
>
> Fortnite는 무기 발사 속도에 대한 자체 장부 관리를 수행합니다. 무기 쿨다운에 GE를 사용하지 않습니다. 이것이 게임에 중요한 문제라면 이 방법을 권장합니다.

8. 게임플레이어빌리티시스템(GameplayAbilitySystem) 플러그인에 대한 에픽의 로드맵은 무엇인가요? 에픽은 2019년 이후에 어떤 기능을 추가할 계획인가요?

> 전반적으로 시스템은 이 시점에서 매우 안정적이라고 생각하며, 주요 신기능 개발에 참여하는 인원은 없습니다. Fortnite 또는 UDN/풀 리퀘스트로 인해 버그 수정 및 작은 개선 사항이 가끔 이루어지지만, 현재로서는 그게 전부입니다.
>
> 장기적으로는 결국 "V2" 또는 큰 변화를 만들 것이라고 생각합니다. 이 시스템을 작성하면서 많은 것을 배우고, 많은 것이 옳았고 많은 것이 틀렸다고 느낍니다. 저는 그러한 실수를 바로잡고 위에서 지적된 치명적인 결함 중 일부를 개선할 기회를 갖고 싶습니다.
>
> 만약 V2가 나온다면, 업그레이드 경로를 제공하는 것이 가장 중요할 것입니다. 우리는 V2를 만들고 Fortnite를 영원히 V1에 두지 않을 것입니다. 가능한 한 많은 부분을 자동으로 마이그레이션하는 경로 또는 절차가 있을 것이며, 일부 수동 재작업이 거의 확실하게 필요할 것입니다.
>
> 우선 순위가 높은 수정 사항은 다음과 같습니다:
> * 캐릭터 이동 시스템과의 더 나은 상호 운용성. 클라이언트 예측 통합.
> * GE 제거 예측 (질문 #4)
> * GE 지연 조정 (질문 #7)
> * RPC 배치 및 프록시 구조와 같은 일반화된 네트워크 최적화. 주로 Fortnite를 위해 수행한 작업이지만, 게임에서 자체적으로 게임별 최적화를 더 쉽게 작성할 수 있도록 더 일반화된 형태로 분해하는 방법을 찾는 것입니다.
>
> 고려할 수 있는 더 일반적인 리팩토링 유형의 변경 사항:
> * GE가 스프레드시트 값을 직접 참조하는 방식에서 근본적으로 벗어나, 대신 매개변수를 방출할 수 있고 그 매개변수는 스프레드시트 값에 바인딩된 일부 상위 수준 객체에 의해 채워질 수 있도록 하고 싶습니다. 현재 모델의 문제는 GE가 커브 테이블 행과의 긴밀한 결합으로 인해 공유할 수 없게 된다는 것입니다. 매개변수화를 위한 일반화된 시스템이 작성되어 V2 시스템의 기반이 될 수 있다고 생각합니다.
> * `UGameplayAbility`의 "정책" 수를 줄이고 싶습니다. `ReplicationPolicy`와 `InstancingPolicy`를 제거할 것입니다. 복제는 제 생각에 실제로는 거의 필요하지 않으며 혼란을 야기합니다. `InstancingPolicy`는 `FGameplayAbilitySpec`을 서브클래싱할 수 있는 `UObject`로 만들어서 대체해야 합니다. 이것이 이벤트가 있고 블루프린트화할 수 있는 "인스턴스화되지 않은 어빌리티 객체"여야 했습니다. `UGameplayAbility`는 "실행별 인스턴스화된" 객체여야 합니다. 실제로 인스턴스화해야 하는 경우 선택 사항일 수 있습니다. 대신 "인스턴스화되지 않은" 어빌리티는 새로운 `UGameplayAbilitySpec` 객체를 통해 구현될 것입니다.
> * 시스템은 "필터링된 GE 적용 컨테이너"(더 높은 수준의 게임플레이 로직으로 어떤 GE를 어떤 액터에 적용할지 데이터 기반으로 결정), "오버랩 볼륨 지원"(충돌 프리미티브 오버랩 이벤트에 기반하여 "필터링된 GE 적용 컨테이너" 적용) 등과 같은 더 "중간 수준"의 구조를 제공해야 합니다. 이는 모든 프로젝트가 자체 방식으로 결국 구현하게 되는 빌딩 블록입니다. 이를 올바르게 구현하는 것은 간단하지 않으므로, 몇 가지 기본 구현을 제공하는 데 더 잘해야 한다고 생각합니다.
> * 일반적으로 프로젝트를 시작하고 실행하는 데 필요한 상용구 코드를 줄이는 것입니다. 수동 어빌리티 또는 기본 히트스캔 무기와 같은 기능을 기본적으로 제공할 수 있는 별도의 "Ex 라이브러리" 모듈 등이 있을 수 있습니다. 이 모듈은 선택 사항이지만 빠르게 시작하고 실행할 수 있도록 할 것입니다.
> * `GameplayCues`를 어빌리티 시스템과 결합되지 않은 별도의 모듈로 이동하고 싶습니다. 여기에서 많은 개선이 이루어질 수 있다고 생각합니다.

> 이것은 저의 개인적인 의견일 뿐이며, 누구의 약속도 아닙니다. 가장 현실적인 행동 방침은 새로운 엔진 기술 이니셔티브가 생겨날 때 어빌리티 시스템을 업데이트해야 할 것이고, 그때 이러한 종류의 작업을 수행할 시기가 될 것이라고 생각합니다. 이러한 이니셔티브는 스크립팅, 네트워킹 또는 물리/캐릭터 이동과 관련될 수 있습니다. 그러나 이 모든 것은 매우 멀리 내다보는 것이므로, 시기나 추정치에 대한 약속을 드릴 수는 없습니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="resources-daveratti-community2"></a>
#### 11.1.2 커뮤니티 질문 2 (Community Questions 2)
커뮤니티 회원 [iniside](https://github.com/iniside)와 Dave Ratti의 Q&A:

1. 디커플된 고정 틱 지원이 계획되어 있나요? 저는 게임 스레드가 고정(예: 30/60fps)되고 렌더링 스레드가 자유롭게 실행되도록 하고 싶습니다. 이것이 미래에 기대해야 할 것인지 아닌지 묻는 이유는 게임플레이가 어떻게 작동해야 하는지에 대한 몇 가지 가정을 하기 위함입니다.
주로 물리 엔진에 이제 고정 비동기 틱이 있기 때문에, 나머지 시스템이 미래에 어떻게 작동할지에 대한 질문을 던지게 됩니다. 엔진의 나머지 부분의 틱 레이트를 고정하지 않고 게임 스레드에 고정 틱을 가질 수 있는 기능이 엄청나게 좋을 것이라는 사실을 숨기지 않겠습니다.

> 렌더링 프레임 속도와 게임 스레드 틱 프레임 속도를 분리할 계획은 없습니다. 이러한 시스템의 복잡성과 이전 엔진 버전과의 하위 호환성을 유지해야 하는 요구 사항으로 인해 이러한 일이 일어날 가능성은 이미 지났다고 생각합니다.
>
> 대신, 우리가 나아간 방향은 게임 스레드와 독립적으로 고정 틱 속도로 실행되는 비동기 "물리 스레드"를 갖는 것입니다. 고정 속도로 실행되어야 하는 것들은 여기서 실행될 수 있고, 게임 스레드/렌더링은 항상 그랬던 것처럼 작동할 수 있습니다.
>
> 네트워크 예측은 독립 틱(Independent Ticking) 모드와 고정 틱(Fixed Ticking) 모드를 지원한다는 점을 명확히 할 가치가 있습니다. 제 장기적인 계획은 독립 틱을 현재 네트워크 예측에서와 거의 동일하게 유지하여 가변 프레임 속도로 게임 스레드에서 실행하고 "그룹/월드" 예측이 없으며, 단지 고전적인 "클라이언트가 자신의 폰과 소유 액터를 예측하는" 모델만 있는 것입니다. 그리고 고정 틱은 비동기 물리 기능을 사용하고 물리 객체 및 다른 클라이언트/폰/차량 등과 같이 클라이언트가 제어/소유하지 않는 액터를 예측할 수 있도록 할 것입니다.

2. 어빌리티 시스템과의 네트워크 예측 통합은 어떻게 될 계획인가요? 예를 들어, 고정 프레임 어빌리티 활성화 (예측 키 대신 어빌리티가 활성화되고 태스크가 실행된 프레임을 서버가 얻도록)?

> 네, 어빌리티 시스템의 예측 키를 다시 작성/제거하고 네트워크 예측 구조로 대체할 계획입니다. NetworkPredictionExtras의 MockAbility 예시는 이것이 어떻게 작동할 수 있는지 보여주지만, GAS가 요구하는 것보다 더 "하드코딩"되어 있습니다.
>
> 주요 아이디어는 ASC의 RPC에서 명시적인 클라이언트->서버 예측 키 교환을 제거하는 것입니다. 더 이상 예측 창이나 스코프 예측 키는 없을 것입니다. 대신 모든 것이 네트워크 예측 프레임을 중심으로 고정될 것입니다. 중요한 것은 클라이언트와 서버가 일이 언제 일어나는지에 동의하는 것입니다. 예시로는 다음과 같습니다:
>
> * 어빌리티가 활성화/종료/취소된 시점
> * 게임플레이 효과가 적용/제거된 시점
> * 어트리뷰트 값 (프레임 X에서의 어트리뷰트 값)
>
> 저는 이것이 어빌리티 시스템 수준에서 일반적으로 수행될 수 있다고 생각합니다. 하지만 `UGameplayAbility` 내의 사용자 정의 로직을 완전히 롤백 가능하게 만드는 것은 여전히 더 많은 작업이 필요할 것입니다. 우리는 결국 완전히 롤백 가능하고 더 제한된 기능 세트 또는 롤백 친화적으로 표시된 어빌리티 태스크에만 접근할 수 있는 `UGameplayAbility`의 서브클래스를 가지게 될 수도 있습니다. 그런 식입니다. 애니메이션 이벤트와 루트 모션, 그리고 이들이 처리되는 방식에도 많은 의미가 있습니다.
>
> 더 명확한 답변을 드리고 싶지만, GAS를 다시 건드리기 전에 기반을 올바르게 구축하는 것이 정말 중요합니다. 이동 및 물리가 견고해야 상위 수준 시스템을 변경할 수 있습니다.

3. 네트워크 예측 개발을 메인 브랜치로 옮길 계획이 있나요? 솔직히 말해서, 최신 코드를 확인하고 싶습니다. 상태와 관계없이요.

> 우리는 그 방향으로 노력하고 있습니다. 시스템 작업은 여전히 NetworkPrediction (NetworkPhysics.h 참조)에서 진행 중이며, 기본 비동기 물리 기능은 모두 사용 가능해야 합니다 (RewindData.h 등). 그러나 우리는 Fortnite에서 공개할 수 없는 특정 사용 사례에 집중하고 있었습니다. 버그 수정, 성능 최적화 등을 진행하고 있습니다.
>
> 더 자세한 설명을 드리자면, 이 시스템의 초기 버전을 작업할 때 우리는 "프론트 엔드"에 매우 집중했습니다. 즉, 상태와 시뮬레이션이 어떻게 정의되고 작성되는지에 대해 말입니다. 거기서 많은 것을 배웠습니다. 그러나 비동기 물리 기능이 온라인화되면서, 우리는 초기 추상화 중 일부를 버리는 대신 이 시스템에서 실제로 작동하는 것을 만드는 데 훨씬 더 집중했습니다. 여기서의 목표는 실제가 작동할 때 다시 돌아와서 모든 것을 통합하는 것입니다. 예를 들어, "프론트 엔드"로 돌아가서 우리가 현재 작업 중인 핵심 기술 위에 최종 버전을 만드는 것입니다.

4. 한동안 메인 브랜치에는 게임플레이 메시지 (이벤트/메시지 버스처럼 보였습니다)를 보내는 플러그인이 있었지만, 제거되었습니다. 복원할 계획이 있나요? Game Features/Modular Gameplay 플러그인과 함께, 일반적인 이벤트 버스 디스패처는 매우 유용할 것입니다.

> GameplayMessages 플러그인을 말씀하시는 것 같습니다. 이것은 아마도 언젠가 다시 돌아올 것입니다. API가 아직 완전히 확정되지 않았고 작성자가 아직 공개할 의도가 없었습니다. 모듈형 게임플레이 디자인에 유용할 것이라는 점에는 동의합니다. 하지만 제 분야가 아니므로 더 많은 정보는 없습니다.

5. 저는 최근 비동기 고정 물리(async fixed physics)로 작업하고 있으며 결과는 유망합니다. 하지만 미래에 NP 업데이트가 있다면, 저는 아마도 그냥 기다릴 것입니다. 왜냐하면 작동시키려면 여전히 전체 엔진을 고정 틱으로 만들어야 하고, 한편으로는 물리를 33ms로 유지하려고 하기 때문입니다. 모든 것이 30fps일 때는 좋은 경험이 되지 않습니다. (:)

Async CharacterMovementComponent에 대한 일부 작업이 있었던 것을 보았지만, 이것이 네트워크 예측을 사용할 것인지, 아니면 별개의 노력인지 확실하지 않습니다.

그것을 알아차린 후, 저는 또한 고정 틱 속도로 나만의 비동기 이동을 구현하려고 시도했는데, 잘 작동했습니다. 하지만 그 위에 보간을 위한 별도의 업데이트를 추가해야 했습니다. 설정은 별도의 워커 스레드에서 고정 33ms 업데이트로 시뮬레이션 틱을 실행하고, 계산을 수행하고, 결과를 저장하고, 게임 스레드에서 현재 프레임 속도에 맞춰 보간하는 것이었습니다. 완벽하지는 않았지만, 작동했습니다.

제 질문은, 이것이 미래에 더 쉽게 설정될 수 있는 것인지입니다. 왜냐하면 작성해야 할 상용구 코드(보간 부분)가 상당히 많고, 각 이동 객체를 개별적으로 보간하는 것이 특히 효율적이지 않기 때문입니다.

비동기적인 것들은 정말 흥미롭습니다. 왜냐하면 고정 업데이트 속도로 게임 시뮬레이션을 실제로 실행할 수 있게 하여 (고정 스레드가 필요 없게 만들고) 더 예측 가능한 결과를 얻을 수 있기 때문입니다. 이것이 앞으로 의도하는 것인지, 아니면 특정 시스템에만 이점이 있는 것인지요? 제가 기억하기로는 액터 변환(actor transforms)은 비동기적으로 업데이트되지 않고 블루프린트는 완전히 스레드 안전하지 않습니다. 다시 말해, 이것이 더 프레임워크 수준에서 지원될 계획인지, 아니면 각 게임이 자체적으로 해결해야 할 문제인지요?

> 비동기 캐릭터 이동 컴포넌트 (Async CharacterMovementComponent)
>
> 이것은 기본적으로 CMC를 물리 스레드로 포팅하는 초기 프로토타입/실험입니다. 저는 아직 이것을 CMC의 미래로 보지는 않지만, 그렇게 발전할 수도 있습니다. 현재로서는 네트워킹 지원이 없으므로 제가 실제로 따를 만한 것은 아닙니다. 이것을 하는 사람들은 주로 이 시스템이 추가할 입력 지연 시간과 이를 완화하는 방법에 대해 측정하는 데 관심이 있습니다.
>
> 저는 여전히 전체 엔진을 고정 틱으로 만들어야 하고, 한편으로는 물리를 33ms로 유지하려고 합니다. 모든 것이 30fps일 때는 좋은 경험이 되지 않습니다. (:)
>
> 비동기적인 것들은 정말 흥미롭습니다. 왜냐하면 고정 업데이트 속도로 게임 시뮬레이션을 실제로 실행할 수 있게 하여 (고정 스레드가 필요 없게 만들고)
>
> 네. 여기서 목표는 비동기 물리가 활성화되면 엔진을 가변 틱 속도로 실행하면서 물리 및 "핵심" 게임플레이 시뮬레이션(예: 캐릭터 이동, 차량, GAS 등)은 고정 속도로 실행할 수 있다는 것입니다.
>
> 이것을 활성화하려면 다음 cvars를 설정해야 합니다: (아마 이미 파악하셨을 겁니다)
> `p.DefaultAsyncDt=0.03333`
> `p.RewindCaptureNumFrames=64`
>
> Chaos는 물리 상태에 대한 보간을 제공합니다 (예: `UPrimitiveComponent`로 푸시되고 게임 코드에 표시되는 변환). 이제 `p.AsyncInterpolationMultiplier`라는 cvar가 있어 이를 제어하므로 살펴보실 수 있습니다. 추가 코드를 작성할 필요 없이 물리 바디의 부드럽고 연속적인 움직임을 보게 될 것입니다.
>
> 비물리 상태를 보간하려면 현재로서는 여전히 직접 해야 합니다. 예를 들어, 비동기 물리 스레드에서 업데이트(틱)하고 싶지만 게임 스레드에서 부드럽고 연속적인 보간을 보고 싶어하는 쿨다운이 있어 렌더링 프레임마다 쿨다운 시각화가 업데이트되는 경우입니다. 우리는 결국 이 문제를 해결할 것이지만 아직 예시가 없습니다.
>
> 작성해야 할 상용구 코드가 상당히 많고,
>
> 네, 그래서 그것이 지금까지 시스템의 큰 일반적인 문제였습니다. 우리는 숙련된 프로그래머가 성능과 안전성을 극대화하기 위해 사용할 수 있는 인터페이스를 제공하고 싶습니다 (수많은 위험과 하지 않는 것이 더 나은 것들 없이 예측적으로 "그냥 작동하는" 게임플레이 코드를 작성할 수 있는 능력). 따라서 CharacterMovement와 같은 것은 성능을 극대화하기 위해 많은 커스텀 작업을 수행할 수 있습니다. 예를 들어, 템플릿 코드를 작성하고 배치 업데이트를 수행하고, 광범위하게 작업하고, 업데이트 루프를 별개의 단계로 분할하는 등입니다. 우리는 이러한 사용 사례를 위해 비동기 스레드 및 롤백 시스템에 대한 좋은 "저수준" 인터페이스를 제공하고 싶습니다. 그리고 이 경우에도 캐릭터 이동 시스템 자체가 자체적인 방식으로 확장 가능하다는 것은 여전히 합리적입니다. 예를 들어, 커스텀 이동 모드를 블루프린트화하고 스레드 안전한 블루프린트 API를 제공하는 방법입니다.
>
> 하지만 우리는 이것이 자체적인 "시스템"이 실제로 필요 없는 더 간단한 게임플레이 객체에는 적합하지 않다는 것을 인식합니다. 언리얼과 더 일치하는 것이 필요합니다. 예를 들어, 리플렉션 시스템을 사용하고, 일반적인 블루프린트 지원 등을 하는 것입니다. 다른 스레드에서 블루프린트가 사용되는 예시가 있습니다 (BlueprintThreadSafe 키워드와 애니메이션 시스템이 지향하는 바 참조). 그래서 저는 언젠가 이런 형태가 있을 것이라고 생각합니다. 하지만 다시 말하지만, 우리는 아직 거기에 도달하지 못했습니다.
>
> 보간에 대해서만 물으셨다는 것을 알고 있지만, 이것이 일반적인 답변입니다. 현재로서는 NetSerialize, ShouldReconcile, Interpolate 등 모든 것을 수동으로 해야 하지만, 결국에는 "리플렉션 시스템만 사용하고 싶다면, 이 모든 것을 수동으로 작성할 필요가 없다"는 방식이 있을 것입니다. 우리는 모든 사람에게 리플렉션 시스템을 *강제*하고 싶지는 않습니다. 왜냐하면 그것이 시스템의 가장 낮은 수준에서 우리가 받아들이고 싶지 않은 다른 제한 사항을 부과하기 때문입니다.
>
> 그리고 제가 앞에서 말한 것과 연결하자면, 현재 우리는 몇 가지 매우 구체적인 예시를 작동시키고 성능을 최적화하는 데 정말 집중하고 있으며, 그런 다음에는 프론트 엔드로 돌아가서 모든 사람이 사용하기에 친숙하고 반복하기 쉽게 만들고 상용구 코드를 줄이는 등의 작업에 집중할 것입니다.

**[⬆ 맨 위로](#table-of-contents)**

<a name="changelog"></a>
## 12. GAS 변경 로그 (GAS Changelog)

이것은 공식 언리얼 엔진 업그레이드 변경 로그와 제가 발견한 문서화되지 않은 변경 사항을 종합하여 GAS에 대한 주요 변경 사항(수정, 변경 및 새로운 기능) 목록입니다. 여기에 나열되지 않은 것을 발견했다면, 문제 또는 풀 리퀘스트를 생성해 주십시오.

<a name="changelog-5.3"></a>
### 5.3
* 크래시 수정: 끊김 없는 이동 후 게임플레이 큐를 적용하려 할 때 발생하던 크래시를 수정했습니다.
* 크래시 수정: Live Coding 사용 시 GlobalAbilityTaskCount로 인해 발생하던 크래시를 수정했습니다.
* 크래시 수정: UAbilityTask::OnDestroy가 UAbilityTask_StartAbilityState와 같은 경우에 재귀적으로 호출될 때 크래시가 발생하지 않도록 수정했습니다.
* 버그 수정: 이제 자식 클래스에서 `Super::ActivateAbility`를 안전하게 호출할 수 있습니다. 이전에는 `CommitAbility`를 호출했습니다.
* 버그 수정: 여러 유형의 FGameplayEffectContext를 올바르게 복제하기 위한 지원을 추가했습니다.
* 버그 수정: FGameplayEffectContextHandle은 이제 "액터"를 검색하기 전에 데이터가 유효한지 확인합니다.
* 버그 수정: 게임플레이 어빌리티 시스템은 유효한 PC를 찾았을 경우에만 PC 검색을 중단합니다.
* 버그 수정: RemoveGameplayCue_Internal에서 기본 매개변수 객체 대신 기존 GameplayCueParameters가 존재하면 사용하도록 수정했습니다.
* 버그 수정: GameplayAbilityWorldReticle이 TargetingActor 대신 Source Actor를 향하도록 수정했습니다.
* 버그 수정: GiveAbilityAndActivateOnce로 전달된 트리거 이벤트 데이터와 어빌리티 목록이 잠겨 있을 경우 트리거 이벤트 데이터를 캐시하도록 수정했습니다.
* 버그 수정: FInheritedGameplayTags가 저장할 때까지 기다리지 않고 CombinedTags를 즉시 업데이트하도록 지원을 추가했습니다.
* 버그 수정: ShouldAbilityRespondToEvent를 클라이언트 전용 코드 경로에서 서버와 클라이언트 모두로 이동했습니다.
* 버그 수정: Curve Simplification으로 인해 Cooked Builds에서 FAttributeSetInitterDiscreteLevels가 작동하지 않던 문제를 수정했습니다.
* 버그 수정: GameplayAbility에서 CurrentEventData를 설정했습니다.
* 버그 수정: 콜백을 잠재적으로 실행하기 전에 MinimalReplicationTags가 올바르게 설정되었는지 확인합니다.
* 버그 수정: instanced GameplayAbility에서 ShouldAbilityRespondToEvent가 호출되지 않던 문제를 수정했습니다.
* 버그 수정: Child Actors에서 실행되는 Gameplay Cue Notify Actors가 gc.PendingKill이 비활성화되었을 때 메모리 누수를 일으키지 않도록 수정했습니다.
* 버그 수정: GameplayCueManager에서 GameplayCueNotify_Actors가 해시 충돌로 인해 '손실'될 수 있던 문제를 수정했습니다.
* 버그 수정: WaitGameplayTagQuery는 액터에 Gameplay Tags가 없더라도 Query를 존중하도록 수정했습니다.
* 버그 수정: PostAttributeChange 및 AttributeValueChangeDelegates가 이제 올바른 OldValue를 가지도록 수정했습니다.
* 버그 수정: FGameplayTagQuery가 네이티브 코드로 구조체가 생성되었을 때 적절한 Query Description을 표시하지 않던 문제를 수정했습니다.
* 버그 수정: 어빌리티 시스템이 사용 중인 경우 UAbilitySystemGlobals::InitGlobalData가 호출되도록 보장합니다. 이전에는 사용자가 호출하지 않으면 게임플레이 어빌리티 시스템이 제대로 작동하지 않았습니다.
* 버그 수정: UGameplayAbility::EndAbility에서 애니메이션 레이어 연결/해제 시 발생하는 문제를 수정했습니다.
* 버그 수정: Ability System Component 함수를 업데이트하여 사용하기 전에 Spec의 어빌리티 포인터를 확인하도록 했습니다.
* 신규: 더 복잡한 요구 사항을 지정할 수 있도록 FGameplayTagRequirements에 GameplayTagQuery 필드를 추가했습니다.
* 신규: SourceAggregateTagQuery를 SourceTagQuery를 보강하기 위해 도입했습니다.
* 신규: 콘솔 명령을 통해 게임플레이 어빌리티 및 게임플레이 이펙트를 실행하고 취소하는 기능을 확장했습니다.
* 신규: 게임플레이 어빌리티 블루프린트에 대한 "감사"를 수행하여 개발 방식 및 의도된 사용 방법에 대한 정보를 표시하는 기능을 추가했습니다.
* 변경: OnAvatarSet은 이제 Instanced per Actor Gameplay Abilities의 CDO 대신 기본 인스턴스에서 호출됩니다.
* 변경: 동일한 게임플레이 어빌리티 그래프에서 Activate Ability와 Activate Ability From Event를 모두 허용합니다.
* 변경: AnimTask_PlayMontageAndWait는 이제 BlendOut 이벤트 후 Completed 및 Interrupted를 허용하는 토글을 가집니다.
* 변경: ModMagnitudeCalc 래퍼 함수가 const로 선언되었습니다.
* 변경: FGameplayTagQuery::Matches는 이제 빈 쿼리에 대해 false를 반환합니다.
* 변경: FGameplayAttribute::PostSerialize를 업데이트하여 포함된 어트리뷰트를 검색 가능한 이름으로 표시했습니다.
* 변경: GetAbilitySystemComponent를 업데이트하여 매개변수를 기본적으로 Self로 설정했습니다.
* 변경: AbilityTask_WaitTargetData의 함수들을 virtual로 표시했습니다.
* 변경: 사용되지 않는 FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters 함수를 제거했습니다.
* 변경: 흔적만 남은 GameplayAbility::SetMovementSyncPoint를 제거했습니다.
* 변경: 게임플레이 태스크 및 어빌리티 시스템 컴포넌트에서 사용되지 않는 복제 플래그를 제거했습니다.
* 변경: 일부 게임플레이 이펙트 기능을 선택적 컴포넌트로 이동했습니다. 기존 모든 콘텐츠는 필요한 경우 PostCDOCompiled 중에 자동으로 컴포넌트를 사용하도록 업데이트됩니다.

https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

<a name="changelog-5.2"></a>
### 5.2
* 버그 수정: `UAbilitySystemBlueprintLibrary::MakeSpecHandle` 함수에서 크래시가 발생하던 문제를 수정했습니다.
* 버그 수정: 게임플레이 어빌리티 시스템에서 비제어 폰이 서버에 로컬로 생성된 경우에도(예: 차량) 원격으로 간주되던 로직을 수정했습니다.
* 버그 수정: 서버에 의해 거부된 예측된 인스턴스 어빌리티에 활성화 정보를 올바르게 설정했습니다.
* 버그 수정: GameplayCues가 원격 인스턴스에서 멈추는 버그를 수정했습니다.
* 버그 수정: WaitGameplayEvent에 대한 호출을 연결할 때 메모리 스톰프(memory stomp)가 발생하던 문제를 수정했습니다.
* 버그 수정: 블루프린트에서 AbilitySystemComponent `GetOwnedGameplayTags()` 함수를 호출할 때 동일한 노드가 여러 번 실행되어도 이전 호출의 반환 값이 유지되지 않도록 수정했습니다.
* 버그 수정: 동적 객체에 대한 참조를 복제하는 GameplayEffectContext와 관련된 문제를 수정했습니다. 이 동적 객체는 복제되지 않았습니다.
  * 이로 인해 `bHasMoreUnmappedReferences`가 항상 true가 되어 GameplayEffect가 `Owner->HandleDeferredGameplayCues(this)`를 호출하지 못했습니다.
* 신규: [게임플레이 타겟팅 시스템](https://docs.unrealengine.com/en-US/gameplay-targeting-system-in-unreal-engine/)은 데이터 기반 타겟팅 요청을 생성하는 방법입니다.
* 신규: GameplayTag Queries에 대한 커스텀 직렬화 지원을 추가했습니다.
* 신규: 파생된 FGameplayEffectContext 유형 복제에 대한 지원을 추가했습니다.
* 신규: 에셋의 게임플레이 어트리뷰트가 저장 시 검색 가능한 이름으로 등록되어, 참조 뷰어에서 어트리뷰트에 대한 참조를 볼 수 있습니다.
* 신규: AbilitySystemComponent에 대한 몇 가지 기본 단위 테스트를 추가했습니다.
* 신규: 게임플레이 어빌리티 시스템 어트리뷰트가 이제 핵심 리디렉션(Core Redirects)을 존중합니다. 즉, 이제 코드에서 어트리뷰트 세트와 해당 어트리뷰트의 이름을 변경하고 DefaultEngine.ini에 리디렉트 항목을 추가하여 이전 이름으로 저장된 에셋에서 올바르게 로드할 수 있습니다.
* 변경: 코드에서 게임플레이 이펙트 수정자의 평가 채널을 변경할 수 있도록 허용했습니다.
* 변경: 게임플레이 어빌리티 플러그인에서 이전에 사용되지 않던 `FGameplayModifierInfo::Magnitude` 변수를 제거했습니다.
* 변경: 어빌리티 시스템 컴포넌트와 Smart Object 인스턴스 태그 간의 동기화 로직을 제거했습니다.

https://docs.unrealengine.com/5.2/en-US/unreal-engine-5.2-release-notes/

<a name="changelog-5.1"></a>
### 5.1
* 버그 수정: 복제된 느슨한 게임플레이 태그가 소유자에게 복제되지 않던 문제를 수정했습니다.
* 버그 수정: 어빌리티 태스크 버그를 수정하여 어빌리티가 적시에 가비지 컬렉션되는 것을 방해할 수 있던 문제를 해결했습니다.
* 버그 수정: 태그를 기반으로 활성화되는 게임플레이 어빌리티가 활성화되지 않던 문제를 수정했습니다. 이는 이 태그를 수신하는 게임플레이 어빌리티가 두 개 이상 있고, 목록의 첫 번째 어빌리티가 유효하지 않거나 활성화 권한이 없을 때 발생했습니다.
* 버그 수정: 데이터 레지스트리를 사용하는 GameplayEffects가 로드 시 경고를 표시하던 문제를 수정하고 경고 텍스트를 개선했습니다.
* 버그 수정: `UGameplayAbility`에서 블루프린트 디버거의 브레이크포인트에 마지막으로 인스턴스화된 어빌리티만 잘못 등록하던 코드를 제거했습니다.
* 버그 수정: ApplyGameplayEffectSpecToTarget 내부의 잠금 중에 EndAbility가 호출될 때 게임플레이 어빌리티 시스템 어빌리티가 멈추던 문제를 수정했습니다.
* 신규: 게임플레이 이펙트가 차단된 어빌리티 태그를 추가할 수 있도록 지원을 추가했습니다.
* 신규: WaitGameplayTagQuery 노드를 추가했습니다. 하나는 UAbilityTask를 기반으로 하고 다른 하나는 UAbilityAsync를 기반으로 합니다. 이 노드는 TagQuery를 지정하며, 구성에 따라 쿼리가 true 또는 false가 될 때 출력 핀을 트리거합니다.
* 신규: 콘솔 변수의 어빌리티 태스크 디버깅을 수정하여 비배포 빌드에서 기본적으로 디버그 기록 및 로그 출력을 활성화하도록 했습니다(필요에 따라 핫픽스 온/오프 가능).
* 신규: `AbilitySystem.AbilityTask.Debug.RecordingEnabled`를 0으로 설정하여 비활성화하고, 1로 설정하여 비배포 빌드에서 활성화하며, 2로 설정하여 모든 빌드(배포 빌드 포함)에서 활성화할 수 있습니다.
* 신규: `AbilitySystem.AbilityTask.Debug.AbilityTaskDebugPrintTopNResults`를 사용하여 로그 스팸을 피하기 위해 로그에서 상위 N개의 결과만 인쇄할 수 있습니다.
* 신규: `STAT_AbilityTaskDebugRecording`을 사용하여 이러한 기본 디버깅 변경 사항의 성능 영향을 테스트할 수 있습니다.
* 신규: GameplayCue 이벤트를 필터링하는 디버그 명령을 추가했습니다.
* 신규: 게임플레이 어빌리티 시스템에 `AbilitySystem.DebugAbilityTags`, `AbilitySystem.DebugBlockedTags`, `AbilitySystem.DebugAttribute` 디버그 명령을 추가했습니다.
* 신규: 게임플레이 어트리뷰트의 디버그 문자열 표현을 가져오는 블루프린트 함수를 추가했습니다.
* 신규: 기존 태스크를 취소하기 위한 새로운 게임플레이 태스크 리소스 오버랩 정책을 추가했습니다.
* 변경: 이제 어빌리티 태스크는 `Super::OnDestroy`를 호출하기 전에 어빌리티 포인터에 필요한 모든 작업을 수행해야 합니다. 호출 후에는 포인터가 null이 됩니다.
* 변경: `FGameplayAbilitySpec/Def::SourceObject`를 약한 참조로 변환했습니다.
* 변경: 어빌리티 태스크 내의 어빌리티 시스템 컴포넌트 참조를 가비지 컬렉션이 삭제할 수 있도록 약한 포인터로 만들었습니다.
* 변경: 중복된 열거형 `EWaitGameplayTagQueryAsyncTriggerCondition`을 제거했습니다.
* 변경: GameplayTasksComponent 및 AbilitySystemComponent가 이제 등록된 서브객체 API를 지원합니다.
* 변경: 게임플레이 어빌리티 활성화에 실패한 이유를 나타내는 더 나은 로깅을 추가했습니다.
* 변경: `AbilitySystem.Debug.NextTarget` 및 `PrevTarget` 명령을 전역 HUD `NextDebugTarget` 및 `PrevDebugTarget` 명령을 선호하여 제거했습니다.

https://docs.unrealengine.com/5.1/en-US/unreal-engine-5.1-release-notes/

<a name="changelog-5.0"></a>
### 5.0

https://docs.unrealengine.com/5.0/en-US/unreal-engine-5.0-release-notes/

<a name="changelog-4.27"></a>
### 4.27
* 크래시 수정: `RootMotionSource` 문제에서 네트워크 클라이언트가 상수 힘 루트 모션 태스크와 시간 경과에 따른 강도 수정자를 사용하는 어빌리티 실행을 완료할 때 크래시가 발생하던 문제를 수정했습니다.
* 버그 수정: GameplayCues 사용 시 에디터 로딩 시간 회귀를 수정했습니다.
* 버그 수정: GameplayEffectsContainer의 `SetActiveGameplayEffectLevel` 메서드가 동일한 EffectLevel을 설정할 경우 FastArray를 더 이상 더티로 만들지 않습니다.
* 버그 수정: `GetNetConnection`에서 명시적으로 네트워크 연결에 소유되지 않았지만 해당 연결을 활용하는 액터가 혼합 복제 업데이트를 받지 못하던 GameplayEffect 혼합 복제 모드의 예외 케이스를 수정했습니다.
* 버그 수정: `K2_OnEndAbility`에서 `EndAbility`를 다시 호출하여 `GameplayAbility`의 클래스 메서드 `EndAbility`에서 발생하던 무한 재귀를 수정했습니다.
* 버그 수정: `GameplayTags` 블루프린트 핀이 태그가 등록되기 전에 로드될 경우 자동으로 지워지지 않습니다. 이제 `GameplayTag` 변수와 동일하게 작동하며, 둘 다에 대한 동작은 프로젝트 설정의 `ClearInvalidTags` 옵션으로 변경할 수 있습니다.
* 버그 수정: `GameplayTag` 작업의 스레드 안전성을 개선했습니다.
* 신규: `SourceObject`를 `GameplayAbility`의 `K2_CanActivateAbility` 메서드에 노출했습니다.
* 신규: Native GameplayTags. 새로운 `FNativeGameplayTag`를 도입하여, 모듈이 로드되고 언로드될 때 올바르게 등록 및 등록 해제되는 일회성 네이티브 태그를 만들 수 있습니다.
* 신규: `GiveAbilityAndActivateOnce`를 `FGameplayEventData` 매개변수를 전달하도록 업데이트했습니다.
* 신규: 게임플레이 어빌리티 플러그인의 ScalableFloats를 개선하여 새로운 데이터 레지스트리 시스템에서 커브 테이블의 동적 조회를 지원합니다. 어빌리티 플러그인 외부에서 일반 구조체를 더 쉽게 재사용할 수 있도록 ScalableFloat 헤더를 추가했습니다.
* 신규: GameplayTagsEditorModule을 통해 다른 에디터 사용자 정의에서 GameplayTag UI를 사용하는 코드 지원을 추가했습니다.
* 신규: `UGameplayAbility`의 `PreActivate` 메서드를 수정하여 선택적으로 트리거 이벤트 데이터를 받을 수 있도록 했습니다.
* 신규: 프로젝트별 필터를 사용하여 에디터에서 `GameplayTags`를 필터링하는 더 많은 지원을 추가했습니다. `OnFilterGameplayTag`는 참조하는 속성과 태그 소스를 제공하여 어떤 에셋이 태그를 요청하는지에 따라 태그를 필터링할 수 있도록 합니다.
* 신규: `GameplayEffectSpec`의 클래스 메서드 `SetContext`가 초기화 후에 호출될 때 원래 캡처된 SourceTags를 보존하는 옵션을 추가했습니다.
* 신규: 특정 플러그인에서 `GameplayTags`를 등록하기 위한 UI를 개선했습니다. 새로운 태그 UI는 새로 추가된 `GameplayTag` 소스에 대해 디스크의 플러그인 위치를 선택할 수 있도록 합니다.
* 신규: `Sequencer`에 새로운 트랙이 추가되어 `GameplayAbiltiySystem`을 사용하여 빌드된 액터에서 노티파이 상태를 트리거할 수 있습니다. 노티파이와 마찬가지로 `GameplayCueTrack`은 범위 기반 이벤트 또는 트리거 기반 이벤트를 활용할 수 있습니다.
* 변경: GameplayCueInterface가 GameplayCueParameters 구조체를 참조로 전달하도록 변경했습니다.
* 최적화: `GameplayTag` 테이블을 로드하고 재생성하는 데 여러 성능 개선이 구현되어 이 옵션이 최적화되었습니다.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_27/

<a name="changelog-4.26"></a>
### 4.26
* GAS 플러그인은 더 이상 베타로 표시되지 않습니다.
* 크래시 수정: 유효한 태그 소스 선택 없이 게임플레이 태그를 추가할 때 발생하던 크래시를 수정했습니다.
* 크래시 수정: `UGameplayCueManager::VerifyNotifyAssetIsInValidPath`에서 크래시를 수정하기 위해 메시지에 경로 문자열 인수를 추가했습니다.
* 크래시 수정: `AbilitySystemComponent_Abilities`에서 포인터 확인 없이 사용할 때 발생하던 접근 위반 크래시를 수정했습니다.
* 버그 수정: 효과의 추가 인스턴스가 적용될 때 지속 시간을 재설정하지 않던 스태킹 `GEs` 버그를 수정했습니다.
* 버그 수정: CancelAllAbilities가 인스턴스화되지 않은 어빌리티만 취소하도록 하던 문제를 수정했습니다.
* 신규: 게임플레이 어빌리티 커밋 함수에 선택적 태그 매개변수를 추가했습니다.
* 신규: PlayMontageAndWait 어빌리티 태스크에 `StartTimeSeconds`를 추가하고 주석을 개선했습니다.
* 신규: `FGameplayAbilitySpec`에 태그 컨테이너 "DynamicAbilityTags"를 추가했습니다. 이는 스펙과 함께 복제되는 선택적 어빌리티 태그입니다. 또한 적용된 게임플레이 효과에 의해 소스 태그로 캡처됩니다.
* 신규: `GameplayAbility IsLocallyControlled` 및 `HasAuthority` 함수는 이제 블루프린트에서 호출할 수 있습니다.
* 신규: 비주얼 로거는 이제 비주얼 로깅 데이터를 기록 중인 경우에만 즉시 `GEs`에 대한 정보를 수집하고 저장합니다.
* 신규: 블루프린트 노드의 게임플레이 어트리뷰트 핀에 대한 리디렉터(redirectors) 지원을 추가했습니다.
* 신규: 루트 모션 이동 관련 어빌리티 태스크가 종료될 때 이동 컴포넌트의 이동 모드를 태스크 시작 전의 이동 모드로 되돌리는 새로운 기능을 추가했습니다.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_26/

<a name="changelog-4.25.1"></a>
### 4.25.1
* 수정! UE-92787 `Get Float Attribute` 노드가 있는 블루프린트 저장 시 크래시 및 어트리뷰트 핀이 인라인으로 설정됨
* 수정! UE-92810 인스턴스 편집 가능한 게임플레이 태그 속성이 인라인으로 변경된 액터 생성 시 크래시

<a name="changelog-4.25"></a>
### 4.25
* `RootMotionSource` `AbilityTasks`의 예측 수정
* [`GAMEPLAYATTRIBUTE_REPNOTIFY()`](#concepts-as-attributes)는 이제 이전 `Attribute` 값을 추가로 받습니다. 이를 `OnRep` 함수의 선택적 매개변수로 제공해야 합니다. 이전에는 어트리뷰트 값을 읽어 이전 값을 가져오려고 했습니다. 그러나 복제 함수에서 호출될 경우, `SetBaseAttributeValueFromReplication`에 도달하기 전에 이전 값이 이미 버려져서 대신 새 값을 얻게 되었습니다.
* `UGameplayAbility`에 [`NetSecurityPolicy`](#concepts-ga-netsecuritypolicy)를 추가했습니다.
* 크래시 수정: 유효한 태그 소스 선택 없이 게임플레이 태그를 추가할 때 발생하던 크래시를 수정했습니다.
* 크래시 수정: 어빌리티 시스템을 통해 공격자가 서버를 크래시시킬 수 있는 몇 가지 방법을 제거했습니다.
* 크래시 수정: 이제 태그 요구 사항을 확인하기 전에 GameplayEffect 정의가 있는지 확인합니다.
* 버그 수정: 게임플레이 태그 카테고리가 함수 종단 노드의 일부인 경우 블루프린트의 함수 매개변수에 적용되지 않던 문제를 수정했습니다.
* 버그 수정: 게임플레이 효과 태그가 여러 뷰포트에서 복제되지 않던 문제를 수정했습니다.
* 버그 수정: `InternalTryActivateAbility` 함수가 트리거된 어빌리티를 반복하는 동안 게임플레이 어빌리티 스펙이 무효화될 수 있던 버그를 수정했습니다.
* 버그 수정: 태그 카운트 컨테이너 내에서 게임플레이 태그 업데이트를 처리하는 방식을 변경했습니다. 게임플레이 태그 제거 시 부모 태그의 업데이트를 지연시킬 때, 이제 부모 태그가 업데이트된 후 변경 관련 델리게이트를 호출합니다. 이는 델리게이트가 브로드캐스트될 때 태그 테이블이 일관된 상태를 유지하도록 보장합니다.
* 버그 수정: 타겟을 확인할 때 생성된 타겟 액터 배열을 반복하기 전에 사본을 만들도록 수정했습니다. 일부 콜백이 배열을 수정할 수 있기 때문입니다.
* 버그 수정: 추가 인스턴스가 적용될 때 지속 시간을 재설정하지 않고 호출자 지정 지속 시간을 가진 스태킹 GameplayEffects가 스택의 첫 번째 인스턴스에 대해서만 지속 시간이 올바르게 설정되던 버그를 수정했습니다. 스택의 다른 모든 GE 스펙은 지속 시간이 1초였습니다. 이 경우를 감지하기 위해 자동화 테스트를 추가했습니다.
* 버그 수정: 게임플레이 이벤트 델리게이트를 처리하는 것이 게임플레이 이벤트 델리게이트 목록을 수정할 수 있던 버그를 수정했습니다.
* 버그 수정: GiveAbilityAndActivateOnce가 일관성 없게 작동하던 버그를 수정했습니다.
* 버그 수정: `FGameplayEffectSpec::Initialize` 내부의 일부 작업을 재정렬하여 잠재적인 순서 의존성을 처리했습니다.
* 신규: `UGameplayAbility`에 `OnRemoveAbility` 함수가 추가되었습니다. `OnGiveAbility`와 동일한 패턴을 따르며, 어빌리티의 기본 인스턴스 또는 클래스 기본 객체에서만 호출됩니다.
* 신규: 차단된 어빌리티 태그를 표시할 때 디버그 텍스트에 차단된 태그의 총 수가 포함됩니다.
* 신규: `UAbilitySystemComponent::InternalServerTryActiveAbility`의 이름을 `UAbilitySystemComponent::InternalServerTryActivateAbility`로 변경했습니다. `InternalServerTryActiveAbility`를 호출하던 코드는 이제 `InternalServerTryActivateAbility`를 호출해야 합니다.
* 신규: 태그가 추가되거나 삭제될 때 게임플레이 태그를 표시하는 데 필터 텍스트를 계속 사용합니다. 이전 동작은 필터를 지웠습니다.
* 신규: 에디터에서 새 태그를 추가할 때 태그 소스를 재설정하지 않습니다.
* 신규: 지정된 태그 집합을 가진 모든 활성 게임플레이 효과를 어빌리티 시스템 컴포넌트에서 쿼리하는 기능을 추가했습니다. 새로운 함수는 `GetActiveEffectsWithAllTags`이며, 코드 또는 블루프린트를 통해 접근할 수 있습니다.
* 신규: 루트 모션 이동 관련 어빌리티 태스크가 종료될 때 이동 컴포넌트의 이동 모드를 태스크 시작 전의 이동 모드로 되돌립니다.
* 신규: `SpawnedAttributes`를 transient로 만들어 만료되고 잘못될 수 있는 데이터를 저장하지 않도록 했습니다. 현재 저장된 만료된 데이터가 전파되는 것을 방지하기 위해 널(null) 검사를 추가했습니다. 이는 `SpawnedAttributes`에 잘못된 데이터가 저장되는 것과 관련된 문제를 방지합니다.
* API 변경: `AddDefaultSubobjectSet`가 더 이상 사용되지 않습니다. 대신 `AddAttributeSetSubobject`를 사용해야 합니다.
* 신규: 게임플레이 어빌리티는 이제 몽타주를 재생할 애니메이션 인스턴스를 지정할 수 있습니다.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_25/

<a name="changelog-4.24"></a>
### 4.24
* 컴파일 시 블루프린트 노드 `Attribute` 변수가 `None`으로 재설정되던 문제를 수정했습니다.
* [`TargetData`](#concepts-targeting-data)를 사용하려면 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)를 호출해야 합니다. 그렇지 않으면 `ScriptStructCache` 오류가 발생하고 클라이언트가 서버에서 연결 해제됩니다. 제 조언은 이제 모든 프로젝트에서 항상 이 함수를 호출하는 것입니다. 4.24 이전에는 선택 사항이었습니다.
* 이전에 변수가 정의되지 않은 블루프린트에 `GameplayTag` 설정자를 복사할 때 발생하던 크래시를 수정했습니다.
* `UGameplayAbility::MontageStop()` 함수가 이제 `OverrideBlendOutTime` 매개변수를 올바르게 사용합니다.
* 컴포넌트의 `GameplayTag` 쿼리 변수가 편집 시 수정되지 않던 버그를 수정했습니다.
* `GameplayEffectExecutionCalculations`가 어트리뷰트 캡처로 뒷받침될 필요가 없는 "임시 변수"에 대한 스코프 수정자를 지원하는 기능을 추가했습니다.
  * 구현은 기본적으로 `GameplayTag`로 식별되는 집계자(aggregator)가 실행(execution)이 스코프 수정자로 조작할 임시 값을 노출하는 수단으로 생성될 수 있도록 합니다. 이제 소스 또는 타겟에서 캡처할 필요가 없는 조작 가능한 값을 원하는 공식을 만들 수 있습니다.
  * 사용하려면, 실행은 새 멤버 변수 `ValidTransientAggregatorIdentifiers`에 태그를 추가해야 합니다. 해당 태그는 아래쪽의 스코프 모드 계산 수정자 배열에 임시 변수로 표시되어 나타납니다. 기능 지원을 위해 세부 정보 사용자 정의도 업데이트되었습니다.
* 제한된 태그 편의성 개선을 추가했습니다. 제한된 `GameplayTag` 소스의 기본 옵션을 제거했습니다. 여러 제한된 태그를 연속으로 추가하기 쉽도록 태그를 추가할 때 소스를 더 이상 재설정하지 않습니다.
* `APawn::PossessedBy()`가 이제 `Pawn`의 소유자를 새 `Controller`로 설정합니다. [`Mixed Replication Mode`](#concepts-asc-rm)가 `ASC`가 `Pawn`에 있는 경우 `Pawn`의 소유자를 `Controller`로 기대하기 때문에 유용합니다.
* `FAttributeSetInitterDiscreteLevels`의 POD(Plain Old Data) 버그를 수정했습니다.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_24/

**[⬆ 맨 위로](#table-of-contents)**
