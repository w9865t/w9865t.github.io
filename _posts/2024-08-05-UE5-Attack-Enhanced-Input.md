---
title: "[UE5] 공격 기능 구현 1 - Enhanced Input 사용하기"
date: 2024-08-05 0:00:00 +09:00
categories: [Unreal Engine 5, 공격 기능 구현]
tags: [unreal]     # TAG names should always be lowercase
description: "마우스 좌클릭 입력 시 실행할 공격 함수 구현(UE5의 향상된 입력 사용)"
---

![](/assets/img/posts/UE5.png)

## 구현 목표
마우스 좌클릭으로 공격 애니메이션을 재생하고 다른 캐릭터에게 대미지를 입히는 기능 구현하기

## 구현해야 할 것들
>1. _**마우스 좌클릭 입력 시 실행할 공격 함수 구현(UE5의 향상된 입력 사용) <- 오늘 할 것**_
>2. 캐릭터가 사용할 무기 클래스 구현
>3. 공격과 피격 매커니즘 구현
>4. 공격 시 재생할 애니메이션과 몽타주 구현

기존 UE4에서 사용하던 방식이 아닌 UE5에서 새로 도입된 Enhanced Input 을 사용하여 구현하는 것이 목표이다.

***
### 언리얼 에디터에서 입력 에셋 만들기


먼저 언리얼 에디터에서 Input Action, Input Mapping Context 에셋을 만들어 주어야 한다.

나의 경우 `IA_Attack`, `IMC_MainCharacter` 이라는 이름으로 생성하였다.

![](/assets/img/posts/UE5-Attack-Enhanced-Input/IA_Attack.png) ![](/assets/img/posts/UE5-Attack-Enhanced-Input/IMC_MainCharacter.png)

`IMC_MainCharacter`을 열어 매핑을 추가하고, 아까 생성한 `IA_Attack`을 등록해 주었다.
왼쪽 마우스 버튼을 클릭했을 때 공격하고 싶으므로 왼쪽 마우스 버튼으로 매핑한다.

![](/assets/img/posts/UE5-Attack-Enhanced-Input/LeftMouse.png)

### 입력 에셋을 캐릭터에 연동하기
캐릭터에 아까 언리얼 에디터에서 만든 `IA_Attack`와 `IMC_MainCharacter`를 연동해주기 위해 캐릭터 클래스의 헤더 파일에 `DefaultMappingContext`와 `AttackAction` 변수를 만들고, UPROPERTY 매크로로 블루프린트에서도 수정할 수 있도록 `EditAnywhere`라고 선언해 주었다.
```cpp
UPROPERTY(EditAnywhere, Category = Input, meta = (AllowPrivateAccess = "true"))
UInputMappingContext* DefaultMappingContext;

UPROPERTY(EditAnywhere, Category = Input, meta = (AllowPrivateAccess = "true"))
UInputAction* AttackAction;
```
이렇게 변수를 만들어 두면 C++ 캐릭터 클래스를 상속한 블루프린트 클래스에서 변수에 다음과 같이 할당해 줄 수 있다.

![](/assets/img/posts/UE5-Attack-Enhanced-Input/DefaultMappingContext.png)
![](/assets/img/posts/UE5-Attack-Enhanced-Input/AttackAction.png)

다음으로 캐릭터 클래스 헤더 파일에 공격 시 실행할 `Attack` 함수와 공격 끝날 때 실행할 `AttackEnd` 함수를 정의하였다.
```cpp
void Attack();
void AttackEnd();
```
지금 당장은 `AttackEnd` 함수가 없어도 될 것 같다는 생각이 드는데 혹시 나중에 써먹을 일이 생길지도 모른다는 생각에 그냥 미리 만들어두기로 했다.


이 함수들을 입력 이벤트와 연동해 주어야 하는데, 캐릭터 클래스의 부모 클래스인 `APawn` 인터페이스 `SetupPlayerInputComponent` 가 준비되어 있으므로 그대로 쓰면 된다. 아마 캐릭터 클래스 헤더 파일에 다음과 같이 이미 선언되어 있을 것이다.
```cpp
virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;
```

이제 캐릭터 클래스의 C++ 파일로 가서 다음과 같이 Cast하여 `EnhancedInputComponent` 를 얻어오자.
```cpp
UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent)
```
`PlayerInputComponent`의 경우 `APawn` 인터페이스 `SetupPlayerInputComponent` 에 인자로 전달되므로 그대로 써먹으면 된다.

이렇게 얻은 `EnhancedInputComponent`을 이용해 아까 만든 두 공격 함수를 Bind 해주었다 (BindAction 전에 null 체크 해주는게 좋음)
```cpp
EnhancedInputComponent->BindAction(AttackAction, ETriggerEvent::Started, this, &Aproject1Character::Attack);
EnhancedInputComponent->BindAction(AttackAction, ETriggerEvent::Completed, this, &Aproject1Character::AttackEnd);
```
나의 경우 `project1Character` 라는 매우 대충 지은 이름으로 캐릭터 클래스를 만들었는데 클래스명에 따라 `&A(클래스명)` 이 부분은 클래스명에 따라 다를 것이다.


이제 게임상에서 마우스 좌클릭 이벤트가 트리거되었을 때 `Attack` 함수가, 이벤트가 완료되었을 때 `AttackEnd` 함수가 실행되게 된다.

잘 작동하는지 테스트 해보고 싶다면 `UE_LOG` 등의 디버그 기능을 `Attack` 함수와 `AttackEnd` 함수에 추가하여 확인해 볼 수 있다.

***
## 다음 목표
캐릭터가 대미지를 입히기 위해 사용할 무기 클래스 구현하기
