## 📍 1. Overview key concept
- 메시지
- 메시지와 메서드
- 퍼블릭 인터페이스 : 디미터의 법칙, 묻지 말고 시켜라, 의도를 드러내는 인터페이스, 명령-쿼리 분리
- 불변성, 부수효과, 참조 무결성

## ✨ 2. Highlights & Takeaways**
#### 0. 개요
- 객체지향 프로그래밍에 대한 가장 흔한 오해는 애플리케이션이 **클래스의 집합**으로 구성된다는 것이다. **클래스는 구현 도구**다. 구현 도구에 집착하면 유연하지 못한 설계를 얻게 된다.
- 중요한 건 클래스가 아닌 객체이며 객체가 수행하는 책임에 집중해야 한다. 또 중요한 건 객체지향에서 가장 중요한 재료가 객체들이 주고 받는 메시지란 것이다.
- **객체가 수신하는 메시지들이 객체의 퍼블릭 인터페이스를 구성한다.**


#### 1. 협력과 메시지
- 객체 간 협력은 `메시지`를 통해 이루어진다.
- 대부분의 사람들은 객체가 수신하는 메시지의 집합에만 초점을 맞추지만, 협력에 적합한 객체 설계를 위해선 외부에 전송하는 메시지의 집합도 함께 고려해야 한다. (Movie는 가격 계산의 요청 메시지를 수신하고 예매 요금을 결과로 반환함)
- 협력 관련 용어를 정리하면 다음과 같다.

  | name      | description                                                                             |
  |-----------|-----------------------------------------------------------------------------------------|
  | 메시지       | 객체들이 협력하기 위해 사용할 수 있는 유일한 의사소통 수단</br> 오퍼레이션 명 + 인자로 구성됨 `ex) isSatisfiedBy(screening)` |
  | 오퍼레이션     | 메서드의 추상체. 메시지는 협력 관계를 강조하지만 오퍼레이션은 메시지를 수신하는 객체의 인터페이스를 강조한다.                           |
  | 메서드       | 메시지를 수신했을 때 실제로 실행되는 함수. 오퍼레이션의 구현이다.                                                   |
  | 퍼블릭 인터페이스 | 객체가 의사소통을 위해 외부에 공개하는 메시지의 집합                                                           |
  | 시그니처      | 오퍼레이션/메서드의 명세를 나타낸 것. 이름과 인자의 목록을 포함한다.                                                 |

- **메시지와 메서드를 구분하라 ⭐️**
- SequenceCondition, PeriodCondition에 정의된 isSatisfiedBy는 구현이므로 메서드라고 부른다. 그리고 두 메서드는 DiscountCondition 인터페이스에 정의된 isSatisfiedBy 오퍼레이션의 여러 가능한 구현 중 하나다.
  <img src="https://github.com/user-attachments/assets/8bbe333a-d9e2-4c7d-9e33-371df19d2585"  width="70%"/>

- 중요한 건 **메시지가 객체의 품질을 결정한다**는 것이다. 객체가 수신할 수 있는 메시지가 객체의 퍼블릭 인터페이스와 그 안에 포함될 오퍼레이션을 결정하기 때문 ⭐️⭐️


#### 2. 인터페이스와 설계 품질
- 좋은 인터페이스란 다음 두 조건을 만족해야 한다. 이를 위해 책임 주도 방법을 따르는 것이 좋다.   
  **① 최소한의 인터페이스** : 꼭 필요한 오퍼레이션만을 인터페이스에 포함한다.   
  **② 추상적인 인터페이스** : 어떻게 수행하는지가 아니라 무엇을 하는지를 표현한다. (= 구현을 인터페이스에 노출시키지 말 것)


- 좋은 인터페이스 설계 기법 ⭐️⭐️⭐️   
  **① 디미터의 법칙**
  ```text
  - 오직 인접한 이웃하고만 말하라. 즉 객체의 내부 구조에 강하게 결합되지 않도록 협력 경로를 제한하라.
  - 디미터의 법칙을 따르는 코드는 메시지 수신자의 내부 구조가 전송자에게 노출되지 않으며, 메시지 전송자는 수신자의 내부 구현에 결합되지 않으므로 낮은 결합도를 가지며,
    정보 처리의 책임을 정보를 알고 있는 객체에게 할당하기 때문에 응집도 높은 객체가 만들어진다.
  - 클래스는 특정한 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그래밍해야 한다.
    1. this 객체
    2. 메서드의 매개변수
    3. this의 속성
    4. this의 속성인 컬렉션의 요소
    5. 메서드 내에 생성된 지역 객체
  - 하지만 항상 좋은 건 아니다!   
  ```
  ```java
  screening.getMovie().getDiscountConditions(); // 디미터의 법칙 위반 : movie의 내부 구조가 노출됨
  screening.calculateFee(audienceCount);        // 디미터의 법칙 준수 : screening 객체에게 작업 처리를 요청하며 내부 구조가 노출되지 않음
  ```
  **② 묻지 말고 시켜라**
  ```
  - 객체의 상태를 묻지 말고, 원하는 것을 시켜라
  - 메시지 전송자는 메시지 수신자의 상태를 기반으로 결정을 내린 후, 메시지 수신자의 상태를 바꿔서는 안 된다. 수신자의 상태로 결정내리는 건 객체의 캡슐화를 위반한 것이다.
  - 묻지 말고 시켜라 원칙을 따르면 밀접하게 연관된 정보와 행동을 함께 가지는 객체를 만들 수 있으므로 웅집도가 높아진다.
  - 하지만 단순하게 객체에 묻지 않고 시킨다해서 모든 문제가 해결되는 건 아니다. 훌륭한 인터페이스를 위해선 객체가 어떻게 작업을 수행하는 지를 노출해선 안 된다.
  ``` 

  **③ 의도를 드러내는 인터페이스**
  ```
  - 인터페이스는 객체가 어떻게가 하는지가 아니라 무엇을 하는지를 서술해야 한다. 즉 구현 방법이 아닌 클라이언트의 의도를 드러내야 한다.
  - 어떻게 수행하는지를 드러낸 이름 → 메서드의 내부 구현을 설명함 👎
  - 메서드가 무엇을 하느냐에 초점을 맞췄을 때 협력에 집중할 수 있고, 클라이언트 관점에서 동일한 작업을 수행하는 메서드들을 하나의 타입 계층으로 묶기 쉬워짐
  ```
  ```java
  // 의도가 드러나지 않는 인터페이스
  // TicketSeller의 setTicket은 티켓 판매, Audience의 setTicket은 티켓 구매, Bag의 setTicket은 티켓 보관의 의도가 있으나 현 메서드명은 그 의도를 드러내지 못하고 있다.
  public class TicketSeller {
    public void setTicket(Audience audience) {...}
  }
  public class Audience {
    public Long setTicket(Ticket ticket) {...}
  }
  public class Bag {
    public Long setTicket(Ticket ticket) {...}
  }
  
  // 의도가 명확히 드러나는 인터페이스 : 클라이언트의 의도를 더 분명하게 표현함
  public class TicketSeller {
    public void sellTo(Audience audience) {...}
  }
  public class Audience {
    public Long buy(Ticket ticket) {...}
  }
  public class Bag {
    public Long hold(Ticket ticket) {...}
  }
  ```

  **④ 명령 - 쿼리 분리**
  ```
  - 어떤 오퍼레이션도 명령인 동시에 쿼리여선 안 된다. 둘을 명확하게 분리하라. 명령과 쿼리가 섞인 메서드는 실행 결과를 예측하기가 어렵다.
  - 명령 : 객체의 상태를 수정하는 오퍼레이션이다. 반환값을 가질 수 없다.
  - 쿼리 : 객체와 관련된 정보를 반환하는 오퍼레이션이다. 상태를 변경할 수 없다.
  - 명령 - 쿼리 분리의 장점
    - 코드가 예측, 이해, 디버깅, 유지보수하기 쉽다. 
    - 한 눈에 어떤 메서드가 부수효과(상태 변경)을 가지는지 확인 가능하다.
    - 참조 투명성(어떤 표현식 e가 있을 때 모든 e를 e의 값인 v로 바꾸더라도 결과가 달라지지 않는 특성)을 만족하며 함수형 프로그래밍의 효과를 얻는다(쿼리 오퍼레이션을 여러 번 호출해도 결과는 동일함. 순서 고려없이 호출 가능)
  ```
  ```java
  public class Event {
    public boolean isSatisfied(RecurringSchedule schedule) {  // 반환값을 가지나 상태를 변경하진 않는다 → 쿼리
      if (from.getDayOfWeek() != schedule.getDayOfWeek() || 
          !from.toLocalTime().equals(schedule.getFrom()) || 
          !duration.equals(schedule.getDuration())) {
        return false;
      }
    }

    public void reschedule(RecurringSchedule schedule) {  // 반환값은 없으나 from과 duration의 상태를 변경한다 → 명령
      from = LocalDateTime.of(from.toLocalDate().plusDays(daysDistance(schedule)),
          schedule.getFrom());
      duration = schedule.getDuration();
    }
  }
  ```


#### 3. 원칙의 함정
- 디미터의 법칙, 묻지 말고 시켜라 원칙은 좋은 설계 원칙이지만 **절대적인 법칙은 아니다**. 설계가 트레이드오프의 산물이란 걸 잊지마라. 원칙이 현재 상황에 부적합하다고 판단된다면 과감하게 원칙을 무시할 줄 알아야 한다.
1. **디미터의 법칙은 하나의 도트(.)를 강제하는 규칙이 아니다.** ⭐️
   ```java
   IntStream.of(1, 15, 20, 3, 9).filter(x → x > 10).distinct().count();
   ```
  - 다음 코드를 보고 디미터의 법칙을 위반한다고 생각할 수 있으나 이는 디미터의 법칙을 이해하지 못한 것이다. 위 코드의 of, filter, distinct 메서드는 모두 IntStream이라는 동일한 클래스의 인스턴스를 반환하며 변환할 뿐이다.
  - 디미터의 법칙은 결합도와 관련있고, 결합도가 문제인 건 **내부 구현이 외부로 드러날 때** 뿐이다. IntStream의 내부 구조가 노출되지 않았으므로 이는 디미터의 법칙을 준수한 것이다!

2. **결합도와 응집도의 충돌**
  - 맹목적으로 위임 메서드를 추가하면 같은 퍼블릭 인터페이스 안에 어울리지 않는 오퍼레이션들이 공존하게 된다. 결과적으로 응집도가 낮아진다.
  - 클래스는 하나의 변경 원인을 가져야 하는데 서로 상관없는 책임들이 뭉쳐있으면 클래스의 변경 원인이 여러 개가 될 수 있고 작은 변경에도 취약해진다.
    ```java
    // 얼핏보기엔 screening의 내부 상태를 가져와서 사용하기 때문에 캡슐화를 위반한 것처럼 보일 수 있다.
    public class PeriodCondition implements DiscountCondition {
      public boolean isSatisfiedBy(Screening screening) {
         return screening.getStartTime().getDayOfWeek().equalsdayOfWeek() &&
           startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
           endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
      }
    }
    
    // 그렇다고 아래와 같이 변경하면 Screening이 할인 조건 판단의 책임을 떠안게 되어 응집도가 낮아진다.
    // 또한 Screening은 PeriodCondition의 인스턴스 변수를 인자로 받기 때문에 PeriodCondition의 인스턴스 변수 목록이 변경될 때도 영향을 받음 👎
    public class Screening {
      public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
          return whenScreened.getDay0fWeek().equals(day0fWeek) &&
                  startTime.compareTo(whenScreened.toLocalTime()) = 0 &&
                  endTime.compareTo(whenScreened.toLocalTime()) >= 0;
      }
    }
    public class PeriodCondition implements DiscountCondition {
      public boolean isSatisfiedBy(Screening screening) {
          return screening.isDiscountable(dayOfWeek, startTime, endTime);
      }
    }
    ```


#### 4. 끝으로
- 메시지를 먼저 선택하고 그 후에 메시지를 처리할 객체를 선택하는 '책임 주도 설계 방법'을 따르자 ⭐️
- 그럼 디미터의 법칙, 묻지 말고 시켜라, 의도를 드러내는 인터페이스, 명령-쿼리 분리 원칙들은 저절로 따라온다


## 🧠 3. Reflections
- 디미터 법칙의 사실과 오해 : "오직 하나의 도트만 사용하라"
- 초보자는 원칙을 맹목적으로 추종한다. 심지어 적용하려는 원칙들이 서로 충돌하는 경우에도 원칙에 정당성을 부여하고 억지로 끼워 맞추려고 노력한다