# 14. 일관성 있는 협력
## 01. 핸드폰 과금 시스템 변경하기
### 기본 정책 확장
기본 정책을 다음과 같이 4가지 방식으로 확장할 것이다.   
<img width="675" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/a38533ec-8d45-4d25-9005-e0e289e2b9f3">

구현하게 될 클래스 구조를 그림으로 나타낸 것이다.   
<img width="645" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/f153e224-f470-461c-b75f-f948d672c28d">

### 고정요금 방식 구현하기
기존의 일반 요금제와 동일 -> 기존의 `RegularPolicy` 를 `FixedFeePolicy`로 수정
```java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;
    
    public FixedFeePolicy(Money amount, Duration seconds){
      this.amount=amount;
      this.seconds=seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

### 시간대별 방식 구현하기
통화 시간을 정해진 시간대별로 나눈 후 각 시간대별로 서로 다른 계산 규칙을 정해야 한다.   
<예시>

<img width="631" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/90f7d8a2-9af5-473a-9d8b-aafa216fd49b">

통화가 여러 날짜에 걸쳐서 이루어진다면 통화 구간을 분리한 후 각 구간에 대해 개별적으로 계산된 요금을 합해야 한다.     
<img width="610" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/ef9891a4-7f2b-47a1-b34d-366a5628682e">

시간대별 방식의 통화 요금을 계산하기 위해서 통화의 시작 시간과 종료 시간뿐 아니라 `시작 일자`와 `종료 일자`도 함께 고려해야 한다.

전체 통화 시간을 분할하는 작업은 다음 세 클래스 사이의 협력으로 구현할 수 있다.   
- `DateTimeInterval` : 기간을 편리하게 관리할 수 있다. (시작 시간과 종료 시간을 인스턴스 변수로 포함)
- `Call` : 통화 기간을 저장 (from -> DateTimeInterval으로 타입 변경)
- `TimeDiscountInterval`: 시간대별 기준을 잘 알고 있는 시간대별로 분할하는 작업을 관리한다.
<img width="657" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/b703c688-3a68-4f73-81ee-61f4955b3e98">

> 0 ~ 19 : 10초당 18원   
> 19 ~ 24 : 10초당 15원   
> 1.1 10시 ~ 1.3 15시 통화   

1. Call은 기간을 저장하고 있는 DateTimeInterval 타입의 인스턴스 변수인 interval에게 splitByDay메서드를 호출
2. splitByDay메서드는 3개의 DateTimeInterval 인스턴스를 포함하는 List를 반환
3. Call은 분리된 LIst를 시간대별 방식을 위한 TimeOfDayDiscountPOlicy 클래스에게 반홭
4. TimeOfDayDiscountPlicy클래스는 일자별로 분리된 각 DateTimeInterval 인스턴스들을 요금 정책에 정의된 각 시간대별로 분할한 후 요금을 부과

<img width="618" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/566fe258-075c-47d7-a5d1-ea61105c573e">   

```java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<LocalTime>();
    private List<LocalTime> ends = new ArrayList<LocalTime>();
    private List<Duration> durations = new ArrayList<Duration>();
    private List<Money>  amounts = new ArrayList<Money>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.splitByDay()) { // call에게 일자별 통화 기간 분리를 요청
            for(int loop=0; loop < starts.size(); loop++) {
                LocalTime from = from(interval, starts.get(loop));
                LocalTime to = to(interval, ends.get(loop));

                long intervalSeconds = Duration.between(from, to).getSeconds();
                long criteriaSeconds = durations.get(loop).getSeconds();

                Money intervalFee = amounts.get(loop);
                result.plus(intervalFee.times(intervalSeconds / criteriaSeconds));
            }
        }
        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        LocalTime intervalFrom = interval.getFrom().toLocalTime();
        return intervalFrom.isBefore(from) ? from : intervalFrom;
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        LocalTime intervalTo = interval.getTo().toLocalTime();
        return intervalTo.isAfter(to) ? to : intervalTo;

```
### 요일별 방식 구현하기
- 요일별로 규칙을 다르게 설정할 수 있다.
- 시간대별 방식은 4개의 List를 이용해서 구현했지만, 요일별 방식은 `DayOfWeekDiscountRule` 클래스 한개로 구현
```java
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>(); //요일의 목록
    private Duration duration = Duration.ZERO; //단위 시간
    private Money amount = Money.ZERO; //단위 요금
    // 생성자 생략
    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds() / duration.getSeconds());
        }
        return Money.ZERO;
    }
}
```
- 시간대별 방식과 동일하게 통화 기간을 날짜 경계로 분리하고 분리된 각 통화 기간을 요일별로 설정된 요금 정책에 따라 적절하게 계산해야 한다.
```java
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();
    // 생성자 생략
    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.getInterval().splitByDay()) {
            for(DayOfWeekDiscountRule rule: rules) { 
                result.plus(rule.calculate(interval));
            }
        }
        return result;
    }
}
```

### 구간별 방식 구현하기
#### 문제점
- FixedFeePOlicy, TimeOfDayDiscountPolicy, DayOfWeekDiscountPolicy의 세 클래스 모두 유사한 문제를 해결하고 있음에도 설계에 일관성이 없다.
  - 새로운 구현을 추가해야 하는 상황에 힘들다.
  - 기존의 구현을 이해해야 할 때 힘들다.
- 유사한 기능을 서로 다른 방싱으로 구현하면 혼돈을 준다.

### 결론
- 유사한 기능은 유사한 방식으로 구현해야 한다.
- DurationDiscountRule 클래스는 FixedFeePolicy 클래스를 상속해 구간별 방식을 구현하고자 했다.
- 하지만 앞서 말했듯, 일관성을 고려하지 않았기에 문제가 있다.


## 02. 설계에 일관성 부여하기
- 협력을 일관성 있게 만들기 위한 두 가지 지침
  - 변하는 개념을 변하지 않는 개념으로부터 분리하라.
  - 변하는 개념을 캡슐화하라.

### 조건 로직 대 객체 탐색
- 객체지향에서는 조건 로직을 객체 사이의 이동으로 바꾼다.
- Movie는 현재 할인 정책이 어떤 종류인지 확인하지 않고, discountPolicy에 필요한 메시지를 전송만 한다.
```java
public class Movie{
  private DiscountPolicy discountPolicy;

  public Money calculateMovieFee(Screening screening){
      return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```
- 조건 로직을 객체 사이의 이동으로 대체하기 위해 커다란 클래스르 더 작은 클래스들로 분리해야 한다.
  - 변경의 이유와 주기를 기준으로 한다.
  - 단일 책임의 원칙을 따르도록 클래스를 분리해야 한다.
- 큰 메서드 안에 있던 조건 로직들을 변경의 압력에 맞춰 작은 클래스들로 분리하면 인스턴스들 사이의 협력 패턴에 일관성을 부여하기 쉽다.

### 캡슐화 다시 살펴보기
- 캡슐화는 데이터 은닉 이상이고, 변할 수 있는 모든 '개념'을 감추는 것이다.
- 객체의 퍼브릭 인터페이스와 구현을 분리하는 것이 캡슐화의 가장 대표적인 예

<img width="664" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/ff772ec2-ad69-4c4e-8894-eaad52031b3b">

- 캡슐화 종류
  - 데이터 캡슐화
  - 메서드 캡슐화
  - 객체 캡슐화
  - 서브타입 캡슐화
- 데이터 캡슐화와 메서드 캡슐화는 개별 객체에 대한 변경을 관리하기 위해 사용
- 객체 캡슐화와 서브타입 캡슐화는 협력에 참여하는 객체들의 관계에 대한 변경을 관리하기 위해 사용
- 협력을 일관성있게 만들기 위한 일반적인 방법 -> 서브타입 캡슐화 + 객체 캡슐화 (조합)
- 서브타입 캡슐화와 객체 캡슐화를 적용하는 방법
  - 변하는 부분을 분리해서 타입 계층을 만든다.
  - 변하지 않는 부분의 일부로 타입 게층을 합성한다.

## 03. 일관성 있는 기본 정책 구현하기
### 변경 분리하기
- 변하는 개념과 변하지 않는 개념을 분리하는 것이다.
- 시간대별, 요일별, 구간별 / 고정요금 이렇게 나누기

### 시간대별, 요일별, 구간별 묶기
공톰점 - 변하지 않는 부분   
- 기본 정책은 한 개 이상의 `규칙`으로 구성된다.
- 하나의 `규칙`은 `적용조건`과 `단위요금`의 조합이다.

차이점 - 변하는 부분
- 각 기본 정책별로 요금을 계산하는 '적용조건'의 형식이 다르다.
  - 시간대별 : 통화 시간이 특정 시간 구간에 포함될 경우에만 요금을 계산
  - 요일병 : 특정 요일에 해당할 경우에만 요금을 계산
  - 구간별 : 통화 시간의 구간이 경과 시간 안에 포함될 경우에만 요금을 계산

### 변경 캡슐화하기
- 변경을 캡슐화해서 파급효과를 줄여야 한다.
- 변하는 부분의 공통점을 추상화하고 변하지 않는 부분이 오직 이 추상화에만 의존하도록 관계를 제한한다.

<img width="658" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/bcb8f459-0196-4154-8320-31a5e4ebed63">

- FeeRule: 규칙을 구현하는 클래스이며, 단위요금은 feePerDuration에 저장
- FeeCondition: 적용조건을 구현하는 인터페이스, 변하는 부분을 캡슐화하는 추상화
- TimeOfDayFeeCondition: 시간대별 방식
- DayOfWeekFeeCondition: 요일별 방식
- DuratinoFeeCondition: 구간별 방식

### 협력 패턴 설계하기
<img width="676" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/984104bf-6308-467f-83ec-70adb51d611a">


- FeeRule: FeeCondition의 인스턴스에게 findTimeIntervals메시지 전송
- findTimeIntervals: 통화 기간 중에서 '적용조건'을 만족하는 구간을 가지는 DateTimeInterval의 List를 반환
- FeeRule: feePerDuratino 정보를 이용해 반환받은 기간만큼의 통화 요금을 계산한 후 반환
<img width="602" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/03c92fac-4ce7-4f2b-9ede-eddc55cd1dae">

- 시간대별 방식으로 요금을 게산하고 싶다면 TimeOfDayFeeCondition -> FeeCondition 대신

### 추상화 수준에서 협력 패턴 구현하기

- 적용조건을 표현하는 추상화인 FeeCondition
```java
public interface FeeCondition {
    List<DateTimeInterval> findTimeIntervals(Call call);
}
```
```java
public class FeeRule {
    private FeeCondition feeCondition; //적용조건
    private FeePerDuration feePerDuration; //단위요금
    // 생성자 생략
    public Money calculateFee(Call call) { 
        return feeCondition.findTimeIntervals(call)
                .stream()
                .map(each -> feePerDuration.calculate(each))  // 단위요금 계산
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```
- FeePerDuration: 단위 시간당 요금이라는 개념을 표현하고 이 정보를 이용해 일정 기간 동안의 요금을 계산

```java
public class FeePerDuration {
    private Money fee;
    private Duration duration;

    //생성자 생략

    public Money calculate(DateTimeInterval interval) {
        return fee.times(Math.ceil((double)interval.duration().toNanos() / duration.toNanos()));
    }
}
```
- BasicRatePolicy: FeeRule 컬렉션을 이용해 전체 통화 요금을 게산하도록 수정
```java
public final class BasicRatePolicy implements RatePolicy {
    private List<FeeRule> feeRules = new ArrayList<>();
    // 생성자 생략
    @Override
    public Money calculateFee(Phone phone) {
        return phone.getCalls()
                .stream()
                .map(call -> calculate(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

    private Money calculate(Call call) {
        return feeRules
                .stream()
                .map(rule -> rule.calculateFee(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```

### 구체적인 협력 구현하기
- 현재의 요금제를 구별하는 기준은 FeeCondition을 대체하는 객체의 타입이 무엇인가에 달려 있다.

#### 시간대별 정책
- DateTimeInterval의 splitByDay메서드를 호출해 날짜별로 시간간격을 분할 -> from과 to 사이의 시간대를 구한다.

```java
public class TimeOfDayFeeCondition implements FeeCondition {
    private LocalTime from;
    private LocalTime to;

    // 생성자 생략

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval().splitByDay() 
                .stream()
                .filter(each -> from(each).isBefore(to(each)))
                .map(each -> DateTimeInterval.of(
                                LocalDateTime.of(each.getFrom().toLocalDate(), from(each)),
                                LocalDateTime.of(each.getTo().toLocalDate(), to(each))))
                .collect(Collectors.toList());
    }
    ...
}
```

#### 요일별 정책
- Call 의 기간 중에서 요일에 해당하는 기간만을 추출해 반환

```java
public class DayOfWeekFeeCondition implements FeeCondition {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();

    // 생성자 생략

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval()
                .splitByDay()
                .stream()
                .filter(each -> dayOfWeeks.contains(each.getFrom().getDayOfWeek()))
                .collect(Collectors.toList());
    }
}
```

#### 구간별 정책
```java
public class DurationFeeCondition implements FeeCondition {
    private Duration from;
    private Duration to;

    // 생성자 생략

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        if (call.getInterval().duration().compareTo(from) < 0) {
            return Collections.emptyList();
        }
        return Arrays.asList(DateTimeInterval.of(
                call.getInterval().getFrom().plus(from),
                call.getInterval().duration().compareTo(to) > 0 ?
                        call.getInterval().getFrom().plus(to) :
                        call.getInterval().getTo()));
    }
}
```

- 변경을 캡슐화해 협력을 일관성 있게 만들면 얻을 수 있는 장점
  - 변하지 않는 부분을 재사용할 수 있다.
  - 새로운 기능을 추가하기 위해 변하는 부분만 구현하면 된다.
  - 코드의 재사용성이 향상되고, 테스트해야 하는 코드의 양이 감소한다.
  - 기능을 추가하거나 변경할 때 설계의 일관성이 무너지지 않는다.
-  공통 코드의 구조와 협력 패턴은 모든 기본 정책에 걸쳐 동일하므로 코드를 한 번 이해하면 다른 코드를 이해하는 데 그대로 적용할 수 있다.
-  유사한 기능에 대해 유사한 협력 패턴을 적용하는 것은 개념적 무결성을 유지할 수 있는 가장 효과적인 방법이다.

### 협력 패턴에 맞추기
- 고정요금 정책은 위 3개 정책과는 다르다.
  - ‘규칙’이라는 개념이 필요하지 않고 ‘단위요금’만 있으면 된다.
- 또 다른 협력 패턴을 만드는 것보다 이상해도 일관성을 유지하는 설계를 선택하는 것이 현명하다.
- FeeCondition을 추가하고, Call의 전체 통화 시간을 반환하게 한다.
- 개념적 무결성을 무너뜨리는 것보다 약간의 부조화를 수용하는 편이 낫다.


```java
public class FixedFeeCondition implements FeeCondition {
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return Arrays.asList(call.getInterval());
    }
}
```

### 패턴을 찾아라
- 일관성 있는 협력의 핵심은 변경을 분리하고 캡슐화하는 것이다.
   - 변경을 캡슐화할 수 있는 적절한 추상화를 찾고, 이 추상화에 변하지 않는 공통적인 책임을 할당하라.
   - 현재의 구조가 변경을 캡슐화하기 적합하지 않다면 코드를 수정하지 않고 변경을 수용할 수 있도록 협력과 코드를 리팩토링

