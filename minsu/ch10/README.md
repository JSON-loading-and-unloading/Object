# 10 상속과 코드 재사용
## 01 상속과 중복 코드
중복 코드는 제거해야 한다.   

### DRY 원칙
- 모든 지식은 시스템 내에서 단일하고, 애매하지 않고, 정말로 믿을 만한 표현 양식을 가져야 한다.
- Don't Repeat Yourself으로, Once and Only Once원칙 또는 Single-Point원칙이라고도 부른다.
- 중복 코드는 변경을 방해한다.
- 코드를 수정하는 데 필요한 노력을 몇 배로 증바시킨다.
- 중복 코드를 판단하는 기준은 변경이다.
  - 요구사항이 변경됐을 때 두 코드를 함께 수정해야 한다면 중복이다.
  - 중복 여부를 결정하는 기준은 코드가 변경에 반응하는 방식이다.

### 중복과 변경
### 중복 코드 살펴보기
- 중복 코드의 문제점을 이해하기 위해 한 달에 한 번씩 가입자별로 전화 요금을 계산
- 전화 요금을 계산하는 규칙은 통화 시간을 단위 시간당 요금으로 나눠준다.

```java
public class Call { 
    private LocalDateTime from;
    private LocalDateTime to;

    public Call(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration getDuration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }
}
```
```java
public class Phone { 
    private Money amount; // 단위 요금
    private Duration seconds; // 단위 시간
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }


    public Money calculateFee() { 
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result;
    }
}
```
- 밤 10시 이후의 통화에 대해 요금을 할인해 주는 방식인 '심야 할인 요금제'라는 새로운 요금 방식을 추가
```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount; // 밤 10시 이후에 적용할 통화 요금
    private Money regularAmount; // 밤 10시 이전에 적용할 통화 요금
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }
        return result;
    }
}
```
### 중복 코드 수정하기
- 통화 요금에 부과할 세금을 계산하는 것이 추가된다면 두 클래스 모두 수정해야 한다.

```java
public class Phone {
    // ...
    private double taxRate;

    public Phone(Money amount, Duration seconds, double taxRate) {
        // ...
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate));
    }
}
```
```java
public class NightlyDiscountPhone {
    ...
    private double taxRate;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        ...
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

         for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }
        return result.minus(result.times(taxRate));
    }
}
```
중복 코드가 가지는 단점 
- 항상 함께 수정해야 하기 때문에 수정할 때 하나라도 빠트린다면 버그로 이어진다.
- 중복 코드를 서로 다르게 수정하기가 쉽다.
- 중복 코드는 새로운 중복 콛를 부른다.

**중복 코드가 늘어날수록 변경에 취약해지고 버그가 발생할 가능성이 높아진다.**

### 타입 코드 사용하기
클래스를 하나로 합치고, 타입 코드를 추가해 타입 코드이 값에 따라 로직을 분기시킨다면 중복 코드를 제거할 수 있다.   
하지만, **낮은 응집도와 높은 결합도**를 가지게 된다.

```java
public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType { REGULAR, NIGHTLY }

    private PhoneType type;

    ...

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            if (type == PhoneType.REGULAR) {
                result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                    result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                } else {
                    result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                }
            }
        }
        return result;
    }
}
```
-> 객체지향 프로그래밍 언어는 타입 코드를 사용하지 않고, **상속**을 사용해서 해결할 수 있다.

### 상속을 이용해서 중복 코드 제거하기
이미 존재하는 클래스와 유사한 클래스가 필요하다면 상속을 이용하라 !   
- NightlyDiscountPhone 클래스와 Phone클래스의 코드가 유사하므로 코드를 중복하게 하지 말고 상속을 이용하자
```java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        super(regularAmount, seconds);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
      // 부모 클래스의 calculateFee 호출
        Money result = super.calculateFee();
        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                        getAmount().minus(nightlyAmount).times(
                                call.getDuration().getSeconds() / getSeconds().getSeconds()));
            }
        }
        return result.minus(nightlyFee);
    }
}
```
-> 일반 요금제에 따라 통화 요금을 계산한 후 이 값에서 통화 시작 시간이 10시 이후인 통화의 요금을 빼줌
- 10시 이전의 요금을 Phone에서 처리
- 10시 이후의 통화 요금을 전체 요금에서 차감
- 값을 차감한 이유는 심야 할인 요금제의 특성상 10시 이전의 요금이 10시 이후의 요금보다 더 비싸기 때문

ex) 
- 밤 10시 이전 : 10초당 5원
- 밤 10시 이후: 10초당 2원

만약 전체 통화 시간 중 처음 40 초 동안은 10시 이전에, 나머지 50초 동안은 10시 이후에 이루어졌다면 ?    
(40초/10초*5원)+(50/10*2원)=(40초/10초*5원)+(50초/10초*5원)-(50초/10초*(5원-2원))=30원   

상속을 이용해 코드를 재사용하려면 자식 클래서의 작성자가 부모 클래스의 구현 방법에 대한 정확한 지식을 가져야 한다.
- 상속은 결합도를 높인다.
- 이 결합도는 코드를 수정하기 어렵게 만든다.

### 강하게 결합된 Phone과 NightlyDiscountPhone
부모 클래스와 자식 클래스 사이의 결합이 문제인 이유
- NightlyDiscountPhone은 부모 클래스인 Phone의 calculateFee 메서드를 오버라이딩
- NightlyDiscountPhone의 calculateFee 메서드는 자신이 오버라이딩한 메서드가 모든 통화에 대한 요금의 총합을 반환한다는 사실에 기반

```java
public class Phone {
    ...

    private double texRate;

    public Phone(Money amount, Duration seconds, double texRate){
    ...
    this.taxRate=taxRate;
    }

    public Money calculateFee() {
         ...
        return result.plus(result.times(taxRate));
    }
}
```

```java
public class NightlyDiscountPhone extends Phone {

    ...

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(regularAmount, seconds, taxRate);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
        // 부모클래스의 calculateFee() 호출
        ...
        return result.minus(nightlyFee.plus(nightlyFee.times(getTaxRate())));
    }
}
```

- NightlyDiscountPhone을 Phone의 자식 클래스로 만든 이유는 Phone의 코드를 재사용하고 중복 코들르 제거하기 위함
- 세금을 부과하는 로직을 추가하기 위해 Phone을 수정할 때 유사한 코드를 NightlyDiscountPhone에도 추가
- 코드 중복을 제거하기 위해 상속을 사용했지만 세금을 계산하는 로직을 추가하기 위해 새로운 중복 코드를 만들어야 함

-> NightlyDiscountPhone이 Phone의 구현에 너무 강하게 결합돼 있기 떄문

> 상속을 위한 경고 1
> 
> 자식 클래스의 메서드 안에서 super참조를 이용해 부모 클래스의 메서드를 직접 호출할 경우 두 클래스는 강하게 결합된다.   
> super 호출을 제거할 수 있는 방법을 찾아 결합도를 제거하라.


## 02 취약한 기반 클래스 문제
부모 클래스의 변경에 의해 자식 클래스가 영향을 받는 현상 : **취약한 기반 클래스 문제** 
- 상속을 사용한다면 피할 수 없는 취약성
- 높은 결합도로 인해 부모 클래스르 점진적으로 개선하는 것이 어렵다.
- 캡슐화를 약화
  - 자식 클래스가 부모 클래스의 구현 세부사항에 의존하도록 만듦
  - 부모 클래스의 퍼블릭 인터페이스가 아닌 구현을 변경해도 자식 클래스가 영향을 받기 쉬워짐
- 결합도를 높임

### 불필요한 인터페이스 상속 문제
자바의 초기 버전에서 상속을 잘못 사용한 대표적인 사례는 ```java.util.Properties```와 ```java.util.Stack```
Stack
- LIFO 자료 구조인 스택을 구현한 클래스

Vector
- 임의의 위치에서 요소를 추출하고 삽입할 수 있는 ㅣㄹ스트
- java.util.List의 초기 버전

Stack -> Vector 상속 관계의 문제점
- Stack의 퍼블릭 인터페이스에 Vector의 퍼블릭 인터페이스가 합쳐짐
- Vector는 임의의 위치에서 요소를 조회, 추가, 삭제 가능 (get, add, remove)
- Stack는 맨 마지막 위치에서만 요소를 추가, 삭제 가능 (push, pop)
  - Stack에게 상속된 Vector의 퍼블릭 인터페이스를 이용하면 임의의 위치에서 요소를 추가, 삭제 가능-> Stack의 규칙을 위반

예를 들어, stack.add(0,"4th")를 할 경우, 이는 Vector의 add메서드를 이용하므로 Stack의 규칙을 무너뜨림

Properites -> Hashtable의 상속 관계의 문제점
- Properties 클래스는 키와 값의 쌍을 보관한다는 점에서 Map과 유사하지만, 키와 값의 타입이 오직 String만 가질 수 있음
- Hashtable은 키와 값의 타입이 다양하다 (Object)
- Hashtable의 인터페이스에 포함돼 있는 put메서드를 이용하면 String 타입 이외의 키와 값이라도 Properties에 저장할 수 있다

> 상속을 위한 경고 2
>
> 상속받은 부모 클래스의 메서드가 자식 클래스의 내부 구조에 대한 규칙을 꺠트릴 수 있다.

## 메서드 오버라이딩의 오작용 문제
HashSet의 구현에 강하게 결합된 InstrumentedHashSet 클래스
문제: 부모 클래스인 HashSet의 addAll 메서드 안에서 add 메서드를 호출하기 때문에 addAll 메서드를 호출하면 중복 호출이 됨

해결 1. InstrumentedHashSet의 addAll 메서드를 제거
- 예상했던 결과가 나오게 됨
- 단, HashSet의 addAll메서드가 add 메시지를 전송하지 않도록 수정되면, 카운트가 누락됨

해결 2. InstrumentedHashSet의 addAll 메서드를 오버라이딩, 추가되는 각 요솓에 대해 한 번씩 add 메시지를 호출
- 미래에 수정이 된다고 해도 아무런 영향이 없음
- 코드가 중복이 됨, 부모 클래스의 코드를 그대로 가져오는 방법이 항상 가능한 것도 아님

> 상속을 위한 경고 3
>
> 자식 클래스가 부모 클래스의 메서드를 오버라이딩할 경우 부모 클래스가 자신의 메서드를 사용하는 방법에 자식 클래스가 결합될 수 있다.

### 부모 클래스와 지식 클래스의 동시 수정 문제
- 자식 클래스가 부모 클래스의 메서드를 오버라이딩하거나 불필요한 인터페이스를 상속받지 않았음에도 부모 클래스르 수정할 때 자식 클래스르 함께 수정해야 함
- 코드 재사용을 위한 상속은 부모 클래스와 자식 클래스를 강하게 결합 -> 함께 수정해야 하는 상황 역시 빈번하게 발생

> 상속을 위한 경고 4
>
> 클래스를 상속하면 결합도로 인해 자식 클래스와 부모 클래스의 구현을 영원히 변경하지 않거나, 자식 클래스와 부모 클래스를 동시에 변경하거나 둘 중 하나를 선택할 수밖에 없다.


## 03. Phone 다시 살펴보기
### 추상화에 의존하자
자식 클래스가 부모 클래스의 구현이 아닌 추상화에 의존핟로고 만들자   
-> 부모 클래스와 자식 클래스 모두 추상화에 의존하도록 수정   

코드 중복을 제거하기 위해 상속을 도입할 때 따르는 두 가지 원칙
- 두 메서드가 유사하게 보인다면 차이점을 메서드로 추출
- 부모 클래스의 코드를 하위로 내리지 말고 자식 클래스의 코드를 상위로 올려라

### 차이를 메서드로 추출하라
변하는 것으로부터 변하지 않는 것을 분리하라, 변하는 부분을 찾고 이를 캡슐화하라   
- Phone와 NightlyDiscountPhone에서 다른 부분은 별도의 메서드로 추출하는 것이다.
- calculateFee의 for문 안에 구현된 요금 계산 로직이 서로 다르므로, 이 부분을 동일한 이름을 가진 메서드로 추출하자.

### 중복 코드를 부모 클래스로 올려라
새로운 부모 클래스 AbstractPhone을 만들고, Phone와 NightlyDiscountPhone이 이 클래스를 상속받도록 수정하자.

``` java
public abstract class AbstractPhone {
    private List<Call> calls = new ArrayList<>();
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }
    abstract protected Money calculateCallFee(Call call);
}
```



``` java
public class Phone extends AbstractPhone {
    private Money amount;
    private Duration seconds;
    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```



``` java
public class NightlyDiscountPhone extends AbstractPhone {
    private static final int LATE_NIGHT_HOUR = 22;
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }
    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```


![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/d20293e2-1f87-4c97-86cb-6249571dffb1)   
리팩토링 후의 상속 계층

### 추상화가 핵심이다
공통 코드를 이동시킨 후, 각 클래스는 서로 다른 변경의 이유를 가진다
- AbstractPhone: 전체 통화 목록을 계산하는 방법이 바뀔 경우에만 변경
- Phone: 일반 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우에만 변경
- NightlyDiscountPhone: 심야 할인 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우에만 변경

장점
- calculateCallFee 메서드의 시그니처가 변경되지 않는 한 부모 클래스의 내부 구현이 변경되더라도 자식 클래스는 영향을 받지 않는다.
- 의존성 역전 원칙도 준수한다.
- 새로운 요금제를 추가하기도 쉽다. -> OCP 원칙 준수

### 의도를 드러내는 이름 선택하기
- Phone -> RegularPhone : 일반 요금제와 관련된 내용을 구현한다는 사실을 명확하게
- AbstractPhone -> Phone : 전화기를 포괄한다는 의미를 명확하게 전달

### 세금 추가하기

- 추상 클래스인 Phone만 수정하면 되는 것은 아니다.
```java
public abstract class Phone {
    private double taxRate;
    private List<Call> calls = new ArrayList<>();

    public Phone(double taxRate) {
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result.plus(result.times(taxRate));
    }

    protected abstract Money calculateCallFee(Call call);
}
```

- Phone의 자식 클래스인 RegularPhone과 NightlyDiscountPhone 의 생성자 역시 taxRate를 초기화하기 위해 수정해야 한다.

책임을 아무리 잘 분리해도 인스턴스 변수의 추가는 종종 상속 계층 전반에 걸친 변경을 유발한다.   
- 인스턴스 초기화 로직을 변경하는 것이 중복 코드보다 낫다.
- 핵심 로직은 한 곳에 모아놓고 캡슐화해야 한다.

상속으로 인한 클래스 사이의 결합을 피할 수 있는 방법은 없다.
- 상속은 어떤 방식으로든 부모 클래스와 자식 클래스를 결합시킨다.
- 행동을 변경하기 위해 인스턴스 변수를 추가해도 상속 계층 전반에 걸쳐 부작용이 퍼지지 않게 막는 것

## 04. 차이에 의한 프로그래밍
차이에 의한 프로그래밍: 기존 코드와 다른 부분만을 추가함으로써 애플리케이션의 기능을 확장하는 방법   
목표: 중복 코드를 제거하고 코드를 재사용하는 것

![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/9eb1d6d0-22ea-4336-bd60-883bc63850fd)

상속이 코드 재사용이라는 측면에서 매우 강력한 도구인 것은 사실이지만 강력한 만큼 잘못 사용할 경우에 돌아오는 피해 역시 크다.


