---
title: "[UE5] Slate 와 UMG, 그리고 HUD"
date: 2024-09-11 0:00:00 +09:00
categories: [Unreal Engine 5, UI]
tags: [unreal]     # TAG names should always be lowercase
description: "UI 를 만드는 여러 수단들"
---

![](/assets/img/posts/UE5.png)

***

## Slate 와 UMG

UE5 에서 UI 를 만들기 위해 가장 보편적으로 사용되는 것이 Slate 와 UMG(Unreal Motion Graphics) 이다.

### Slate

Slate 는 코드 기반의 UI 설계 도구이다. 플랫폼에 구애받지 않으며 코드 기반인 만큼 아주 자유롭게 커스텀이 가능하다. 언리얼 에디터 자체도 Slate 를 통해 만들어진 것이다.

선언형 문법을 사용하며 인디렉션(포인터를 통한 변수 접근) 레이어가 필요 없고, 위젯을 선언하고 생성하기 위한 매크로들이 다 준비되어 있으므로 갖다 쓰기만 하면 된다. 에픽이 올려둔 예시 코드를 보자.

```cpp
SLATE_BEGIN_ARGS( SSubMenuButton )
    : _ShouldAppearHovered( false )
    {}
    /** 버튼에 표시할 라벨 */
    SLATE_ATTRIBUTE( FString, Label )

    /** 버튼 클릭 시 호출됨 */
    SLATE_EVENT( FOnClicked, OnClicked )

    /** 버튼에 넣을 컨텐츠 */
    SLATE_NAMED_SLOT( FArguments, FSimpleSlot, Content )

    /** 버튼이 호버 상태에서 표시되어야 하는지 여부 */
    SLATE_ATTRIBUTE( bool, ShouldAppearHovered )
SLATE_END_ARGS()
```

`SLATE_ATTRIBUTE`, `SLATE_EVENT`, `SLATE_NAMED_SLOT` 등 매크로들을 써서 버튼을 생성하고 있다.

스타일 문법 등 예시는 [언리얼 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/slate-overview-for-unreal-engine) 에 잘 나와 있다.

### UMG

UMG 는 블루프린트와 비주얼 에디터를 통해 쉽고 직관적으로 UI 를 설계할 수 있게 만든 시스템이다. **위젯 블루프린트** 를 이용해 디자인할 수 있으며, 화면 위에 드래그해서 여러 요소를 배치하는 것과 같이 설계할 수 있다.

사실 UMG 는 Slate 를 래핑한 것으로, UMG로 만들어도 내부적으로 Slate 코드가 호출되어 돌아간다. Slate 를 쓰기 편하게 만든 것일 뿐 같은 기술 기반이다.

위젯 블루프린트만으로 로직을 짜는 데 한계가 있을 때 다음 방법을 이용해 C++ 코드로 위젯을 제어할 수 있다.

> 1. 위젯 블루프린트 생성
> 2. `UserWidget` 상속해 C++ 클래스 생성
> 3. 위젯 블루프린트에 추가한 위젯과 **같은 이름으로** 변수를 만들고 `UPROPERTY(meta = (BindWidget))` 매크로 사용
> 4. 위젯 블루프린트의 부모 클래스를 C++ 클래스로 변경

이렇게 하면 C++ 코드와 위젯 블루프린트를 묶어서 사용할 수 있다. 다만 `BindWidget` 을 사용할 경우 변수의 이름이 위젯 블루프린트에 추가해둔 위젯과 다르면 컴파일 오류가 발생하게 된다. `UPROPERTY(meta = (BindWidgetOptional))` 로 매크뢰를 지정하면 컴파일은 정상적으로 진행하되 같은 이름의 위젯이 없을 때 디버그 메시지를 띄우게 할 수 있다.

### 위젯

UI 를 구성하는 구성 요소들로 버튼이나 진행바, 슬라이더 같은 것들이 바로 위젯이다. Slate 는 `SWidget`, UMG 는 `UWidget` 을 사용한다. 버튼, 슬라이더 등 자주 사용되는 것들은 구현되어 있어서 갖다 쓰기만 하면 되며 필요 시 위젯 클래스를 상속하여 커스텀 위젯을 만드는 것도 당연히 가능하다.

위젯과 캐릭터가 서로 데이터를 주고받아야 할 때 위젯 안에서 `GetCharacter` 나 `GetController` 를 사용하는 경우가 있는데, 위젯은  기본적으로 플레이어 컨트롤러가 소유하게 되므로 순환 참조 문제가 발생할 수 있어 하지 않는 것이 좋다. 대신 인터페이스나 델리게이트를 사용하는 것이 좋다.

***

## HUD

`AHUD` 클래스로 정의되어 있으며 언리얼 엔진의 초기 버전부터 지원하던 기능이다. 화면 위에 항상 표시되는 체력바 같은 걸 생각하면 이해가 쉽다. 정보 표시가 주 용도이며 대부분의 경우 플레이어와 상호작용할 수 없다. (클릭 등의 동작이 불가능하다는 말)

HUD 만의 고유 기능들이 있긴 하지만 UE5 에 이르러서는 HUD 클래스를 전혀 사용하지 않고도 UMG 만으로 모든 게임 UI를 만드는 것도 가능하다. `UserWidget` 을 `Add To Viewport` 를 통해 게임에 표시하는 순간 HUD의 일부로 작동한다.

꼭 HUD 클래스를 이용해 UI를 그리지 않더라도, 위젯들을 담아두는 컨테이너처럼 사용하는 방법도 있다.

게임 모드 클래스에서 기본 HUD 클래스를 지정할 수 있으며, 해두면 레벨 시작 시 설정한 HUD가 기본적으로 표시된다.

***

## 참고한 링크

[언리얼 5.4 공식 문서 - 유저 인터페이스 만들기](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/creating-user-interfaces-with-umg-and-slate-in-unreal-engine?application_version=5.4)

[언리얼 5.4 공식 문서 - 유저 인터페이스 및 HUD](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/user-interfaces-and-huds-in-unreal-engine?application_version=5.4)

[언리얼 엔진 개발자 포럼 - HUD and a simple widget - what the different?](https://forums.unrealengine.com/t/hud-and-a-simple-widget-what-the-different/483667)

[언리얼 엔진 개발자 포럼 - What is the appropiate relationship between HUD and Widgets?](https://forums.unrealengine.com/t/what-is-the-appropiate-relationship-between-hud-and-widgets/21138/4)

[명령형 프로그래밍 vs 선언형 프로그래밍](https://dmdwn3979.tistory.com/14)

[위키백과 - Indirection](https://en.wikipedia.org/wiki/Indirection)