# 04장. 설계 품질과 트레이드 오프

# 1. 데이터 중심의 관점이란?

데이터 중심의 관점은 **객체의 상태에 초점**을 맞춘다.

- 객체 내부에 저장되는 데이터를 기반으로 시스템을 분할하는 방법이다.

객체의 행동이 아닌 포함해야 하는 데이터에 집중한다면 데이터 중심의 설계에 매몰돼 있을 확률이 높다.

데이터 중심의 관점과 행동 중심의 관점의 차이를 표로 나타내면 아래와 같다.

| **관점** | **객체 지향 설계 (Object-Oriented Design)** | **데이터 중심 설계 (Data-Centric Design)** |
| --- | --- | --- |
| **중심 개념** | 행동 중심 | 데이터 중심 |
| **모델링 대상** | 객체(협력의 문맥 안에서의 행동) | 객체가 가져야 할 데이터 |
| **캡슐화(Encapsulation)** | 잘 지켜짐 (정보 은닉 강조) | 위반하기 쉬움 (데이터 노출 가능성 높음) |
| **유지 보수성** | 높은 편 (변화에 유연) | 낮은 편 (변화가 여러 객체에 영향) |
| **코드 구조** | 메시지와 행동 중심으로 구성 | 데이터를 중심으로 행동을 결정 |
| **예시** | `User.changePassword()` | `changePassword(user)` (user 구조 노출) |

---

# 2. 캡슐화, 응집도, 결합도

우리는 뒤이어 나올 영화 예매 시스템의 구조를 **캡슐화,응집도,결합도**의 관점에서 평가한다.

평가에 앞서  이 세 가지 요인이 무엇인지 정리하고 가자.

### 2-1. 캡슐화

캡슐화란? 

- 객체의 **내부 구현을 숨기고 필요한 기능만 외부에 공개**하여 변경에 유연하게 대응하는 설계 원칙

구현을 숨기는 여유는 변경될 가능성이 높기 때문이다.

만약 객체 내부의 구현이 외부에 드러나게 되면 변경의 영향이 여러 곳에 닿게 된다.

본 서적의 저자는 캡슐화를 이렇게 말하고 있다.

*“캡슐화는 **외부에서 알 필요가 없는 부분을 감춤**으로써 대상을 단순화하는 **추상화의 한 종류**이다.”*

![객체_지향_캡슐화 drawio](https://github.com/user-attachments/assets/2e5c3a25-1f4e-48b1-be03-ed5cd93fdc4a)

우리가 좋은 설계를 지향해야하는 이유는 유지 보수성에 있다.

- 두려움 없이, 저항감 없이 코드를 변경할 수 있는 능력이 유지 보수성이며 가장 중요한 동료는 캡슐화이다.

설계에서 캡슐화가 중요한 이유는 변경이 발생할 불안정한 부분(구현)을 분리하여 변경의 영향을 통제할 수 있기 때문이다.

### 2-2. 응집도

응집도란?

- 하나의 객체나 모듈이 **자신이 맡은 책임에 얼마나 집중**되어 있는지를 나타내는 척도

객체 지향의 관점에서 응집도는 객체 또는 클래스에 얼마나 관련 높은 책임들을 할당했는지를 나타낸다.

![image](https://github.com/user-attachments/assets/18d75596-5dc4-4375-89c3-8741ad070fb1)

위 그림에서 음영은 변경 발생 시  수정되는 영역을 표현한 것이다.

응집도는 변경이 발생할 때 모듈 내부에서 발생하는 변경의 정도를 기준으로 측정할 수 있다.

- 변경에 적은 모듈이 영향을 받을수록 응집도가 높다고 측정할 수 있다.

### 2-3. 결합도

결합도란?

- 서로 다른 객체나 모듈 간의 **의존성과 연결 정도**를 나타내며, 낮을수록 변경에 유리한 구조

결합도 역시 변경의 관점에서 설명할 수 있다.

![image 1](https://github.com/user-attachments/assets/0b6a6bbc-9a8c-4425-bdeb-62a5b67bf4db)

결합도는 한 모듈이 변경되기 위해서 다른 모듈의 변경을 요구하는 정도로 측정할 수 있다.

- 변경에 많은 모듈이 영향을 받을수록 결합도가 높다고 측정할 수 있다.

결합도가 높아도 상관 없는 경우

- 일반적으로 변경될 확률이 매우 적은 안정적인 모듈
- 예) 자바의 String, ArrayList

하지만, **직접 작성한 코드는 언제라도 변경될 가능성이 높다.**

따라서 클래스의 구현이 아닌 **인터페이스에 의존하도록 코드를 작성**해 낮은 결합도를 얻을 수 있도록 노력해야 한다.

---

# 3. 데이터 중심 설계 평가

아래는 본 서적의 티켓 예매 어플리케이션 코드의 일부이다.

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening,Customer customer,int audienceCount) {
        Money fee = screening.calculateFee(audienceCount);
        return new Reservation(customer,screening,fee,audienceCount);
    }
}
```

- 걷보기에는 좋은 설계처럼 보인다. Agency에서 자신이 할 수 없는 역할을  Screening에 요청했기 때문이다.

### 3-1. Sceening의 문제점

```java
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
    }

    public Money calculateFee(int audienceCount) {
        switch (movie.getMovieType()) {
            case AMOUNT_DISCOUNT -> {
                if (movie.isDiscountable(whenScreened,sequence)) {
                    return movie.calculateAmountDiscountFee().times(audienceCount);
                }
            }

            case PERCENT_DISCOUNT -> {
                if (movie.isDiscountable(whenScreened,sequence)) {
                    return movie.calculatePercentDiscountFee().times(audienceCount);
                }
            }
        }

        return movie.calculateNoneDiscountFee();
    }
}
```

- 구현이 드러나 있으며 변경에 취약하다. 근거는 아래와 같다.
    - Movie의 속성에 Screening이 영향을 받는다. (switch 조건)
    - Movie의 할인 여부를 판단하는데 쓰이는 속성들이 파라미터를 통해 드러난다.

### 3-2. Movie의 문제점

```java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditionList;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public Money calculateAmountDiscountFee() {
        if (movieType != MovieType.AMOUNT_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee.minus(discountAmount);
    }

    public Money calculatePercentDiscountFee() {
        if (movieType != MovieType.PERCENT_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee.minus(fee.times(discountPercent));
    }

    public Money calculateNoneDiscountFee() {
        if (movieType != MovieType.NONE_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee;
    }

    public boolean isDiscountable(LocalDateTime whenScreened,int sequence) {
        for (DiscountCondition condition : discountConditionList) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                if (condition.isDiscountable(whenScreened.getDayOfWeek(),whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }

        return false;
    }
}
```

- 할인 요금을 계산하는데 여러 구현체들이 드러난다. (추상화가 미약함)
- 할인 여부를 판단하는데 DiscountCondition의 Type 변경에 영향을 받는다.
- 영화는 할인 여부 판단에 대한 책임을 가지지 않는다. (할인 조건의 책임)

### 3-3. DiscountCondition의 문제점

할인을 계산하는데 필요한 필드들을 외부에 드러내고 있다. (캡슐화 위반)

```java
    public boolean isDiscountable(**DayOfWeek dayOfWeek,LocalTime time**) {
        if (type != DiscountConditionType.PERIOD) {
            throw new IllegalArgumentException();
        }

        return this.dayOfWeek.equals(dayOfWeek) &&
                this.startTime.compareTo(time) <= 0 &&
                this.endTime.compareTo(time) >=0;

    }

    public boolean isDiscountable(**int sequence**) {
        if (type != DiscountConditionType.SEQUENCE) {
            throw new IllegalArgumentException();
        }

        return this.sequence == sequence;
    }
```

- 할인 금액을 계산하는 행동은 할인 정책의 책임이다. 그런데, 할인을 결정하는데 필요한 속성들이 Movie에 드러나 캡슐화가 깨지는 점을 확인할 수 있다.
- 할인 결정 속성을 할인 정책에 숨길 필요가 있다. (변경 전파 방지)

---

# 4. 개선 실습

### 4-1. DiscountCondition 개선

할인 정책을 확인하고, 할인 조건을 확인하는 행동에 초점을 맞추어 개선한다.

- 금액/비율 할인 정책을 추상화 한다.
- 기간/순번 할인 조건을 추상화 한다.

- 할인 정책
    - 공통 관심사 : 할인 요금 계산
    - 여러 할인 조건을 알 책임이 있다.
    - 할인 금액을 계산할 행동의 책임이 있다.

```java
public abstract class DiscountPolicy {
    private List<DiscountCondition> discountConditionList;

    public DiscountPolicy(List<DiscountCondition> discountConditionList) {
        this.discountConditionList = discountConditionList;
    }

    public List<DiscountCondition> getDiscountConditionList() {
        return discountConditionList;
    }

    public abstract Money calculateDiscountFee(Screening screening);
}
```

- 할인 조건
    - 공통 관심사 : 할인 여부 체크
    - 할인 여부를 확인하는 행동의 책임을 가진다.

```java
public abstract class DiscountCondition {
    public abstract boolean isDiscountable(Screening screening);
}
```

### 4-2. Screening 개선

```java
 public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    ..........................................
    
    public Money calculateFee(Screening screening,int audienceCount) {
        return movie.calculateFee(screening,audienceCount);
    }
}
```

- 요금 계산을 요청하는데 Screening이 알고 있는 정보만 전달한다.
- 구체적인 파라미터는 몰라도 상관이 없게 되었다.

### 4-3. Movie

```java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Money calculateFee(Screening screening,int audienceCount) {
        return discountPolicy.calculateDiscountFee(screening).multiply(audienceCount);
    }

    public Money getFee() {
        return fee;
    }
}
```

- 요금 계산을 위해 할인 정보를 알고 있는 DiscountPolicy에 메시지를 요청한다.
    - 각 할인의 종류를 구체적으로 알 필요가 없어졌다.
- 할인 조건의 책임이 DiscountCondition 내부로 숨겨졌다. (캡슐화)

---

# 5. 트레이드오프

개선에도 불구하고, 캡슐화는 완벽하지 않다. 

⇒ 영화 요금에 대한 정보가 밖으로 노출된다.

- Screening에서 getter를 통해 영화 요금 조회

```java
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDate startTime;
    private LocalDate endTime;
    private LocalDate dayOfWeek;

   ..................................

    public Money getMovieFee() {
        **return movie.getFee();**
    }
}
```

- 할인 요금 계산시 screening을 통해 영화 요금을 조회한다.

```java
    @Override
    public Money calculateDiscountFee(Screening screening) {
        for (DiscountCondition discountCondition : this.getDiscountConditionList()) {
            if (discountCondition.isDiscountable(screening)) {
                return **screening.getMovieFee()**.times(discountPercent);
            }
        }

        return screening.getMovieFee();
    }
```

⇒ 영화 객체가 알아야 할 책임으로 요금에 대한 정보가 있다.

해당 문제점을 용인할 수 있다고 판단되는 이유는 아래와 같다.

- 일반적으로 영화 요금은 돈에 관련된 것이므로 **변경의 소지가 거의 없다.**

---

감사합니다.
