<h1>의존성 관리하기</h1>

협력은 필수적이지만 과도한 협력은 설계를 곤경에 빠트릴 수 있다. </br>
협력은 객체가 다른 객체에 대해 알 것을 강요한다.</br>
다른 객체와 협력하기 위해서는 그런 객체가 존재한다는 사실을 알고 있어야 한다.</br>
객체가 수신할 수 있는 메시지에 대해서도 알고 있어야 한다. </br></br>

충분히 협력적이면서도 유연한 객체를 만들기 위해 의존성을 관리하는 방법을 알아보자</br>

<h2>의존성 이해하기</h2>


<h3>변경과 의존성</h3>

의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가진다.
</br></br>
실행 시점 : 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.</br>
구현 시점 : 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.</br></br>

```

public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0&&
                endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }



```


실행 시점에 PeriodCondition의 인스턴스가 정상적으로 동작하기 위해서는 Screening의 인스턴스가 존재해야 한다.</br>
만약 Screening의 인스턴스가 존재하지 않거나 getStartTime 메시지를 이해할 수 없다면 PeriodCondition의 isSatisfiedBy 메서드는 예상했던 대로 동작하지 않을 것이다.</br></br>

이처럼 어떤 객체가 예정된 작업을 정상적으로 수행하기 위해 다른 객체를 필요로 하는 경우 두 객체 사이의 의존성을 존재한다고 말한다.</br></br>

구현 시점에서 PeriodConditon이 DayOfWeek, LocalTime, Screening에 대해 의존성을 가지는데 이 중 이름이 변경되면 결국 PeroidCondition의 코드도 바뀐다.</br>


<h3>의존성 전이</h3>

Screening이 가지고 있는 의존성이 Screening에 의존하고 있는 PeriodCondition으로도 전파되는 것이다.</br>
따라서, Screening이 Movie, LocalDateTime, Customer에 의존하기 때문에 PeriodCondition 역시 간접적으로 Movie, LocalDateTime, Customer에 의존하게 된다.</br></br>

직접 의존성 : 한 요소가 다른 요소에 직접 의존하는 경우</br>
간접 의존성 : 직접적인 관계는 존재하지 않지만 의존성 전이에 의해 영향이 전파되는 경우를 가르킴</br>



<h3>런타임 의존성과 컴파일타임 의존성</h3>

런타임 : 애플리케이션이 실행되는 시점</br>
컴파일 타임 : 작성된 코드를 컴파일하는 시점을 가리키지만 문맥에 따라서 코드 그 자체를 가리키기도 함.</br></br></br>


객체지향 애플리케이션</br></br>

런타임 의존성 : 런타임의 주인공 : 객체 => 객체 사이의 의존성</br>
컴파일타임 의존성 : 코드 관점에서 주인공은 클래스 => 클래스 사이의 의존성</br></br>


컴파일타임 의존성에서는</br>


이미지


Movie가 AmountDiscountPolicy 등에 직접 의존하지 않고 이의 추상 클래스인 DiscounstPolicy에 의존한다. 이로 인해 각 정책에 대해서 상세하게 알지 못한다.</br></br>



런타임 의존성에서는</br>

이미지

Movie가 AmountDiscountPolicy와 PercentDiscountPolicy에 각각 의존하게 된다.</br>
이로 인해 전체적인 결합도를 높이고 새로운 할인 정책을 추가하기 어렵다.</br></br>

컴파일 구조와 런타임 구조 사이의 거리가 멀면 멀수록 설계가 유연해지고 재사용 가능해진다.</br>



