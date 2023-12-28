## 책임 할당하기

책임을 할당하는 것은 쉽지 않다. 이번 장에서 살펴볼 **GRASP 패턴**은 책임 할당의 어려움을 해결하기 위한 답을 제시해줄 것이다.

## 01 책임 주도 설계를 향해

데이터 중심의 설계에서 책임 중심의 설계로 전환하기 위해서는 다음의 2가지 원칙을 따라야 한다.

- 데이터보다 행동을 먼저 결정하라
- 협력이라는 문맥 안에서 책임을 결정하라

### 데이터보다 행동을 먼저 결정하라

데이터에 초점을 맞추게 되면 낮은 응집도와 높은 결합도를 가진 객체들로 넘쳐나게 된다. 이로 얻는 결과는 변경에 취약한 설계다. 따라서 객체를 설계하기 위해서는 객체가 수행해야 하는 책임을 물은 후 데이터를 결정해야 한다.

그렇다면 객체에게 어떤 책임을 부여해야 하는가 ?

### 협력이라는 문맥 안에서 책임을 결정하라

객체에게 할당된 책임의 품질은 협력에 적합한 정도로 결정된다. 객체의 입장에서는 조금 어색하더라도 협력에 적합하다면 그 책임은 좋은 것이다.

## 02 책임 할당을 위한 GRASP 패턴

GRASP는 "General Responsibility Assignmnet Software Pattern(일반적인 책임 할당을 위한 소프트웨어 패턴)"을 뜻한다.

### 도메인 개념에서 출발하기

설계를 시작하기 전 도메인에 대한 개략적인 모습을 그려봐라. 도메인 안에는 무수히 많은 개념들이 존재하며 이 개념들을 책임 할당의 대상으로 사용하면 도메인의 모습을 코드에 투영하기 쉬워진다.

### 정보 전문가에게 책임을 할당하라

책임에 필요한 정보를 가장 많이 알고 있는 전문가(객체)에게 책임을 할당하라.

### 높은 응집도와 낮은 결합도

설계에는 다양한 방법이 존재하는데 이때 높은 응집도와 낮은 결합도를 염두에 두면 설계의 품질이 좋아진다.

영화 예매 시스템에서 할인 요금을 계산하기 위해 Movie가 DiscountCondition에 할인 여부를 판단하라 메시지를 전송했다. 대신 Screening이 직접 DiscountCondition과 협력하게 하는 것은 어떨까?

- LOW COUPLING(낮은 결합도)
    - Movie와 DiscountCondition은 이미 결합돼 있기 때문에  Movie와 DiscountCondition이 협력하게 되면 결합도를 추가하지 않고도 협력이 가능하다. 하지만 Screening과 협력한다면 새로운 결합도가 추가된다.
    - 의존성을 낮추어 변화의 영향을 줄이기 위해서는 Movie와 DiscountCondition이 협력하는 것이 더 나은 설계이다.

- HIGH COHESION(높은 응집도)
    - Screening의 가장 중요한 책임은 예매를 생성하는 것이다. 그런데 DiscountCondition과 협력하게 되면 Screening은 서로 다른 이유로 변경되는 책임을 짊어지게 되므로 응집도가 낮아진다.

### 창조자에게 객체 생성 책임을 할당하라

객체 A를 생성해야 할 때 다음 조건을 최대한 많이 만족하는 B에게 객체 생성을 할당하라.

- B가 A 객체를 포함하거나 참조한다.
- B가 A 객체를 기록한다.
- B가 A 객체를 긴밀하게 사용한다.
- B가 A 객체를 초기화하는데 필요한 데이터를 가지고 있다.

긴밀하게 연결되어 있는 객체에게 생성 책임을 할당하는 것은 설계의 전체적인 결합도에 영향을 미치지 않는다.

## 03 구현을 통한 검증

Screening을 구현하는 것으로 시작하자. Screening은 영화를 예매할 책임을 맡으며 Reservation을 생성할 책임을 수행한다.

```
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
    }
}
```

책임이 결정된 후 필요한 인스턴스 변수를 결정한다.

```
public class Screening {

    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Reservation reserve(Customer customer, int audienceCount) {
    }
}
```

영화를 예매하기 위해서는 Movie에게 가격을 계산하라 라는 메시지를 전송해야 한다.

```
public class Screening {

    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
    }

    private Money calculateFee(int audienceCount) {
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```

Screening이 Movie의 내부 구현에 대한 어떤 지식도 없이 전송할 메시지를 결정했다. 이로써 Movie 내부 구현을 깔금하게 캡슐화했다.

---

위와 같은 방식으로 Movie의 예매 금액을 계산하기 위한 책임을 할당한다. 그런데 코드에 몇가지 개선할 부분이 있다.

### DiscountCondition 개선하기

<사진>

해당 클래스는 낮은 응집도를 암시하는 3가지 징후가 모두 들어 있다.

- 클래스가 하나 이상의 이유로 변경된다. 변경의 이유를 기준으로 클래스를 분리해야 한다.
- 클래스의 인스턴스를 초기화하는 시점에 경우에 따라 서로 다른 속성들을 초기화하고 있다. 초기화되는 속성의 그룹을 기준으로 클래스를 분리해야 한다.
- 메서드 그룹이 속성 그룹을 사용하는지 여부로 나뉜다. 이들 그룹을 기준으로 클래스를 분리해야 한다.

### 타입 분리하기

DiscountCondition의 가장 큰 문제는 순번 조건과 기간 조건이라는 두 개의 독립적인 타입이 하나의 클래스 안에 공존하고 있다는 점이다. 두 타입을 SequenceCondition과 PeriodCondition으로 분리한다.

클래스를 분리하면 SequenceCondition과 PeriodCondition은 자신의 모든 인스턴스 변수를 함께 초기화한다. 또한 모든 메서드는 동일한 인스턴스 변수 그룹을 사용함으로써 개별 클래스의 응집도가 향상됐다.

하지만 Movie 클래스에서 SequenceCondition과 PeriodCondition 클래스 양쪽 모두에게 결합하게 되면서 변경과 캡슐화라는 관점에서 전체적으로 설계의 품질이 나빠졌다.

### 다형성을 통해 분리하기

사실 Movie 입장에서는 협력하는 객체의 구체적인 타입을 몰라도 상관없다. Movie 입장에서 SequenceCondition과 PeriodCondition는 동일한 책임, 즉 동일한 역할을 수행한다. 따라서 Movier가 구체적인 클래스는 알지 못한 채 오직 역할에 대해서만 결합되도록 의존성을 제한할 수 있다.

두 클래스의 구체적인 타입을 추상화하여 DiscountCondtion라는 이름의 인터페이스를 만들고 Movie와 DiscountCondtion의 다형적인 협력을 이용한다.

객체 타입에 다라 변하는 행동이 있다면 타입을 분리하고 변화하는 행동을 각 타입의 책임으로 할당하는 패턴을 POLYMORPHISM(다형성) 패턴이라고 부른다.

### 변경으로부터 보호하기

SequenceCondition과 PeriodCondition은 서로 다른 이유로 변경된다. 또한 DiscountConditon이라는 추상화가 구체적인 타입을 캡슐화했기 때문에 타입을 추가하더라도 Movie는 영향을 받지 않는다. 

이처럼 변경을 캡슐화하도록 책임을 할당하는 것을 PROTECTED VARIATIONS(변경 보호) 패턴이라고 부른다.

### Movie 클래스 개선하기

POLYMORPHISM(다형성) 패턴과 PROTECTED VARIATIONS(변경 보호) 패턴을 적용해 할인 정책과 관련된 코드를 개선하자.

금액 할인 정책과 관련된 인스턴스 변수와 메서드를 AmountDiscountMovie 클래스로 옮긴다. 그리고 비율 할인 정책은 PercentDiscountMovie 클래스에서 구현하고 Movie를 상속받는다. 각각의 클래스는 Movie에서 선언된 calculateDiscountAmount 메서드를 오버라이딩한 후 할인할 금액을 반환한다.

### 변경과 유연성

설계를 주도하는 것은 변경이다. 유사한 변경이 반복적으로 발생하고 있다면 복잡성이 상승하더라도 유연성을 추가하는 방법이 좋다.

예를 들어, 영화에 설정된 할인 정책을 실행 중에 변경할 수 있다는 요구사항이 추가되었다.

현재는 **상속 구조**를 사용하고 있기 때문에 실행 중에 변경하기 위해서는 새로운 인스턴스를 생성한 후 필요한 정보를 복사해야 한다. 또한 식별자를 관리하는 코드를 추가하는 일은 번거롭다.

이를 상속 대신 **합성**으로 설계를 변경하면 새로운 정책이 추가될 때마다 새로 생성된 클래스의 인스턴스만 추가하면 되는 더 간단한 작업으로 변한다.

