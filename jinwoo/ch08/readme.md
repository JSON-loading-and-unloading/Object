## 의존성 관리하기

이번 장은 충분히 협력적이면서도 유연한 객체를 만들기 위해 **의존성을 관리하는 방법**을 살펴본다.

먼저 의존성이 무엇인지 알아보자.

## 01 의존성 이해하기

### 변경과 의존성

의존성은 실행 시점과 구현 시점에 서로 다른 의미를 가진다.

- 실행 시점 : 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.
- 구현 시점 : 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.

```
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    ...

    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0&&
                endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }
}
```

- 실행 시점에 PeriodCondition이 정상적으로 동작하기 위해서는 Screening의 인스턴스가 존재해야 한다.
- 어떤 형태로든 DayOfWeek, LocalTime, Screening, DiscountCondition이 변경된다면 PeriodCondition도 함께 변경될 수 있다.

### 의존성 전이

의존성이 전이된다는 것은 PeriodCondition이 Screeing에 의존하는 경우 PeriodConditon이 Screeing이 의존하는 대상에 대해서도 자동적으로 의존하게 된다는 것을 의미한다.

```
PeriodConditon --> Screening --> Movie
```

모든 경우에 의존성이 전이되는 것은 아니다. 의존성은 함께 변경될 가능성을 의미하기 때문에 Screening의 캡슐화 정도에 따라 PeriodCondition에 의존성이 전이되지 않을 수도 있다.

### 런타임 의존성과 컴파일타임 의존성

**런타임**은 애플리케이션이 실행되는 시점을 가리킨다. **컴피알** 타임은 원래는 작성된 코드를 컴파일하는 시점을 가리키지만 여기서는 코드 그 자체를 가리킨다.

런타임 의존성과 컴파일타임 의존성은 다를 수 있다. 유연하고 재사용 가능한 코드를 설계하기 위해서는 두 종류의 의존성을 서로 다르게 만들어야 한다.

```
public class Movie {
    ...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = discountPolicy;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

코드 작성 시점에서 Movie 클래스는 AmountDiscountPolicy 클래스와 PercentDiscountPolicy 클래스로 향하는 어떤 의존성도 존재하지 않는다. 오직 추상 클래스인 DiscountPolicy 클래스에만 의존한다.

금액할인 정책이나 비율할인 정책을 적용하기 위해서는 DiscountPolicy라는 추상 클래스에 의존하도록 만들고 이 컴파일타임 의존성을 실행 시에 PercentDiscountPlicy 인스턴스나 AmountDisciountPolicy 인스턴스에 대한 런타임 의존성으로 대체해야 한다.

### 컨텍스트 독립성

위의 예시에서 보았듯 클래스는 자신과 협력할 객체의 구체적인 클래스에 대해 알아서는 안 된다. 구체적인 클래스를 알면 알수록 그 클래스가 사용되는 특정한 문맥에 강하게 결합되기 때문이다. 이를 **컨텍스트 독립성**이라고 부른다.

클래스가 실행 컨텍스트에 독립적인데도 어떻게 런타임에 실행 컨텍스트에 적절한 객체들과 협력할 수 있을까?

### 의존성 해결하기

컴파일타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는 것을 **의존성 해결**이라고 부른다. 의존성을 해결하기 위해서는 다음과 같은 3가지 방법을 사용한다.

- 객체를 생성하는 시점에 생성자를 통해 의존성 해결
- 객체 생성 후 setter 메서드를 통해 의존성 해결
- 메서드 실행 시 인자를 이용해 의존성 해결

**객체를 생성하는 시점에 생성자를 통해 의존성 해결**

```
Movie avatar = new Movie("아바타",
    Duration.ofMinutes(120),
    Money.wons(1000),
    new AmountDiscountPolicy(...));
```
**객체 생성 후 setter 메서드를 통해 의존성 해결**

```
Movie avatar = new Movie(...);
avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```

setter 메서드를 이용하는 방법은 실행 시점에 의존 대상을 변경할 수 있기 때문에 설계를 좀 더 유연하게 만들 수 있다. 단점은 객체 생성 후 필요한 의존 대상을 설정하기 때문에 그전까지는 객체 상태가 불완전할 수 있다. 

**메서드 실행 시 인자를 이용해 의존성 해결**

생성자 방식과 setter 방식을 혼합하는 방법도 있다. 항상 객체를 생성할 때 의존성을 해결해서 완전한 상태의 객체를 생성한 후, 필요에 따라 setter 메서드를 이용해 의존 대상을 변경할 수 있게 할 수 있다.

```
Movie avatar = new Movie(..., new PercentDiscountPolicy(...));

...

avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
```

## 02 유연한 설계

의존성을 관리하는 데 유용한 몇 가지 원칙과 기법을 소개한다.

### 의존성과 결합도

모든 의존성이 나쁜 것은 아니다. 객체지향 패러다임의 근간은 협력이며 협력을 위해서는 객체들이 서로의 존재와 수행 가능한 책임을 알아야 한다. 의존성은 협력을 위해 반드시 필요한 것이다.

문제는 바람직하지 못한 의존성이다. 이는 **재사용성**과 관련이 있다. 어떤 의존성이 다양한 환경에서 클래스를 재사용할 수 없도록 제한한다면 그 의존성은 바람직하지 못하다. 반면 어떤 의존성이 다양한 환경에서 재사용할 수 있다면 바람직하다고 말할 수 있다. 

Movie 클래스가 PercentDiscountPolicy(구체 클래스)에 의존한다면 다른 종류의 할인 정책이 필요한 문맥에서 Movie를 재사용할 수 없게 된다. 이를 가능하게 하는 것은 코드에서 DiscountPolicy의 타입을 모두 변경하는 수 밖에 없다. 하지만 이는 바람직하지 못한 의존성을 바람직하지 못한 또 다른 의존성으로 대체한 것일 뿐이다.

두 요소 사이에 존재하는 의존성이 바람직할 때 두 요소가 **느슨한 결합도 또는 약한 결합도**를 가진다고 표현한다. 반대로 두 요소 사이의 의존성이 바람직하지 못할 때는 **단단한 결합도 또는 강한 결합도**를 가진다고 말한다.

> 보통 의존성과 결합도를 동의어로 사용하지만 사실 두 용어는 서로 다른 관점에서의 관계의 특성을 설명하는 용어다. 의존성은 두 요소 사이의 관계 유무를 설명한다. 그에 반해 결합도는 두 요소 사이에 존재하는 의존성의 정도를 상대적으로 표현한다.

### 지식이 결합을 낳는다

더 많이 알수록 더 많이 결합된다. 결합도를 느슨하게 만들기 위해서는 협력하는 대상에 대해 필요한 정보 외에는 최대한 감추는 것이 중요하다.

이를 달성할 수 있는 효과적인 방법은 **추상화**이다.

### 추상화에 의존하라

추상화란 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 숨김으로써 복잡도를 극복하는 방법이다. 추상화를 사용하면 현재 다루고 있는 문제를 해결하는 데 불피룡한 정보를 감출 수 있어 결합도를 느슨하게 유지할 수 있다.

일반적으로 추상화와 결합도의 관점에서 의존 대상을 다음과 같이 구분하는 것이 유용하다.

- 구체 클래스 의존성
- 추상 클래스 의존성
- 인터페이스 의존성

결합도를 느슨하게 만들기 위해서는 구체적인 클래스보다 추상 클래스에, 추상 클래스보다 인터페이스에 의존하도록 만드는 것이 효과적이다.

### 명시적 의존성

```
public class Movie {
    ...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = new AmountDiscountPolicy(...);
    }
```

discountPolicy는 DiscountPolicy 타입으로 선언돼 있지만 생성자에서 구체 클래스인 AmountDiscountPolicy의 인스턴스를 직접 생성해서 대입하고 있다. 언뜻 보기에는 문제가 없어 보인다.

Movie 내부에서 AmountDiscountPolicy의 인스턴스를 직접 생성하는 방식은 Movie가 DiscountPolicy에 의존한다는 사실을 감춘다. 의존성이 퍼블릭 인터페이스에 표현되지 않는다. 이를 숨겨진 의존성이라고 부른다. 

의존성이 명시적이지 않으면 의존성 파악을 위해 내부 구현을 살펴봐야 한다. 커다란 클래스 내부에 정의된 긴 메서드 내부에서 해당 코드를 파악하는 일은 쉽지 않을 수 있다. 또한 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야 한다는 더 커다란 문제를 가지고 있다. 코드 수정은 언제나 잠재적인 버그의 발생 가능성을 내포한다.

의존성은 명시적으로 표현해야 된다. 의존성을 구현 내부에 숨겨두지 마라!

### new는 해롭다

### 표준 클래스에 대한 의존은 해롭지 않다

변경확률이 거의 없는 클래스라면 의존성이 문제가 되지 않는다.

```
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();
}
```

ArrayList의 코드가 수정될 확률은 0에 가깝기 때문에 인스턴스를 직접 생성해도 문제되지 않는다.

비록 클래스를 직접 생성하더라도 타입을 List로 지정해준 것과 같이 가능한 한 추상적인 타입을 사용하면 확장성 측면에서 유리하다.

### 컨텍스트 확장하기

2가지 요구사항이 추가됐다.

- 할인 혜택을 제공하지 않는 영화의 예매 요금을 계산해야 한다.
- 다수의 할인 정책을 중복해서 적용하는 영화의 예매 요금을 계산해야 한다.

**할인 혜택을 제공하지 않는 영화**

가장 먼저 떠올릴 수 있는 방법은 DiscountPolicy에 null 값을 할당하는 것이다. 하지만 calculateMovie메서드에서 discountPolicy 값이 null인지 판단하는 코드가 추가된다.

코드는 제대로 동작하겠지만 Movie와 DiscountPolicy사이의 협력 방식에 어긋나는 예외 케이스가 추가되면서 코드를 수정했다. 이는 버그 가능성을 높인다.

해결방법으로는 할인할 금액으로 0원을 반환하는 NoneDiscountPolicy 클래스를 추가하고 DiscountPolicy의 자식 클래스로 만드는 것이다.

```
public class NoneDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;
    }
}
```

이제 Movie 클래스에 If 문을 추가하지 않고도 할인 혜택을 제공하지 않는 영화를 구현할 수 있다.

**다수의 할인 정책을 중복해서 적용하는 영화**

Movie 클래스를 수정하지 않고 다수의 할인 정책을 중복해서 적용하도록 해보자. NoneDiscountPolicy와 같은 방법을 사용해서 해결할 수 있다. 중복 할인 정책을 할인 정책의 한 가지로 간주하는 것이다. 

```
public class OverlappedDiscountPolicy extends DiscountPolicy {
    private List<DiscountPolicy> discountPolicies = new ArrayList<>();

    public OverlappedDiscountPolicy(DiscountPolicy ... discountPolicies) {
        this. discountPolicies = Arrays.asList(discountPolicies);
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        Money result = Money.ZERO;
        for(DiscountPolicy each : discountPolicies) {
            result = result.plus(each.calculateDiscountAmount(screening));
        }
        return result;
    }
}
```

이 예제들은 Movie를 수정하지 않고도 할인 정책을 적용하지 않는 새로운 기능을 추가하는 것이 얼마나 간단한지 보여준다. 이렇게 설계를 유연하게 만들 수 있었던 이유는 Movie가 

- DiscountPolicy라는 추상화에 의존하고, 
- 생성자를 통해 DiscountPolicy에 대한 의존성을 명시적으로 드러냈으며, 
- new 와 같이 구체 클래스를 직접적으로 다뤄야 하는 책임을 Movie 외부로 옮겼기 때문이다. 

**조합 가능한 행동**

유연하고 재사용 가능한 설계는 객체가 어떻게(how) 하는지를 장황하게 나열하지 않고도 객체들의 조합을 통해 무엇(what)을 하는지를 표현하는 클래스들로 구성된다.