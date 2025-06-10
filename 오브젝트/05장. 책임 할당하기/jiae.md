## 📍 1. Overview key concept
- 협력, 책임
- 변경
- GRASP : 도메인 개념, 정보 전문가
- 다형성
- 합성
- 자율성


## ✨ 2. Highlights & Takeaways
#### 0. 개요
- 책임에 초점을 맞춰 설계할 때 가장 어려운 건 어떤 객체에게 어떤 책임을 할당할지 결정하기가 쉽지 않다는 것이다. **상황과 문맥**에 따라 최선의 방법은 달라진다.
- 이번 장에 등장하는 **GRASP 패턴**은 책임 할당 어려움 해결의 답을 제시해 줄 것이다. 다양한 기준에 따라 책임을 할당하고 결과를 트레이드오프할 수 있는 기준을 알 수 있게 될 것!

#### 1. 책임 주도 설계를 향해 ⭐️
데이터 중심 설계 -> 책임 중심 설계로 전환하기 위해선 다음 두 원칙을 따라야 한다
1. **데이터보다 행동을 먼저 결정하라** : (1) 이 객체가 수행해야 하는 책임이 무엇인지 결정한 후에 (2) 이 책임을 수행하는데 필요한 데이터를 결정하자 👍
2. **협력이라는 문맥 안에서 책임을 결정하라** : 메시지가 객체를 선택하게 해야 한다. 만약 객체에 할당된 책임이 협력에 어울리지 않는다면 그 책임은 나쁜 것이다 👎 <ins>객체의 입장에선 책임이 조금 어색해보여도 협력에 적합하다면 그 책임은 좋은 것 !</ins>


#### 2. 책임 할당 기법 : GRASP 패턴
- GRASP는 General Responsibility Assignment Software Pattern의 약자로 객체에게 책임을 할당할 때 지침으로 쓸 수 있는 원칙을 패턴화해서 정리한 것이다.
- GRASP 패턴은 다음과 같다.
1. **도메인 개념에서 출발하라** ⭐️ : 설계 전 도메인에 대해 대략적으로 그려보자. 도메인을 그려보면 책임을 할당하기가 수월해진다. (그러나 도메인 개념 정리에 너무 많은 시간을 들이진 말 것! 우리에게 필요한 건 완벽한 도메인 모델이 아닌(존재하지도 않음) 구현에 도움이 되는 도메인 모델이다.)

   <img src="https://github.com/user-attachments/assets/5d00c4f9-c714-498c-8849-dbc26f3f164b" width="70%"/>

3. **정보 전문가에게 책임 할당하라** : 메시지를 정하고 그 메시지를 처리할 정보를 아는 객체에게 책임을 할당하자. 응집도는 높이고 결합도는 낮출 수 있다. 단 책임을 수행하는 객체가 정보를 '알고'있다고 해서 그 정보를 '저장'하고 있을 필요는 없다. 객체는 해당 정보를 제공할 수 있는 다른 객체를 알거나 필요한 정보를 계산해서 제공해도 된다.
4. **LOW COUPLING, HIGH COHENSION 패턴** ⭐️ : 구현의 방법은 다양하지만 결합도는 낮추고 응집도는 높이는 방향을 선택해라. 재사용성은 높이면서 변화의 영향도는 작아진다.
5. **창조자에게 객체 생성 책임을 할당하라** : 객체 A를 생성해야 할 때 어떤 객체에게 객체 생성의 책임을 할당해야 할까? 다음 조건을 최대한 많이 만족하는 객체에게 책임을 할당하자
  - B가 A 객체를 포함하거나 참조함
  - B가 A 객체를 기록함
  - B가 A 객체를 긴밀하게 사용함
  - B가 A 객체를 초기화하는데 필요한 데이터를 가지고 있다

   → 영화 예매의 결과인 Reservation 객체 생성의 책임은 예매 정보를 생성하는 데 필요한 영화, 상영시간, 상영순번 등에 대한 정보 전문가이며, 예매 요금을 계산하는데 필수인 Movie도 알고있는 Screening에게 책임을 할당할 수 있다   
   <img src="https://github.com/user-attachments/assets/7dafc342-5c1d-4e57-9b49-1fb6f717447d" width="60%"/>



#### 3. 구현을 통해 협력과 책임의 동작을 확인하기
1. **변경의 이유에 따라 클래스를 분리하기** ⭐️ : 하나 이상의 변경 이유를 가지는 순간 객체의 응집도는 낮아진다. 서로 연관성 높은 기능, 데이터만 뭉쳐놓을 수 있도록 클래스를 분리해라.
   ```
   [변경이 일어날 것 같은 클래스 감지하는 방법]
   1. 인스턴스 변수가 초기화되는 시점을 확인해봐라 : 응집도가 높으면 인스턴스 생성 시에 모든 속성을 함께 초기화할 거고, 낮으면 일부는 초기화하고 일부는 초기화하지 않은 상태로 남겨진다. 함께 초기화되는 속성을 기준으로 코드를 분리할 것.
   2. 메서드들이 인스턴스 변수를 어떻게 사용하는지 확인해봐라 : 모든 메서드가 객체의 모든 속성을 사용한다면 클래스의 응집도는 높은 것이다.
   ```
3. **다형성을 이용해 클래스 분리하기** : 객체 타입에 따라 다른 행동을 가지고 있다면, 타입을 분리하고 변화하는 행동을 각 타입의 책임(퍼블릭 인터페이스)로 할당하라. 응집도가 높아진다. 분기문을 이용한다면 조건 추가/삭제마다 코드 수정이 필요하며 이는 변경에 취약한 설계다.
4. **변경으로부터 보호하기** : 변경될 가능성이 높다면 캡슐화해라
5. **상속 대신 합성을 사용해라** ⭐️ : 새로운 구현체가 필요해도 추상은 수정할 필요가 없어진다. 할인 정책을 퍼센트 할인에서 금액 할인으로 바꿔야 한다면 상속으로 구현했을 때와 합성으로 구현했을 때의 차이는 다음과 같다.
    ```java
    // 상속으로 구현한 경우 런타임에 Movie 인스턴스 자체의 상속 관계를 변경할 수 없으므로 새로운 인스턴스를 생성하고 모든 정보를 다시 복사해야 한다
    Movie titanic = new AmountDiscountMovie("타이타닉",
        Duration.ofMinutes(180),
        Money.wons(10000),
        Money.wons(1000));
    Movie updatedTitanic = new PercentDiscountMovie("타이타닉", // 개념적으로 같은 '타이타닉' 영화지만, 물리적으로는 다른 두 객체
        Duration.ofMinutes(180), 
        Money.wons(10000), 
        0.1); 
    
    // 합성을 통한 변경의 유연성
    Movie movie = new Movie("타이타닉", 
        Duration.ofMinutes(120),
        Money.wons(10000),
        new AmountDiscountPolicy(...));
    movie.changeDiscountPolicy(new PercentDiscountPolicy(...)); // 새로운 인스턴스를 생성할 필요 없이 할인 정책만 바꿔 끼울 수 있음
    ```
    ```
    [상속과 합성] ⭐️⭐️⭐️
    상속 
     - 부모 클래스의 로직을 하위 클래스에서 물려 받아 코드를 재사용하는 방법
     - is-a 관계 (예: Dog is-a Animal, Square is-a Rectangle, ArrayList is-a List, AmountPolicy is-a DiscountPolicy)
     - 구현에 의존 (부모의 구현에 강결합)
     - 상속 관계는 컴파일 시점에 결정된다
    
     합성
     - 클래스 A가 클래스 B의 기능을 가져다 쓰는 방법
     - has-a 관계 (예 : Car has-a Engine, Computer has-a CPU, UserServiceImpl has-a UserRepository, Movie has-a DiscountPolicy)
     - 퍼블릭 인터페이스에 의존 (세부 구현엔 의존 X)
     - 합성 관계는 런타임 시점에 결정된다

    상속 vs 합성: 언제 무엇을 사용할 것인가?
     - "is-a" 관계가 명확하고 확고한 경우 상속을 고려한다.
     - 코드 재사용만을 위한 상속은 피하라. 단순히 코드 재사용이 목적이라면 합성을 사용하는 것이 좋다.
     - 기능 확장(Decorator Pattern), 행위 변경(Strategy Pattern) 등 유연성이 필요한 경우에도 합성이 더 적합하다. 유지보수하기도 쉬움
    ```


#### 4. 리팩터링 : 책임 주도 설계의 대안
- 책임을 올바르게 할당하면서 설계하긴 쉽지 않다. 일단 실행되는 코드를 짠 다음 책임들을 올바른 위치로 이동시키는 방법도 괜찮다. (이때 동작은 변하면 안 된다)
1. **메서드를 응집도 있게 분해하기** : 메서드를 작게 분해해서 각 메서드의 응집도를 높여라. 그럼 각 메서드를 적절한 클래스로 이동시키기가 수월해진다.
2. **객체를 자율적으로 만들기** : 자신이 소유한 데이터는 스스로 처리하도록 메서드를 이동시키자. 메서드 안의 접근자 메서드인 `getter`를 통해 어디로 옮겨야 할 지에 대한 힌트를 얻을 수 있다.
    ```java
    // as-is : condition 객체에 getSequence()로 접근하고 있다.
    public class ReservationAgency {
      private boolean isSatisfiedBySequence(DiscountCondition condition, Screening screening) {
        return condition.getSequence() == screening.getSequence();
      }
    }
    
    // to-be : 순번 체크의 책임을 DiscountCondition으로 이동시켜 객체가 스스로 작업하게끔 만들었다.
    public class DiscountCondition {
      private int sequence;
      
      private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
      }
    }
    ```


## 🧠 3. Reflections
- 접근 제어자를 public, private말고 protected도 잘 활용할 것
- 상속 관련 견해들. 상속이 적절한 케이스를 제외하곤 합성을 고려하자
  ```
  "내가 자바를 만들면서 가장 후회하는 일은 상속을 만든 점이다."
  - James Arthur Gosling의 인터뷰
  
  "상속을 위한 설계와 문서를 갖추거나 그럴 수 없다면 상속을 금지하라"
  - Effective Java by 조슈아 블로크
  
  "An object is dead if it allows other objects to inherit its encapsulated code and data"
  - David West (Object Thinking 저자)의 인터뷰
  ```