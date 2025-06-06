## 📍 1. Overview key concept
- 책임
- 변경
- 데이터 중심 관점 / 책임 중심 관점
- 캡슐화, 응집도, 결합도

## ✨ 2. Highlights & Takeaways
#### 0. 개요
- 좋은 설계란 적은 비용으로 쉽게 변경할 수 있는 구조를 만드는 것
- 좋은 설계를 만들기 위해선 객체의 `책임`에 초점을 맞춰야 한다 : 책임에 집중함으로서 객체의 상태가 아닌 행동에 집중하게 되고 객체 간 상호작용과 낮은 결합도, 높은 응집도, 캡슐화된 구현을 가진 애플리케이션을 만들 수 있다.

#### 1. 데이터 중심 관점과 책임 중심 관점
- **데이터 중심 관점**
  - 객체는 자신이 가진 데이터를 조작하는데 필요한 메서드를 정의한다
  - **변경에 취약하다** : 객체의 상태는 구현에 해당하는데 구현은 불안정하기 때문에 변하기 쉽다. 상태 변경 → 인터페이스 변경 → 변경한 인터페이스에 의존하는 모든 객체도 변경 필요로 이어진다 👎
- **책임 중심 관점**
  - 객체는 다른 객체가 요청할 수 있는 메시지를 처리하기 위해 필요한 상태를 보관한다
  - **변경의 영향이 작다** : 객체의 책임은 인터페이스에 속한다. 책임에 필요한 상태는 캡슐화되어있어서 구현이 변경돼도 파장이 외부로 전파되지 않는다 👍


#### 2. 설계 트레이드오프
좋은 설계를 판단할 수 있는 기준인 캡슐화, 응집도, 결합도에 대해 알아보자. 셋 다 **변경**과 관련된 것이다.

**① 캡슐화**
- 변경 가능성이 높은 구현을 객체 내부로 숨기고 상대적으로 안정적인 인터페이스만 노출시키는 것
- 효과 : 내부에 변경 포인트가 생겨도 외부엔 영향이 없다
- 변경될 수 있는 모든 것을 캡슐화해야 한다! ⭐️

**② 응집도**
- 모듈 내의 구성 요소간 서로 연관되어 있는 정도
- 하나의 변경이 발생할 때 모듈 전체가 함께 변경되면 응집도가 높은 것이고, 모듈의 일부만 변경되면 응집도가 낮은 것이다.
- 하나의 변경에 대해 하나의 모듈만 변경되면 응집도가 높은 것이고, 다수의 모듈이 함께 변경되면 응집도가 낮은 것이다.
<img src="https://github.com/user-attachments/assets/9200ef10-6090-4775-a71d-fd1460ccb254" width="50%"/>


**③ 결합도**
- 모듈과 모듈 간의 상호 의존 정도
- 하나의 변경에 대해 하나의 모듈만 변경하면 결합도가 낮은 것이고, 다수의 모듈을 변경해야 하면 결합도가 높은 것이다.
- 자바의 String이나 ArrayList는 변경될 확률이 매우 적은 안정적인 모듈이므로 결합도가 높아도 상관 없다.
   ```java
  // 높은 결합도 : List의 구체인 ArrayList를 직접적으로 사용함
  private ArrayList<String> students = new ArrayList<>();
  ```
<img src="https://github.com/user-attachments/assets/cea75470-ab04-40f8-858c-ba2993937bd8"  width="48%"/>


**⭐️ 캡슐화의 정도가 응집도와 결합도에 영향을 미친다.** 캡슐화를 지키면 모듈 안의 응집도는 높아지고 모듈 사이의 결합도는 낮아진다. 캡슐화를 위반하면 모듈 안의 응집도는 낮아지고 모듈 사이의 결합도는 높아진다.   
→ 따라서 응집도, 결합도를 고려하기 전에 캡슐화를 향상시키기 위해 노력하라


#### 3. 데이터 중심 설계의 문제점
① 캡슐화 위반 사례
```java
public class Movie {
  private Money fee;

  public Money getFee() {
    return fee;
  }

  public void setFee(Money fee) {
    this.fee = fee;
  }
}
```
- 위 코드는 객체 내부에 접근할 수 없어서 캡슐화를 잘 지킨 것처럼 보인다. 그러나 getter, setter는 객체 상태에 대해 전혀 캡슐화하지 않는다. getFee(), setFee()는 Movie 내부에 Money 타입의 fee 변수가 있다는 걸 public interface로 드러낸다. 데이터에 초점을 맞췄기 때문에 캡슐화를 어긴 것. 협력을 고려해야 캡슐화를 강화할 수 있다.


② 낮은 응집도 사례
```java
public enum DiscountConditionType {
  SEQUENCE, // 순번 조건
  PERIOD // 기간 조건
}

public class ReservationAgency {
  public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
    Movie movie = screening.getMovie();

    for (DiscountCondition condition : movie.getDiscountConditions()) {
      if (condition.getType() = DiscountConditionType.PERIOD) {...} 
      else if (condition.getType() = DiscountConditionType.SEQUENCE) {...}
      else {...}
    }
  }
}
```
- 만약 할인 조건이 추가되거나 삭제된다면 DiscountConditionType과 ReservationAgency 코드를 수정해야 한다. 
- 할인 조건 변경이 여러 모듈에 흩어져 있음 → 하나의 변경에 대해 하나 이상의 모듈이 변경되어야 함 → 낮은 응집도를 가짐


③ 높은 결합도 사례
```java
public class ReservationAgency {
  public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
    ...
    Money fee;
    if (discountable) {
      ...
      fee = movie.getFee().minus(discountedAmount).times(audienceCount);
    } else {
      fee = movie.getFee();
    }
    ...
  }
} 
```
- fee의 타입을 변경한다고 가정하자. 그럼 getFee()의 return type도 변경해야 하고 getFee()를 호출하는 ReservationAgency의 구현(minus(discountedAmount) 부분) 등도 변경된 타입에 맞춰 수정해야 한다.
- fee 타입 변경으로 다수 모듈의 변경이 필요해지고 이는 즉 높은 결합도를 의미한다.

<img src="https://github.com/user-attachments/assets/e29d7195-cf5e-4e19-98a3-c324924c00aa" width="70%"/>   

- 영화 예매 시스템 diagram을 보면 대부분의 제어 로직을 가진 ReservationAgency가 모든 데이터 객체에 의존하고 있음을 알 수 있다.
- 만약 DiscountCondition이 변경되면 이 모듈뿐만 아니라 ReservationAgency도 함께 수정해야 한다. Screening도 마찬가지. 시스템의 어떤 변경도 ReservationAgency의 변경을 유발한다.


#### 4. 책임 중심 설계로 개선하기
```java
public class Movie {
  private Money fee;
  private MovieType movieType;

  public MovieType getMovieType() {...}
  public Money calculateAmountDiscountedFee() {...}
  public Money calculatePercentDiscountedFee() {...}
  public Money calculateNoneDiscountedFee() {...}
}
```
① 캡슐화 강화하기
 - ReservationAgency에서 getFee(), setFee()로 가격 데이터에 접근해 요금을 계산하는 것이 아닌 calculateXXXDiscountedFee()라는 인터페이스를 만들어 Movie가 스스로 요금을 계산할 수 있게 만들었다.
 - 요금 계산의 주체가 외부의 객체에서 Movie로 이동하였으며, 이는 '책임이 이동'한 것임을 알 수 있다.
 - 1차적으로 개선되었지만 여전히 할인 정책이 외부에 드러나 내부 구현이 외부에 노출되고 있다. 새로운 할인 정책이 추가되거나 삭제되면 클라이언트 측에서도 변경이 일어난다. (calculateDiscountedFee()로 요금 계산 메서드를 만들고 내부적으로 할인 정책을 판단 후 계산해야 할듯)

② 응집도 높임, 결합도 낮춤
 - 기존엔 getter, setter만 있었지만 fee와 관련된 요금 계산 로직이 ReservationAgency → Movie로 이동하며 응집도가 높아지고 결합도는 낮아졌다.


#### 5. 데이터 중심 설계의 문제점
2절에서 말했다시피 데이터 중심 설계는 변경에 취약하다. 그 이유는

**① 데이터 중심의 설계는 너무 이른 시기에 데이터에 관해 결정하게 한다.**
 - 데이터 중심 설계는 객체의 행동보다 상태에 초점을 맞췄었다. 그래서 일단 getter, setter를 만들게 되고 데이터를 사용하는 절차는 개선 전 ReservationAgency 처럼 별도의 객체 안에 구현하게 된다. 접근자/수정자를 가진 데이터는 public field와 마찬가지이므로 캡슐화에 실패하고 변경에 취약해진다.

**② 데이터 중심 설계는 협력을 고려하지 않고 오퍼레이션을 결정한다.**
 - 데이터 중심 설계는 협력이 아닌 객체 내부에 집중한다. 따라서 협력 고려 없이 데이터의 오퍼레이션을 결정하고, 객체 구현이 완료된 상태에서 다른 객체와의 협력을 위해 인터페이스를 억지로 끼워맞추게 된다. 객체의 내부 구현이 외부에 노출되어 있으므로 협력하는 객체와의 결합도가 높아지고 변경 시 영향이 커진다.


## 🧠 3. Reflections
- bad practice로 보여주는 코드들이 너무 익숙했음
- getter도 필요한 상황에서만 생성하는 습관 필요
