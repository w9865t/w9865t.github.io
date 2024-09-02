---
title: "[UE5] TObjectPtr<> 이 도대체 뭐야?"
date: 2024-09-02 0:00:00 +09:00
categories: [Unreal Engine 5, 기타]
tags: [unreal]     # TAG names should always be lowercase
description: "원시 포인터를 대체하는 UE5의 표준"
---

![](/assets/img/posts/UE5.png)

***

## TObjectPtr<> 가 뭔데?

언리얼 엔진 5를 이용해 작업을 하면서 아무 생각 없이 포인터를 써오고 있었는데, 언리얼 엔진 5 마이그레이션 가이드에서 `TObjectPtr<>` 에 관한 내용을 보게 되었다.

[언리얼 엔진 5 마이그레이션 가이드](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/unreal-engine-5-migration-guide?application_version=5.4)

문서를 읽어 보니 `TObjectPtr<>` 는 기존의 원시 포인터를 대체하기 위한 것이지만, 반드시 대체해야 하는 것은 아니고 **선택 사항** 이라고 한다.

기존 원시 포인터와 똑같은 역할을 하고 자동으로 원시 포인트로 변환되지만, 다이내믹 액세스와 액세스 트래킹이 추가되었다고 한다. 사용법은 기존 원시 포인터와 동일하다.

```cpp
UObject* MyObject; // 이걸
TObjectPtr<UObject> MyObject; // 이렇게 바꾸라는 거다
```

### 그래서 왜 써야 하는데?

에픽이 문서에 써둔 개선점은 다이내믹 액세스와 액세스 트래킹이 전부이고, 나머지는 어떻게 기존 원시 포인터를 `TObjectPtr<>` 로 손쉽게 변환할 수 있는지에 관한 내용이다. 심지어 다음과 같은 설명까지 있다.

> 비에디터 빌드에 대한 원시 포인터로 변환되므로 출시된 제품의 동작 또는 성능에는 영향을 미치지 않지만, 에디터 빌드에서 개발할 때 경험이 향상될 수 있습니다.

당연히 새로 나온 것이니 기존에 쓰던 것보다야 좋겠지만, 코드 쓰는 입장에서는 장점은 그닥 체감되지 않고 성능 향상도 없고 필수 사항도 아니라고 하니 굳이 바꿔야 하나 싶기도 하다.

그래서 더 검색을 해 보니 왜 이걸 써야 하는지에 대한 [포럼 질문글](https://forums.unrealengine.com/t/why-should-i-replace-raw-pointers-with-tobjectptr/232781)도 있었는데, 요약해 보니 대충 이런 내용이었다.

> 1. 포인터 변수 생성 시 자동으로 `nullptr` 로 초기화가 되어서 편하다.
> 2. 에픽이 그렇게 하기로 결정했으니까 써야 한다.

1번은 확실히 장점이긴 한데 2번이 뭔소린가 싶을 수도 있다. 하지만 에픽에서 `TObjectPtr<>` 을 쓰기로 결정했고, 많은 엔진 클래스에서 이미 원시 포인터를 대체했다고 하니 앞으로 나올 기능들은 개발자가 `TObjectPtr<>` 을 사용하고 있다고 가정하고 나올 가능성이 높다. 물론 지금 당장은 선택 사항인 만큼 전환하지 않아도 문제가 없지만 앞날을 생각한다면 개발자들 입장에서는 좋든 싫든 따라갈 수밖에 없다.

### 단점 - UFUNCTION 의 매개 변수로 사용 불가

`TObjectPtr<>` 을 매개 변수로 사용하는 함수는 `UFUNCTION` 매크로를 사용할 수 없다! 포럼을 검색해 봤지만 왜 안되는 거냐고 성토하는 사람들만 있을 뿐 이유에 대한 명쾌한 해답은 없었다. 

에픽게임즈에서도 원시 포인터가 필요한 경우 다음과 같이 패스스루 함수를 만들어서 사용하라고 설명해 두었다.

```cpp
// 대부분의 사례에서 사용할 원시 포인터를 사용하는 원본 함수 시그니처:
static bool MyFunction(UObject* FirstParameter);

// 드물게 묵시적 변환을 사용할 수 없을 때 이 패스스루 함수를 사용합니다.
// TObjectPtr를 사용하는 패스스루 함수 시그니처:
static bool MyFunction(TObjectPtr<UObject> FirstParameter);

// (소스 파일의) 패스스루 함수 바디:
bool UMyClass::MyFunction(TObjectPtr<UObject> FirstParameter)
{
    return ShouldShowResetToDefault(FirstParameter.Get());
}
```

로직을 수정해서 매개변수로 포인터를 받는 상황을 만들지 말던가 정 안되겠으면 패스스루 함수를 만들어서 쓸 수밖에 없을 것 같다.

***

## 결론

세 줄 요약하면 다음과 같다.

> 1. 에픽이 앞으로 원시 포인터 대신 `TObjectPtr<>` 을 쓰기로 결정했다.
> 2. 지금 당장은 원시 포인터 그대로 써도 되지만 앞날을 생각하면 바꾸는게 좋다.
> 3. 다만 함수의 매개 변수에 쓰면 `UFUNCTION` 매크로를 못 쓴다는 점은 유의하자.