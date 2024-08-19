---
title: "[UE5] 공격 기능 구현 2 - 무기 클래스 만들기"
date: 2024-08-06 0:00:00 +09:00
categories: [Unreal Engine 5, 공격 기능 구현]
tags: [unreal]     # TAG names should always be lowercase
description: "캐릭터가 사용할 무기 클래스 구현"
---

![](/assets/img/posts/UE5.png)

## 구현 목표
마우스 좌클릭으로 공격 애니메이션을 재생하고 다른 캐릭터에게 대미지를 입히는 기능 구현하기
## 구현해야 할 것들
>1. [마우스 좌클릭 입력 시 실행할 공격 함수 구현(UE5의 향상된 입력 사용)](https://w9865t.github.io/posts/UE5-Attack-Enhanced-Input/)
>2. _**캐릭터가 사용할 무기 클래스 구현 <- 오늘 할 것**_
>3. 공격과 피격 매커니즘 구현
>4. 공격 시 재생할 애니메이션과 몽타주 구현

캐릭터가 대미지를 입히기 위해서는 무기가 필요하므로 무기 클래스도 만들어 주어야 한다.

무기를 사용하지 않는 캐릭터(예를 들어 깨무는 공격을 하는 캐릭터라던가 등등...) 라고 하더라도 무기를 투명화 처리하여 캐릭터의 해당 위치(물기 공격이라면 입) 에 부착하여 구현하는 경우가 많은 것 같다...

결론은 공격 기능 구현을 위해선 므로 무기 클래스를 만들어야 한다는 것!

***
## 무기 클래스에 필요한 요소들
무기 클래스는 `AActor` 클래스를 상속하여 구현하는데, 최소한 다음과 같은 요소들이 필요하다.
> 1. 무기의 외형을 할당할 스태틱 메시 컴포넌트
2. 무기와 다른 물체의 충돌을 감지할 콜리전 컴포넌트

### 스태틱 메시 컴포넌트
무기 클래스의 헤더 파일에 다음과 같이 선언해 주었다.
```cpp
UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* WeaponMesh;
```
`VisibleAnywhere` 로 선언하였기 때문에 전에 만든 캐릭터 클래스와 마찬가지로, C++ 무기 클래스를 상속한 블루프린트 무기 클래스의 디테일 패널에서 쓰고 싶은 에셋을 설정해 줄 수 있다.

![](/assets/img/posts/UE5-Attack-Weapon-Class/WeaponMesh.png)

나중에 변경할지도 모르지만 일단 언리얼 마켓플레이스의 [Free Fantasy Weapon Sample Pack](https://www.unrealengine.com/marketplace/ko/product/e4494c76c3b348aba7ef9b263a6dd496?lang=ko) 을 임시로 사용하기로 했다.

다음으론 무기 클래스의 CPP 파일로 가서 다음과 같이 오브젝트를 생성하고 루트 컴포넌트로 지정해 주었다.
```cpp
WeaponMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("WeaponMesh"));
RootComponent = WeaponMesh;
```
루트 컴포넌트는 액터의 트랜스폼(위치, 회전 등) 의 기준이 되는 컴포넌트로 다른 컴포넌트들을 붙일 수도 있다.

눈에 잘 보이는 스태틱 메시를 루트 컴포넌트로 설정하는 것이 편하지 않을까 싶다.

### 콜리전 컴포넌트
콜리전 메시의 형태에 따라 `UBoxComponent`, `UCapsuleComponent`, `USphereComponent` 와 같이 다양한 옵션들이 있으므로 무기 모양에 따라 적절히 고르면 된다. 나는 박스 형태의 `UBoxComponent` 를 사용하기로 했다
```cpp
UPROPERTY(VisibleAnywhere)
UBoxComponent* WeaponCollision;
```

스태틱 메시 컴포넌트와 마찬가지로 CPP 파일에서 오브젝트를 생성하고 루트 컴포넌트에 붙여주었다.
```cpp
WeaponCollision = CreateDefaultSubobject<UBoxComponent>(TEXT("WeaeponCollision"));
WeaponCollision->SetupAttachment(GetRootComponent());
```


다음과 같이 콜리전이 시작될 때와 끝날 때 실행할 함수도 만들어 주었다.

```cpp
// 콜리전 시 실행되는 함수
UFUNCTION()
virtual void WeaponBeginCollision(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

// 콜리전 끝날 시 실행되는 함수
UFUNCTION()
virtual void WeaponEndCollision(UPrimitiveComponent* OverlappedComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
```

인수가 굉장히 복잡한데 왜 이렇게 만들었는지 알아보기 위해 `PrimitiveComponent.h` 를 보자.

```cpp
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_SixParams( FComponentBeginOverlapSignature, UPrimitiveComponent, OnComponentBeginOverlap, UPrimitiveComponent*, OverlappedComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, int32, OtherBodyIndex, bool, bFromSweep, const FHitResult &, SweepResult);
```
`OnComponentBeginOverlap` 부분을 보면 다음과 같이 멀티캐스트 델리게이트로 구성되어 있는 것을 알 수 있다.

여기서 델리게이트는 어떤 이벤트가 발생했을 때 실행할 특정 함수를 바인딩하는 기능이고, 멀티캐스트는 한 이벤트에 여러 함수를 바인딩하여 실행할 수 있다는 뜻이다. (네트워크 공부하면서 접한 개념인데 여기서 보니 뭔가 반갑다)

[언리얼 5.4 델리케이트 공식 문서 ](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/delegates-and-lamba-functions-in-unreal-engine?application_version=5.4)

어쨌든 `WeaponCollision` 을 통해 이렇게 이벤트가 시작될 때, 끝날 때 실행할 함수를 묶어주었다.
```cpp
WeaponCollision->OnComponentBeginOverlap.AddDynamic(this, &AWeapon::WeaponBeginCollision);
WeaponCollision->OnComponentEndOverlap.AddDynamic(this, &AWeapon::WeaponEndCollision);
```

### 콜리전 시 실행되는 함수
콜리전 이벤트가 발생했을 때 무엇과 충돌했는지 알아야 한다. `UKismetSystemLibrary` 를 사용할 것이다.

[언리얼 5.4 UKismetSystemLibrary 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/API/Runtime/Engine/Kismet/UKismetSystemLibrary)

엄청나게 기능이 많은데 이번에 쓸 것은 `BoxTraceSingle` 이다. 주어진 시작점과 끝점을 한번 쓸듯이 스캔하여 제일 첫번째 충돌한 것을 판정하는 함수이다. 모든 결과를 반환하는 `BoxTraceMulti` 도 있지만 무기에 사용할 용도로는 아마 Single로도 충분할 것 같다. (여러번 스캔하면 성능도 저하될 것 같고...)

```cpp
BoxTraceSingle (
const UObject* WorldContextObject, // 충돌 판정할 대상
const FVector Start, // 판정 시작 지점
const FVector End, // 판정 끝 지점
const FVector HalfSize, // 충돌 볼륨의 크기 (박스의 중심으로부터 가장자리까지의 거리)
const FRotator Orientation, // 박스의 회전 정보
ETraceTypeQuery TraceChannel, // 트레이스 채널
bool bTraceComplex, // 더욱 정밀한 판정을 할 것인지 여부
const TArray< AActor* >& ActorsToIgnore, // 충돌 판정을 무시할 액터들 (Owner, Instigator 는 일단 넣는게 좋음)
EDrawDebugTrace::Type DrawDebugType, // 디버그 트레이스 표시 여부
FHitResult& OutHit, // !!중요!! 판정 결과
bool bIgnoreSelf, // 스스로와의 충돌 판정 무시 여부
FLinearColor TraceColor, // 이하 디버그 관련, 생략해도 무관
FLinearColor TraceHitColor,
float DrawTime
)
```
여기서 시작점과 끝점은 `USceneComponent` 를 두개 만들어서 무기 양 끝 (칼이라면 칼날의 양 끝) 에 배치한 후 루트 컴포넌트에 부착하는 것으로 쉽게 파악할 수 있다.

결과적으로 참조자를 통해 `FHitResult` 형 변수에 결과값을 저장하는 구조로 되어 있다.

[언리얼 5.4 FHitResult 공식 문서](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Engine/FHitResult?application_version=5.4)

이를 통해 많은 정보를 알 수 있는데 `GetActor()` 를 통해 충돌한 액터가 무엇인지 판정할 수 있게 된다.

***
### 추가 기능
캐릭터가 무기를 장착하도록 하는 기능 등을 무기 클래스에 구현해도 될 듯

***
## 다음 목표
`UGameplayStatics` 을 사용하여 충돌 판정한 대상에 대미지 입히는 기능
