잘 설계된 애플리케이션은 작고 응집도 높은 객체들로 구성된다.

이런 작은 객체들은 단독으로 작업을 수행하기보다, 다른 객체에게 도움을 요청한다.

이러한 협력을 위해 필요한 의존성을 유지하며, 변경을 방해하는 의존성은 제거하는 설계가 필요하다.

이 장에서는 유연한 설계를 위한 의존성을 관리하는 방법에 대해 살펴본다.

---

# 1. 의존성 이해하기

두 요소 사이의 의존성은 의존되는 요소가 변경될 때 의존하는 요소도 함께 변경될 수 있다는 것을 의미한다.

## 변경과 의존성

아래 PeriodCondition을 보자.

```java
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalDate startTime;
    private LocalDate endTime;

    public PeriodCondition(DayOfWeek dayOfWeek, LocalDate startTime, LocalDate endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(ChronoLocalDate.from(screening.getStartTime().toLocalTime())) <= 0 &&
                endTime.compareTo(ChronoLocalDate.from(screening.getStartTime().toLocalTime())) >= 0;
    }
}
```

이 클래스의 의존성을 도식화해보자.

<img width="785" height="286" alt="그림 8-2" src="https://github.com/user-attachments/assets/955ab979-0db1-4131-8eac-14c961628723" />

PeriodCondition 객체는 DiscountCondition의 오퍼레이션 명이 수정되거나 함수 인자 및 클래스 속성이 달라지면 변경될 여지가 존재한다.

여기서 **의존성의 근본적인 특성**을 알아볼 수 있는데,

**자신이 의존하는 대상이 변경될 때 함께 변경될 수 있다**는 것이다.

## 의존성 전이

앞서 살펴본 PeriodCondition의 의존성 전이 경우의 일부를 표현해보자.

<img width="966" height="105" alt="그림 8-4" src="https://github.com/user-attachments/assets/1b46b2d4-f739-4940-8d98-541ab4583eee" />

여기서 주의할 것이, **의존성은 함께 변경될 “가능성”을 의미**하기 때문에 모든 경우에 의존성이 전이되는 것은 아니다.

의존성이 실제로 전이될지 여부는 변경의 방향과 캡슐화의 정도에 따라 달라진다.

## 런타임 의존성과 컴파일타임 의존성

의존성과 관련해서 다뤄야 하는 두가지는 **런타임 의존성**과 **컴파일타임 의존성**이다.

객체지향 어플리케이션에서 런타임의 주인공은 객체이다.

코드 관점에서 주인공은 클래스이다. (컴파일타임)

Movie 객체가 추상 클래스 / 구체에 의존하는 상황을 보자.

- Movie가 추상체에 의존

<img width="752" height="242" alt="그림 8-5" src="https://github.com/user-attachments/assets/3f25c1ba-c6a4-46be-89fc-af491d40fd16" />

- Movie가 구체에 의존

<img width="712" height="215" alt="그림 8-6" src="https://github.com/user-attachments/assets/5883190c-07b2-4653-afb0-cdd043123cef" />

구체에 의존하면, Movie에서 다른 할인 정책이 필요한 경우 코드의 수정이 필요하다.

이는 변경에 유연하지 못하며, 버그의 소지를 높힌다.

반면 추상체에 의존하면, Movie가 런타임에서 어떤 구체와 협력할지 모르기에 동일한 소스코드로 다양한 할인 정책 구체와 협력할 수 있다.

## 컨텍스트 독립성

앞선 Movie와 할인 정책 의존 관계에서 보았듯,

구체적인 클래스를 알면 알수록 그 클래스가 사용되는 특정한 문맥에 강하게 결합된다.

클래스가 특정한 문맥에 강하게 결합될수록 다른 문맥에서 사용하기가 어려워진다.

**특정한 문맥에 대해 최소한의 가정**만으로 이루어진다면 다른 문맥에서 **재사용하기 더 수월**해진다.

- 이를 **컨텍스트 독립성**이라고 부른다.

## 의존성 해결하기

컴파일타임 의존성을 **실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체**하는 것을 **의존성 해결**이라고 부른다.

일반적으로 아래 세 가지 방법을 사용한다.

### 1. 생성자를 통한 의존성 해결

생성자를 통한 의존성 해결은 **객체의 필수 의존성을 명시적으로 정의**하는 방법이다.

**객체 생성 시점에 모든 의존성을 주입**하여 안정적이고 테스트하기 쉬운 코드를 작성할 때 유용하다.

### 2. setter 메서드를 통해 의존성 해결

객체를 생성한 이후에도 의존하고 있는 대상을 변경할 수 있는 가능성을 열어 놓고 싶은 경우 유용하다.

단점은 객체가 생성된 후에 협력에 필요한 의존 대상을 설정하기 때문에 객체를 생성하고 의존 대상을

설정하기 전까지는 객체의 상태가 불완전할 수 있다.

### 3. 메서드 실행 시 인자를 이용해 의존성 해결

지속적으로 의존 관계를 맺을 필요 없이 메서드가 실행되는 동안만 일시적으로 의존 관계가 존재하도

무방한 경우와

메서드가 실행될 때마다 의존 대상이 매번 달라져야 하는 경우에 유용하다.

---

# 2. 유연한 설계

모든 의존성이 나쁜 것은 아니다.

문제는 의존성의 존재가 아니라 의존성의 정도다.

결합도가 높은 나쁜 의존성의 경우를 예시를 통해 확인해보자.

```java
public class Movie {
		private PercentDiscountPolicy percentDiscountPolicy;
		
		public Movie(PercentDiscountPolicy percentDiscountPolicy) {
				this.percentDiscountPolicy = percentDiscountPolicy;
		}
}
```

이 코드는 Movie를 percentDiscountPolicy라는 **구체적인 클래스에 의존**하게 만들었기 때문에 다른 할인 정책이 필요한 문맥에서 **재사용 가능성을 없애** 버렸다.

해결 방법은 구체가 아닌 추상적인 타입과 이해할 수 있는 메시지를 정의해 재사용 가능하도록 하는 것이다.

바람직한 의존성이란 **컨텍스트에 독립적인 의존성을 의미**하며 **다양한 환경에서 재사용될 수 있는 가능성**을 열어두는 의존성이다.

## 지식이 결합을 낳는다.

결합도의 정도는 한 요소가 자신이 의존하고 있는 다른 요소에 대해 알고 있는 정보의 양으로 결정된다.

더 많이 알수록 더 많이 결합된다. 

더 많이 알고 있다는 것은 더 적은 컨텍스트에서 재사용 가능하다는 것을 의미한다. 

결합도를 느슨하게 유지하려면 협력하는 대상에 대해 적게 알아야 한다.

## 추상화에 의존하라

추상화를 사용하면 현재 다루고 있는 문제를 해결하는 데 불필요한 정보를 감출 수 있다.

중요한 것은 실행 컨텍스트에 대해 알아야 하는 정보를 줄일수록 결합도가 낮아진다는 것이다.

## 명시적인 의존성

### 숨겨진 의존성

```java
public class Movie {
		private DiscountPolicy discountPolicy;
		
		public Movie(...) {
				this.discountPolicy = new AmountDiscountPolicy(...);
		}
}
```

인스턴스 변수가 추상 클래스이지만 구체 클래스를 생성자에서 직접 선언함으로써 결합도가 불필요하게 높아졌다.

이 예제에서 알 수 있듯이 결합도를 느슨하게 만들기 위해서는 인스턴스 변수의 타입을 추상 클래스나 인터페이스로 선언하는 것만으로는 부족하다.

### 의존성 명시

이를 생성자에서 의존성을 명시적으로 드러냄으로써 해결할 수 있다.

```java
public class Movie {
		private DiscountPolicy discountPolicy;
		
		public Movie(DiscountPolicy discountPolicy) {
				this.discountPolicy = discountPolicy;
		}
}
```

두가지 예시를 비교하며 알 수 있는점은 의존성이 명시적이지 않으면 의존성을 파악하기 위해 내부 구현을 직접 살펴볼 수밖에 없다는 것이다.

- 문제는 다른 컨텍스트에서 재사용하기 위해서 내부 구현을 변경해야하며, 이는 버그의 가능성을 높힌다.

결론은 의존성은 숨기지 말아야 하며 **퍼블릭 인터페이스를 통해 명시적으로 의존성이 드러나야 한다.**

## new는 해롭다.

new 키워드는 좋은 설계에 해롭다.

클라이언트가 클래스를 생성하기 위해 어떤 정보가 필요한지 구체적으로 알아야하기 때문이다.

이는 결합도를 높히게 되며 여러 컨텍스트에서 재사용성을 떨어뜨리는 결과를 낳는다.

## 가끔은 생성해도 무방하다.

협력하는 기본 객체를 설정하고 싶은 경우 (예 - 자주 사용되는 디폴트 클래스) 직접 생성하는 방식이 유용하기도 하다.

디폴트 클래스 외 다른 클래스도 가끔 사용된다면 생성자 체이닝 방식을 고려할 수 있다.

```java
public class Movie {
		private String title;
		private DiscountPolicy discountPolicy;
		
		public Movie(String title) {
				this(title,new AmountDiscountPolicy(...));
		}
		
		public Movie(String title,DiscountPolicy discountPolicy) {
				this.title = title;
				this.discountPolicy = discountPolicy;
		}
}
```

혹은 변경의 여지가 거의 없는 표준 클래스의 경우도 직접 생성하는 방식이 유용하다.

- 예 - JDK 표준 컬렉션 라이브러리에 속하는 ArrayList

## 조합 가능한 행동

어떤 객체와 협력하느냐에 따라 객체의 행동이 달라지는 것은 유연하고 재사용 가능한 설계가 가진 특징이다.

유연하고 재사용 가능한 설계는 객체가 어떻게가 아닌 무엇을 하는지를 표현하는 클래스들로 구성된다.

이를 통해 객체가 어떤 객체와 연결됐는지를 보는 것만으로도 객체의 행동을 쉽게 이해할 수 있다.

즉, 선언적으로 객체의 행동을 정의할 수 있는 것이다.

- 선언적의 사전적 정의 : **어떤 사실이나 의지를 명확히 밝히는 것**
- 객체지향의 선언적 : 어떻게 동작하는지 드러내지 않고 **무엇을 하는지를 중심으로 객체의 책임과 협력을 표현**
