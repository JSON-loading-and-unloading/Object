## 유연한 설계

이번 장은 유연하고 재사용 가능한 설계를 만들기 위해 적용할 수 있는 다양한 의존성 관리 기법들을 원칙이라는 관점에서 정리한다.

## 01 개방-폐쇄 원칙(OCP)

> 소프트웨어 개체는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.

- 확장에 대해 열려 있다 : 애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 '동작'을 추가해서 애플리케이션의 기능을 확장할 수 있다.
- 수정에 대해 닫혀 있다 : 기존 '코드'를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다.

### 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라

OCP는 런타임 의존성과 컴파일타임 의존성에 관한 이야기다.

영화 예매 시스템의 할인 정책을 의존성 관점에서 다시 한번 살펴보자.

![KakaoTalk_Photo_2024-01-17-14-42-12 002](https://github.com/JSON-loading-and-unloading/Object-Study/assets/98975580/ff4b008f-f8e3-4de8-92cc-7663dd861cea)


- 컴파일타임 의존성 관점 : Movie는 추상 클래스인 DiscountPolicy에 의존한다.
- 런타임 의존성 관점 : Movie는 AmountDiscountPolicy와 PercentDiscountPolicy에 의존한다.

OCP를 수용하는 코드는 확장을 위해 컴파일타임의 의존성을 수정하지 않고도 런타임 의존성을 쉽게 변경할 수 있다. **중복 할인 정책**이라는 새로운 클래스를 추가하기 위해서는 OverlappedDiscountPolicy 클래스를 추가한다.

![KakaoTalk_Photo_2024-01-17-14-42-12 003](https://github.com/JSON-loading-and-unloading/Object-Study/assets/98975580/4186719e-7b3c-462f-81cc-703f3a66e740)

여전히 Movie는 DiscountPolicy 클래스에만 의존하며 컴파일타임 의존성은 변하지 않는다. 하지만 런타임에 Movie는 OverlappedDiscountPolicy 인스턴스와 협력할 수 있다. 런타임 의존성은 변한다.

### 추상화가 핵심이다

OCP의 핵심은 **추상화**에 의존하는 것이다. 

**추상화**는 핵심적인 부분만 남기고 불필요한 부분은 생략함으로써 복잡성을 극복하는 기법이다. 공통적인 부분은 문맥이 바뀌더라도 변하지 않아야 한다. 이러한 공통적인 부분을 추상화함으로써 문맥이 바뀌더라도 수정에 대해 닫혀있다. 또한 추상화를 통해 생략된 부분은 확장의 여지를 남기며 확장을 가능하게 한다.


## 02 생성 사용 분리

Movie가 오직 DiscountPolicy라는 추상화에만 의존하기 위해서는 내부에서 AmountDiscountPolicy와 같은 구체 클래스의 인스턴스를 생성해서는 안 된다.

```
public class Movie {
    ...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = new AmountDiscountPolicy(...);
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

Movie의 할인 정책을 비율 할인 정책으로 변경하기 위해서는 AmountDiscountPolicy를 PercentDiscountPolicy로 바꿔야 한다. 이것은 동작을 추가하거나 변경하기 위해 기존의 코드를 수정하도록 만들기 때문에 OCP 원칙을 위반한다.

이를 해결하기 위해서는 **생성과 사용을 분리(separating use from creation)**해야 한다.

가장 보편적인 방법은 객체를 생성할 책임을 클라이언트로 옮기는 것이다.

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

![KakaoTalk_Photo_2024-01-17-14-42-12 004](https://github.com/JSON-loading-and-unloading/Object-Study/assets/98975580/691b39bc-9d58-4da6-9b15-535a475e62c2)

두 그림을 비교해봤을 때 Movie의 의존성을 DiscountPolicy로만 제한하기 떄문에 확장에 대해서는 열려 있으면서도 수정에 대해서는 닫혀 있는 코드를 만들 수 있다.

### FACTORY 추가하기

위의 그림을 보면, Client도 여전히 생성과 사용이 공존한다는 것을 알 수 있다. Movie을 사용하는 Client도 특정한 컨텍스트에 묶이지 안힉를 바란다고 가정하자.

이 경우 객체 생성과 관련된 책임만 전담하는 별도의 객체를 추가하고 Client는 이 객체를 사용하도록 만들 수 있다. 이처럼 생성과 사용을 분리하기 위햇 객체 생성에 특화된 객체를 **FACTORY**라고 부른다.

```
public class Factory {
    return new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(10000),
                new AmountDiscountPolicy(
                    Money.wons(800),
                    new SequenceCondition(1),
                    new SequenceCondition(10)));
}
```

이제 Client는 Factory 를 사용해서 생성된 Movie의 인스턴스를 반환받아 사용하기만 하면 된다.

```
public class Client {
    private Factory factory;

    public Client(Factory factory) {
        this.factory = factory;
    }

    public Money getAvatarFee() {
        Movie avator = factory.createAvatarMovie();
        return avatar.getFee();
    }
}
```

### 순수한 가공물에게 책임 할당하기

객체 지향 설계를 위한 가장 기본적인 접근법은 도메인에 존재하는 사물 또는 개념을 표현하는 객체들을 이용해 시스템을 분해하는 것이다. 이를 **표현적 분해**라고 한다.

하지만 종종 도메인 개념을 표현하는 객체에게 책임을 할당하는 것만으로는 부족한 경우가 발생한다. 모든 책임을 도메인 객체에 할당하면 낮은 응집도, 높은 결합도, 재사용성 저하와 같은 심각한 문제에 봉착하게 될 가능성이 높아진다.

이러한 경우에는 도메인 개념을 표현한 객체가 아닌 설계자가 편의를 위해 임의로 만들어낸, 마치 FACTORY클래스처럼, 가공의 객체에게 책임을 할당해서 문제를 해결해야 한다. 그리고 이와 같은 객체를 **PURE FABRICATION(순수한 객체)**라고 부른다.

## 03 의존성 주입

의존성 주입은 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 의존성을 해결하는 방법이다. 의존성 해결은 컴파일타임 의존성과 런타임 의존성의 차이점을 해소학 위한 다양한 메커니즘을 포괄한다. 8장에서는 대표적인 방식으로 **생성자 주입**, **setter 주입**, **메서드 주입**을 알아봤다.

### 숨겨진 의존성은 나쁘다

의존성 주입 외에도 의존성을 해결할 수 있는 다양한 방법이 존재하며 대표적으로 **SERVICE LOCATOR 패턴**이 있다. SERVICE LOCATOR는 의존성을 해결할 객체들을 보관하는 일종의 저장소다. 외부에서 객체에게 의존성을 전달하는 의존성 주입과 달리 직접 SERVICE LOCATOR에게 의존성을 해결해줄 것을 요청한다.

Movie의 인스턴스가 AmountDiscountPolicy의 인스턴스에 의존하길 원한다면 다음과 같이 ServiceLocator에 인스턴스를 등록한 후 Movie를 생성하면 된다.

```
ServiceLocator.provide(new AmountDiscountPolicy(...));
Movie avatar = new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(1000));
```

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

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

SERVICE LOCATOR 패턴은 의존성을 해결할 수 있는 가장 쉽고 간단한 도구처럼 보인다.

하지만 ! 의존성을 감춘다는 큰 단점을 가지고 있다. Movie는 DiscountPolicy에 의존하고 있지만 Movie의 퍼블릭 인터페이스 어디에도 이 의존성에 대한 정보가 표시돼있지 않다.

다음과 같은 코드를 마주쳤다고 가정해보자. 이 코드를 처음보는 개발자는 Movie가 온전한 상태로 생성될 것이라고 예상한다. 하지만 아래 코드는 NPE를 발생시킨다.
```
Movie avatar = new Movie("아바타",
                Duration.ofMinutes(120),
                Money.wons(1000));
```

디버깅을 통해 개발자는 Movie의 생성자가 ServiceLocator를 이용해 의존성을 해결한다는 사실을 알게 되고 코드를 이를 해결하기 위한 코드를 추가한다. 이와 같이 의존성을 내부로 감출 경우 의존성과 관련된 문제가 컴파일타임이 아닌 런타임에 가서야 발견된다는 사실을 알 수 있다.

또한 의존성을 숨기는 코드는 단위 테스트 작성도 어렵다. 위에서 구현한 ServiceLocator는 내부적으로 정적 변수를 사용해 객체들을 관리하기 때문에 모든 단위 테스트 케이스에 걸쳐 Service의 상태를 공유하게 된다. 이는 각 단위 테스트는 서로 고립돼야 한다는 단위 테스트의 기본 원칙을 위반한다.

## 04 의존성 역전 원칙

> "추상화에 의존해야지, 구체화에 의존하면 안된다" 즉, 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다.

**역전**이라는 단어를 사용한 이유는 DIP를 따르는 설계는 의존성의 방향이 전통적인 절차형 프로그래밍과는 반대 방향으로 나타나기 때문이다.

객체 사이의 협력이 존재할 때 그 협력의 본질을 담고 있는 것은 상위 수준의 정책이다. Movie와 AmountDiscountPolicy 중 협력의 본질인 영화의 가격을 계산하는 책임을 담당하는 Movie는 상위 수준 클래스이다.

하위 수준의 클래스가 빈번하게 변경되고 새로운 것들이 추가될 때마다 상위 클래스가 영향을 받으므로 의존성의 방향을 역전시켜야 한다. **추상화**를 통해 문제를 해결할 수 있다.

![KakaoTalk_Photo_2024-01-17-14-42-12 005](https://github.com/JSON-loading-and-unloading/Object-Study/assets/98975580/0560e5ed-7918-4715-897b-937d165be49f)

Movie는 추상 클래스인 DiscountPolicy에 의존하고 AmountDiscountPolicy도 추상 클래스인 DiscountPolicy에 의존한다.

### 의존성 역전 원칙과 패키지 🤔

역전은 의존성의 방향뿐만 아니라 인터페이스의 소유권에도 적용된다.

`AmountDiscountPolicy` `PercentDiscountPolicy` `DiscountPolicy` 가 하나의 패키지에 있고 `Movie` 는 다른 패키지에 있다면 `Movie` 를 다양한 컨텍스트에서 재사용하려면 불필요한 클래스들(`AmountDiscountPolicy` `PercentDiscountPolicy`)이 함께 배포되어야 한다. 컴파일 측면에서 이처럼 불필요한 클래스들을 같은 패키지에 두면 전체적인 빌드 시간을 가파르게 상승시킨다. 따라서 재사용될 필요가 없는 클래스들은 별도의 독립적인 패키지에 모아야 한다. 이를 SEPARATED INTERFACE 패턴이라고 한다.

## 05 유연성에 대한 조언

### 유연한 설계는 유연성이 필요할 때만 옳다

유연성은 항상 복잡성을 수반한다. 불필요한 유연성은 불필요한 복잡성을 낳는다. 단순하고 명확한 해법이 그런대로 만족스럽다면 유연성을 제거하라. 유연성은 코드를 읽는 사람들이 복잡함을 수용할 수 있을 때만 가치가 있다.

### 협력과 책임이 중요하다

의존성을 관리해야 하는 이유는 역할, 책임, 협력의 관점에서 설계가 유연하고 재사용 가능해야 하기 때문이다. 

다양한 컨텍스트에서 협력을 재사용할 필요가 없다면 설계를 유연하게 만들 당위성도 함께 사라진다. 객체들이 메시지 전송자의 관점에서 동일한 책임을 수행하는지 여부를 판단할 수 없다면 공통의 추상화를 도출할 수 없다. 동일한 역할을 통해 겍채들을 대체 가능하게 만들지 않았다면 협력에 참여하는 객체들을 교체할 필요가 없다.

따라서 역할, 책임, 협력에 먼저 집중하라.



