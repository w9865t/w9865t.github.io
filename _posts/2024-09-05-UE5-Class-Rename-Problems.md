---
title: "[UE5] C++ 클래스 이름 변경 후 생긴 문제들과 해결 방법"
date: 2024-09-05 0:00:00 +09:00
categories: [Unreal Engine 5, 문제 해결]
tags: [unreal]     # TAG names should always be lowercase
description: "웬만하면 하지 말자"
---

![](/assets/img/posts/UE5.png)

## 시작하며

UE5 를 공부하며 아무 생각 없이 프로젝트 이름을 `project1` 으로 지었고 자연스럽게 캐릭터 클래스도 `project1Character` 가 되었다.

그런데 개발을 하다 보니 기본 캐릭터 클래스에서 상속하여 여러 클래스들을 만들 일이 많겠다고 느꼈고, 이름 역시 `BaseCharacter` 같은 게 더 적절하겠다고 생각했다.

그 결과 너무나 안이하게도 `project1Charcter` 의 이름을 `BaseCharacter` 로 바꾸려고 했고, 거하게 삽질을 하고야 말았다. 다시 바보짓을 안하기 위해 그 과정을 기록해 두고자 한다.

## 첫 번째 시도

처음엔 그냥 비주얼스튜디오에서 일괄 변경한 다음 자손 블루프린트 클래스는 에디터에서 부모클래스 재설정 해주면 되겠거니 하고 생각했다.

제일 먼저 소스 코드에서 `Ctrl` + `H` 를 이용해 `project1Character` 를 `BaseCharacter` 로 변경했고, `.vs`, `Binaries`, `Intermediate`, `project1.sln` 을 삭제한 후 프로젝트 파일을 재생성하고 빌드했다.

## 첫번째 문제 - 부모 클래스 변경 불능

![](/assets/img/posts/UE5-Class-Rename-Problems/Error.png)

빌드는 문제 없이 됐으나 원래 `project1Character` 을 상속해 만든 `BP_project1Character` 의 부모 클래스를 `BaseCharacter` 로 바꾸려고 보니 변경이 안 되는 문제에 직면했다...

부모 클래스는 **없음** 이라고 표기되었고, `BaseCharacter` 뿐만 아니라 `Actor` 나 `Pawn` 등 다른 언리얼 클래스도 선택지에 뜨지 않아 변경이 불가능했다.

### 해결 - 클래스 리다이렉트

안이하게 되겠거니 했던 게 안되니 당황스러웠지만 검색을 해 보니 다음과 같은 해결책이 있었다.

`Config/DefaultEngine.ini` 파일을 열고, 맨 끝에 다음 리다이렉트 코드를 추가한다.

```ini
[CoreRedirects]
+ClassRedirects=(OldName="project1Character", NewName="BaseCharacter")
```

그리고 다시 빌드하면 다음과 같이 부모 클래스가 **BaseCharacter** 로 리다이렉트 된 것을 볼 수 있다.

![](/assets/img/posts/UE5-Class-Rename-Problems/AfterRedirect.png)

리다이렉트 코드를 계속 남겨두긴 싫으니 이 상태에서 블루프린트 클래스를 **저장** 하고, `Config/DefaultEngine.ini` 파일 맨 끝에 추가했던 리다이렉트 코드를 지운 후 다시 빌드하면 블루프린트 클래스의 부모 관련 문제는 해결할 수 있다.

## 남은 문제

위 방법으로 블루프린트 클래스에서 발생하는 문제는 해결했지만, 월드에 배치한 캐릭터들에서 문제가 발생했다.

![](/assets/img/posts/UE5-Class-Rename-Problems/Outliner.png)

이 문제의 해결 방법은 찾지 못했고 결국 문제가 발생한 액터를 지운 후 수정한 블루프린트 클래스를 다시 배치하는 수밖에 없었다.

나의 경우 배치했던 수가 많지 않아 다행이었지만, 맵 곳곳에 잔뜩 배치해 둔 액터였다면 정말 막막했을 것이다.

## 결론

> 클래스 이름 변경 웬만하면 하지 말자.
{: .prompt-danger }

애초에 처음부터 이름을 잘 짓는게 좋겠지만 부득이하게 바꿔야 할 때는 새 클래스를 만들고 기존 클래스의 내용을 복붙한 뒤 의존 관계를 해결하는 것이 훨씬 바람직한 방법이라고 느꼈다.