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


<h2>설계에 일관성 부여하기</h2>

일관성 있는 설계를 만드는데 훌륭한 조건~~</br></br>

1. 다양한 설계 경험을 익히자</br>
2. 널리 알려진 디자인 패턴을 학습하고 변경이라는 문맥 안에서 디자인 패턴을 적용해보자</br>


협력을 일관성 있게 만들기 위한 지침</br>
 - 변하는 개념을 변하지 않는 개념으로부터 분리하라.</br>
 - 변하는 개념을 캡슐화하라.</br>

<h3>조건 로직 대 객체 탐색</h3>

```
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                        condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                        condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }

            if (discountable) {
                break;
            }
        }

        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }

            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}

```

하나는 할인 조건의 종류를 결정하는 부분이고 다른 하난는 할인 정책을 결정하는 부분이다.</br>
이 설계가 나쁜 이유는 변경의 주기가 서로 다른 코드가 한 클래스 안에 뭉쳐져 있다.</br></br>

![KakaoTalk_20240217_164059492_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/ea3f2f0b-ede0-491e-8dfc-e399635b3590)

하지만 DiscountPolicy와 DiscountCondition을 이미지를 보면</br>
Movie는 DiscountPolicy로 향하는 참조를 통해 메시지를 전달할 뿐이다.</br></br>

위와 같이 조건 로직을 객체 사이의 이동으로 대체하기 위해서는 커다란 클래스를 더 작은 클래스들로 분리해야 한다.</br>
클래스를 분리하기 위해 어떤 기준을 따르면 좋을까??</br>
기준은 변경의 이유와 주기다.</br>
클래스는 명확히 단 하나의 이유에 의해서만 변경돼야 하고 클래스 안의 모든 코드는 함께 변경돼야 한다.(단일 책임 원칙)</br></br>

큰 메서드 안에 뭉쳐있던 조건 로직들을 변경의 압력에 맞춰 작은 클래스들로 분리하고 나면 인스턴스들 사이의 협력 패턴에 일관성을 부여하기가 더 쉬워진다.</br>
유사한 행동을 수행하는 작은 클래스들이 자연스럽게 역할이라는 추상화로 묶이게 되고 역할 사이에서 이뤄지는 협력 방식이 전체 설계의 일관성을 유지할 수 있게 이끌어준다.</br>

일관성 있는 렵력을 위한 지침</br></br> 

1. 변하는 개념을 변하지 않는 개념으로부터 분리하라.
2. 변하는 개념을 캡슐화하라.


1 => 할인 정책과 할인 조건의 타입을 체크하는 하나하나의 조건문이 개별적인 변경이었다는 점에서 각 조건문을 개별적인 객체로 분리했고 이 객체들과 일관성 있게 협력하기 위해 타입 계층을 구성했다.</br>

2 => 추상 클래스인 DiscountPolicy를 부모로 삼아 상속 계층을 구성한 이유가 바로 Movie로부터 구체적인 할인 정책들을 캡슐화하기 위해서다.</br>
    실행 시점에 Movie는 자신과 협력하는 객체의 구체적인 타입에 대해 알지 못한다.</br>

<h3>캡슐화 다시 살펴보기</h3>

캡슐화란 단순히 데이터를 감추는 것이 아니다. 소프트웨어 안에서 변할 수 있는 모든 '개념'을 감추는 것이다.</br></br>

캡슐화의 가장 대표적인 예는 객체의 퍼블릭 인터페이스와 구현을 분리하는 것이다.</br>

![KakaoTalk_20240217_164059492](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/551b3afb-321b-4e24-b7cd-16799233154e)


데이터 캡슐화 : Movie 클래스의 인스턴스 변수 title 가시성은 private이기 때문에 외부에서 접근할 수 없다.</br>
메서드 캡슐화 : DiscountPolicy 클래스에서 정의돼 있는 getDiscountAmount 메서드의 가시성은 protect다. 클래스의 외부에서는 이 메서드에 직접 접근할 수 없고 클래스 내부와 서브클래스에서만 접근이 가능하다.</br>
객체 캡슐화   : Movie 클래스는 DiscountPolicy 타입의 인스턴스 변수 discountPolicy를 포함한다. 이 인스턴스 변수는 private 가시성을 가지기 때문에 Movie와 DiscountPolicy 사이의 관계를 변경하더라도 외부에서 영향을 미치지 않는다.</br>

서브타입 캡슐화 : Movie는 DiscountPolicy에 대해서는 알고 있지만 AmountDiscountPolicy와 PercentDiscountPolicy에 대해서는 알지 못한다. 그러나 실제로 실행 시점에는 이 클래스들의 인스턴스와 협력할 수 있다. 이것은 기반 클래스인 DiscountPolicy와의 추상적인 관계가 AmountDiscountPolicy와 PercentDiscountPolicy의 존재를 감추고 있기 때문이다.</br>

서브타입 캡슐화와 객체 캡슐화를 적용하는 방법</br>


1. 변하는 부분을 분리해서 타입 계층을 만든다. </br>
 - 변하지 않는 부분으로부터 변하는 부분을 분리. 변하는 부분들의 공통적인 행동을 추상 클래스나 인터페이스로 추상화한 후 변하는 부분들이 이 추상 클래스나 인터페이스를 상속받게 만든다.</br>

2. 변하지 않는 부분의 일부로 타입 계층을 합성한다.</br>
   - 앞에서 구현한 타입 계층을 변하지 않는 부분에 합성한다. 변하지 않는 부분에서는 변경되는 구체적인 사항에 결합돼서는 안된다. 의존성 주입과 같이 결합도를 느슨하게 유지할 수 있는 방법을 이용해 오직 추상화에만 의존하게 만든다.</br>



<h2>일관성 있는 기본 정책 구현하기</h2>

<h3>변경 분리하기</h3>

단위 요금은 말 그대로 단위시간당 요금 정보를 의미한다. 적용조건은 통화 요금을 계산하는 조건을 의미한다.</br></br>

단위 요금과 적용 조건이 모여 하나의 규칙을 구성한다.</br></br>

시간대별, 요일별, 구간별 방식의 차이점은 각 기본 정책별로 요금을 계산하는 '적용조건'의 형식이 다르다는 것이다.</br>
모든 규칙에 '적용 조건'이 포함된다는 사실은 변하지 않지만 실제 조건의 세부적인 내용은 다르다.</br>

변하지 않는 것과 변하는 것을 분리하라</br>
=> 변하지 않는 규칙으로부터 변하는 '적용 조건'을 분리해야 한다.</br>

<h3>변경 캡슐화하기</h3>

여기서 변하지 않는 것은 규칙이고 변하는 것은 적용 조건이다.</br>
따라서 규칙으로부터 적용조건을 분리해서 추상화한 후 시간대별, 요일별, 구간별 방식을 이 추상화의 서브타입으로 만든다.</br>

이미지

FeeRule이 FeeCondition을 합성 관계로 연결하고 있다.</br>
FeeRule의 인스턴스 변수인 feePerDuration에 저장돼 있다.</br>
FeeCondtion은 적용조건을 구현하는 인터페이스이며 변하는 부분을 캡슐화하는 추상화다.</br>
TimeOfDayFeeCondition은 시간대별 방식, DayOfWeekFeeCondition은 요일별 방식, DurationFeeCondition은 구간별 방식이다.</br></br>

FeeRule은 추상화인 FeeCondtion에 대해서만 의존하기 때문이 적용조건이 변하더라도 영향을 받지 않는다. 즉, 적용조건이라는 변경에 대해 캡슐화돼 있다.</br>

<h3>협력 패턴 설계하기</h3>

이미지

1. BasicRatePolicy의 calculateFee 메서드는 인자로 전달받은 통화 목록의 전체 요금을 계산
2. BasicRatePolicy는 목록에 포함된 각 Call별로 FeeRule의 calculateFee메서드를 실행
3. FeeRule은 하나의 Call에 대해 요금을 계산하는 책임을 수행( FeeRule은 단위 시간당 요금인 feePerDuration과 요금을 적용할 조건을 판단하는 적용조건인 FeeCondition의 인스턴스를 알고 있다.)

하나의 Call 요금을 계산하기 위해서는 두 개의 작업이 필요하다.</br>
1. 전체 통화 시간을 각 규칙의 적용조건을 만족하는 구간들로 나누는 것
2. 분리된 통화 구간에 단위요금을 적용해서 요금을 계산하는 것


이미지

FeeRule은 FeeCondtion의 인스턴스에게 findTimeIntervals 메시지를 전송한다.</br>
findTimeIntervals는 통화 기간 중에서 적용조건을 만족하는 구간을 가지는 DateTimeIntervals의 List를 반환한다.</br>
FeeRule은 feePerDuration 정보를 이용해 반환받은 기간 만큼의 통화 요금을 계산한 후 반환한다.</br></br>


<h3>추상화 수준에서 협력 패턴 구현하기</h3>

```
public interface FeeCondition {
    List<DateTimeInterval> findTimeIntervals(Call call);
}

```


```
public class FeeRule {
    private FeeCondition feeCondition;
    private FeePerDuration feePerDuration;

    public FeeRule(FeeCondition feeCondition, FeePerDuration feePerDuration) {
        this.feeCondition = feeCondition;
        this.feePerDuration = feePerDuration;
    }

    public Money calculateFee(Call call) {
        return feeCondition.findTimeIntervals(call)
                .stream()
                .map(each -> feePerDuration.calculate(each))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}

```

FeeRule의 calculateFee메서드는 FeeCondition에게 findTimeIntervals 메시지를 전송해서 조건을 만족하는 시간의 목록을 반환받은 후 feePerDuration의 값을 이용해 요금을 계산한다.</br>

```
public class FeePerDuration {
    private Money fee;
    private Duration duration;

    public FeePerDuration(Money fee, Duration duration) {
        this.fee = fee;
        this.duration = duration;
    }

    public Money calculate(DateTimeInterval interval) {
        return fee.times(Math.ceil((double)interval.duration().toNanos() / duration.toNanos()));
    }
}

```

FeePerDuration클래스는 단위 시간당 요금이라는 개념을 표현하고 이 정보를 이용해 일정 기간 동안의 요금을 계산하는 calculate 메서드를 구현</br>



```
public final class BasicRatePolicy implements RatePolicy {
    private List<FeeRule> feeRules = new ArrayList<>();

    public BasicRatePolicy(FeeRule ... feeRules) {
        this.feeRules = Arrays.asList(feeRules);
    }

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

BasicRatePolicy가 FeeRule의 컬렉션을 이용해 전체 통화 요금을 계산하도록 수정할 수 있다.</br>
