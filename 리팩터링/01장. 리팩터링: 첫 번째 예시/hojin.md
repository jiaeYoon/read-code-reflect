# Chapter 01

# 0. 들어가며

리팩토링 2판의 첫번째 시나리오는 연극을 공연하는 극단이며, 아래 객체들이 존재한다.

- 극 (Play)
    - 극 타입 (비극, 희극)
    - 극 이름
- 청구서 (Invoice)
    - 고객 식별자
    - 신청된 공연 리스트
- 공연 (Performance)
    - 관객 수
    - 공연 식별자

위 3가지 객체를 클래스로 구조화하면 아래와 같다.

- Play

```java
public class Play {
    private String name;
    private String type;

    public Play(String name, String type) {
        this.name = name;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public String getType() {
        return type;
    }
}
```

- Invoice

```java
public class Invoice {
    private String customer;
    private List<Performance> performances;

    public Invoice(String customer, List<Performance> performances) {
        this.customer = customer;
        this.performances = performances;
    }

    public String getCustomer() {
        return customer;
    }

    public List<Performance> getPerformances() {
        return performances;
    }
}

```

- Performance

```java
public class Performance {
    private String playId;
    private int audience;

    public Performance(String playId, int audience) {
        this.playId = playId;
        this.audience = audience;
    }

    public String getPlayId() {
        return playId;
    }

    public int getAudience() {
        return audience;
    }
}

```

위 3가지 객체가 서로 상호작용하며 **공연 요청 관련한 명세서를 만들어내는 시나리오**를 구현한 코드는 

아래와 같다.

```java
    public static String createStatement(Invoice invoice,Map<String,Play> plays) throws Exception {
        int totalAmount = 0;
        int volumeCredits = 0;
        String result = String.format("청구 내역 (고객명: %s)\n", invoice.getCustomer());

        NumberFormat format = NumberFormat.getCurrencyInstance(new Locale("en", "US"));
        format.setMinimumFractionDigits(2);

        for (Performance perf : invoice.getPerformances()) {
            Play play = plays.get(perf.getPlayId());
            int thisAmount = 0;

            switch (play.getType()) {
                case "tragedy":
                    thisAmount = 40000;
                    if (perf.getAudience() > 30) {
                        thisAmount += 1000 * (perf.getAudience() - 30);
                    }
                    break;

                case "comedy":
                    thisAmount = 30000;
                    if (perf.getAudience() > 20) {
                        thisAmount += 10000 + 500 * (perf.getAudience() - 20);
                    }
                    thisAmount += 300 * perf.getAudience();
                    break;

                default:
                    throw new Exception(String.format("알 수 없는 장르: %s", play.getType()));
            }

            volumeCredits += Math.max(perf.getAudience() - 30, 0);
            if ("comedy".equals(play.getType())) {
                volumeCredits += Math.floorDiv(perf.getAudience(), 5);
            }

            result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(thisAmount / 100.0), perf.getAudience());
            totalAmount += thisAmount;
        }

        result += String.format("총액: %s\n", format.format(totalAmount / 100.0));
        result += String.format("적립 포인트: %d점\n", volumeCredits);

        return result;
    }
```

- 지저분하며, 구현의 의도를 직관적으로 알아내기가 어렵다.

단순히 미적인 관점에서만 지저분한 코드가 나쁘다는 것이 아니다.

- 설계가 나쁜 시스템은 수정하기 어렵고, 원하는 동작을 수행하도록 하기 위해 수정해야할 부분을 찾기 어려워진다.
- 모듈끼리 서로 맞물려 작동하는지 파악하기 어렵다.

위 예시의 문제점은 아래와 같다.

- 함수가 길어 구현을 파악하기 어렵다.
- 연극 장르/공연료 정책이 달라질 때마다 함수 본문을 수정해야한다.

# 1. 함수 쪼개기

먼저, statement 전체 구현 중 각기 다른 동작을 나눌 지점을 찾는다. 

## 1-1. 공연 요금 계산 로직

먼저, **switch문**을 주목하자. 이는 한번의 공연에 대한 요금을 계산한다.

- calculateFee()로 명명한 함수로 switch 문을 추출한다.
- thisAmount의 네이밍을 명시적인 것 (result)로 수정한다. ⇒ 저자는 컴퓨터가 이해하는 코드가  아닌, 사람이 이해하도록 코드를 작성하는 것이 실력자라고 말한다.

```java
    private static int calculateFee(Performance perf,Play play) throws Exception {
        int result = 0;

        switch (play.getType()) {
            case "tragedy":
                result = 40000;
                if (perf.getAudience() > 30) {
                    result += 1000 * (perf.getAudience() - 30);
                }
                break;

            case "comedy":
                result = 30000;
                if (perf.getAudience() > 20) {
                    result += 10000 + 500 * (perf.getAudience() - 20);
                }
                result += 300 * perf.getAudience();
                break;

            default:
                throw new Exception(String.format("알 수 없는 장르: %s", play.getType()));
        }

        return result;
    }
```

- 아래는 추출 후 createStatement 전체 코드. 요금 계산의 의도가 더 명확하게 보인다.

```java
    public static String createStatement(Invoice invoice,Map<String,Play> plays) throws Exception {
        int totalAmount = 0;
        int volumeCredits = 0;
        String result = String.format("청구 내역 (고객명: %s)\n", invoice.getCustomer());

        NumberFormat format = NumberFormat.getCurrencyInstance(new Locale("en", "US"));
        format.setMinimumFractionDigits(2);

        for (Performance perf : invoice.getPerformances()) {
            Play play = plays.get(perf.getPlayId());
            int thisAmount = 0;
            **totalAmount += calculateFee(performance,play);**

            volumeCredits += Math.max(perf.getAudience() - 30, 0);
            if ("comedy".equals(play.getType())) {
                volumeCredits += Math.floorDiv(perf.getAudience(), 5);
            }

            result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(thisAmount / 100.0), perf.getAudience());
            totalAmount += thisAmount;
        }

        result += String.format("총액: %s\n", format.format(totalAmount / 100.0));
        result += String.format("적립 포인트: %d점\n", volumeCredits);

        return result;
    }
```

### 1-1-1. 변수 인라인하기 (지역 변수 제거)

또한,  thisAmount를 포멧팅하는 부분을 함께 보자.

```java
**int thisAmount = calculateFee(performance,play);**

volumeCredits += Math.max(performance.getAudience() - 30, 0);

if ("comedy".equals(play.getType())) {
      volumeCredits += Math.floorDiv(performance.getAudience(), 5);
}

result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(**thisAmount** / 100.0), performance.getAudience());
**totalAmount += thisAmount;**
```

여기서, thisAmount 변수는 전달되는 매개변수가 같다면, 계산될 때마다 변하지 않는다. 

따라서 지역변수를 제거하고 변수 인라인 기법을 도입할 수 있다.

```java
volumeCredits += Math.max(performance.getAudience() - 30, 0);

if ("comedy".equals(play.getType())) {
      volumeCredits += Math.floorDiv(performance.getAudience(), 5);
}

result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance,play) / 100.0), performance.getAudience());
totalAmount += calculateFee(performance,play);
```

## 1-2.포인트 적립 로직

매 for loop 마다 포인트를 적립하는 로직이 반복되고 있고, 이 또한 함수 추출을 통해 의도를 분리해 낼 수 있다.

아래 추출 케이스를 보자.

```java
    public static String createStatement(Invoice invoice,Map<String,Play> plays) throws Exception {
        int totalAmount = 0;
        int volumeCredits = 0;
        String result = String.format("청구 내역 (고객명: %s)\n", invoice.getCustomer());

        NumberFormat format = NumberFormat.getCurrencyInstance(new Locale("en", "US"));
        format.setMinimumFractionDigits(2);

        for (Performance perf : invoice.getPerformances()) {
            Play play = plays.get(perf.getPlayId());
            int thisAmount = 0;
            totalAmount += calculateFee(performance,play);

            **volumeCredits += Math.max(perf.getAudience() - 30, 0);**
            **if ("comedy".equals(play.getType())) {**
                **volumeCredits += Math.floorDiv(perf.getAudience(), 5);**
            **}**

            result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(thisAmount / 100.0), perf.getAudience());
            totalAmount += thisAmount;
        }

        result += String.format("총액: %s\n", format.format(totalAmount / 100.0));
        result += String.format("적립 포인트: %d점\n", volumeCredits);

        return result;
    }
```

아래와 같이 포인트 적립 의도를 분리한다.

```java
private static int calculateCredit(Performance performance,Play play) {
    int result = 0;
    result += Math.max(performance.getAudience() - 30, 0);

    if ("comedy".equals(play.getType())) {
        result += Math.floorDiv(performance.getAudience(), 5);
    }

    return result;
}
```

포인트 적립 로직 함수 추출 후

```java
    public static String createStatement(Invoice invoice,Map<String,Play> plays) throws Exception {
        int totalAmount = 0;
        int volumeCredits = 0;
        String result = String.format("청구 내역 (고객명: %s)\n", invoice.getCustomer());

        NumberFormat format = NumberFormat.getCurrencyInstance(new Locale("en", "US"));
        format.setMinimumFractionDigits(2);

        for (Performance performance : invoice.getPerformances()) {
            Play play = plays.get(performance.getPlayId());
            **volumeCredits = calculateCredit(performance,play);**

            result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance,play)/ 100.0), performance.getAudience());
            totalAmount += calculateFee(performance,play);
        }

        result += String.format("총액: %s\n", format.format(totalAmount / 100.0));
        result += String.format("적립 포인트: %d점\n", volumeCredits);

        return result;
    }
```

### 1-2-1. 반복문 쪼개기

- 이번에는 반복문을 쪼개 의도를 분리해보고, 변수의 등장을 사용하는 시점에 더 가까이 가도록 수정해보자.
- 아래 반복문은 포인트 적립과 청구 내역 출력 두가지 의도를 가진다.

```java
for (Performance performance : invoice.getPerformances()) {
      Play play = plays.get(performance.getPlayId());
      
      // 포인트 적립
      volumeCredits = calculateCredit(performance,play);

     //청구 내역 출력
      result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance,play)/ 100.0), performance.getAudience());
      totalAmount += calculateFee(performance,play);
}
```

- 이 중, 포인트 적립 의도를 반복문 쪼개기로 분리해보자.
    - 다만, 이 부분은 상황 / 개인차가 존재할 것으로 보인다.
    - 기존 하나의 for - loop에서 두 가지 의도를 메서드 추출을 통해 충분히 분리했다고 볼 수도 있기 때문이다.

```java
  for (Performance performance : invoice.getPerformances()) {
       Play play = plays.get(performance.getPlayId());
       result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance,play)/ 100.0), performance.getAudience());
       totalAmount += calculateFee(performance,play);
  }

  for (Performance performance : invoice.getPerformances()) {
        Play play = plays.get(performance.getPlayId());
        volumeCredits = calculateCredit(performance,play);
   }
```

### 1-2-2.청구 내역 추출하기

아래 코드에서 String.format의 의도가 어떤 문자열을 만들기 위함인지 분명하지 않다. 

해당 내용도 메서드 추출을 통해 명시적으로 의도를 드러내자.

```java
  for (Performance performance : invoice.getPerformances()) {
       Play play = plays.get(performance.getPlayId());
       **result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance,play)/ 100.0), performance.getAudience());**
       totalAmount += calculateFee(performance,play);
  }

  for (Performance performance : invoice.getPerformances()) {
        Play play = plays.get(performance.getPlayId());
        volumeCredits = calculateCredit(performance,play);
   }
```

아래는 메서드 추출 밑 변수명 수정을 통해 변경된 코드이다.

```java
 for (Performance performance : invoice.getPerformances()) {
       Play play = plays.get(performance.getPlayId());
       **invoiceReport = printInvoiceReport(performance, invoiceReport, play, format);**
       totalAmount += calculateFee(performance,play);
 }

 for (Performance performance : invoice.getPerformances()) {
       Play play = plays.get(performance.getPlayId());
       volumeCredits = calculateCredit(performance,play);
 }
 
 private static String printInvoiceReport(Performance performance, String result, Play play, NumberFormat format) throws Exception {
        result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance, play)/ 100.0), performance.getAudience());
        return result;
 }
```

앞서 함수 추출, 변수 인라인, 함수 옮기기 3가지 기법으로 변경된 코드를 중간 점검 차 확인해보자.

```java
    public static String createStatement(Invoice invoice,Map<String,Play> plays) throws Exception {
        int totalAmount = 0;
        int volumeCredits = 0;
        String invoiceReport = String.format("청구 내역 (고객명: %s)\n", invoice.getCustomer());

        NumberFormat format = NumberFormat.getCurrencyInstance(new Locale("en", "US"));
        format.setMinimumFractionDigits(2);

        for (Performance performance : invoice.getPerformances()) {
             Play play = plays.get(performance.getPlayId());
             volumeCredits = calculateCredit(performance,play);

             invoiceReport = printInvoiceReport(performance, invoiceReport, play, format);
             totalAmount += calculateFee(performance,play);
        }

        invoiceReport += String.format("총액: %s\n", format.format(totalAmount / 100.0));
        invoiceReport += String.format("적립 포인트: %d점\n", volumeCredits);

        return invoiceReport;
    }
    

    private static String printInvoiceReport(Performance performance, String result, Play play, NumberFormat format) throws Exception {
        result += String.format("  %s: %s (%d석)\n", play.getType(), format.format(calculateFee(performance, play)/ 100.0), performance.getAudience());
        return result;
    }

    private static int calculateCredit(Performance performance,Play play) {
        int result = 0;
        result += Math.max(performance.getAudience() - 30, 0);

        if ("comedy".equals(play.getType())) {
            result += Math.floorDiv(performance.getAudience(), 5);
        }

        return result;
    }

    private static int calculateFee(Performance performance,Play play) throws Exception {
        int result = 0;

        switch (play.getType()) {
            case "tragedy":
                result = 40000;
                if (performance.getAudience() > 30) {
                    result += 1000 * (performance.getAudience() - 30);
                }
                break;

            case "comedy":
                result = 30000;
                if (performance.getAudience() > 20) {
                    result += 10000 + 500 * (performance.getAudience() - 20);
                }
                result += 300 * performance.getAudience();
                break;

            default:
                throw new Exception(String.format("알 수 없는 장르: %s", play.getType()));
        }

        return result;
    }
```

- 전체 코드의 양은 다소 많아졌지만, 최상위 함수 라인이 줄고 의도가 더 명확히 나뉘었다.

자, 그런데 calculateFee의 switch문을 확인해보면 Play 객체의 타입마다 요금 계산 구현이 다름을 확인할 수 있다.

이를 다형성을 활용하여 의도를 드러내고 확장에 더 용이한 코드로 개선해보자.

# 2. 다형성 활용하기

앞서 언급했던 calculateFee 로직을 생각해보면….

Play의 타입에 따라 요금 계산 구현이 달라짐을 확인할 수 있는데, 이들의 **공통 관심사가 요금 계산**이라는 점을 확인할 수 있다.

기존 Play를 요금을 계산하는 추상 메서드를 가지는 추상 클래스로 정의하고,

type에 따라 해당 메서드의 구현을 달리하는 구체 클래스들을 만들어 리팩토링한다.

- Play : 추상클래스로 수정

```java
public abstract class Play {
    private String name;
    private String type;

    public Play(String name, String type) {
        this.name = name;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public String getType() {
        return type;
    }

    **public abstract int calculateFeeWithPerformance(Performance performance);**
}
```

구체 클래스 - Comedy, Tragedy

```java
public class Comedy extends Play {
    public Comedy(String name,String type) {
        super(name,type);
    }

    @Override
    public int calculateFeeWithPerformance(Performance performance) {
        int result = 30000;
        if (performance.getAudience() > 20) {
            result += 10000 + 500 * (performance.getAudience() - 20);
        }
        result += 300 * performance.getAudience();
        return result;
    }
}

public class Tragedy extends Play {
    public Tragedy(String name,String type) {
        super(name,type);
    }

    @Override
    public int calculateFeeWithPerformance(Performance performance) {
        int result = 40000;
        if (performance.getAudience() > 30) {
            result += 1000 * (performance.getAudience() - 30);
        }
        return result;
    }
}
```

리팩토링 결과 - switch 문을 없애고 매우 간결해졌다. Play의 구체 타입은 런타임에서 결정되므로 가능한 구조이다.

```java
    private static int calculateFee(Performance performance,Play play) {
        return play.calculateFeeWithPerformance(performance);
    }
```

# 마무리하며

이번 장에서는 **함수 추출, 변수 인라인, 함수 옮기기, 조건부 로직을 다형성으로 바꾸기** 이렇게 4가지 기법을 선보였다.

해당 장의 핵심을 요약하면 아래와 같다.

- 코드는 수정하기 좋고 명확해야 좋다고 말할 수 있다.
    - 고쳐야 할 곳을 쉽게 찾을 수 있고 오류 없이 빠르게 수정할 수 있어야 하기 때문이다.

감사합니다.
