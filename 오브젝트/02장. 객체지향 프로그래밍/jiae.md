## 📍 1. Overview key concept

## ✨ 2. Highlights & Takeaways
#### 영화 예매 시스템
개념 설명을 위해 영화 예매 시스템을 만들어보자. 영화당 할인 정책은 최대 1개이며, 할인의 조건은 여러 개가 될 수 있다. 할인 정책엔 금액 할인 / 비율 할인이 있고, 할인 조건엔 순번 조건 / 기간 조건이 있다.


#### 클래스가 아닌 객체를 먼저 고려하라
- 요구사항이 정립되고 코드를 짠다고 생각했을 때 <ins>가장 먼저 클래스를 떠올리기 쉽다.</ins> 하지만 객체지향은 객체 간의 협력을 통해 실현되므로 클래스가 아닌 객체에 초점을 맞춰야 한다. 이를 위해선
  1. 어떤 객체가 필요한지 고민해라 : 객체들이 어떤 상태와 행동을 가지는지 결정하자
  2. 객체를 협력하는 공동체의 일원으로 바라봐라 : 객체를 고립된 존재가 아닌 상호작용하는 협력자로 생각하면 설계가 유연해지고 확장 가능성이 커진다


#### 효율적인 협력
- 효율적인 협력을 위해선 객체들이 자율적인 존재가 되어야 한다. 내가 맡은 일은 스스로 처리하고 본인의 일이 아니라면 다른 객체에게 요청해야 한다. 이렇게 일의 경계를 명확히 하기 위해선 **캡슐화**가 필요하다.
  - 캡슐화하는 방법은 다음과 같다. 
    1. **접근 제어자 활용** : 외부에 노출이 필요한 부분만 공개하고 나머지는 감춘다. 무분별한 공개는 클라이언트 입장에서 너무 많은 정보를 알게 하고, 객체 입장에서도 외부의 간섭이 많아져 자율적인 객체가 될 수 없다. 일반적으로 객체의 상태는 숨기고 행동만 외부에 공개해야 한다. 
    2. **구현 은닉** : 인터페이스와 구현을 분리하자. 클라이언트가 요청할 수 있는 창구만 public하게 열고 그 안의 로직은 모두 private으로 처리한다. 클라이언트는 요청 창구가 필요할뿐, 내부 구현을 알 필요가 없다. 객체는 인터페이스를 바꾸지 않는 이상 외부에 미치는 영향 없이 내부 구현을 마음껏 변경할 수 있다.
- 객체는 다른 객체에 메시지를 전송/수신하며 협력한다. 이때 '메서드 호출'이 아닌 '메시지 송수신'이란 표현을 사용해 협력을 인지하자.


#### 유연한 설계 방법 : 상속과 다형성 활용하기
```java
public class Movie {
	private String title;
	private Duration runningTime;
	private Money fee;
	private DiscountPolicy discountPolicy;

	public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
		this.title = title;
		this.runningTime = runningTime;
		this.fee = fee;
		this.discountPolicy = discountPolicy;
	}

	public Money getFee() {
		return fee;
	}

	// 할인된 영화 가격을 계산한다.
	public Money calculateMovieFee(Screening screening) {
		return fee.minus(discountPolicy.calculateDiscountAmount(screening));
	}
}
```
영화 가격을 계산하는 calculateMovieFee 메서드를 보자. 이 메서드엔 어떤 할인 정책을 사용할지를 결정하는 코드가 없다. 그저 discountPolicy에 할인 금액 계산을 요청할 뿐이다. 구체적인 할인 정책 지정 없이 할인 금액 계산이 가능한 이유는 상속과 다형성 덕분이다.
이를 다이어그램으로 나타내면 다음과 같다.
[이미지]
여기서 Movie와 DiscountPolicy의 관계에 집중하자. 영화 계산엔 Amount / Percent DiscountPolicy가 필요하지만 코드상 연결되어있는 건 DiscountPolicy다. 어떤 할인 정책이 선택되는지는 Movie의 생성자에서 알 수 있다.
```java
Movie avatar = new Movie("아바타",
        Duration.ofMinutes (120), 
        Money. wons (10000), 
        new AmountDiscountPolicy (Money. wons (800), ...)); // 영화 생성 시, 할인 정책을 결정한다
```
즉 코드의 의존성과 실행 시점의 의존성이 서로 다를 수 있다. 코드상으론 Movie 클래스가 DiscountPolicy와 연결되어있지만, 실행시점엔 AmountDiscountPolicy에 의존하게 된다. 이러한 설계의 장점은 높은 유연성과 확장 가능성이지만 컴파일 시점과 실행 시점의 의존성이 달라지며 코드를 이해하기가 어려워진다는 단점이 있다.

그렇다면 코드상 존재하는 DiscountPolicy로의 의존성이 실행 시점엔 어떻게 AmountDiscountPolicy로 바뀔 수 있는 걸까? 상속이 이를 가능케 한다.

#### 상속
- 상속이란 기존 클래스인 super class가 가진 모든 속성, 행동을 새로운 클래스인 sub class 포함시킬 수 있는 특성을 말한다.
- 코드 중복을 제거하고 재사용하기 위해 가장 많이 사용되는 방법이다. 그러나 상속의 주목적은 업캐스팅을 통한 유연성 향상이다. 업캐스팅이란 자식 클래스가 부모 클래스를 대신하는 것이다. 자식 클래스는 상속을 통해 부모 클래스의 인터페이스를 물려받기 때문에, 즉 같은 메세지를 처리할 수 있기 때문에 부모 클래스 대신 자식 클래스가 사용될 수 있고, Movie의 discountPolicy 자리에 AmountDiscountPolicy가 할당될 수 있었던 것도 업캐스팅 덕분이다.

#### 다형성
- 다형성이란 동일한 메시지를 수신했을 때, 객체의 타입에 따라 다르게 응답할 수 있는 능력을 말한다. 
- 코드 상 Movie는 DiscountPolicy에 메시지를 전송하지만 실행 시점에 실제로 실행되는 메서드는 Movie와 협력하는 실제 클래스가 무엇인지에 따라 달라진다.
- 다형성 구현 방법은 매우 다양하지만 동적 바인딩(실행될 메서드가 컴파일 시점이 아닌 실행 시점에 결정됨)에 의해 다형성이 실현된다는 건 동일하다.
- DiscountPolicy는 인터페이스와 내부 구현을 모두 상속받게 만들었다. 만약 구현 없이 인터페이스만 물려주고 싶다면 <\<interface>>를 사용하자.

#### 추상화 : 상속과 다형성의 근간 원리
- 추상화란 공통으로 가질 수 있는 인터페이스를 정의하고, 구현의 결정권은 자식 클래스가 가지는 것이다.
- 추상화의 장점
  1. 요구사항을 높은 수준에서 표현할 수 있다 : "영화엔 금액/비율 할인이 있으며 각각은 기간/순번의 할인을 가진다" → "영화엔 할인 정책과 할인 조건이 있다"로 심플하게 표현할 수 있다.
  2. 설계가 유연해진다 : 새로운 할인 정책이 필요한 경우, 기존 DiscountPolicy 클래스 수정 없이 이를 상속하는 새로운 자식 클래스를 생성 / 구현하기만 하면 된다! (OCP)

#### 합성 : 코드 재사용
앞에서도 언급했다시피 상속은 코드 재사용을 위해 널리 쓰이는 방법이다. 하지만 두 가지 단점이 있는데 
1. 캡슐화를 위반한다 : 상속을 이용하기 위해선 부모 클래스의 내부 구조를 잘 알고 있어야 한다. 부모 클래스가 work 메서드를 step1 -> step2 과정으로 호출한다고 예를 들어보자. 자식 클래스는 부모 클래스의 코드를 재사용하기 위해 step2만 재정의 후 work를 step1 -> step2 과정으로 동일하게 호출해야 한다. 이때 부모의 구현을 알고 있어야 하므로 이는 강결합이며 부모 클래스가 변경되면 자식 클래스도 변경될 가능성이 높아진다. 과한 상속은 코드를 변경하기 어렵게 만든다.
2. 설계를 유연하지 못하게 만든다 : 상속은 부모 클래스 - 자식 클래스 사이의 관계가 컴파일 시점에 결정된다. 만약 실행 시점에 영화의 할인 정책을 금액 -> 비율 할인으로 바꿔야 하는 상황이 생겼다고 가정해보자. 상속을 사용하면 

================================ 여기서부터 보완 필요

이제 discountPolicy를 살펴보자

```java
public abstract class DiscountPolicy {
	private List<DiscountCondition> conditions = new ArrayList<>();

	public DiscountPolicy(DiscountCondition... conditions) {
		this.conditions = Arrays.asList(conditions);
	}

	public Money calculateDiscountAmount(Screening screening) {
		for (DiscountCondition each : conditions) {
			if (each.isSatisfiedBy(screening)) {
				return getDiscountAmount(screening);
			}
		}
		return Money.ZERO;
	}

	abstract protected Money getDiscountAmount(Screening Screening);
}
```
할인 정책엔 금액 할인 / 비율 할인이 있다. 두 정책을 AmountDiscountPolicy, PercentDiscountPolicy로 구현할 것이다. 두 클래스는 비슷한 코드를 가지므로 공통 코드를 보관할 장소가 필요한데 여기선 부모 클래스인 DiscountPolicy에 중복 코드를 두고 상속받게 할 것이다. 실제 애플리케이션에선 DiscountPolicy의 인스턴스를 생성할 필요가 없기 때문에 abstract class로 구현했다.

️✔ 이처럼 부모 클래스에 기본 알고리즘 흐름을 구현하고, 중간에 필요한 처리는 자식 클래스가 구현하도록 하는 디자인 패턴을 TEMPLATE METHOD 패턴이라고 한다.

영화 가격 계산에 참여하는 모든 클래스를 다이어그램으로 나타내면 다음과 같다.