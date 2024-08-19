---
title: "[UE5] 공격 기능 구현 3 - 공격과 피격 매커니즘 구현하기"
date: 2024-08-11 0:00:00 +09:00
categories: [Unreal Engine 5, 공격 기능 구현]
tags: [unreal]     # TAG names should always be lowercase
description: "ApplyDamage를 사용한 공격과 피격 매커니즘 구현"
---

![](/assets/img/posts/UE5.png)

## 구현 목표
마우스 좌클릭으로 공격 애니메이션을 재생하고 다른 캐릭터에게 대미지를 입히는 기능 구현하기

## 구현해야 할 것들
>1. [마우스 좌클릭 입력 시 실행할 공격 함수 구현(UE5의 향상된 입력 사용)](https://w9865t.github.io/posts/UE5-Attack-Enhanced-Input/)
>2. [캐릭터가 사용할 무기 클래스 구현](https://w9865t.github.io/posts/UE5-Attack-Weapon-Class/)
>3. _**공격과 피격 매커니즘 구현 <- 오늘 할 것**_
>4. 공격 시 재생할 애니메이션과 몽타주 구현

저번에 만든 무기 클래스를 이용해서 피격 매커니즘을 구현해보려고 한다. 대충 생각해 보니까 다음과 같은 점들에 유의하면서 구현하면 될 것 같다.
>1. 무기와 충돌한 캐릭터에 설정한 대미지 입히기
>2. 무기를 장착한 캐릭터 본인에게 대미지를 입히지 않도록 하기
>3. 공격 상태가 아닐 때 닿은 것만으로 대미지를 입히지 않도록 하기

1. 무기의 대미지를 설정하는 것은 간단히 헤더 파일에 변수로 설정해도 되지만 더 많은 스탯을 붙이고 싶은 경우 커스텀 컴포넌트를 만들어서 붙이는게 더 깔끔할 수도  있다. 충돌한 캐릭터 판정은 저번에 `BoxTraceSinge` 을 사용해서 구현했고 대미지를 입히는 동작은 아마 `UGameplayStatics` 의 `ApplyDamage` 을 사용하면 될듯?
2. `SetOwner` 나 `SetInstigator` 을 사용해서 무기의 주인을 장착한 캐릭터로 지정한다. 이후 무기 클래스에서 `GetOwner` 로 오너를 얻어오고 `UKismetSystemLibrary` 의 `BoxTraceSingle` 에서 `ActorsToIgnore` 배열에 넣어주기만 하면 된다.
3. 애니메이션 몽타주에서 애님 노티파이를 사용하여 공격 모션이 시작될 때 콜리전을 켜고 공격 모션이 끝나면 끄도록 한다.

***

### 무기와 충돌한 캐릭터에 설정한 대미지 입히기

`BoxHitResult` 로 얻어온 `FHitResult` 변수를 통해 충돌한 액터를 알아내고 `UGameplayStatics` 의 `ApplyDamage` 를 사용할 것이다.

[언리얼 5.4 UGamePlayStatics 공식 문서](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Kismet/UGameplayStatics?application_version=5.4)

공식 문서를 보니 `ApplyPointDamage`, `ApplyRadialDamage` 같은 것도 있으니 상황에 맞게 쓰면 될듯 하다.

먼저 `GameplayStatics.cpp` 에 정의되어 있는 함수를 보자.

```cpp
float UGameplayStatics::ApplyDamage(AActor* DamagedActor, float BaseDamage, AController* EventInstigator, AActor* DamageCauser, TSubclassOf<UDamageType> DamageTypeClass)
{
	if ( DamagedActor && (BaseDamage != 0.f) )
	{
		// make sure we have a good damage type
		TSubclassOf<UDamageType> const ValidDamageTypeClass = DamageTypeClass ? DamageTypeClass : TSubclassOf<UDamageType>(UDamageType::StaticClass());
		FDamageEvent DamageEvent(ValidDamageTypeClass);

		return DamagedActor->TakeDamage(BaseDamage, DamageEvent, EventInstigator, DamageCauser);
	}

	return 0.f;
}
```

`DamagedActor` 의 `TakeDamage` 함수를 호출해주는 것을 알 수 있다.

`TakeDamage` 는 `AActor` 와 `APawn` 에 구현되어 있으므로 캐릭터 클래스에서 override 해서 쓰면 손쉽게 구현할 수 있다!

일단 무기 클래스에는 다음과 같이 구현했다. `BoxHitResult` 는 `FHitResult` 형 변수이다.
```cpp
if (BoxHitResult.GetActor())
{
	AActor* OverlapActor = BoxHitResult.GetActor();

	UGameplayStatics::ApplyDamage(
		OverlapActor, // 충돌한 액터
		20.f, // 대미지, 일단 20으로 해둠
		GetInstigatorController(), // 이벤트 Instigator 가져오기
		this, // 대미지의 원인, this(무기)
		AttackDamageType // UDamageType 클래스
	);
}
```

캐릭터 클래스 쪽 구현은 아까 쓴 대로 그냥 `TakeDamage` override 해서 구현했다.

캐릭터 체력과 같은 요소들은 보통 컴포넌트 클래스 하나 만들어서 관리하므로 컴포넌트에 현재체력 리턴하는 함수, 대미지 인수로 받아서 체력깎는 함수 하나씩 만들어 두고 쓰면 될듯 하다.

추가로 체력 0 됐을 때 Dead 상태로 지정하도록 구현하면 기본적인 기능은 다 들어간듯 하다.

***

### 무기를 장착한 캐릭터 본인에게 대미지를 입히지 않도록 하기

나의 경우 일단 무기 장착 기능은 구현하지 않았고 게임 시작과 동시에 무기를 스폰하여 장착하도록 해 뒀는데, 스폰하면서 무기의 Owner 와 Instigator 를 장착한 캐릭터로 지정하도록 구현했다.

```cpp
void Aproject1Character::SpawnAttachWeapon(TSubclassOf<AWeapon>& TargetWeapon, FName TargetName)
{
	if (TargetWeapon != nullptr)
	{
		AWeapon* WeaponToSpawnAttach;
		WeaponToSpawnAttach = GetWorld()->SpawnActor<AWeapon>(TargetWeapon, FVector::ZeroVector, FRotator::ZeroRotator);
		WeaponToSpawnAttach->Equip(this, TargetName);

		WeaponToSpawnAttach->SetOwner(this);
		WeaponToSpawnAttach->SetInstigator(this);
	}
}
```
다음은 무기 클래스의 `BeginPlay` 같은데서 무시할 배열에 넣어주면 무기 장착한 캐릭터와의 콜리전을 무시할 수 있다.
```cpp
	if (GetOwner())
	{
		ActorsToIgnore.Add(GetOwner());
	}
```

`ActorsToIgnore` 는 `BoxTraceSingle` 호출할 때 인수로 넘겨주면 된다.

***

### 공격 상태가 아닐 때 닿은 것만으로 대미지를 입히지 않도록 하기

이건 애니메이션 몽타주를 만든 후 노티파이를 추가해서 구현할 예정이므로 일단 애니메이션 구현한 다음에...

## 다음 목표
믹사모에서 주워온 애니메이션을 적용하기
개인적으로 아직 이해 못한 부분이 많아 두려운 작업이지만 열심히 해봐야지...
