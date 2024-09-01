---
title: "[UE5] 애니메이션 노티파이란?"
date: 2024-09-01 0:00:00 +09:00
categories: [Unreal Engine 5, 애니메이션]
tags: [unreal]     # TAG names should always be lowercase
description: "애니메이션 노티파이란 무엇인가"
---

![](/assets/img/posts/UE5.png)

## 애니메이션 노티파이란?

[언리얼 5.4 애니메이션 노티파이 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/animation-notifies-in-unreal-engine?application_version=5.4)

애니메이션 노티파이는 애니메이션의 특정 구간에서 이벤트를 호출할 수 있는 수단이다. 예를 들어, 공격 애니메이션을 재생할 때 무기를 내리치는 순간에 이펙트 재생을 호출한다던가 하는 식으로 사용할 수 있다.

애니메이션 시퀀스나 애니메이션 몽타주에서 생성할 수 있으며, 트랙으로 그룹을 만들어 보기 쉽게 관리할 수 있다. 기본적으로 `1` 이라는 트랙이 생성되는데 이름 부분을 세 번 클릭하여 이름을 변경할 수 있고, 새 트랙을 추가할 수도 있다.

![](/assets/img/posts/UE5-Animation-Notify/AddTrack.png)



## 노티파이의 종류

### 스켈레톤 노티파이

![](/assets/img/posts/UE5-Animation-Notify/AddNotify.png)

시퀀스나 몽타주의 트랙 창에서 우클릭 후 `노티파이 추가` 를 선택했을 때 생성할 수 있는 가장 기본적인 노티파이다.  사운드 재생이나 파티클 이펙트 재생 같이 자주 쓰이는 것들은 언리얼에 구현되어 있기 때문에 그냥 갖다 쓰면 되고, 직접 만든 함수 같은 걸 트리거하고 싶다면 최상단의 `새 노티파이` 를 눌러 생성할 수 있다.

이렇게 생성한 노티파이는 **스켈레톤** 에 공유 데이터로 추가되기 때문에, 시퀀스나 몽타주에서 만든 노티파이가 별도의 처리 없이 애니메이션 블루프린트에 노출되므로 ABP에서 노티파이가 트리거됐을 때의 동작을 구현하면 된다.

![](/assets/img/posts/UE5-Animation-Notify/Skeleton.png)
_노티파이를 추가해 보면 스켈레톤에도 변경사항이 있다는 별 표시가 뜨는 것으로 알 수 있다._

예를 들어, `EnableCollision` 스켈레톤 노티파이를 몽타주에 다음과 같이 생성했다고 하자.

![](/assets/img/posts/UE5-Animation-Notify/EnableCollisionNotify.png)

그러면 별도의 처리를 해주지 않아도 애니메이션 블루프린트에서 다음과 같이 `EnableCollision` 노티파이 이벤트를 추가할 수 있게 되며, 노티파이가 트리거됐을 때의 동작을 구현할 수 있게 된다.

![](/assets/img/posts/UE5-Animation-Notify/ABP_EnableCollisionNotify.png)

### 노티파이 스테이트

노티파이 스테이트는 스켈레톤 노티파이와 거의 비슷하지만 단발성으로 트리거하는 스켈레톤 노티파이와는 다르게 일정 기간 동안 실행된다는 점이 다르다.

![](/assets/img/posts/UE5-Animation-Notify/AddNotifyState.png)

미리 구현된 기능들도 이펙트 등 일정 시간동안 실행해야 하는 것들이다.

![](/assets/img/posts/UE5-Animation-Notify/NotifyState.png)

추가해 보면 위와 같이 양 옆을 잡아 늘릴 수 있게 되어 있으므로 적절히 기간을 조절하고 실행하고 싶은 구간에 배치하면 된다.

## 노티파이 속성

![](/assets/img/posts/UE5-Animation-Notify/DetailPanel.png)

**팔로워일 때 트리거** 옵션은 [싱크 그룹](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/animation-sync-groups-in-unreal-engine) 을 사용할 때 선택할 수 있는 옵션이다. 싱크 그룹은 서로 길이가 다른 애니메이션을 블렌딩할 때 더 자연스럽게 동기화해주는 기능인데, 기준이 되는 리더 애니메이션을 정하고 나머지 애니메이션을 팔로워로 설정하여 싱크를 맞추게 된다. 이 때 기본적으로 리더 애니메이션에 설정된 노티파이만 트리거되는데, 이 옵션을 체크하면 팔로워 애니메이션의 노티파이도 트리거된다.

**몽타주 틱 타입**은 노티파이의 정확도나 트리거 순서와 관련된 옵션이다. `Queued` 와 `Branching Point` 의 두가지 옵션이 있고, 기본값은 `Queued` 인데 조금 부정확하지만 성능 면에서 이점이 있다. 분기를 나누는 것과 같이 게임에 크게 영향을 주는 노티파이라면 `Branching Point` 를 사용해 최대 정확도로 트리거할 수도 있다. 더 정확히 계산하는 만큼 당연히 성능은 `Queued` 보다 떨어진다. 성능, 정확도 중 하나를 선택해야 하는 트레이드 오프의 개념이라 보면 될 것 같다.

## 그 외

![](/assets/img/posts/UE5-Animation-Notify/CustomClass.png)

반복적으로 사용하는 기능일 경우 `AnimNotify` 를 상속한 커스텀 클래스를 만들어 구현해 두면 편하게 재사용할 수 있다.