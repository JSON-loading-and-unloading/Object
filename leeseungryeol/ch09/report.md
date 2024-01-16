<h1>유연한 설계</h1>

<h2>개방-폐쇄 원칙</h2>

개방-폐쇄 원칙 : 소프트웨어 개체는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.( 확장, 수정 )</br>

확장에 대해 열려 있다 : 애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 '동작'을 추가해서 애플리케이션의 기능을 확장할 수 있다.</br>
수정에 대해 닫혀 있다 : 기존 '코드'를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다.</br></br>


개방-폐쇄 원칙은 유연한 설계란 기존의 코드를 수정하지 않고도 애플리케이션의 동작을 확장할 수 있는 설계라고 이야기한다.</br>
개발-폐쇄 원칙은 런타임 의존성과 컴파일 타임 의존성에 관한 이야기다.</br></br>


※런타임 의존성 : 실행시에 협력에 참여하는 객체들 사이의 관계</br>
※컴파일 타임 의존성 : 코드에서 드러나는 클래스들 사이의 관계</br>


![KakaoTalk_20240111_182950558](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/0f6b5f55-0339-4997-bc0c-b806ef1ea8fb)


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



<h2>생성 사용 분리</h2>



![KakaoTalk_20240111_182950558_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/f3c88cbf-2ef7-4ac0-89dc-e18a5e09633d)


유연하고 재사용 가능한 설계를 원한다면 객체와 관련된 두 가지 책임을 서로 다른 객체에 분리해야 한다.</br>
하나는 객체를 <strong>생성</strong>하는 것이고, 다른 하나는 <strong>객체</strong>를 사용하는 것이다.</br>
=> 객체애 대한 생성과 사용을 분리해야 한다.</br></br>


사용으로부터 생성을 분리하는 데 사용되는 가장 보편적인 방버</br>
 - 객체를 생성할 책임을 클라이언트로 옮기는 것


```


public class Client {
    public Money getAvatarFee() {
        Movie avatar = new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(10000),
                new AmountDiscountPolicy(
                    Money.wons(800),
                    new SequenceCondition(1),
                    new SequenceCondition(10)));
        return avatar.getFee();
    }
}


```


![KakaoTalk_20240111_182950558_02](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/5f2dd28b-785a-4b8f-a42a-3136d2cf138f)





그림 9.4에서 Movie는 AmountDiscountPolicy에 대한 의존성 때문에 금액 할인 정책이라는 구체적인 컨텍스트에 묶여 있다. </br>
반면 그림 9.5에서는 AmountDiscountPolicy의 인스턴스를 생성하는 책임을 클라이언트에게 맡김으로써 구체적인 컨텍스트와 관련된 정보는 클라이언트로 옮기고 </br>
Movie는 오직 DiscountPolicy의 인스턴스를 사용하는 데만 주력하고 있다.</br></br>



<h3>Factory 추가하기</h3>

Movie를 사용하는 Client도 특정한 컨텍스트에 묶이지 않기를 바란다면 </br> </br>

각체 생성과 관련된 책임만 전담하는 별도의 객체를 추가하고 Client는 이 객체를 사용하도록 만들 수 있다. </br>
생성과 사용을 분리하기 위해 객체 생성에 특화된 객체를 Factory라고 부른다. </br>


```

public class Client {
    private Factory factory;

    public Client(Factory factory) {
        this.factory = factory;
    }

    public Money getAvatarFee() {
        Movie avatar = factory.createAvatarMovie();
        return avatar.getFee();
    }
}


```



```

public class Factory {
    public Movie createAvatarMovie() {
        return new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(10000),
                new AmountDiscountPolicy(
                    Money.wons(800),
                    new SequenceCondition(1),
                    new SequenceCondition(10)));
    }
}


```


![KakaoTalk_20240111_182950558_03](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/67e0b564-4d41-46df-95c8-cc7e8d50595a)



Factory를 사용하면 Movie와 AountDiscountPolicy를 생성하는 책임 모두를 Factory로 이동할 수 있다. </br>
=> Client는 오직 사용과 관련된 책임만 지고 생성과 관련된 어떤 지식도 가지지 않을 수 있다. </br> </br>


<h3>순수한 가공물에게 책임 할당하기</h3>


전에 Factory를 추가한 이유는 순수하게 기술적인 결정이다. </br>
전체적인 결합도를 낮추고 재사용성을 높이기 위해 도메인 객체를 할당돼 있던 객체 생성 책임을 도메인 개념과는 아무런 상관이 없는 객체로 이동시킨 것이다. </br> </br>


시스템을 객체로 분해하는 데는 크게 두 가지 방식이 존재한다. </br>

1. 표현적 분해 
2. 행위적 분해

표현적 분해는 도메인에 존재하는 사물 또는 개념을 표현하는 객체들을 이용해 시스템을 분해한다. </br>
=> 모든 책임을 도메인 객체에게 할당하면 낮은 응집도, 높은 결합도, 재사용성 저하와 같은 심각한 문제점에 봉착하게 될 가능성이 높아진다. </br> </br>

이를 위해 도메인과 무관한 인공적인 객체 <strong>PURE FABRICATION(순수한 가공물)</strong>이라고 부른다. </br> </br>

어떤 행동을 추가하려고 하는데 이 행동을 책임질 마땅한 도메인 개념이 존재하지 않느다면 PURE FABRICATION을 추가하고 이 객체에게 책임을 할당한다. </br> </br>


설계자로서 우리의 역할은 도메인 추상화를 기반으로 애플리케이션 로직을 설계하는 동시에 품질의 측면에서 균형을 맞추는 데 필요한 객체들을 창조하는 것이다. </br> </br>

먼저 도메인의 본질적인 개념을 표현하는 추상화를 이용해 애플리케이션을 구축하기 시작하라. </br>
만약 도메인 개념이 만족스럽지 못하다면 주저하지 말고 인공적인 객체를 창조하라 </br>

<h2>의존성 주입</h2>

의존성 주입 : 사용하는 객체가 아닌 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 의존성을 해결하는 방법</br>
의존성 주입은 의존성을 해결하기 위해 의존성을 객체의 퍼블릭 인터페이스에 명시적으로 드러내서 외부에서 필요한 런타임 의존성을 전달할 수 있도록 만드는 방법을 포괄하는 명칭</br>

1. 생성자 주입 : 객체를 생성하는 시점에 생성자를 통한 의존성 해결</br>

```

 Movie avatar = new Movie (
             "아바타",
              Duration.ofMinutes(120),
               Money.wons(10000),
              new AmountDiscountPolicy(...));

```

2. setter 주입 : 객체 생성 후 setter 메서드를 통한 의존성 해결</br>

```

avatar.setDiscountPolicy(new AmountDiscountPolicy(..));


```
장 : 의존성의 대상을 런타임에 변경할 수 있다</br>
단 : 객체가 올바로 생성되기 위해 어떤 의존성이 필수적인지를 명시적으로 표현할 수 없다.</br>

3. 메서드 주입  : 메서드 실행 시 인자를 이용한 의존성 해결</br>

```

avatar.calculateDiscountAmount(screening, new AmountDiscountPolicy(..));

```

의존성이 한 두개의 메서드에만 사용된다면 각 메서드의 인자로 전달하는 방법이 나을 수 있다.</br>


<h3>숨겨진 의존성은 나쁘다.</h3>

외부에서 객체에게 의존성을 전달하는 의존성 주입과 달리 SERVICE LOCATOR의 경우 객체가 직접 SERVICE LOCATOR에게 의존성을 해결해줄 것을 요청한다.</br></br>

※SERVICE LOCATOR패턴은 서비스를 사용하는 코드로부터 서비스가 누구인지, 어디에 있는지를 몰라도 되게 해준다.</br>

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
        this.discountPolicy = ServiceLocator.discountPolicy();
    }



```

Moive는 직접 ServiceLocator의 메서드를 호출해서 DiscountPolicy에 대한 의존성을 해결</br>


```


public class ServiceLocator {
    private static ServiceLocator soleInstance = new ServiceLocator();
    private DiscountPolicy discountPolicy;

    public static DiscountPolicy discountPolicy() {
        return soleInstance.discountPolicy;
    }

    public static void provide(DiscountPolicy discountPolicy) {
        soleInstance.discountPolicy = discountPolicy;
    }

    private ServiceLocator() {
    }
}



```

ServiceLocator는 DiscountPolicy의 인스턴스를 등록하기 위한 provide메서드와 인스턴스를 반환하는 discountPolicy메서드를 구현</br></br>



```

ServiceLocator.provide(new AmountDiscountPolicy(...));
 Movie avatar = new Movie (
             "아바타",
              Duration.ofMinutes(120),
               Money.wons(10000));



```

SERVICE LOCATOR의 단점은 의존성을 감춘다.</br>
=> 의존성과 관련된 문제가 컴파일타임이 아닌 런타임에 가서야 발견된다는 사실을 알 수 있다.</br>
=> 의존성을 숨기는 코드는 단위 테스트 작성도 어렵다.</br></br>

❗캡슐화는 코드를 읽고 이해하는 행위와 관련이 있다. 클래스의 퍼블릭 인터페이스만으로 사용 방법을 이해할 수 있는 코드가 캡슐화의 관점에서 훌륭한 코드다.❗</br>
가급적 의존성을 객체의 퍼블릭 인터페이스에 노출하라!
</br>


<h2>의존성 역전 원칙</h2>

<h3>추상화와 의존성 역전</h3>

이미지

Movie와 AmountPolicy 사이의 협력이 가지는 본질은 영화의 가격을 계산하는 것이다.</br>
어떤 협력에서 중요한 정책이나 의사결정, 비즈니스의 본질을 담고 있는 것은 상위 수준의 클래스다.</br>
※상위 수준 클래스가 하위 수준의 클래스에 의존한다면 하위 수준의 변경에 대해 상위 수준 클래스가 영향을 받게 될 것이다.</br>
(상위 수준의 Movie 변경으로 하위 수준의 AmountDiscountPolicy가 영향을 받아야한다.)</br></br></br>

위 이미지는 의존성 방향이 잘못됐다.</br>
상위 수준의 클래스는 어떤 식으로든 하위 수준의 클래스에 의존해서는 안된다.</br>
=> 상위 수준의 클래스가 하위 수준의 클래스에 의존하면 상위 수준의 클래스를 재사용할 때 하위 수준의 클래스도 필요하기 때문에 재사용하기가 어려워진다.</br></br></br>


하위 수준의 이슈로 인해 상위 수준에 위치하는 클래스들을 필요하기 때문에 재사용하기가 어려워진다.</br>
해결사는 <strong>추상화</strong>이다.</br>

이미지


Movie와 AmountDiscountPolicy 모두가 추상화에 의존하도록 수정하면 하위 수준 클래스의 변경으로 인해 상위 수준의 클래스가 영향을 받는 것을 방지할 수 있다.</br>
또한, 상위 수준을 재사용할 때 하위 수준의 클래스에 얽매이지도 않고도 다양한 컨텍스트에서 재사용이 가능하다.</br></br></br>


❗가장 중요한 조언은 추상화에 의존하는 것이다. 유연하고 재사용 가능한 설계를 원한다면 모든 의존성의 방향이 추상 클래스나 인터페이스와 같은 추상화를 따라야 한다.❗</br>

정리</br>
 1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
 2. 추상화는 구체적인 사항에 의존해서는 안 된다. 구체적인 사항은 추상화에 의존해야 한다.

이를 의존성 역전 원칙이라고 부른다.</br>


<h3>의존성 역전 원칙과 패키지</h3>

역전은 의존성의 방향 뿐만 아니라 인터페이스의 소유권에도 적용된다.</br>

이미지


위 이미지에서 Movie를 다양한 컨텍스트에서 재사용하기 위해서는 불필요한 클래스들이 Movie와 함께 배포돼야만 한다.</br>
=> Movie는 DiscountPolicy에 의존한다. </br>
=> 하지만 DiscountPolicy는 하위 클래스들과 묶여있다.</br>
=> 빈번한 재컴파일과 재배포가 발생할 수 있다.</br></br>


이미지

다음과 같이로 묶어 해결할 수 있다.</br>
추상화를 별도의 독립적인 패키지가 아니라 클라이언트가 속한 패키지에 포함시켜야 한다.</br>
그리고 함께 재사용될 필요가 없는 클래스들은 별도의 독립적인 패키지에 모아야 한다.</br></br>

