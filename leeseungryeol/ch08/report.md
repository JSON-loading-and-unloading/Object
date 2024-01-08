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



![KakaoTalk_20240108_020625309_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/0af16ba7-bb80-4f1a-b327-ee1ba900a90c)


Movie가 AmountDiscountPolicy 등에 직접 의존하지 않고 이의 추상 클래스인 DiscounstPolicy에 의존한다. 이로 인해 각 정책에 대해서 상세하게 알지 못한다.</br></br>



런타임 의존성에서는</br>

![KakaoTalk_20240108_020625309_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/c31fb597-8258-43ed-9197-bb1c95f206e0)


Movie가 AmountDiscountPolicy와 PercentDiscountPolicy에 각각 의존하게 된다.</br>
이로 인해 전체적인 결합도를 높이고 새로운 할인 정책을 추가하기 어렵다.</br></br>

컴파일 구조와 런타임 구조 사이의 거리가 멀면 멀수록 설계가 유연해지고 재사용 가능해진다.</br>

<h3>컨텍스트 독립성</h3>

클래스가 특정한 문맥에 강하게 결합될수록 다른 문맥에서 사용하기 더 어려워진다.</br>
클래스가 사용될 특정한 문맥에 대해 최소한의 가정만으로 이뤄져 있다면 다른 문맥에서 재사용하기가 더 수월해진다.</br>
=> 설계가 유연해지기 위해서는 가능한 한 자신이 실행될 컨텍스트에 대한 구체적인 정보를 최대한 적게 알아야한다.</br></br>

<h3>의존성 해결하기</h3>

의존성 해결 : 컴파일 타임 의존성을 실행 컨텍스트에 맞게 적절한 런타임 의존성으로 교체하는 것


의존성 해결 방법
- 객체를 생성하는 시점에 생성자를 통해 의존성 해결
- 객체 생성 후 setter 메서드를 통해 의존성 해결
- 메서드 실행 시 인자를 이용해 의존성 해결


1. 객체를 생성하는 시점에 생성자를 통해 의존성 해결

```

Movie avatar = new Movie();
avatar.caculateFee(); //discount policy가 아직 설정되지 않았기 때문에 NPE가 발생한다
avatar.setDiscountPolicy(new AmountDiscountPolicy());


```


2. 객체 생성 후 setter 메서드를 통해 의존성 해결

```
Movie avatar = new Movie(new AmountDiscountPolicy());
avatar.setDiscountPolicy(new PercentDiscountPolicy());

```

setter메서드를 이용하면 객체를 생성한 이유에도 의존하고 있는 대상을 변경할 수 있는 가능성을 열어 놓고 싶은 경우에 유용


   ```

Movie movie = new Movie(new AmountDiscountPolicy()); //1
movie.setDiscountPolicy(new PercentDiscountPolicy()); //2


   ```

   완전 상태의 객체를 생성 후, 필요에 따라 setter 메서드를 이용해 의존 대상을 변경


3. 메서드 실행 시 인자를 이용해 의존성 해결

```
   
public class Movie{

   public Money calculateMovieFee(Screening screening, DiscountPolicy discountPolicy){

    return fee.minus(discountPolicy.calculateDiscountAmount(screening));

   }

}


```

협력 대상에 대해 지속적으로 의존 관계를 맺을 필요 없이 메서드가 실행되는 동안만 일시적으로 의존 관계가 존재해도 무방하거나, 메서드가 실행될 때마다 의존 대상이 매번 달라져야 하는 경우에 유용


<h2>유연한 설계</h2>

<h3>의존성과 결합도</h3>

```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, PercentDiscountPolicy percentDiscountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

```

위 코드에서 PercentDiscountPolicy라는 구체적인 클래스에 의존하게 만들었기 때문에 다른 종류의 할인 정책이 문맥에서 Movie를 재사용할 수 있는 가능성을 없애 버렸다.</br>

해결 : 의존성을 바람직하게 만들기
</br></br>
추상 클래스 DiscountPolicy는 사용 </br>
바람직한 의존성은 재사용성과 관련이 있다.</br>
어떤 의존성이 다양한 환경에서 클래스를 재사용할 수 없도록 제한한다면 그 의존성은 바람직하지 못한 것이다.</br></br>

바람직한 의존성이란 컨텍스트에 독립적인 의존성을 의미하며 다양한 환경에서 재사용될 수 있는 가능성을 열어놓는 의존성을 의미한다.</br></br>


<h3>지식이 결합을 낳는다</h3>

Movie 클래스가 추상 클래스인 DiscountPolicy 클래스에 의존하는 경우에는 구체적인 계산 방법은 알 필요가 없다.</br>
따라서 Movie가 PercentDiscountPolicy에 의존하는 것보다 DiscountPolicy에 의존하는 경우 알아야 하는 지식의 양이 적기 때문에 결합도가 느슨해진다.</br>

<h3>추상화에 의존하라</h3>

추상화란 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로서 복잡도를 극복하는 방법이다.</br></br>

DiscountPolicy클래스는 PercentDiscountPolicy클래스가 비율 할인 정책에 따라 할인 요금을 계산한다는 사실을 숨겨주기 때문에 PercentDiscountPolicy의 추상화다.</br>

 - 구체 클래스 의존성
 - 추상 클래스 의존성
 - 인터페이스 의존성

• 구체 클래스에 비해 추상 클래스는 메서드의 내부 구현과 자식 클래스의 종류에 대한 지식을 클라이언트에게 숨길 수 있다.</br>
• 따라서, 클라이언트가 알아야 하는 지식의 양이 더 적기 때문에 구체 클래스보다 추상 클래스에 의존하는 결합도가 더 낮다.</br>
• 인터페이스에 의존하면 상속 계층을 모르더라도 협력이 가능해진다.</br>
• 인터페이스 의존성은 협력하는 객체가 어떤 메시지를 수신할 수 있는지에 대한 지식만을 남기기 때문에 추상 클래스 의존성보다 결합도가 낮다.</br>
=> 결합도를 느슨하게 만들기 위해서는 구체적인 클래스보다 추상 클래스에, 추상 클래스보다 인터페이스에 의존하도록 만드는것이 더 효과적이다.</br>


```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = new AmountDiscountPolicy(..);
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}


```

위 코드같은 경우 생성자 내부에서 인스턴스 변수를 생성하므로 결국 DiscountPolicy와 AmountDiscountPolicy 둘 다 결합을 하게 된다.

```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}


```

위 코드는 DiscountPolicy에 의존한다.
setter 메서드를 사용하는 방식과 메서드 인자를 사용하는 방식의 경우에도 동일하다.
모든 경우에 의존성은 명시적으로 퍼블릭 인터페이스에 노출된다. 이를 명시적인 의존성이라고 부른다.

반면 이전 코드처럼 생성자에 인스턴스가 직접 생성되는 방식은 Movie가 DiscountPolicy에 의존한다는 사실을 감춘다.
다시 말해 의존성이 퍼블릭 인터페이스에 표현되지 않는다. 이를 숨겨진 의존성이라고 부른다.

❗커다란 문제는 의존성이 명시적이지 않으면 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야 한다.❗



