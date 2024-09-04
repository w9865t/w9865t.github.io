---
title: "[UE5] 엔진 버전 업데이트 하는 방법"
date: 2024-09-04 0:00:00 +09:00
categories: [Unreal Engine 5, 기타]
tags: [unreal]     # TAG names should always be lowercase
description: "언리얼 엔진 5 버전 업데이트 하기"
---

![](/assets/img/posts/UE5.png)

## 시작하며

언리얼 엔진 5 는 새로운 버전이 출시되어도 자동으로 업데이트되지 않는다. 에셋이나 플러그인들과의 호환성 문제같은 게 발생하면 개발 일정에 큰 차질이 생기기 때문이다. 

그래서 이번 게시글에선 수동으로 언리얼 엔진 버전을 업데이트하는 과정을 정리해 보았다.

Visual Studio 를 사용할 때 기준으로, VSCode 등 다른 에디터를 사용중이라면 방법이 다를 수 있으므로 유의하자. 또 만약을 위한 백업은 꼭 해놓길 권장한다.

***

### 1. 최신 버전의 엔진 설치

![](/assets/img/posts/UE5-Engine-Version-Update/Install.png)

먼저 에픽게임즈 런처에서 라이브러리에 들어간 후, `+` 버튼을 눌러 새로운 버전의 엔진을 설치한다. 이 때 기존 버전은 만일을 위해 삭제하지 않고 그대로 둔다.

### 2. 프로젝트 폴더에서 파일 삭제

프로젝트 폴더에 들어가 다음 파일들과 폴더를 삭제한다.

> 1. .vs 폴더: 비주얼스튜디오 임시 파일
> 2. Binaries 폴더: 빌드된 바이너리 파일
> 3. Intermediate 폴더: 컴파일 과정에서 생성된 파일
> 4. `프로젝트 이름`.sln 파일: 비주얼스튜디오 솔루션 파일

위 파일, 폴더들은 빌드 시 다시 생성된다. 비주얼스튜디오에서 열어둔 창 정보같은 자잘한 설정같은게 사라질 수는 있다.

### 3. 버전 전환

![](/assets/img/posts/UE5-Engine-Version-Update/Attribute1.png)

![](/assets/img/posts/UE5-Engine-Version-Update/Attribute2.png)

`프로젝트 이름`.uproject 파일을 우클릭 후 `추가 옵션 표시`(윈도우 11일 경우에만), `Switch Unreal Engine Version` 을 차례로 누른다.

![](/assets/img/posts/UE5-Engine-Version-Update/SwitchVersion.png)

전환할 버전을 선택 후 `OK` 를 누르면 다시 비주얼스튜디오 솔루션 파일이 생성된다.

### 4. 빌드

`프로젝트 이름`.sln 파일을 열고 `Ctrl` + `Shift` + `B` 를 눌러 프로젝트를 다시 빌드한다.

만약 빌드 오류가 발생하거나 `Ctrl` + `F5` 단축키가 동작하지 않는다면 다음과 같이 솔루션 탐색기에서 `Games` 하단에 위치한 본인의 프로젝트를 우클릭 후 `시작 프로젝트로 설정` 을 선택하고 다시 시도해 본다.

![](/assets/img/posts/UE5-Engine-Version-Update/StartProject.png)

### 5. 이전 버전 엔진 삭제

![](/assets/img/posts/UE5-Engine-Version-Update/Remove.png)

빌드 및 호환성 테스트를 해 보고 문제가 없다면 에픽게임즈 런처에서 이전 버전을 제거해도 된다.