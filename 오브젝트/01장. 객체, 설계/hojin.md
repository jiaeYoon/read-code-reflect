# 0. 패러다임?

![image](https://github.com/user-attachments/assets/fdc01ec1-ae3b-4ba1-9099-0f279e1cfbe1)
수십년 전인 1962년, 사람들에게 패러다임이라는 단어는 생소한 단어였다.

하지만 오늘날 우리에게는 매우 익숙하다. 예를 들어 “객체 지향 패러다임” 이라는 단어를 개발 업계에서도 흔히 쓰고 있다는 점이 그러하다.

![image 1](https://github.com/user-attachments/assets/7f0af2eb-5405-4fd3-a72d-913e7c7a9803)
소프트웨어 업계에서의 패러다임의 의미는, 기존 과학계의 그것과는 다소 상이하다.

새로운 패러다임은 **과거의 그것과 공존이 가능하며, 단점을 보완하는 과정**을 거친다.

즉, 소프트웨어에서의 패러다임은 혁명적(revolutionary)이 아니라 **발전적인(evolutionary)** 것이며,

본 서적에서는 객체 지향 프로그래밍 패러다임에 대해 다룬다.

---

# 1. 시나리오 소개

객체 지향 패러다임을 이해하기 위해 활용될 첫번째 시나리오는,

소극장 티켓 판매이며, 아래와 같은 시나리오를 가진다.

- 관람객은 초대장을 받을 수 있다.
- 초대장이 있는 경우, 돈을 지불하지 않아도 티켓을 획득한다.
- 초대장이 없는 경우, 티켓을 구매해야 한다.

      해당 애플리케이션의 핵심 클래스간 관계를 나타내면 아래와 같다.

![티켓_판매_어플리케이션_핵심_클래스 drawio](https://github.com/user-attachments/assets/048b49fe-19b0-48f2-b6bf-f669bfae401a)

**그림 1.1 티켓 판매 어플리케이션 핵심 클래스**

시나리오를 토대로 소극장에서의 입장을 구현하면 아래와 같다.

```java
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    /**
     * 절차지향 패러다임을 따르는 코드
     * 연관 있는 객체 구현이 바뀌면, 호출부의 구현도 수정해야한다.
     * 결합도가 지나치게 높다.
     */
    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

---

## 1-1. 무엇이 문제인가?

![image 2](https://github.com/user-attachments/assets/bdde1f77-ab0c-4642-a9e5-25a30b7dc00b)

기존이 문제점은 현실의 행위에 비추어 보았을 때, **프로그래머의 예상에서 크게 벗어나는 코드**이다.

무엇이 예상에서 벗어난다는 것인가?

- 소극장 객체는 실제 세계에서 직접 티켓을 팔거나 초대장 유무를 검사하는 역할을 하지 않는다.
- **관람객, 판매원의 역할이 소극장의 통제를 받아 수동적**이다.

또한, 하나의 클래스에 **과도한 의존 관계가 성립**된다.

소극장은 audience,bag,ticketseller 등 많은 객체와 의존 관계를 맺고 있다.

즉, **결합도가 높아 변경이 용이하지 않다**.

![너무많은_클래스에_의존하는_Theater drawio](https://github.com/user-attachments/assets/56ab3346-6969-4e10-b2bf-66e2060df73b)

**그림 1.2 너무 많은 클래스와 의존 관계를 맺는 Theater**

# 2. 개선

이를 개선하는 방법으로는 **각 객체에 자율성을 보장하는 방법**이 있다.

자신과 관련 있는 **데이터는 자신이 직접 처리**하게 함으로써 **응집성을 높히는 방법**으로 볼 수 있다.

그럼, TicketSeller와 Audience의 자율성을 높히는 방향으로 개선해보자.

TicketSeller에 자율성 부과

- 티켓을 관람객에게 판매하는 역할을 부여한다.
- 매표소 매출을 티켓 가격만큼 더하는 일을 준다.

Audience에 자율성 부과

- 입장권이 있는지 직접 체크한다.
- 자신의 현금에서 티켓 가격만큼 제한다.
- 티켓을 가져간다.

- 첫번째 개선 후 코드. (내부의 사항을 감춘다. 캡슐화 적용)

```java
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        ticketSeller.sellTicketTo(audience);
    }
}
```

- Audience : 티켓을 구매하는 행위를 능동적으로 수행한다.

```java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public void buyTicket(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return;
        }

        Long ticketFee = ticket.getFee();
        minusAmount(ticketFee);
        setTicket(ticket);
    }

    private void setTicket(Ticket ticket) {
        this.bag.setTicket(ticket);
    }

    private void minusAmount(Long amount) {
        this.bag.minusAmount(amount);
    }

    private boolean hasInvitation() {
        return this.bag.hasInvitation();
    }
}
```

- TicketSeller : 티켓을 판매하는 행위를 능동적으로 수행한다.

```java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public void sellTicketTo(Audience audience) {
        Ticket ticket = showTicket();
        audience.buyTicket(ticket);

        Long ticketFee = ticket.getFee();
        plusAmount(ticketFee);
    }

    private Ticket showTicket() {
        return this.ticketOffice.getTicket();
    }

    private void plusAmount(Long amount) {
        this.ticketOffice.plusAmount(amount);
    }
}
```

개념적이나 물리적으로 **객체 내부의 세부적인 사항을 감추는 것을 캡슐화**라고 부르며,

다시 클래스 다이어그램으로 보면 아래와 같다.

- TicketOffice와 Ticket이 Theater의 의존성에서 벗어났다.

![Theater의_결합도를_낮춘_설계 drawio](https://github.com/user-attachments/assets/72561305-28fe-4b9e-a17d-cd3449493554)

**그림 1-3. Theater의 결합도를 낮춘 설계**

---

## 2-1. 무엇이 개선되었는가?

Audience, TicketSeller는 **자신의 데이터를 직접 관리**한다.

Audience와 TicketSeller의 내부 구현을 Theater가 몰라도 되는 상태가 되었다. 

이전 보다 변경하기 쉬운 상태가 되었다.

여기서 객체 지향 패러다임에서의 변경 하기 쉬운 상태를 정의할 필요가 있는데, 이는 **“변경하기 쉬운 설계는 한 번에 하나의 클래스만 변경할 수 있는 설계”** 라는 것 뜻이다.

- 책임이 중앙 집중된 기존의 구조

![절차지향_구조 drawio](https://github.com/user-attachments/assets/c5872d75-b4b7-4983-a9ba-086c256b2f5e)

**그림 1-4. 중앙 집중형 절차적 프로그래밍 구조**

- 책임이 분산된 개선된 구조

![객체지향_프로그래밍_구조 drawio](https://github.com/user-attachments/assets/bb4c4f22-b6a0-4ea8-9b97-49ec08e5a49e)

**그림 1-5. 책임이 분산된 객체지향 프로그래밍 구조**

## 2-2. 더 개선할 포인트는?

여전히 아쉬움은 존재한다. TicketOffice와 Bag이 자율적인 존재가 아니기 때문이다.

- TicketOffice는 여전히 TicketSeller에게 종속적이다. (데이터 호출이 수동적이다.)

```java
    /**TicketSeller에서 TicketOffice의 데이터를 직접 질의/변경한다.**/
    
    private Ticket showTicket() {
        return this.ticketOffice.getTicket();
    }

    private void plusAmount(Long amount) {
        this.ticketOffice.plusAmount(amount);
    }
```

- Bag 역시 자신의 데이터를 관리하는데 수동적이다.

```java
    /**Audience에서 bag의 데이터를 직접 질의/변경한다.**/
    
    private void setTicket(Ticket ticket) {
        this.bag.setTicket(ticket);
    }

    private void minusAmount(Long amount) {
        this.bag.minusAmount(amount);
    }

    private boolean hasInvitation() {
        return this.bag.hasInvitation();
    }
```

나머지 의존성들은 어떻게 개선해 낼 것인가?

- TicketOffice의 데이터 변경은 TicketSeller에 의해 수행된다.
- Bag 역시 Audience에 끌려다니는 수동적인 존재이다.

마찬가지이다. 각 객체가 데이터들을 직접 관리하도록 자율성을 부과하면 된다.

**Bag 개선 :** 티켓을 직접 관리한다.

```java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public void buyTicket(Ticket ticket) {
        this.bag.hold(ticket);
    }
}

public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

   ...................

    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        }

        setTicket(ticket);
        minusAmount(ticket.getFee());
        return ticket.getFee();
    }

    private boolean hasInvitation() {
        return this.invitation != null;
    }

    private void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    private void plusAmount(Long amount) {
        this.amount += amount;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

**TicketOffice 개선** : 티켓을 청중에게 직접 제공한다.

```java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public void sellTicketTo(Audience audience) {
        ticketOffice.offerTicketTo(audience);
    }
}

public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }

    public void offerTicketTo(Audience audience) {
        Ticket ticket = getTicket();
        audience.buyTicket(ticket);

        Long ticketFee = ticket.getFee();
        plusAmount(ticketFee);
    }

    private Ticket getTicket() {
        return tickets.remove(0);
    }

    private void plusAmount(Long amount) {
        this.amount += amount;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

변경 후 기존에 존재하지 않았던 의존성이 생겨나는 단점도 있다.

- TicketOffice에서 Audience의 의존성이 추가됨.

하지만 언제나 **완벽한 설계는 존재하지 않으며 트레이드 오프를 고려해 차선책을 선택**해야 함을 명심해야 한다. 

- TicketOffice의 데이터를 외부 변경을 허용하느냐 vs 응집도를 높히고, Audience 종속성을 추가하느냐

앞서 예시에서 실제 세계와 프로그래밍 세계에서의 큰 차이점을 하나 더 꼽자면……

![image 3](https://github.com/user-attachments/assets/3226fb0e-894c-47c8-a32e-3d86df3ee2f4)

실제로는 가방, 매표소가 살아있는 유기체가 아니라 능동적일 수 없지만 **프로그래밍 세계에서는 능동적인 개체가 된다**는 것이다.

- 이것을 **의인화**라고 부른다.

---

# 3. 설계가 왜 필요할까

해당 서적에서 설계에 대해 흥미로운 정의가 있었다.

**“설계란 코드를 배치하는 것이다.”**

바로 설계는 코드를 작성하는 **매 순간 코드를 어떻게 배치할 것인지**를 결정하는 과정에서 나온다는 것이다.

프로그래머는 코드의 배치를 고민하는 과정에서 객체간 의존성, 메서드의 위치 등을 재조정하기도 한다.

이런 점에서 매 순간 코드의 배치를 설계의 관점에서 바라볼 수 있는 듯 하다.

이번 장에서는 좋은 코드/설계란 무엇인가?를 토대로 티켓 판매 어플리케이션을 객체 지향적으로 개선하는 과정을 살펴보았다.

1장에서 정의한 좋은 설계를 요약하면 아래와 같다.

1. **이해하기 쉬운 코드**
2. **변화에 유연한 의존성**

- 감사합니다-
