---
title:  "언리얼엔진 탑다운 뷰 예제 분석하기"
excerpt: "코드의 구조를 분석하자"

categories:
  - UE5 Insight
tags:
  - [UE5, 분석, C++]

toc: true
toc_sticky: true
 
date: 2023-11-06
last_modified_at: 2023-11-06
---

언리얼엔진의 구조는 여기에서 확인할 수 있다.
[Unreal Engine API Reference | Unreal Engine 5.2 Documentation](https://docs.unrealengine.com/5.3/en-US/API/)
{: .notice--primary}

PlayerController.cpp를 기반으로 작성되었으며, 처음으로 분석한 글이기에 읽기에 구조가 불편할 수 있습니다.  
또한 오류가 있다면 지적해주시면 감사드리겠습니다😌
{: .notice--warning}
---


# 1. 생성자
```cpp
AThirdPlayerController::AThirdPlayerController() {
	bShowMouseCursor = true;
	DefaultMouseCursor = EMouseCursor::Default;
	CachedDestination = FVector::ZeroVector;
	FollowTime = 0.f;
}
```

이곳은 생성자이다. 마우스를 통한 이동을 위해 초기화를 진행하고 있다.

bShowMouseCursor : 마우스 커서가 표시될지 결정한다.
- 파일 : PlayerController.h
- 유형 : 변수
- 접근자 : public
- 자료형 : uint32: 1

[DefaultMouseCursor](https://docs.unrealengine.com/5.3/en-US/API/Runtime/Engine/GameFramework/APlayerController/DefaultMouseCursor/) : 기본적으로 표시할 마우스 커서 유형
- 파일 : PlayerController.h
- 유형 : 변수
- 접근자 : public
- 자료형 : TEnumAsByte<EMouseCursor::Type>

[EMouseCursor](https://docs.unrealengine.com/4.26/en-US/API/Runtime/ApplicationCore/GenericPlatform/EMouseCursor__Type/) : 여기선 마우스의 모양을 정하기 위해 사용되고 있다. Default면 기본 윈도우 커서이다.
- 클래스: ICursor.h
- 유형 : enum

CachedDestination : Destination을 기록해두기 위한 변수이다.
- 파일 : 헤더에 있음
- 유형 : 변수
- 접근자 : private
- 자료형 : FVector

FollowTime : 마우스 클릭 혹은 터치를 얼마나 오래 했는지 기록하는 변수이다.
- 파일 : 헤더에 있음
- 유형 : 변수
- 접근자 : private
- 자료형 : float

---
# 2. BeginPlay()
```cpp
void AThirdPlayerController::BeginPlay()
{
	// Call the base class  
	Super::BeginPlay();

	//Add Input Mapping Context
	if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(GetLocalPlayer()))
	{
		Subsystem->AddMappingContext(DefaultMappingContext, 0);
	}
}
```

입력 액션(Input Action)을 트리거하려면 입력 매핑 컨텍스트(Input Mapping Context)에 포함시키고, 해당 입력 매핑 컨텍스트를 로컬 플레이어의 향상된 입력 로컬 플레이어 서브시스템(Enhanced Input Local Player Subsystem)에 추가해야한다. ([언리얼 엔진의 입력 개요](https://docs.unrealengine.com/5.3/ko/input-overview-in-unreal-engine/))

순서는 다음과 같다.
Input Action > Input Mapping Context > Input Local Player Subsystem

[UEnhancedInputLocalPlayerSubsystem]([UEnhancedInputLocalPlayerSubsystem | Unreal Engine 5.2 Documentation](https://docs.unrealengine.com/5.3/en-US/API/Plugins/EnhancedInput/UEnhancedInputLocalPlayerSubsyst-/)) : 입력과 관련된 역할을 하는 클래스
- 파일 : EnhancedInputComponent.h
- 유형 : 클래스
- 부모 : UInputComponent

[ULocalPlayer](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/ULocalPlayer/) : 현재 클라이언트에서 활성 상태인 각 플레이어는 로컬 플레이어(LocalPlayer)를 가지고 있다. 로컬 플레이어는 맵 간에 유지되며, 스플릿 스크린/co-op 모드의 경우 여러개가 스폰될 수 있다. 서버에서는 스폰된 로컬 플레이어가 없을 수도 있다.
- 파일 : LocalPlayer.h
- 유형 : 클래스
- 부모 : UPlayer

[GetSubsystem](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/ULocalPlayer/GetSubsystem/2/) : 지정된 유형의 서브시스템을 제공된 로컬 플레이어로부터 가져온다. 서브시스템을 찾을 수 없거나 로컬 플레이어가 null인 경우 nullptr을 반환한다.
- 파일 : LocalPlayer.h
- 유형 : static 함수
- 인자 : const ULocalPlayer *
- 반환값 : TSubsystemClass (템플릿)
- 클래스 : ULocalPlayer

[GetLocalPlayer](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/GameFramework/APlayerController/GetLocalPlayer/) : 만약 존재한다면, 이 컨트롤러에 대한 ULocalPlayer를 반환하고, 그렇지 않다면 null을 반환한다.
- 파일 : PlayerController.h
- 유형 : 함수
- 인자 : void
- 반환값 : ULocalPlayer
- 클래스 : APlayerController

---
# 3. SetupInputComponent()

```cpp
void AThirdPlayerController::SetupInputComponent()
{
    // set up gameplay key bindings
    Super::SetupInputComponent();
    // Set up action bindings
    if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(InputComponent))
    {
        // Setup mouse input events
        EnhancedInputComponent->BindAction(SetDestinationClickAction, ETriggerEvent::Started, this, &AThirdPlayerController::OnInputStarted);
        EnhancedInputComponent->BindAction(SetDestinationClickAction, ETriggerEvent::Triggered, this, &AThirdPlayerController::OnSetDestinationTriggered);
        EnhancedInputComponent->BindAction(SetDestinationClickAction, ETriggerEvent::Completed, this, &AThirdPlayerController::OnSetDestinationReleased);
        EnhancedInputComponent->BindAction(SetDestinationClickAction, ETriggerEvent::Canceled, this, &AThirdPlayerController::OnSetDestinationReleased);
        // Setup touch input events
        EnhancedInputComponent->BindAction(SetDestinationTouchAction, ETriggerEvent::Started, this, &AThirdPlayerController::OnInputStarted);
        EnhancedInputComponent->BindAction(SetDestinationTouchAction, ETriggerEvent::Triggered, this, &AThirdPlayerController::OnTouchTriggered);
        EnhancedInputComponent->BindAction(SetDestinationTouchAction, ETriggerEvent::Completed, this, &AThirdPlayerController::OnTouchReleased);
        EnhancedInputComponent->BindAction(SetDestinationTouchAction, ETriggerEvent::Canceled, this, &AThirdPlayerController::OnTouchReleased);
    }
    else
    {
        UE_LOG(LogTemplateCharacter, Error, TEXT("'%s' Failed to find an Enhanced Input Component! This template is built to use the Enhanced Input system. If you intend to use the legacy system, then you will need to update this C++ file."), *GetNameSafe(this));
    }
}
```

[UEnhancedInputComponent](https://docs.unrealengine.com/4.27/en-US/API/Plugins/EnhancedInput/UEnhancedInputComponent/) : 향상된 입력 컴포넌트(Enhanced Input Component)는 액터(Actor)가 향상된 액션을 대리 함수(delegate function)에 바인딩하거나 해당 액션을 모니터링할 수 있게 하는 임시 컴포넌트
- 파일 : EnhancedInputComponent.h
- 유형 : 클래스
- 부모 : UInputComponent

SetDestinationClickAction : UInputAction을 담기위한 변수를 헤더에서 따로 선언해준 것
- 파일 : 헤더파일에 있음
- 유형 : 변수
- 접근자 : public
- 자료형 : UInputAction*

[ETriggerEvent](https://docs.unrealengine.com/4.27/en-US/API/Plugins/EnhancedInput/ETriggerEvent/) : 액션의 동작을 제어하고 이해하는데 사용된다.
- 파일 : InputTriggers.h
- 유형 : enum
- 값
	- None : 중요한 트리거 상태 변경이 없으며 활성 장치 입력이 없다.
	- Started : 트리거 평가가 시작된 이벤트가 발생
	- Ongoing: 트리거가 아직 처리중
	- Canceled: 트리거 처리가 취소됨
	- Triggered : 하나 이상의 처리 틱 이후 트리거 발생
	- Completed : 트리거 상태가 이 프레임에서 Triggered에서 None으로 전환되었다. 즉, 트리거 처리가 완료되었다.

---
# 4. OnInputStarted()

```cpp
void AThirdPlayerController::OnInputStarted()
{
    StopMovement();
}
```

[StopMovement](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/GameFramework/AController/StopMovement/) : 움직임을 멈춘다.
- 파일 : Controller.h
- 유형 : 함수
- 인자 : void
- 반환값 : void
- 클래스 : AController

---
# 5. OnSetDestinationTriggered()

```cpp
// Triggered every frame when the input is held down
void AThirdPlayerController::OnSetDestinationTriggered()
{
    // We flag that the input is being pressed
    FollowTime += GetWorld()->GetDeltaSeconds();
    // We look for the location in the world where the player has pressed the input
    FHitResult Hit;
    bool bHitSuccessful = false;
    if (bIsTouch)
    {
        bHitSuccessful = GetHitResultUnderFinger(ETouchIndex::Touch1, ECollisionChannel::ECC_Visibility, true, Hit);
    }
    else
    {
        bHitSuccessful = GetHitResultUnderCursor(ECollisionChannel::ECC_Visibility, true, Hit);
    }
    // 만약 표면에 충돌했다면, 위치를 캐시한다.
    if (bHitSuccessful)
    {
        CachedDestination = Hit.Location;
    }
    // 마우스 포인터나 터치 방향으로 이동
    APawn* ControlledPawn = GetPawn();
    if (ControlledPawn != nullptr)
    {
        FVector WorldDirection = (CachedDestination - ControlledPawn->GetActorLocation()).GetSafeNormal();
        ControlledPawn->AddMovementInput(WorldDirection, 1.0, false);
    }
}
```

[GetWorld](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/Engine/UWorld/GetWorld/) : 월드를 반환한다.
- 파일 : World.h
- 유형 : 함수
- 인자 : void
- 반환값 : UWorld
- 클래스 : UWorld

[GetDeltaSeconds](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/UWorld/GetDeltaSeconds/) : 프레임 델타 타임을 초 단위로 반환하며, 시간 이완(time dilation)과 같은 요소에 조정된다.
- 파일 : World.h
- 유형 : 함수
- 인자 : void
- 반환값 : float
- 클래스 : UWorld

[FHitResult](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/FHitResult/) : 추적(trace)의 하나의 충돌(hit)에 대한 정보를 포함하는 구조체(structure)로, 충돌 지점(point of impact) 및 해당 지점에서의 표면 법선(surface normal)과 같은 정보를 포함하고 있다.
- 파일 : EngineTypes.h
- 유형 : 구조체

[GetHitResultUnderCursor](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/GameFramework/APlayerController/GetHitResultUnderCursor/) : 화면 상에서 커서 아래의 충돌 정보를 반환하는 데 사용된다.
- 파일 : PlayerController.h
- 유형 : 함수
- 인자
	- Ecollision TraceChannel
	- bool bTraceComplex
	- FHitREsult & HitResult
- 반환값 : bool
- 클래스 : APlayerController

[ECollisionChannel](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/Engine/ECollisionChannel/) : 게임에서 다양한 종류의 오브젝트 간 충돌을 관리하는 데 사용된다. 이를 통해 충돌 처리를 미세하게 제어하고 원하는 동작을 구현할 수 있다.
- 파일 : EngineTypes.h
- 유형 : enum
- 값 : 너무 많아서 생략

[APawn](https://docs.unrealengine.com/5.3/en-US/API/Runtime/Engine/GameFramework/APawn/) : 플레이어 또는 AI에 의해 소유될 수 있는 모든 액터의 기본 클래스이다. 이들은 레벨에서 플레이어와 생물체의 물리적 표현이다.
- 파일 : Pawn.h
- 유형 : 클래스
- 부모 : AActor

[GetPawn](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/GameFramework/AController/GetPawn/1/) : 폰의 Getter
- 파일 : Controller.h
- 유형 : 함수
- 인자 : void
- 반환값 : APawn*
- 클래스 : AController

[GetActorLocation](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/GameFramework/AActor/GetActorLocation/) : 이 액터의 RootComponent의 위치를 반환한다.
- 파일 : Actor.h
- 유형 : 함수
- 인자 : void
- 반환값 : FVector
- 클래스 : AActor

[GetSafeNormal](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Core/Math/FVector/GetSafeNormal/) : 벡터의 정규화된 복사본을 가져오며, 길이를 기준으로 안전하게 정규화할 수 있는지 확인한다. 벡터 길이가 너무 작아 안전하게 정규화할 수 없으면 영 벡터를 반환한다.
- 파일 : Vector.h
- 유형 : 함수
- 인자값
	- float Tolerance
- 반환값 : FVector
- 클래스 : FVector

[AddMovementInput](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Engine/GameFramework/APawn/AddMovementInput/) : 주어진 월드 방향 벡터(보통 정규화된 벡터)를 기반으로 'ScaleValue'로 스케일링된 움직임 입력을 추가한다. 만약 'ScaleValue'가 0보다 작으면, 움직임은 반대 방향으로 될 것이다. 기본 Pawn 클래스는 움직임을 자동으로 적용하지 않으며, 이를 수행하는 것은 Tick 이벤트에서 사용자의 역할이다. Character 및 DefaultPawn과 같은 하위 클래스는 이 입력을 자동으로 처리하고 움직인다.
- 파일 : Pawn.h
- 유형 : 함수
- 인자
	- FVector WorldDirection
	- float ScaleValue
	- bool bForce
- 반환값 : void
- 클래스 : APawn

---
# 6. OnSetDestinationReleased()

```cpp
void AThirdPlayerController::OnSetDestinationReleased()
{
    // 짧게 눌렀다면,
    if (FollowTime <= ShortPressThreshold)
    {
        // 그곳으로 이동하고 파티클을 생성한다.
        UAIBlueprintHelperLibrary::SimpleMoveToLocation(this, CachedDestination);
        UNiagaraFunctionLibrary::SpawnSystemAtLocation(this, FXCursor, CachedDestination, FRotator::ZeroRotator, FVector(1.f, 1.f, 1.f), true, true, ENCPoolMethod::None, true);
    }
    FollowTime = 0.f;
}
```

ShortPressThreshold : 짧게 누른 텀이 어느정도인지 따로 설정해준다.
- 파일 : 헤더파일에 있음
- 유형 : 변수
- 접근자 : public
- 자료형 : float

[UAIBlueprintHelperLibrary](https://docs.unrealengine.com/4.26/en-US/API/Runtime/AIModule/Blueprint/UAIBlueprintHelperLibrary/) : 사용자 지정 AI 블루프린트 및 블루프린트에서 AI 동작을 조작하기 위한 다양한 도우미 함수와 유틸리티 함수를 제공하는 클래스. 이 클래스를 사용하면 AI 행동을 스크립트화하고 조작할 수 있다.
- 파일 : AIBlueprintHelperLibrary.h
- 유형 : 클래스
- 부모 : UBlueprintUFunctionLibrary

[SimpleMoveToLocation](https://docs.unrealengine.com/4.26/en-US/API/Runtime/AIModule/Blueprint/UAIBlueprintHelperLibrary/SimpleMoveToLocation/) :
- 파일 : AIBlueprintHelperLibrary.h
- 유형 : 함수
- 인자
	- AController * Controller
	- const FVector & Goal
	- 반환값 : void
- 클래스 : UAIBlueprintHelperLibrary

[UNiagaraFunctionLibrary](https://docs.unrealengine.com/4.27/en-US/API/Plugins/Niagara/UNiagaraFunctionLibrary/) : Niagara 시뮬레이션에 액세스하기 위한 C++ 및 블루프린트에서 사용 가능한 유틸리티 함수 라이브러리. 모든 위치와 방향 정보는 언리얼엔진의 참조 프레임과 단위로 반환되며, Leap Device가 원점에 위치한다고 가정한다.
- 파일 : NiagaraFunctionLibrary.h
- 유형 : 클래스
- 부모 : UBlueprintFunctionLibrary

[SpawnSystemAtLocation](https://docs.unrealengine.com/4.27/en-US/API/Plugins/Niagara/UNiagaraFunctionLibrary/SpawnSystemAtLocation/) : 지정된 월드 위치/회전에 Niagara 시스템 생성
- 파일 : NiagaraFunctionLibrary.h
- 유형 : 함수
- 인자
	- const UObject * WorldContextObject
	- class UNiagaraSystem * SystemTemplate
	- FVector Location
	- FRotator Rotation
	- FVector Scale
	- bool bAutoDestroy
	- bool bAutoActivate
	- ENCPoolMethod PoolingMethod
	- bool bPreCullCheck
- 반환값 : UNiagaraComponent*
- 클래스 : UNiagaraFunctionLibrary

FXCursor :
- 파일 : 헤더파일에 있음
- 유형 : 변수
- 접근자 : public
- 자료형 : UNiagaraSystem*

[UNiagaraSystem](https://docs.unrealengine.com/4.27/en-US/API/Plugins/Niagara/UNiagaraSystem/) : 여러 개의 이미터를 포함하는 컨테이너로, 이들이 결합하여 파티클 시스템 이펙트를 생성한다.
- 파일 : NiagaraSystem.h
- 유형 : 클래스
- 부모 : UFXSystemAsset

[ENCPoolMethod](https://docs.unrealengine.com/4.26/en-US/API/Plugins/Niagara/ENCPoolMethod/) : Niagara 시스템의 풀링(pooling) 방법을 정의한다. 풀링은 메모리 사용 및 성능 최적화를 위해 사용한다.
- 파일 : NiagaraComponentPool.h
- 유형 : enum
- 값
	- None : 시스템 풀링을 사용하지 않고, 시스템 인스턴스가 만들어지면 즉시 소멸
	- AutoRelease : 시스템 인스턴스가 실행을 완료하면 자동으로 반환되어 풀에 다시 넣는다.
	- ManualRelease : 사용자가 수동으로 시스템 인스턴스를 반환해야 한다.
	- ManualRelease_OnComplete : 시스템 인스턴스가 실행을 완료한 경우에만 사용자가 수동으로 반환해야 한다.
	- FreeInPool : 시스템 인스턴스를 생성할 때마다 즉시 풀에 반환한다. 사용자는 반환을 직접 처리할 필요가 없다.