## 📍 1. Overview key concept
<img src="https://github.com/user-attachments/assets/361bae56-1b01-48e7-a55d-58f3b2ba3e09" width=80%/>

## ✨ 2. Highlights & Takeaways
#### 0. 개요
- 2장에선 클래스, 상속, 지연 바인딩은 구현 측면에 집중했다. 하지만 **협력, 역할, 책임**에 대한 고려 없이 구현에만 집중한다면 변경하기 어렵고 유연하지 않은 코드가 탄생할 것이다.
- 시작 전 영화 예매 시스템을 돌아보자   
  <img src="https://github.com/user-attachments/assets/39c5d199-d05e-4406-a8d6-3f14e74ecad6" width="60%"/>
   

#### 1. 협력
- 객체는 `메세지` 전송, 응답을 통해 다른 객체와 협력한다. 객체는 다른 객체의 내부 구현에 접근할 수 없기 때문에 **메시지 전송을 통해서만 소통할 수 있다.**
- 객체의 `상태와 행동`은 협력에 따라 결정된다. **협력이 행동을 결정하고, 행동이 상태를 결정한다.** 영화 Movie 객체는 요금 계산의 책임을 가지고 있으며, 기본 요금과 할인 정책을 상태로 가지는 이유는 요금 계산이라는 행동에 필요한 정보들이기 때문
  ```java
  public class Movie {
    private Money fee;
    private DiscountPolicy discountPolicy;
  
    public Money calculateMovieFee(Screening screening) {
      return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
  }
  ```
   

#### 2. 책임 ⭐⭐⭐
- 객체 지향에서 가장 중요한 개념이다.
- 앞서 협력이 행동을 결정하고, 행동이 상태를 결정한다고 했다. 이때 **행동이 객체의 책임**이다.
- 객체의 책임은 `아는 것 + 하는 것`으로 구성된다. 객체는 본인이 맡은 책임을 다 하는데 필요한 정보를 알고 있어야 하며, 자신이 할 수 없는 작업에 대해선 누가 도와줄 수 있는지를 알아야 하는 책임이 있다.
- 책임 할당은 그 일을 처리하는데 필요한 지식과 방법을 가장 많이 아는 **정보 전문가**에게 한다.   
'영화 가격을 계산하라'는 메시지가 있다면 영화 가격과 관련된 정보를 가장 많이 가진 요금 계산 전문가 Movie 객체에 가격 계산의 책임을 할당할 수 있다.
  - 이처럼 객체지향 설계는 협력에 필요한 메시지 찾기 → 메시지를 처리할 객체 선택하기 → 필요한 메시지 찾기...의 반복적인 과정을 거친다. 이를 `책임 주도 설계(RDD)`라고 부름   
    <img src="https://github.com/user-attachments/assets/1c2639b6-95c3-43f7-abe2-d0cc7314207f" width="60%"/>   
  - **객체가 메시지를 선택하기(X) 메시지가 객체를 선택하기(O)** : 메시지가 객체를 선택해야 객체는 비로소 최소한의 인터페이스를 가지게 되고(무분별한 getter, setter 등을 안 만들게 됨), 추상적인 인터페이스(내부 프로세스를 노출하지 않을 수 있음)를 가질 수 있게 된다.
  - 경우에 따라 응집도/결합도의 관점에서 정보 전문가가 아닌 다른 객체에게 책임을 할당하는 것이 더 적절할 수도 있음!
   

#### 3. 역할
- 객체가 수행하는 책임의 집합을 역할이라고 한다. 실제로 협력을 모델링할 때는 특정 객체가 아닌 역할에 책임을 할당한다고 생각하자.
- 역할이란 동일한 책임을 가지는 객체들을 묶어서 지칭하는 말로 객체를 `추상화`한 것이다. 그렇게 때문에 추상화의 장점인 유연함, 재사용성을 가지며 상위 수준에서 협력 설명이 가능하다.   
  **① 유연함** : 동일한 책임을 수행하는 객체들은 동일한 역할을 수행하므로 서로 대체 가능하다. 또한 동일한 책임을 가지는 새로운 객체가 필요할 때, 기존 코드 변경 없이 새로운 행동을 추가할 수 있다.   
  **② 재사용성** : 할인 요금 계산이라는 동일한 책임을 가지는 객체가 2개 이상 있을 때, 역할을 통해 두 개의 협력을 하나로 통합하며 중복 코드를 제거할 수 있다.   
      <img src="https://github.com/user-attachments/assets/334c11e3-5457-4ecc-8cec-1796c053ba1c" width="60%"/>   
  **③ 상위 수준에서 협력 설명** : 영화의 금액 할인 정책, 비율 할인 정책, 순번 할인 조건, 기간 할인 조건 ... 을 구구절절 말할 필요없이 '영화엔 할인 정책과 여러 개의 할인 조건이 있다'로 간단하게 표현 가능하다.
- 역할을 구현하는 가장 일반적인 방법은 `추상 클래스`와 `인터페이스`다. 공통의 구현이 필요하면 추상 클래스, 책임 목록 정의만 필요하다면 인터페이스를 사용한다. 
- 역할의 특징은 다음과 같다.   
  **① 역할을 수행하는 객체는 하나 이상일 수 있다** : 하나의 배역을 여러 배우가 할 수 있음. 즉 대체 가능하다.   
  **② 객체는 다양한 역할을 가질 수 있다** : 배우는 여러 연극에 참여하면서 여러 배역을 연기할 수 있음. 특정한 협력 안에서는 일시적으로 하나의 역할만 보인다!   
  **③ 역할은 객체가 협력에 참여하는 동안에만 존재하는 일시적인 개념이다** : 연극에 출연하는 배우처럼 연극이 상영되는 동안에만 배역이란 개념이 존재하고, 연극이 끝나면 배우는 배역이라는 역할을 벗고 본연의 모습으로 돌아간다.
   

## 🧠 3. Reflections
**Q. 객체는 다양한 역할을 가질 수 있다고 했는데 SRP와 충돌하는 개념이 아닌가?**   
A. 책임과 역할은 객체의 관점에서, SRP 원칙은 클래스의 관점에서 바라봐야 한다. SRP는 변경의 이유가 하나임을 보장하는 클래스의 구현 방법이므로 두 개념은 충돌하지 않는다. (클래스는 설계도고 객체는 설계도로 만든 실제 주체)

**Q. 역할, 객체, 클래스의 관계?**   
A. DiscountCondition은 역할(행동 정의), PeriodCondition는 클래스(역할 구현), periodCondition1은 객체(클래스로부터 만든 실제 객체)로 이해할 수 있다.
```java
// 행동 정의
public interface DiscountCondition {
  boolean isSatisfiedBy(Screening screening);
}

// 행동 구현
public class PeriodCondition implements DiscountCondition {
  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;

  public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {...}

  public boolean isSatisfiedBy(Screening screening) {
    return screening.getStartTime().getDay0fWeek().equals(day0fWeek) &&
      startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
      endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
  }
}

// 객체 생성
PeriodCondition periodCondition1 = new PeriodCondition(DayOfWeek.MONDAY, LocalTime.of(10, 0), LocalTime.of(11, 59));
```   
 
> 초보들은 객체에 필요한 상태를 결정한 다음, 상태에 필요한 행동을 결정한다.

앞으로는 책임 주도 설계 과정을 떠올리며 코드 작성할 것
