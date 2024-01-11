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

위 코드같은 경우 생성자 내부에서 인스턴스 변수를 생성하므로 결국 DiscountPolicy와 AmountDiscountPolicy 둘 다 결합을 하게 된다.</br>

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

위 코드는 DiscountPolicy에 의존한다.</br>
setter 메서드를 사용하는 방식과 메서드 인자를 사용하는 방식의 경우에도 동일하다.</br>
모든 경우에 의존성은 명시적으로 퍼블릭 인터페이스에 노출된다. 이를 명시적인 의존성이라고 부른다.</br></br>

반면 이전 코드처럼 생성자에 인스턴스가 직접 생성되는 방식은 Movie가 DiscountPolicy에 의존한다는 사실을 감춘다.</br>
다시 말해 의존성이 퍼블릭 인터페이스에 표현되지 않는다. 이를 숨겨진 의존성이라고 부른다.</br></br>

❗커다란 문제는 의존성이 명시적이지 않으면 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야 한다.❗</br>

<h3>new는 해롭다</h3>

대부분의 언어네서는 클래스의 인스턴스를 생성할 수 있는 new 연산자를 제공한다.</br>
하지만 안타깝게도 new를 잘못 사용하면 클래스 사이의 결합도가 극단적으로 높아진다.</br></br>

new가 해로운 이유</br>
 - new 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야 한다. 따라서 new를 사용하는 클라이언트는 추상화가 아닌 구체 클래스에 의존할 수밖에 없기 때문에 결합도가 높아진다.
 - new연산자는 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 한다. 따라서 new를 사용하면 클라이언트가 알아야 하는 지식의 양이 늘어나기 때문에 결합도가 높아진다.</br>

```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee) {

       this.discountPolicy = new AmountDiscountPolicy( Money.wons(500),
                        new SequenceCondition(1),
                        new SequenceCondition(10),
                        new PeriodCondition(DayOfWeek.MONDAY,
                                       LocalTime.of(10,0), LocalTime.of(11,59)),
                        new PeriodCondition(DayOfWeek.THURSDAY,
                                       LocalTime.of(10,0), LocalTime.of(20,59)),

    }


}

```

위 코드에서 new를 사용할 경우 AmountDiscountPolicy의 생성자의 인자를 알아야하므로 더 강하게 결합됨.</br>
또한, PeriodCondition,  SequenceCondition에도 의존하도록 만든다.</br></br>


해결 : 인스턴스를  생성하는 로직과 생성된 인스턴스를 사용하는 로직을 분리하는 것</br>

인스턴스를</br>
1. 생성자의 인자로 전달
2. setter 메서들 사용
3. 실행 시에 메서드 인자로 전달

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


```

위처럼 생성자를 인자로 받으면 위와 같은 코드의 문제점이 사라진다.</br></br>


※사용과 생성의 책임을 분리하고, 의존성을 생성자에 명시적으로 드러내고, 구체 클래스가 아닌 추상 클래스에 의존하게 함으로써 설계를 유연하게 만들 수 있다.</br>


<h3>가끔은 생성해도 무방하다</h3>

클래스 안에서 객체의 인스턴스를 직접 생성하는 방식이 유용한 경우도 있다.</br>

```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee){

         this(title, runningTime, fee, AmountDiscountPolicy(..))

}

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }
}


```

이 코드에서는 첫번째 생성자의 내부에서 두 번째 생성자를 호출한다.</br>
=> 클라이언트는 대부분의 경우에 추가된 간략한 생성자를 통해 AmountDiscountPolicy의 인스턴스와 협력하면서도 컨텍스트에 적절한 DiscountPolicy의 인스턴스로 의존성을 교체할 수 있다.</br>


```

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie calculateMovieFee(Screening screening){
           return calculateMovieFee(screening, new AmountDiscountPolicy(...)));
     }

    public Movie calculateMovieFee(Screening screening, DiscountPolicy discountPolicy){
           return fee.minus(discountPolicy.calculateDiscountAmount(screening));
     }


}

```

위 방법은 메서드 오버로딩하는 경우에도 사용할 수 있다.</br>
DiscountPolicy의 인스턴스를 인자로 받는 메서드와 기본값을 생성하는 메서드를 함께 사용한다면 클래스의 사용성을 향상시키면서 다양한 컨텍스트에서 유연하게 사용될 수 있는 여지를 제공할 수 있다.</br></br>

=> 이 예제를 통해 트레이드오프 활동이라는 사실을 상기시킴.</br>


<h3>표준 클래스에 대한 의존은 해롭지 않다</h3>

jdk의 표준 컬렉션 라이브러리에 속하는 ArrayList의 경우에는 다음과 같이 생성한다.</br>

```
List<A> conditions = new ArrayList<>();

```

비록 클래스를 직접 생성하더라고 가능한 한 추상적인 타입을 사용하는 것이 확장성 측면에서 유리하다.</br>
여기서 ArrayList 등와 같은 가장 추상적인 인터페이스 List를 선언하여 설계의 유연성을 높일 수 있다.</br>


<H3>컨텍스트 확장하기</H3>

1. 할인이 없을 경우
2. 여러 개의 할인을 받을 경우
</br>
할인이 없을 경우</br>

```

public class NoneDiscountPolicy extends DiscountPolicy{

   @override
   protected Money getDiscountAmount(Screening screening){
      return Money.ZERO;
}

}


```

DiscountPolicy를 상속받는 NondeDiscountPolicy 클래스를 만들어서 자식 클래스를 추가한다. Movie 변동 x
</br></br>
여러 개의 할인을 받을 경우</br>

```

public class OverlappedDiscountPolicy extends DiscountPolicy{

   private List<DiscountPolicy> discountPolicies = new ArrayList();

   public OverlappedDiscountPolicy( DiscountPolicy ... discountPolicies){

      this.discountPolicies = Arrays.asList(discountPolicies);
}


}

```

여러 개의 할인을 받을 경우 OverlappedDiscountPolicy 클래스를 생성하여 할인 정책을 List로 받아 적용한다. Moive 변동 x</br></br>


정리 : 설계를 유연하게 만들 수 있었던 이유는 Movie가 DiscountPolicy라는 추상화에 의존하고, 생성자를 통해 DiscountPolicy에 대한 의존성을 명시적으로 드러냈으며, new와 같이 구체 클래스를 직접적으로 다뤄야 하는 책임을 Movie 외부로 옮겼기 때문이다.</br>
우리는 Movie가 의졶나는 추상화인 DiscountPolicy클래스에 자식 클래스를 추가함으로써 간단하게 Movie가 사용될 컨텍스트를 확장할 수 있었다</br>
=> 결합도를 낮춤으로써 얻게 되는 컨텍스트의 확장이라는 개념이 유연하고 재사용 가능한 설계를 만드는 핵심이다.</br>


<h3>조합 가능한 행동</h3>

유연하고 재사용 가능한 설계는 객체가 어떻게 하는지를 장황하게 나열하지 않고도 객체들의 조합을 통해 무엇을 하는지를 표현하는 클래스들로 구성된다.</br>
따라서 클래스의 인스턴스를 생성하는 코드를 보는 것만으로 객체가 어떤 일을 하는지를 쉽게 파악할 수 있다.</br></br>


유연하고 재사용 가능한 설계는 작은 객체들의 행동을 조합함으로써 새로운 행동을 이끌어낼 수 있는 설계다.</br>
훌륭한 객체지향 설계란 어떻게 하는지를 표현하는 것이 아니라 객체들의 조합을 선언적으로 표현함으로써 객체들이 무엇을 하는지를 표현하는 설계이다.</br>
