---
title: "[UE5] 공격 기능 구현 5 - 애니메이션 구현 (애니메이션 몽타주, 애니메이션 블루프린트란?)"
date: 2024-08-26 0:00:00 +09:00
categories: [Unreal Engine 5, 공격 기능 구현]
tags: [unreal]     # TAG names should always be lowercase
description: "애니메이션 몽타주, 애니메이션 블루프린트란 무엇인가"
---

![](/assets/img/posts/UE5.png)

## 구현 목표
마우스 좌클릭으로 공격 애니메이션을 재생하고 다른 캐릭터에게 대미지를 입히는 기능 구현하기

## 구현해야 할 것들
>1. [마우스 좌클릭 입력 시 실행할 공격 함수 구현(UE5의 향상된 입력 사용)](https://w9865t.github.io/posts/UE5-Attack-Enhanced-Input/)
>2. [캐릭터가 사용할 무기 클래스 구현](https://w9865t.github.io/posts/UE5-Attack-Weapon-Class/)
>3. [공격과 피격 매커니즘 구현](https://w9865t.github.io/posts/UE5-Attack-Mechanism/)
>4. [공격 시 재생할 애니메이션과 몽타주 구현 1편](https://w9865t.github.io/posts/UE5-Attack-Root-Motion-Animation/)
>5. **_공격 시 재생할 애니메이션과 몽타주 구현 2편 <- 오늘 할 것_**

>1. 루트 본, 루트 모션 애니메이션이란 무엇인가?
>2. Mixamo에서 다운받은 에셋을 언리얼 엔진에서 활용하는 방법
>3. 애니메이션 몽타주와 애니메이션 블루프린트

저번 포스트에서 1번과 2번에 대해 간단히 다루었고, 오늘은 3번에 대해 써보려고 한다.

***
### 애니메이션 몽타주란?
[언리얼 5.3 애니메이션 몽타주 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/animation-montage-in-unreal-engine?application_version=5.3)

5.4 버전 문서 번역본이 없어서 5.3 문서를 참조했다.

> 애니메이션 몽타주, 즉 몽타주 를 사용하면 블루프린트로 여러 애니메이션 시퀀스 를 단일 에셋으로 결합하고 재생을 제어할 수 있습니다.
>***
>언리얼 공식 문서

여러 애니메이션들을 하나로 묶어서 특정 상황에 맞춰 재생할 수 있도록 만든 것이다.

예를 들어, 콤보 액션을 만들고 싶다면 각각의 애니메이션을 묶어 하나의 몽타주로 만들고, 조건에 따라 연이어 재생하도록 만들 수 있다.

연이어 사용하는 애니메이션이 아니더라도 같은 상황에 여러 애니메이션을 무작위로 골라 쓰고 싶을 때도 활용할 수 있다.

말만 들어선 잘 모르겠으니 실제로 한번 만들어 보자.

#### 공격 애니메이션 몽타주

두 개의 공격 애니메이션 중 하나를 무작위로 골라 재생하는 몽타주를 만들어 보자.

![](/assets/img/posts/UE5-Attack-Montage-Blueprint/MontageWindow.png)

위 사진은 공격에 쓸 애니메이션 몽타주 `AM_AttackMontage` 를 생성한 모습이다.

여기서 빨간색 네모 쪽의 공간에 애니메이션 에셋을 드래그 앤 드롭으로 배치할 수 있다. 위 사진은 두 개의 에셋 `AS_Attack` 과 `AS_Attack2` 를 배치한 것이다.

기본적으로 `DefaultGroup.DefaultSlot` 이라는 슬롯에 배치될텐데, 여기서 슬롯은 애니메이션을 재생할 **캐릭터의 부위** 라고 이해하면 쉽다.

예를 들어, `UpperBody` 라는 슬롯을 만들어서 공격 애니메이션을 할당하고 적절히 블렌딩을 해 주면, 상반신은 공격 애니메이션을 재생하면서 하반신은 다른 걸 재생할 수 있다. 이해하기 쉬운 예시로 **달리면서 공격** 하는 동작을 구현하고 싶을 때 유용하게 사용할 수 있다. 기본값인 `DefaultSlot` 의 경우 캐릭터의 전신에 애니메이션을 적용한다.

일단 지금은 기본 슬롯 그대로 두고, 애니메이션에 섹션을 붙여 보자. 여러 애니메이션 중 하나를 골라서 재생하려면 각 애니메이션을 뭐라 부를지 이름이 필요할 텐데 그게 바로 섹션이다.

초록색으로 배치된 에셋 위에서 마우스 우클릭 후 새 몽타주 섹션을 선택하여 추가할 수 있다. 나는 각각 `AttackSection1`, `AttackSection2` 라고 붙였다.

마지막으로 노란색 네모 쪽의 공간을 보면 기본적으로 섹션들이 화살표로 연결되어 있을 것이다. 그 순서대로 섹션을 자동 재생하겠다는 뜻인데, 콤보 액션을 만들고 싶을 경우 사용하면 된다. 나는 섹션 딱 하나만 재생하도록 만들 생각이므로 `지우기` 를 눌러 연결을 끊어 주었다.

다음으로 몽타주를 재생하는 함수를 만들었다.

```cpp
void Aproject1Character::PlayAttackMontage()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && AttackMontage)
	{
		// AttackMontage는 BP_캐릭터에서 설정됨
		AnimInstance->Montage_Play(AttackMontage);

		// TArray에 재생할 섹션명 추가
		TArray<FName> SectionArray;
		SectionArray.Add(FName("AttackSection1"));
		SectionArray.Add(FName("AttackSection2"));

		// 랜덤으로 선택하여 재생
		if (!SectionArray.IsEmpty())
		{
			int32 RandomIndex = FMath::RandRange(0, SectionArray.Num() - 1);
			FName SelectedSection = SectionArray[RandomIndex];

            // 어떤 섹션이 선택되었는지 로그에 표시해 보기
			FString str = SelectedSection.ToString();
			UE_LOG(LogTemp, Warning, TEXT(": %s"), *str);

			AnimInstance->Montage_JumpToSection(SelectedSection);
		}
	}
}
```

배열에 두 개의 섹션을 담고, 그 중 하나를 랜덤으로 선택해서 재생하는 함수이다.

여기서 주의할 점은 언리얼 엔진에선 C++ 표준 라이브러리 대신 언리얼 엔진에서 제공하는 전용 기능들을 활용해야 한다는 점이다. 그래야 엔진의 가비지 컬렉터가 인식해서 관리해 줄 수 있고 무엇보다 게임 개발에 자주 쓰는 기능들 위주로 구현되어 있어 편하다. 위 코드에선 `std::vector` 가 아니라 언리얼 엔진 전용 `TArray`를, `cmath` 대신 `FMath` 를 사용하였다. 

```cpp
void Aproject1Character::Attack()
{
	// 애니메이션 몽타주 재생
	PlayAttackMontage();
}
```

다음으로 Enhanced Input과 연동된 `Attack` 함수에서 재생해주도록 처리하면 된다.

그런데... 여기까지 작업하고 언리얼 에디터로 돌아가서 시험해 보면 아마 아무 일도 일어나지 않을 것이다. 아까 `DefaultSlot` 에 애니메이션 에셋을 배치했던 것이 기억나는가? 여기서 애니메이션 블루프린트를 사용해서 `DefaultSlot` 을 사용하도록 해줘야 한다.

***

### 애니메이션 블루프린트란?
[언리얼 5.4 애니메이션 블루프린트 공식 문서](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/animation-blueprints-in-unreal-engine?application_version=5.4)

> 애니메이션 블루프린트(Animation Blueprints) 는 시뮬레이션 또는 게임플레이 도중 스켈레탈 메시(Skeletal Mesh) 의 애니메이션을 제어하는 특수 블루프린트 입니다. 그래프(Graphs) 는 애니메이션 블루프린트 에디터(Animation Blueprint Editor) 안에서 편집되며, 여기서 애니메이션을 블렌딩하거나 스켈레톤의 본을 제어하거나, 각 프레임에 사용할 스켈레탈 메시의 최종 애니메이션 포즈를 정의할 로직을 생성할 수 있습니다.
>***
>언리얼 공식 문서

말 그대로 애니메이션의 블루프린트(설계도) 라 생각하면 이해하기 쉽다. 캐릭터의 상태에 따라서 어떤 애니메이션을 재생할 지 결 정하는 설계도이며, 서로 애니메이션 간 전환될 때 자연스럽게 섞어주는 블렌딩 기능도 지원한다.

애니메이션 블루프린트에는 이벤트그래프 말고도 **애님 그래프** 라는 탭이 있다. 바로 이걸 통해 상태에 따라 재생할 애니메이션을 고르는 동작을 어떻게 구현하게 된다.

![](/assets/img/posts/UE5-Attack-Montage-Blueprint/EGAG.png)

이벤트그래프는 다른 블루프린트 클래스에서 본 것과 별 차이가 없지만, 애님 그래프에서는 오토마타 수업 때 들었던 **상태 기계 (State Machine)** 라는 개념이 등장한다. 뭔가 있어보이는 이름이지만 여기선 게임의 상태에 따라 어떻게 애니메이션을 재생할 지 선택하는 로직이라 이해하고 넘어가도 된다. 

![](/assets/img/posts/UE5-Attack-Montage-Blueprint/StateMachine.png)

위 사진은 내가 구상해 본 상태 기계이다. 크게 캐릭터가 땅 위에 있을 때, 공중에 떠 있을 때, 떨어지다가 땅에 착지할 때 3개의 상태로 나누고, 땅 위에 있을 땐 가만히 서 있을 때(Idle), 걷고 있을 때, 달리고 있을 때로 나누었다. 이렇게 상태 기계를 통해 결정된 애니메이션은 마지막으로 **슬롯** 을 통해 드디어 캐릭터에 적용되게 된다.

![](/assets/img/posts/UE5-Attack-Montage-Blueprint/UseAnimBP.png)

애니메이션 블루프린트를 만들었다면 캐릭터 블루프린트로 돌아가 애니메이션 모드를 `Use Animation Blueprint` 로 변경하고, 방금 만든 애니메이션 블루프린트를 고르면 진짜 끝이다.

![](/assets/img/posts/UE5-Attack-Montage-Blueprint/AttackDemo.gif)

랜덤으로 선택하는 동작, 애니메이션 재생 모두 문제 없이 동작하는걸 확인했다.

여기서 마우스를 광클하면 기존 애니메이션이 중단되고 바로 다음 애니메이션이 재생되는 문제가 있는데 이건 `enum` 을 사용해 캐릭터의 상태(공격중인지, Idle인지, 죽었는지 등) 을 관리해주고 Enhanced Input 과 연결된 공격함수에서 계속 공격하지 못하게 수정해주면 된다. 자세한 구현에 대해선 나중에 다시 써 볼 생각이다.

***

## 다음 목표

노티파이를 어떻게 설정했는지, 캐릭터 체력 컴포넌트나 enum은 어떻게 했는지 같이 너무 상세한 것까진 쓰지 않았는데 이런 자잘한 것까지 전부 다듬은 다음 깃허브에 첫 커밋을 할 생각이다.

그 다음은 UI 부분을 만들어보려고 한다. 이유는 눈에 이쁘게 보이면 더 의욕이 나서.........
> 1. 플레이어의 체력 등을 표시하는 HUD
> 2. 적의 체력 등을 표시하는 UI. 가까이 가면 나타나고 멀어지면 사라지는 동작

그다음 적의 AI 동작(경로탐색, 폰 센싱 컴포넌트 사용한 탐지 등) 을 구현해볼 생각.