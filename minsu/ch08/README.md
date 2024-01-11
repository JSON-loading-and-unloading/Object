# 8. 의존성 관리하기
## 1. 의존성 이해하기
### 변경과 의존성
- 실행 시점: 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재
- 구현 시점: 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경

```java
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayofWeek) &&
            ...
    }
}
```
- **실행 시점**에 ```PeriodCondition``` 의 인스턴스가 정상적으로 동작하기 위해서는 ```Screening```의 인스턴스가 존재해야 함 -> 두 객체 사이에 의존성이 존재한다.
- 의존성은 항상 단방향이다.
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/9cd041dc-f112-47ea-8510-412ef7c4d1f8)
- ```PeriodCondition``` 은 ```DayOfWeek``` , ```LocalTime``` , ```Screening```에 대해 의존성을 가짐
- ```PeriodCondition``` 은 ```DiscountCondition```에도 의존, 그 이유는 인터페이스에 정의된 오퍼레이션들을 퍼블릭 인터페이스의 일부로 포함시키기 위함
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/08533f5a-eb7c-4f54-8944-dfb28fe28ad7)

### 의존성 전이
- ```PeriodCondition```이 ```Screening```에 의존할 경우 ```PeriodCondition```은 ```Screening```이 의존하는 대상에 대해서도 자동적으로 의존하게 된다. (전파)
- ```Screening```이 ```Movie```,```LocalDateTime```,```Customer```에 의존하기 떄문에 ```PeriodCondition```역시 ```Movie```,```LocalDateTime```,```Customer```에 간접적으로 의존
- 의존성이 실제로 전이될지 여부는 변경의 방향과 캡슐화의 정도에 따라 달라진다. (변경에 의해 영향이 널리 전파될 수도 있다는 경고)
- 한 요소가 다른 요소에 직접 의존해, 코드에 명시적으로 드러나는 **직접 의존성**과 직접적인 관계는 존재하지 않지만 의존성 전이에 의해 영향이 전파되는 **간접 의존성**으로 나눈다.

### 런타임 의존성과 컴파일타임 의존성
- 런타임 : 애플리케이션 실행되는 시점
  - 런타임 의존성의 주제는 객체
- 컴파일타임: 작성된 코드를 컴파일 시점 또는 코드 그 자체를 가리킴
  - 컴파일타임 의존성이 다루는 주제는 클래스 사이의 의존성
- 런타임 의존성과 컴파일타임 의존성이 다를 수 있고, 유연하고 재사용한 코드를 설계하기 위해서는 두 종류의 의존성을 서로 다르게 만들어야 함
- 어떤 클래스의 인스턴스가 다양한 클래스의 인스턴스와 협력하기 위해서는 협력할 인스턴스의 구체적인 클래스를 알아서는 안됨
  - ㄴ>실제로 협력할 객체가 어떤 것인지는 런타임에 해결해야 함

### 컨텍스트 독립성
- 클래스가 사용될 특정한 문맥에 대해 최소한의 가정만으로 이뤄져 있다면 다른 문맥에서 재사용하기가 더 수월해지며, 이를 컨텍스트 독립성이라고 함
- 설계가 유연해지기 위해서는 가능한 자신이 실행될 컨텍스트에 대해 구체적인 정보를 최대한 적게 알아야 함

### 의존성 해결하기
- 컴파일타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는 것   
- 방법   
  - 객체를 생성하는 시점에 생성자를 통해 의존성 해결
  - 객체 생성 후 setter 메서드를 통해 의존성 해결
  - 메서드 실행 시 인자를 이용해 의존성 해결
- 생성자 방식과 setter 방식을 혼합하는 방식이 항상 객체를 생성할 때 의존성을 해결해서 완전한 상태의 겍체를 생성한 후, 필요에 따라 setter 메서드를 이용해 의존 대상을 변경할 수 있게 할 수 있으므로 제일 좋다.   
- 협력 대상에 대해 지속적으로 의존 관계를 맺을 필요 없이 메서드가 실행되는 동안만 일시적으로 의존 관계가 존재해도 무방하거나, 메서드가 실행될 때마다 의존 대상이 매번 달라져야 하는 경우에는 메서드의 인자를 이용해 의존성을 해결할 수 있다.   

## 2. 유연한 설계

### 의존성과 결합도
- 문제는 의존성의 존재가 아닌 의존성의 정도이다.
- 바람직한 의존성은 재사용성과 관련되어 있다.
  - 컨텍스트에 독립적인 의존성은 바람직한 의존성이다.
  - 특정한 컨텍스트에 강하게 의존하는 클래스를 다른 컨텍스트에서 재사용할 수 있는 유일한 방법은 구현을 변경하는 것 뿐
- 의존성이 바람직하다 -> 느슨한 결합도 / 약한 결합도
- 의존성이 바람직하지 못하다 -> 단단한 결합도 / 강한 결합도

### 지식이 결합을 낳는다
- 서로에 대해 알고 있는 지식의 양이 결합도롤 결정한다.
- 결합도를 느슨하게 만들기 위해서는 협력하는 대상에 대해 필요한 정보 외에는 최대한 감추는 것이 중요하다
- 이게 바로 '추상화'

### 추상화에 의존하라
- 추상화를 사용하면 현재 다루고 있는 문제를 해결하는 데 불필요한 정보를 감출 수 있다.
- 목록에서 아래쪽으로 갈수록 결합도가 느슨해진다.
   - 구체 클래스 의존성
   - 추상 클래스 의존성
   - 인터페이스 의존성
- 구체 클래스에 비해 추상 클래스는 메서드의 내부 구현과 자식 클래스의 종류에 대한 지식을 클라이언트에게 숨길 수 있다.
- 하지만 여전히 협력하는 대상이 속한 클래스 상속 계층이 무엇인지에 대해서는 알고 있어야 한다.
- 인터페이스에 의존하면 상속 계층을 모르더라고 협력이 가능해진다.
- 핵심은 의존하는 대상이 더 추상적일수록 결합도는 더 낮아진다.

### 명시적인 의존성
- 의존성이 퍼블릭 인터페이스에 표현되지 않는 숨겨진 의존성
```java
public class Movie {
...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee) {
        ...
        this.discountPolicy = new AmountDiscountpolicy(...);
    }
}
```
- 클래스 안에서 구체 클래스에 대한 모든 의존성을 제거해야 한다.
- 생성자를 사용해 의존성을 해결할 때, 생성자 안에서 인스턴스를 직접 생성하지 않고, 생성자의 인자로 선언해야 한다.
- 이 경우에는 의존성이 명시적으로 퍼블릭 인터페이스에 노출되므로 명시적인 의존성이라고 한다.
```java
public class Movie {
...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = discountPolicy;
    }
}
```
의존성이 명시적이지 않은 경우
- 의존성을 파악하기 위해 내부 구현을 직접 살펴보아야 한다.
- 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야 하낟.

### new는 해롭다
- new가 해로운 이유
  - new 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야 한다. 따라서 new를 사용하는 클라이언트는 추상화가 아닌 구체 클래스에 의존할 수밖에 없기 떄문에 결합도가 높아진다.
  - new 연산자는 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 한다. 따라서 new를 사용하면 클라이언트가 알아야 하는 지식의 양이 늘어나기 때문에 결합도가 높아진다.
- 요약하면 new 연산자를 사용하면 결합도가 높아지고, 결합도가 높아지면 변경에 의해 영향을 받기 쉬워진다.
- 해결 방법
   - 인스턴스를 생성하는 로직과 생성된 인스턴스를 사용하는 로직을 분리하라

### 가끔은 생성해도 무방하다.
- 주로 협력하는 기본 객체를 설정하고 싶은 경우, 예를 들어 대부분의 경우에는 A인스터와 협력하고 아주 가끔만 B 인스턴스와 협력할 때
- 기본 객체를 생성하는 생성자를 추가하고, 이 생성자에서 다른 인스턴스를 인자로 받는 생성자를 체이닝하는 것이다.
```java
  public class Movie {
  ...
      private DiscountPolicy discountPolicy;

      public Movie(String title, Duration runningTime, Money fee) {
          this(title, runningTime, fee, new AmountDiscountPolicy(...));
      }

      public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
          ...
          this.discountPolicy = discountPolicy;
      }
  }
  ```
- 위 코드를 보면, 클라이언트는 대부분의 경우에 추가된 간략한 생성자를 통해 ```AmountDiscountPolicy``` 의 인스턴스와 협력하게 하면서도 컨텍스트에 적절한 ```DiscountPolicy```의 인스턴스로 의존성을 교체할 수 있다.
- 이 경우는 메서드를 오버로딩할때도 사용할 수 있다.
- 구체 클래스에 의존하게 되더라도 클래스의 사용성이 더 중요하다면 결합도를 높일 '수'는 있지만 가급적 결합도를 낮추는 것이 바람직하다.

### 표준 클래스에 대한 의존은 해롭지 않다
- 변경될 확률이 거의 없는 클래스라면 의존성이 문제가 되지 않는다.
- 예를 들어 ArrayList의 경우에는 직접 생성해서 대입하는 것이 일반적이다.
```java
  public abstract class DiscountPolicy {
      private List<DiscountCondition> conditions = new ArrayList<>();

  public void switchConditions(List<DiscountCondition> conditions){
    this.conditions=conditions;
  }
}
```
- 비록 클래스를 직접 생성하더라도 가능한 추상적인 타입을 사용해 확장성 측면에서 유리하게 하자.

### 컨텍스트 확장하기
1. 할인 혜택을 제공하지 않는 영화의 예매 요금을 계산하는 경우 -> ```discountPolicy```에 ```null```값을 할당
```java
public class Movie {
...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee) {
        this(title, runningTime, fee, new NoneDiscountPolicy(...));
    }

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = discountPolicy;
    }

    public Movie claculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```
- 이 코드는 제대로 동작하지만, 지금까지의 ```Movie```와 ```DiscountPolicy``` 사이의 협력 방식에 어긋나는 에외 케이스가 추가된 것이다.
- 해결책은 할인 정책이 존재하지 않는다는 사실을 예외 케이스로 처리하지 말고 할인 정책의 한 종류로 간주하는 것이다.
```java
public class NoneDiscountPolicy extends DiscountPolicy {

  @Override
  protected Money getDiscountAmount(Screening screening) {
     return Money.ZERO;
  }
}
```

2. 중복 적용이 가능한 할인 정책을 구현하는 것이다.
중복 할인: 금액 할인 정책과 비율 할인 정책을 혼합해서 적용할 수 있는 할인 정책
- 해결책은 마찬가지로 중복 할인 정책을 할인 정책의 한 가지로 간주하는 것이다.
  
```java
public class OverlappedDiscountPolicy extends DiscountPolicy {
    private List<DiscountPolicy> discountPolicies = new ArrayList<>();

    public OverlappedDiscountPolicy(DiscountPolicy ... discountPolicies) {
        this.discountPolicies = Arrays.asList(discountPolicies);
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        Money result = Money.ZERO;
        for (DiscountPolicy each : discountPolicies) {
            result = result.plus(each.calculateDiscountAmount(screening));
        }
        return result;
    }
}
```
### 조합 가능한 행동
- Movie가 비율 할인 정책에 따라 요금을 계산 -> PercentDiscountPolicy 인스턴스를 연결
- Movie가 금액 할인 정책에 따라 요금을 계산 -> AmountDiscountPolicy 인스턴스를 연결
- Movie가 중복할인 -> OverlappedDiscountPolicy 인스턴스를 연결
- Movie가 할인을 제공하고 싶지 않음 -> NoneDiscountPolicy 인스턴스를 연결

유연하고 재사용 가능한 설계는 객체가 어떻게 하는지를 장황하게 나열하지 않고도 객체들의 조합을 통해 무엇을 하는지를 표현하는 클래스들로 구성된다.
