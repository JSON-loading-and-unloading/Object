## 객체지향 프로그래밍

### 01 영화 예매 시스템
> 사용자는 영화 예매 시스템을 이용해 쉽고 빠르게 보고 싶은 영화를 예매할 수 있다.

### 요구사항 정리
- ```영화```와 ```상영```을 구분한다. 영화는 제목, 상영시간, 가격 정보와 같이 영화가 가지고 있는 기본 정보를 가리키며, 상영은 실제로 관람객들이 영화를 관람하는 사건을 표현한다.
- ```할인조건```과 ```할인정책```이 존재한다.
- 할인조건은 ```순서조건```과 ```기간조건```으로 구성된다.
    - 순서 조건은 상영 순번을 이용해 할인 여부를 결정한다.
    - 기간 조건은 영화 상영 시작 시간을 이용해 할인 여부를 결정한다
    - 할인 조건은 여러개 지정 가능하다.
- 할인정책은 ```금액할인정책```과 ```비율할인정책```으로 구성된다.
    - 금액할인정책은 일정 금액을 할인해주는 방식이다.
    - 비율할인정책은 일정 비율의 요금을 할인해주는 방식이다.
    - 할인 정책은 1개, 혹은 지정하지 않는 것만 가능하다.
- 할인조건 확인 후 할인 정책에 따라 할인 요금을 계산한다.

### 02 객체지향 프로그래밍을 향해

### 프로그램 구조

![](https://github.com/JINU-CHANG/dongguk-cafe/assets/98975580/cc9f6519-aae7-4501-93b1-cf41e4ec3674)

### 클래스 구현하기

객체는 상태와 행동을 함께 가지는 복합적인 존재이며, ```스스로 판단하고 행동하는 자율적인 존재```이어야 한다.

객체가 자율적인 존재가 되기 위해서는 외부의 간섭을 최소화해야 하며, 외부의 요청이 있을 때 객체 스스로 최선의 방법을 결정할 수 있어야 한다. 객체는 public, protected, private와 같은 접근 수정자를 통해 외부에서의 접근을 제어한다.

외부에서 접근 가능한 부분을 ```퍼블릭 인터페이스(public interface)```, 접근 불가능한 부분을 ```구현(implementation)```이라고 부른다. 이처럼 인터페이스와 구현을 분리하는 것은 훌륭한 객체지향 프로그램을 만들기 위한 핵심 원칙이다.

### 협력하는 객체들의 공동체

영화를 예매하는 기능을 통해 객체들이 자율적인 존재인지 확인해보자.

```
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
    }
}
```

Screening 객체는 reserve() 메서드를 통해 private 메서드인 calculateFee()를 호출한다. 그리고 이는 다시 Movie의 calculateMovieFee와 Money의 times()를 호출한다. Screening 객체는 스스로 Reservation을 생성하며 이에 대해 외부에서 간섭할 수 없다.

```
public class Screening {
    private Money calculateFee(int audienceCount) {
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```

### 협력에 관한 짧은 이야기

객체는 다른 객체와 ```메시지를 전송(send a message)```을 통해 상호작용한다. 메시지를 수신한 객체는 스스로 결정에 따라 자율적으로 메시지를 처리할 방법을 결정한다. 이때 자신만의 방법을 ```메서드(method)```라고 부른다.

메시지와 메서드를 구분하는 것은 매우 중요하다. 객체는 메서드가 아닌 메시지로 소통해야 한다. Screening 객체로부터 calculateFee()를 호출하여 Reservation 객체를 생성하는 것은 메시지가 아닌 메서드로 소통한 것이다. 메시지로 소통하기 위해서는 reserve()를 호출하고, 어떻게 Reservation이 생성될지는 객체 스스로에게 맡겨야 한다.

### 03 할인 요금 구하기

### 할인 정책과 할인 조건

- 할인 정책

    할인 정책은 금액 할인 정책(AmountDiscountPolicy)과 비율 할인 정책(PercentDiscountPolicy)으로 구분된다. 두 클래스는 대부분의 코드가 유사하고 할인 요금을 계산하는 방식만 다르다. 따라서 부모 클래스인 DiscountPolicy 안에 중복 코드를 두고, AmountDiscountPolicy와 PercentDiscountPolicy가 이 클래스를 상속받게 한다. DiscountPolicy는 인스턴스를 생성할 필요가 없기 때문에 abstract으로 구현한다.

    DiscountPolicy는 할인 여부와 요금 계싼에 필요한 전체적인 흐름을 정의하지만 실제 요금을 계산하는 부분은 추상 메서드인 getDiscountAmount 메서드에 위임한다. 이처럼 부모 클래스에 기본적인 알고리즘의 흐름을 구현하고, 자식 클래스에 위임하는 디자인 패턴을 ```TEMPLATE METHOD```패턴이라고 부른다.

- 할인 조건

    할인 조건들은 순번조건(SequenceCondition)과 기간조건(PeriodCondition)이 존재한다. DiscountCondition은 인터페이스로 선언되어 있고, isSatisfiedBy 오퍼레이션은 인자로 전달된 screening이 할인이 가능한 경우 true를, 불가능한 경우 false를 반환한다.
    SequenceCondition과 PeriodCondition은 DiscountConditon을 구현해 isSatisfiedBy를 오버라이딩한다.

### 할인 정책 구성하기

하나의 영화에 대해 단 하나의 할인 정책만 설정할 수 있고, 할인 조건은 여러개 설정할 수 있다. Movie의 생성자를 통해 오직 하나의 DiscountPolicy인스턴스만 받도록 선언한다.

![](https://github.com/JINU-CHANG/dongguk-cafe/assets/98975580/1db67284-3fa6-4708-a222-2060ce9cb612)

```
public class Movie {
    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...

        this.discountPolicy = discountPolicy;
    }
}
```

반면 DiscountPolicy의 생성자는 여러 개의 DiscountConditon 인스턴스를 허용한다.

```
public abstract class DiscountPolicy {
    public DiscountPolicy(Discount ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }
}
```

이처럼 생성자의 파라미터 목록을 이용해 초기화에 필요한 정보를 전달하도록 강제하면 올바른 상태를 가진 객체 생성을 보장할 수 있다.

### 04 상속과 다형성

Movie 클래스 어디에도 할인 정책이 금액 할인 정책인지, 비율 할인 정책인지 판단하지 않는다. 그럼에도 할인 정책에 따라 할인 금액을 계산할 수 있는 이유는 무엇일까 ?

### 컴파일 시간 의존성과 실행 시간 의존성

Movie는 DiscountPolicy와 직접적으로 의존관계를 가지며, DiscountPolicy는 AmountDiscountPolicy, PercentDiscountPolicy와 상속 관계에 있다.

Movie 인스턴스를 생성하는 코드를 보면, DiscountPolicy 타입의 객체를 받을 때 ```AmountDiscountPolicy```를 받는다. 

```
Movie avatar = new Movie("아바타",
    Duration.ofMinutes(120),
    ...
    new AmountDiscountPolicy(Money.wons(800), ...));
```

코드와의 의존성은 DiscountPolicy와 가지고 있지만, 이렇게 하면 실행 시 AmountDiscountPolicy에 의존하게 된다. 코드의 의존성과 실행 시점의 의존성을 서로 다를 수 있다. 이는 유연하고, 쉽게 재사용할 수 있다는 장점을 준다. 반면 코드를 이해하기 어려워진다는 단점도 있다.

### 상속과 인터페이스

상속이 가치있는 이유는 부모 클래스가 제공하는 모든 인터페이스를 자식 클래스가 물려받을 수 있기 때문이다. 자식 클래스는 부모 클래스가 수신할 수 있는 모든 메시지를 수신할 수 있기 때문에 외부 객체는 자식 클래스를 부모 클래스와 동일한 타입으로 간주한다.

```
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

Movie 입장에서는 calculateDiscountAmount()가 어떤 구현체에서 실행되는지 중요한게 아니라, 해당 메시지를 수신할 수 있다는 사실이 중요하다. 따라서 구현체가 AmountDiscountPolicy에서 PercentDiscountPolicy로 변하더라도 Movie 클래스 내부에서 변경되는 코드는 없다.

### 다형성

위와 같이 동일한 메시지를 수신했을 때 객체의 타입에 따라 다르게 응답할 수 있는 능력을 다형성이라고 한다.

다형성을 구현하는 방법은 매우 다양하지만 메시지에 응답하기 위해 실행될 메서드를 컴파일 시점이 아닌 실행 시점에 결정한다는 공통점이 있다. 메시지와 메서드를 실행 시점에 바인딩하면 이를 지연 바인딩 또는 동적 바인딩이라고 부른다. 반면 컴파일 시점에 실행된 함수나 프로시저를 결정하는 것은 초기 바인딩 또는 정적 바인딩이다.

### 인터페이스와 다형성

앞서는 DiscountPolicy를 추상 클래스로 구현함으로써 자식 클래스들이 인터페이스와 내부 구현을 함께 상속받도록 했다. 그러나 종종 구현은 공유할 필요가 없고 순수하게 인터페이스만 공유하고 싶을 때가 있다. 이를 위해 자바에서는 ```인터페이스```라는 프로그래밍 요소를 제공한다.

할인 조건은 구현을 공유할 필요가 없어 인터페이스로 설계한다. SequenceCondition과 PeriodCondition은 동일한 인터페이스를 공유하며 다형적인 협력에 참여한다.

 ### 05 추상화와 유연성

### 추상화의 장점

1. 추상화의 계층만 따로 떼어 놓고 살펴보면 요구사항의 정책을 높은 수준에서 서술화할 수 있다.

    Movie, DiscountPolicy, DiscountConditon 의 관계를 "영화 예매 요금은 최대 하나의 '할인 정책'과 다수의 '할인 조건'을 이용해 계산할 수있다"라고 표현할 수 있다.

2. 설계가 유연해진다.

    설계가 구체적인 상황에 결합되는 것을 방지하기 때문에 어떤 클래스와도 협력이 가능해진다. 예를 들어 NoneDiscountPolicy라는 할인 정책을 추가할 때 기존의 Movie와 DiscountPolicy를 수정하지 않고 새로운 클래스를 추가하는 것만으로 애플리케이션 기능을 확장할 수 있다.

### 상속과 합성 🤔

코드를 재사용하기 위해 상속보다는 합성이 더 좋은 방법일 수 있다. 합성은 다른 객체의 인스턴스를 자신의 인스턴스 변수로 재사용하는 방법이다.

상속의 가장 큰 문제점은 캡슐화를 위반한다는 것이다. 상속은 자식 클래스가 부모 클래스에 강하게 결합되도록 만들기 때문에 부모 클래스를 변경할 때 자식 클래스도 함께 변경될 확률을 높인다.

두번째로 설계가 유연하지 않다. 상속은 부모 클래스와 자식 클래스 사이의 관계를 컴파일 시점에 결정한다. 따라서 실행 시점에 객체의 종류를 변경하는 것이 불가능하다.












