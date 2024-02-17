<h1>일관성 있는 협력</h1>

재사용을 위해서는 객체들의 협력 방식을 일관성 있게 만들어야한다.</br>
일관성은 설계에 드는 비용을 감소시킨다.</br>

<h2>핸드폰 과금 시스템 변경하기</h2>

기본 정책</br></br>

1. 고정요금 방식 : FixedFeePolicy
2. 시간대별 방식 : TimeOfDayDiscsountPolicy
3. 요일별 방식   : DayOfWeekDiscountPolicy
4. 구간별 방식   : DurationDiscountPolicy


![KakaoTalk_20240216_150336859](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/1b08cf06-f41f-442b-94bf-3ece77010c95)



<h3>고정요금 방시 구현하기</h3>

```
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;

    public FixedFeePolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

```

11장 내용과 동일하고 RegularPolicy 클래스의 이름을 FixedFeePolicy로 수정한다.</br>

<h3>시간대별 방식 구현하기</h3>

경우의 수</br>

1. 하루동안 일정 시간에 통화
2. 하루동안 시간에 겹처 통화
3. 며칠에 걸처 통화


```
public class DateTimeInterval {
    private LocalDateTime from;
    private LocalDateTime to;

    public static DateTimeInterval of(LocalDateTime from, LocalDateTime to) {
        return new DateTimeInterval(from, to);
    }

    public static DateTimeInterval toMidnight(LocalDateTime from) {
        return new DateTimeInterval(from, LocalDateTime.of(from.toLocalDate(), LocalTime.of(23, 59, 59, 999_999_999)));
    }

    public static DateTimeInterval fromMidnight(LocalDateTime to) {
        return new DateTimeInterval(LocalDateTime.of(to.toLocalDate(), LocalTime.of(0, 0)), to);
    }

    public static DateTimeInterval during(LocalDate date) {
        return new DateTimeInterval(
                LocalDateTime.of(date, LocalTime.of(0, 0)),
                LocalDateTime.of(date, LocalTime.of(23, 59, 59, 999_999_999)));
    }

    private DateTimeInterval(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration duration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }

    public LocalDateTime getTo() {
        return to;
    }
}

```


```
public class Call {
	private DateTimeInterval interval;

	public Call(LocalDateTime from, LocalDateTime to) {
		this.interval = DateTimeInterval.of(from, to);
	}

	public Duration getDuration() {
		return interval.duration();
	}

	public LocalDateTime getFrom() {
		return interval.getFrom();
	}

	public LocalDateTime getTo() {
		return interval.getTo();
	}

	public DateTimeInterval getInterval() {
		return interval;
	}

	public List<DateTimeInterval> splitByDay() {
		return interval.splitByDay();
	}
}

```

통화 기간을 일자 단위로 나누는 작업의 정보 전문가는 Call 객체이다.</br>
하지만 Call은 통화 기간은 잘 알아도 기간 자체를 처리하는 방법에 대해서는 전문가가 아니다.</br>
기간을 처리하는 방법에 대한 전문가는 바로 DateTimeInterval이다</br></br>

따라서, 통화 기간을 일자 단위로 나누는 책임은 DateTimeInterval에게 할당하고 Call이 DateTimeInterval에게 분할을 요청하도록 협력을 설계하는 것이 적절하다.</br>

두 번째 작업인 시간대별로 분할하는 작업의 정보 전문가는 TimeOfDayDiscountPolicy이다.</br>

로직</br>
1. TimeOfDayDiscountPolicy는 통화 기간을 알고 있는 Call에게 일자별로 통화 기간을 분리할 것을 요청
2. Call은 이 요청을 DateTimeInterval에게 위임
3. DateTimeInterval은 기간을 일자 단위로 분할한 후 분할된 목록을 반환한다.
4. Call은 반환된 목록을 그대로 TimeOfDayDiscountPolicy에게 반환
5. TimeOfDayDiscountPolicy는 일자별 기간의 목록을 대상으로 루프를 돌리면서 각 시간대별 기준에 맞는 시작시간과 종료기간을 얻음


```
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<LocalTime>();
    private List<LocalTime> ends = new ArrayList<LocalTime>();
    private List<Duration> durations = new ArrayList<Duration>();
    private List<Money>  amounts = new ArrayList<Money>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.splitByDay()) {
            for(int loop=0; loop < starts.size(); loop++) {
                result.plus(amounts.get(loop).times(Duration.between(from(interval, starts.get(loop)),
                        to(interval, ends.get(loop))).getSeconds() / durations.get(loop).getSeconds()));
            }
        }
        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        return interval.getFrom().toLocalTime().isBefore(from) ? from : interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        return interval.getTo().toLocalTime().isAfter(to) ? to : interval.getTo().toLocalTime();
    }
}

```

시간 시간의 List, 종료 시간의 List, 단위 시간의 List, 단위 요금 List로 나뉨.</br></br>

Call의 splitByDay 메서드는 DateTimeInterval에 요청을 전달한 후 응답을 반환하는 위임 메서드</br>
DateTimeInterval 클래스의 splitByDay메서드는 통화 기간을 일자별로 분할해서 반환.</br>
days 메서드는 from과 to 사이에 포함된 날짜 수를 반환</br>    

<h3>요일별 방식 구현하기</h3>

```
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;

    public DayOfWeekDiscountRule(List<DayOfWeek> dayOfWeeks,
                                 Duration duration, Money  amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this.amount = amount;
    }

    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds() / duration.getSeconds());
        }

        return Money.ZERO;
    }
}

```

요일별 방식을 개발 규칙은 DayOfWeekDiscountRule이라는 하나의 클래스를 구현하는 것이 더 나은 설계라고 판단됐다고 한다.</br>
DayOfWeekDiscountWeek 클래스는 규칙을 정의하기 위해 필요한 요일의 목록, 단위시간, 단위 요금을 인스턴스 변수로 포함한다.</br>
calculate 메서드는 파라미터로 전달된 interval이 요일 조건을 만족시킬 경우 단위 시간과 단위 요금을 이용해 통화 요금을 계산한다.</br>

 - 시간대별 방식과 동일하게 통화 기간을 날짜 경계로 분리하고 분리된 각 통화 기간을 요일별로 설정된 요금 정책에 따라 적절하게 계산해야 한다.

```
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();

    public DayOfWeekDiscountPolicy(List<DayOfWeekDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.getInterval().splitByDay()) {
            for(DayOfWeekDiscountRule rule: rules) { result.plus(rule.calculate(interval));
            }
        }
        return result;
    }
}

```


<h3>구간별 방식 구현하기</h3>

지금까지 구현 방식은 일관성이 맞지 않는다.</br></br>

시간대별 방식 : TimeOfDayDiscountPolicy는 규칙을 구성하는 시작 일자와 종료 일자, 단위 시간, 단위 요금 각각을 별도의 List로 관리한다.</br>
요일별   방식 : DayOfWeekDiscountPolicy는 DayOfWeekDiscountRule이라는 별도의 클래스를 사용했다.</br>
고정요금 방식 : 하나의 규칙만으로 구성 돼있다.</br></br>

이 상태에서 새로운 기본 정책을 추가하면 추가할수록 코드 사이의 일관성은 점점 더 어긋나게 된다.</br>
일관성 없는 코드가 가지는 두 번째 문제점은 코드를 이해하기 어렵다.</br>
=> 유사한 기능은 유사한 방식으로 구현해야 한다.</br>

```
public class DurationDiscountRule extends FixedFeePolicy {
    private Duration from;
    private Duration to;

    public DurationDiscountRule(Duration from, Duration to, Money amount, Duration seconds) {
        super(amount, seconds);
        this.from = from;
        this.to = to;
    }

    public Money calculate(Call call) {
        if (call.getDuration().compareTo(to) > 0) {
            return Money.ZERO;
        }

        if (call.getDuration().compareTo(from) < 0) {
            return Money.ZERO;
        }

        // 부모 클래스의 calculateFee(phone)은 Phone 클래스를 파라미터로 받는다.
        // calculateFee(phone)을 재사용하기 위해 데이터를 전달할 용도로 임시 Phone을 만든다.
        Phone phone = new Phone(null);
        phone.call(new Call(call.getFrom().plus(from),
                            call.getDuration().compareTo(to) > 0 ? call.getFrom().plus(to) : call.getTo()));

        return super.calculateFee(phone);
    }
}

```

```
public class DurationDiscountPolicy extends BasicRatePolicy {
    private List<DurationDiscountRule> rules = new ArrayList<>();

    public DurationDiscountPolicy(List<DurationDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DurationDiscountRule rule: rules) {
            result.plus(rule.calculate(call));
        }
        return result;
    }
}

```

요일별 방식처럼 규칙을 정의하는 새로운 클래스를 추가한다.</br>
요일별 방식과 다른 점은 코드를 재사용하기 위해 FixedFeepolicy 클래스를 상속한다.</br>
