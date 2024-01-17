# 9. 유연한 설계
## 01. 개방-패쇄 연칙 (OCP)
- 소프트웨어 개체는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.
    - 확장에 대해 열려 있다: 요구사항이 변경될 때 새로운 '동작'을 추가해서 기능을 확장할 수 있다.
    - 수정에 대해 닫혀 있다: 기존의 '코드'를 수정하지 않고 동작을 추가하거나 변경할 수 있다.

### 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라
영화 예매 시스템의 할인 정책을 봐도 알 수 있듯이, 컴파일타임 의존성과 런타임 의존성은 동일하지 않다.   

![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/68333170-be20-4fc3-9b52-37305a9a22c3)

**사실 할인 정책 설계는 이미 개방-패쇄 원칙을 따르고 있다.**   
- 중복 할인 정책을 추가할 때 ```OverlappedDiscountPolicy``` 클래스를 추가한 것 뿐이다.
- 할인 정책을 적용하지 않을 때 ```NondeDiscountPolicy``` 클래스를 추가한 것 뿐이다.   

-> 두 경우 모두 기존 클래스는 전혀 수정하지 않은 채 애플리케이션의 동작을 확장했다.
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/6e75f6d6-8aaa-4e54-a218-894755c9e058)

### 추상화가 핵심이다
- 개방-패쇄 원칙의 핵심은 **추상화에 의존하는 것**
- 추상화 부분은 수정에 대해 닫혀 있다. 추상화를 통해 생략된 부분은 확장의 여지를 남긴다.

```java
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
DiscountPolicy는 추상화이다.
- 변하는 부분: 할인된 요금을 계산하는 방법 
- 변하지 않는 부분: 할인 여부를 판단하는 로직

추상화를 했다고 해서 수정에 대해 닫혀 있는 설게를 만들 수 있는 것은 아니다.
- 폐쇄를 가능하게 하는 것은 **의존성의 방향**이다. 
- 수정에 대한 영향을 최소하기 위해서는 모든 요소가 추상화에 의존해야 한다.
- ```Movie```가 할인 정책을 추상화한 ```DiscountPolicy```에 대해서 의존하기 떄문에 수정에 대해 닫혀 있다고 할 수 있다.

## 02. 생성 사용 분리
- 유연하고 재사용 가능한 설계를 원한다면 객체와 관련된 두 가지 책임을 분리해야 한다.
- 생성과 사용을 분리해야 한다.
- 가장 보편적인 방법은 객체를 생성할 책임을 클라이언트로 옮기는 것이다.
```java
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
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/a2eafaa2-c587-427a-af1d-fedff8423cfc)

### FACTORY 추가하기
Client의 코드를 보면 ```Movie```의 인스턴스를 생성하는 동시에 ```getFee``` 메시지도 함께 전송한다.
- 객체 생성과 관련된 책임만 전담하는 별도의 객체를 추가한다. -> FACTORY
- Client는 객체 생성하는 객체를 사용하도록 한다.

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
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/2818d3a3-e547-4d2e-a13b-abec61147807)

### 순수한 가공물에게 책임 할당하기
객체를 분해하는 두 가지 방법 : 표현적 분해와 행위적 분해
- 표현적 분해는 도메인에 존재하는 사물 또는 개념을 표현하는 객체들을 이용해 시스템을 분해하는 것
- 모든 책임을 도메인 객체에게 할당하면 낮은 응집도, 높은 결합도, 재사용성 저하 초래
- 설계자가 편의를 위해 임의로 만들어낸 가공의 객체 - **PURE FABRICATION** - 에게 책임을 할당해서 문제 해결
- PURE FABRICATION은 표현적 분해보다 행위적 분해에 의해 생성되는 것이 일반적이다.
- ```FACTORY```는 객체의 생성 책임을 할당할만한 도메인 객체가 존재하지 않을 때 선택할 수 있는 PURE FABRICATION

## 03. 의존성 주입
의존성 주입 : 사용하는 객체가 아닌 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 의존성을 해결하는 방법
- 생성자 주입 : 객체를 생성하는 시점에 생성자를 통한 의존성 해결
- setter 주입 : 객체 생성 후 setter 메서드를 통한 의존성 해결
- 메서드 주입 : 메서드 실행 시 인자를 이용한 의존성 해결

### 숨겨진 의존성은 나쁘다
SERVICE LOCATOR   
- 의존성을 해결할 객체들을 보관하는 일종의 저장소
- 외부에게서 객체에게 의존성을 전달하는 것이 아닌, 객체가 직접 SERVICE LOCATOR에게 의존성을 해결해줄 것을 요청

```java
public class Movie {
 ...
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Movie fee) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = ServiceLocator.discountPolicy();
    }
}
```
SERVICE LOCATOR 패턴의 가장 큰 단점은 **의존성을 감춘다는 것**
- ```Movie```는 ```DiscountPolicy```에 의존하고 있지만 ```Movie```의 퍼블릭 인터페이스 어디에도 의존성에 대한 정보가 표시돼 있지 않다.
- 컴파일 시에는 에러를 발견하지 못하고, 런타임 시에 널포인트 에러가 발생할 수 있다.

의존성을 구현 내부로 감출 경우 
- 컴파일 타임이 아닌 런타임에 가서야 발견된다.
- 문제점을 코드 작성 시점이 아니라 실행 시점을 미뤄진다.

의존성을 숨기는 코드는 테스트 작성도 어렵다.
- 내부적으로 정적 변수를 사용해 객체들을 관리하기 때문에 모든 단위 테스트 케이스에 걸쳐 ServiceLocator의 상태를 공유하게 된다.
- 각 단위 테스트는 서로 고립돼야 한다는 단위 테스트의 기본 원칙을 위반할 수 있다.

숨겨진 의존성은 캡슐화를 위반한다.
- 클래스의 퍼블릭 인터페이스만으로 사용 방법을 이해해야 하지만, 구현 내부를 샅샅이 뒤져야 한다.

숨겨진 의존성은 의존성의 대상을 설정하는 시점과 해결하는 시점을 멀리 떨어트려 놓는다.
- 코드를 이해하고 디버깅하기 어렵게 만든다.
- 의존성 주입은 이 문제를 해결할 수 있다.


> 명시적인 의존성이 숨겨진 의존성보다 좋다.
> 
> 가급적 의존성을 객체의 퍼블릭 인터페이스에 노출하라

## 04. 의존성 역전 원칙
### 추상화와 의존성 역전
- 객체 사이의 협력이 존재할 때 그 협력의 본질을 담고 있는 것은 상위 수준의 정책이다.
  - 중요한 정책, 의사결정, 비즈니스의 본질을 담고 있는 것은 상위 수준의 클래스
- 상위 수준의 클래스가 하위 수준의 클래스에 의존한다면 하위 수준의 변경에 의해 상위 수준의 클래스가 영향을 받게 될 것이다.
- 상위 수준의 변경에 의해 하위 수준이 변경될 수는 있으나 그 역은 문제가 된다.
-> 이를 **추상화**로 해결할 수 있다.

추상화에 의존하라
- 유연하고 재사용 가능한 설계를 원한다면 모든 의존성의 방향이 추상 클래스나 인터페이스와 같은 추상화를 따라야 한다.
- 구체 클래스는 의존성의 시작점이어야 하며, 목적지가 돼서는 안 된다.

**의존성 역전 원칙 (DIP)**
1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안된다. 둘 모두 추상화에 의존해야 한다.
2. 추상화는 구체적인 사항에 의존해서는 안된다. 구체적인 사항은 추상화에 의존해야 한다.

### 의존성 역전 원칙과 패키지
역전은 의존성의 방향뿐만 아니라 인터페이스의 소유권에도 적용된다.
❗️
인터페이스가 서버 모듈 쪽에 위치하는 전통적인 모듈 구조
- {```Movie```} | {```DiscountPolicy```, ```AmountDiscountPolicy```, ```PercentDiscountPolicy```}
- DiscountPolicy 클래스에 의존하기 위해서는 반드시 같은 패키지에 포함된 AmountDiscountPolicy 클래스와 PercentDiscountPolicy 클래스도 함께 존재해야 한다.
- DiscountPolicy 가 포함된 패키지 안의 어떤 클래스가 수정되더라도 패키지 전체가 재배포 -> 불필요한 클래스들을 같은 패키지에 두는 것은 전체적인 빌드 시간을 상승

SEPARATED INTERFACE 패턴 (인터페이스의 소유권을 역전시킨 객체지향적인 모듈 구조)
- {```Movie```} | {```DiscountPolicy```} | {```AmountDiscountPolicy```, ```PercentDiscountPolicy```}
- 인터페이스의 소유권을 서버가 아닌 클라이언트에 위치시킨다.
- 새로운 할인 정책을 위해 새로운 패키지를 추가하고 새로운 DiscountPolicy의 자식 클래스를 구현하기만 하면 상위 수준의 협력 관계를 재사용할 수 있다.
- 불필요하게 연관이 없는 클래스를 함께 배포할 필요가 없다.

전통적인 패러다임과 객체지향 패러다임의 차이
- 전통적인 패러다임에서는 상위 수준 모듈이 하위 수준 모듈에 의존했다면 객체지향 패러다임에서는 상위 수준 모듈과 하위 수준 모듈이 모두 추상화에 의존한다.

❗️ 훌륭한 객체지향 설계를 위해서는 의존성을 역전시켜야 하며 의존성을 역전시켜야만 유연하고 재사용 가능한 설계를 얻을 수 있다.

## 05. 유연성에 대한 조언
### 유연한 설계는 유연성이 필요할 때만 옳다
유연하고 재사용 가능한 설계는 항상 좋은 것은 아니다.
- 변경하기 쉽고 확장하기 쉬운 구조를 만들기 위해서는 단순함과 명확함의 미덕을 버리게 될 가능성이 높다.
- 유연한 설계를 단순하고 명확하게 만드는 유일한 방법은 사람들 간의 긴밀한 커뮤니케이션뿐이다.
- 복잡성이 필요한 합리적인 근거를 제시해야 한다.
  

유연성은 항상 복잡성을 수반한다.
- 코드 상에 표현된 정적인 클래스의 구조와 실행 시점의 동적인 객체 구조가 다르다.
- 특정 시점의 객체 구조를 파악하는 유일한 방법은 클래스를 사용하는 클라이언트 코드 내에서 객체를 생성하거나 변경하는 부분을 직접 살펴보는 것 뿐이다.

단순하고 명확한 해법이 만족스럽다면 유연성을 제거하라
- 유연성은 코드를 읽는 사람들이 복잡함을 수용할 수 있을 때만 가치가 있다.
- 복잡성에 대한 걱정 < 유연하고 재사용 가능한 설계의 필요성 이라면 코드의 구조와 실행 구조를 다르게 만들자.

 ### 협력과 책임이 중요하다
 설계를 유연하게 만들기 위해서는 먼저 역할, 책임, 협력에 초점을 맞춰라
 - 다양한 컨텍스트에서 협력을 재사용할 필요가 없다면 유연하게 만들 필요 x
 - 객체들이 메시지 전송자의 관점에서 동일한 책임을 수행하는지 여부를 판단x -> 공통의 추상화를 도출 x

객체의 역할과 책임이 자리를 잡은 후 객체를 생성 매커니즘을 결정하라
- 책임 관점에서 객체들 간의 균형이 잡혀 있는 상태라면 생성과 관련된 책임을 지게 될 객체를 선택하는 것은 간단
- 책임의 불균형이 심화되고 있는 상태에서 객체의 생성 책임을 지우는 것은 설계를 하부의 특정한 메커니즘에 종속적으로 만들 확률이 높다.
- 불필요한 싱글톤패턴은 객체 생성에 관해 너무 이른 시기에 고민하고 결정할 때 도입되는 경향이 있다. 
