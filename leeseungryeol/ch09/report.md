<h1>유연한 설계</h1>

<h2>개방-폐쇄 원칙</h2>

개방-폐쇄 원칙 : 소프트웨어 개체는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.( 확장, 수정 )</br>

확장에 대해 열려 있다 : 애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 '동작'을 추가해서 애플리케이션의 기능을 확장할 수 있다.</br>
수정에 대해 닫혀 있다 : 기존 '코드'를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다.</br></br>


개방-폐쇄 원칙은 유연한 설계란 기존의 코드를 수정하지 않고도 애플리케이션의 동작을 확장할 수 있는 설계라고 이야기한다.</br>
개발-폐쇄 원칙은 런타임 의존성과 컴파일 타임 의존성에 관한 이야기다.</br></br>


※런타임 의존성 : 실행시에 협력에 참여하는 객체들 사이의 관계</br>
※컴파일 타임 의존성 : 코드에서 드러나는 클래스들 사이의 관계</br>


이미지


위 그림에서 알 수 있는 것처럼 중복 할인 정책을 구현하는 OverlapDiscountPolicy 클래스를 추가하더라도 Movie클래스는 여전히 DiscountPolicy 클래스에만 의존한다.</br>
따라서, 컴파일타임 의존성은 변하지 않는다. 하지만 런타임에 Movie 인스턴스 OverlappedDiscountPolicy 인스턴스와 협력할 수 있다. 따라서 런타임 의존성은 변경된다.</br>




<h2>추상화가 핵심이다</h2>

개방-폐쇄 원칙의 핵심은 추상화에 의존하는 것이다.</br></br>


※추상화란 핵심적인 부분만 남기고 불필요한 부분은 생략함으로써 복장섭을 극복하는 기법이다.</br></br>


개방-폐쇄 원칙의 관점에서 생략되지 않고 남겨지는 부분은 다양한 상황에서의 공통점을 반영한 추상화의 결과물이다.</br>
=> 공통적인 부분은 문맥이 바뀌더라도 변하지 않아야 한다.</br>
=> 수정할 필요가 없어야 한다.</br></br>


```

public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return screening.getMovieFee();
    }

    abstract protected Money getDiscountAmount(Screening Screening);
}


```


변하는 부분 : 할인된 요금을 계산하는 방법</br>
변하지 않는 부분 : 할인 여부를 판단하는 로직</br></br>

변하지 않는 부분을 고정하고 변하는 부분을 생량하는 추상화 메커니즘이 개방-폐쇄 원칙의 기반이 된다는 사실에 주목</br>




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

Movie가 DiscountPolicy와 결합하기 때문에 기능이 추가되더라도 DiscountPolicy는 영향을 받지 않는다.</br></br>


❗변경에 의 한 파급 효과를 최대한 피하기 위해서는 변하는 것과 변하지 않는 것이 무엇인지를 이해하고 이를 추상화 목적으로 삼아야 한다.❗</br>
