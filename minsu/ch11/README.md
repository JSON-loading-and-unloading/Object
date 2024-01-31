# 11 합성과 유연한 설계
상속과 합성의 차이점   
- 합성은 내부에 포함되는 객체의 구현이 아닌 퍼블릭 인터페이스에 의존하기에 구현에 의존하지 않는다.
- 상속 관계는 클래스 사이의 정적인 관계인 데 비해 합성 관계는 객체 사이의 동적인 관계이다.
- 상속은 부모 클래스 안에 구현된 코드 자체를 재사용하지만 합성은 포함되는 객체의 퍼블릭 인터페이스를 재사용한다.


## 01. 상속을 합성으로 변경하기
10장에 나왔던 상속을 남용했을 때 직면할 수 있는 세 가지 문제   
- 불필요한 인터페이스 상속 문제
- 메서드 오버라이딩 오작용 문제
- 부모 클래스와 자식 클래스의 동시 수정 문제

를 합성을 사용하면 해결할 수 있다.

### 불필요한 인터페이스 상속 문제 : java.util.Properties와 java.util.Stack
Properties 클래스에서 상속 관계를 제거하고 Hashtable을 Properties의 인스턴스 변수로 포함시키면 합성 관계로 변경 가능   
```java
public class Properties {
    private HashTable<String, String> properties = new HashTable<>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}
```

Vector을 상속받는 Stack 역시 Vector의 인스턴스 변수를 Stack 클래스의 인스턴스 변수로 선언함으로써 합성 관계로 변경 가능   
```java
public class Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw enw EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```
이점
- Properties는 Hashtable의 오퍼레이션을, Stack은 Vector의 오퍼레이션을 포함하지 않으므로 규칙을 어길 위험성이 사라진다.
- 상속 관계일 시에는 내부 구현에 밀접하게 결합되지만, 합성 관계는 내부 구현에 관해 알지 못한다.


### 메서드 오버라이딩의 오작용 문제 : InstrumentedHashSet

```java
public class InstrumentedSet<E> {

	private int addCount = 0;
	private final Set<E> set;

	public InstrumentedSet(Set<E> set) {
		this.set = set;
	}

	public boolean add(E e) {
		addCount++;
		return set.add(e);
	}

	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return set.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```
- InstrumentedHashSet의 경우에는 HashSet이 제공하는 퍼블릭 인터페이스를 그대로 제공해야 한다.
- HashSet은 Set 인터페이스를 실체화하는 구현체 중 하나이며, InstrumentedHashSet이 제공해야 하는 모든 오퍼레이션들을 Set 인터페이스에 정의돼 있다.
- InstrumentedHashSet이 Set 인터페이스를 실체화하면서 내부에 HashSet 인스턴스를 합성하면 HashSet에 대한 구현 결합도는 제거하면서도 퍼블릭 인터페이스는 그대로 유지 가능

```java
public class InstrumentedHashSet<E> implements Set<E> {
    ...
    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }
    ...
}
```

포워딩 : 상위 클래스의 오퍼레이션을 오버라이딩한 인스턴스 메서드에서 내부의 하위 클래스의 인스턴스에게 동일한 메서드 호출을 그대로 전달하는 것   
포워딩 메서드 : 동일한 메서드를 호출하기 위해 추가된 메서드   

### 부모 클래스와 자식 클래스의 동시 수정 문제 : PersonalPlaylist

- 합성으로 변경해도 가수별 노래 목록을 유지하기 위해 두 클래스 모두 수정해야 하는 문제는 해결 x
- 향후에 Playlist의 내부 구현을 변경하더라고 파급효과를 최대한 PersonalPlaylist 내부로 캡슐화 가능 -> 그래도 합성이 더 낫다.

```java
public class PersonalPlaylist {
    private Playlist playlist = new Playlist();
    public void append(Song song) {
        playlist.append(song);
    }
    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
```

## 02. 상속으로 인한 조합의 폭발적인 증가
작은 기능들을 조합해 더 큰 기능을 수행하는 객체를 만들어야 하는 경우, 상속으로 인해 다음과 같은 문제가 발생   
- 하나의 기능을 추가하거나 수정하기 위해 불필요하게 많은 수의 클래스를 추가하거나 수정해야 한다.
- 단일 상속만 지원하는 언어에서는 상속으로 인해 오히려 중복 코드의 양이 늘어날 수 있다.

### 기본 정책과 부가 정책 조합하기

부가 정책이 다음과 같은 특성을 가진다고 가정.   
- 기본 정책의 계산 결과에 적용된다.
- 선택적으로 적용할 수 있다.
- 조합 가능하다.
- 부가 정책은 임의의 순서로 적용 가능하다.
![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/2d818b8e-c1f9-4d59-be18-d4f5325e3ad6)

### 상속을 이용해서 기본 정책 구현하기
- 기본 정책: 추상 클래스 `Phone`
    - 일반 요금제 : `RegularPhone`
    - 심야 할인 요금제: `NightlyDiscountPhone`
- RegularPhone과 NightlyDiscountPhone의 인스턴스만 단독으로 생성 -> 부가 정책은 적용되지 않은 기본 정책만으로 요금을 계산한다.

### 기본 정책에 세금 정책 조합하기
- 일반 요금제에 세금 정책을 조합해야 한다면?
  - RegularPhone 클래스를 상속받은 TexableRegularPhone 클래스를 추가
  - TexableRegularPhone 클래스는 부모 클래스의 calculateFee 메서드를 오버라이딩, super 호출을 통해 부모 클래스에게 calculateFee 메시지 전송

```java
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;
    public TaxableRegularPhone(Money amount, Duration seconds,
                               double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }
    @Override
    public Money calculateFee() {
        Money fee = super.calculateFee();
        return fee.plus(fee.times(taxRate));
    }
}
```
결합도를 낮추기 위해 자식 클래스가 부모 클래스의 메서드를 호출하지 않도록 부모 클래스에 추상 메서드를 제공한다.   
```java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return afterCalculated(result);
    }

    protected abstract Money calculateCallFee(Call call);
    protected abstract Money afterCalculated(Money fee);
}
```
자식 클래스는 afterCalculated 메서드를 오버라이딩해서 계산된 요금에 적용할 작업을 추가한다.
```java
public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;
    public RegularPhone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
    @Override
    protected Money afterCalculated(Money fee) {
        return fee;
    }
}
```

모든 추상 메서드의 구현이 동일하므로 Phone에서 afterCalculated 메서드에 대한 기본 구현을 함꼐 제공한다.   
```java
public abstract class Phone {
  ...
  protected Money afterCalculated(Money fee) {
    return fee;
  }

  protected abstract Money calculatedCallFee(Call call);
}
```
TaxableRegularPhone은 RegularPhone이 계산한 요금에 세금을 부과하므로, afterCalculated 메서드를 오버라이딩한 후 fee에 세금을 더해서 반환하도록 구현한다.   
```java
public class TaxableNightlyDiscountPhone extends NightlyDiscountPhone {
    private double taxRate;

    public TaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(nightlyAmount, regularAmount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

심야 할인 요금제인 NightlyDiscountPhone에도 세금을 부과할 수 있도록 NightlyDiscountPhone의 자식 클래스인 TaxableNightlyDiscountPhone을 추가한다.
```java
public class TaxableNightlyDiscountPhone extends NightlyDiscountPhone {
    private double taxRate;

    public TaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(nightlyAmount, regularAmount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

<img width="411" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/f949fc9d-f883-4635-96c6-afb41210cd31">   

문제는 TableNightlyDiscountPhone과 TaxableRegularPhone 사이에 코드가 중복한다.

### 기본 정책에 기본 요금 할인 정책 조합하기
기본 요금 할인 정책이란 메달 청구되는 요금에서 고정된 요금을 차감하는 부가 정책   
  - 일반 요금제와 기본 요금 항인 정책을 조합하고 싶다면 RegularPhone을 상속받는 RateDiscountableRegularPhone클래스를 추가한다.
  - 심야 할인 요금제와 기본 요금 할인 정책을 조합하고 싶다면 NightlyDiscountPhone을 상속받는 RateDiscountableNightlyDiscountPhone 클래스를 추가한다.
<img width="656" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/65b7c4b7-5f46-473a-a9f6-f15ca8bbb2eb">   

문제는 부가 정책을 구현한 RateDiscountableRegularPhone 클래스와 RateDiscountableNightlyDiscountPhone 클래스 사이에 중복 코드를 추가했다.

### 중복 코드의 덫에 걸리다
상속을 이용한 해결 방법은 모든 가능한 조합별로 자식 클래스를 하나씩 추가하는 것이다.    
<img width="666" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/8dcbe64b-3022-45a3-b812-01dac6b063bc">   
여기에 새로운 기본 정책을 추가해야 한다고 하면, 다음과 같아진다.   
<img width="647" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/66a5528b-a8e4-4a3b-8b2e-97ce88acdc36">   
만약 새로운 부가 정책을 추가해야 한다면, 모든 기본정책에 약정 할인 정책을 선택적으로 적용할 수 있어야 할뿐만 아니라, 다른 부가정책도 임의 순서로 조합 가능해야 한다.   

클래스 폭발문제 또는 조합의 폭발문제: 상속의 남용으로 하나의 기능을 추가하기 위해 필요 이상으로 많은 수의 클래스를 추가해야 하는 경우   
- 이는 자식 클래스가 부모 클래스의 구현에 강하게 결합되도록 강요하는 상속의 근본적인 한계 때문에 발생하는 문제
- 자식 클래스와 부모 클래스의 다양한 조합이 필요한 상황에서 유일한 해결 방법은 조합의 수만큼 새로운 클래스를 추가하는 것

이 문제를 해결하기 위해서는 상속을 포기해야 한다!

## 03. 합성 관계로 변경하기
### 기본 정책 합성하기
각 정책을 별도의 클래스로 구현해야 한다.

기본 정책과 부가 정책을 포괄하는 인터페이스   
```java
public interface RatePolicy {
    Money calculateFee(Phone phone);
}
```

기본 정책 추상 클래스   
```java
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;
        for(Call call : phone.getCalls()) {
            result.plus(calculateCallFee(call));
        }
        return result;
    }

    protected abstract Money calculateCallFee(Call call);
}
```

일반 요금제
```java
public class RegularPolicy extends BasicRatePolicy { ... }
```

심야 할인 요금제
```java
public class NightlyDiscountPolicy extends BasicRatePolicy { ... }
```

- 이제 기본 정책을 이용해 요금을 계산하도록 Phone을 수정
    - Phone 내부에 RatePolicy에 대한 참조자가 포함돼 있다 : 합성
    - 합성하는 객체 타입을 인터페이스나 추상 클래스로 선언하고 의존성 주입을 사용해 런타임에 필요한 객체를 설정할 수 있도록 구현.

```java
public class Phone {
    private RatePolicy ratePolicy;
    private List<Call> calls = new ArrayList<>();

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }

    public Money calculateFee() {
        return ratePolicy.calculateFee(this);
    }
}
```

### 부가 정책 적용하기

- 부가 정책은 기본 정책이나 다른 부가 정책의 인스턴스를 참조할 수 있어야 한다.
- 기본 정책과 부가 정책은 협력 안에서 동일한 '역할'을 수행해야 한다.

부가 정책을 AdditionalRatePolicy 추상 클래스로 구현   
- AdditionalRatePolicy는 컴파일타임 의존성을 런타임 의존성으로 쉽게 대체할 수 있도록 RatePolicy 타입의 인스턴스를 인자로 받는 생성자를 제공
- AdditionalRatePolicy를 상속받는 자식 클래스는 afterCalculated 메서드를 오버라이딩해서 적절한 부가 정책을 구현할 수 있다.
```java
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }

    @Override
    public Money calculateFee(Phone phone) {
        Money fee = next.calculateFee(phone);
        return afterCalculated(fee) ;
    }

    abstract protected Money afterCalculated(Money fee);
}
```

세금 정책   
```java
public class TaxablePolicy extends AdditionalRatePolicy {
    private double taxRatio;

    public TaxablePolicy(double taxRatio, RatePolicy next) {
        super(next);
        this.taxRatio = taxRatio;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRatio));
    }
}
```

기본 요금 할인 정책   
```java
public class RateDiscountablePolicy extends AdditionalRatePolicy {
    private Money discountAmount;

    public RateDiscountablePolicy(Money discountAmount, RatePolicy next) {
        super(next);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```


### 기본 정책과 부가 정책 합성하기

이제 정책들을 조합해서 다양한 요금제 계산을 구현할 수 있다.

일반 요금제에 세금 정책 조합하고 싶다면? 

```java
Phone phone = new Phone(
        new TaxablePolicy(0.05, new RegularPolicy(...));
```

일반 요금제에 기본 요금 할인 정책을 조합한 결과에 세금 정책을 조합하고 싶다면?
```java
Phone phone = new Phone(
        new TaxablePolicy(0.05, 
            new RateDiscountablePolicy(Money.wons(1000),
                new RegularPolicy(...)));
```

세금 정책과 기본 요검 할인 정책이 적용되는 순서를 바꾸고 싶다면 위에서 의존성이 주입되는 순서를 변경하면 된다.   

합성의 장점: 객체를 조합하고 사용하는 방식이 더 예측 가능하고 일관성이 있다.   

### 새로운 정책 추가하기
- 상속과 달리 합성을 기반으로 한 설계에서는 고정 요금제가 필요하다면 고정 요금제를 구현한 클래스 '하나'만 추가하면 된다.
<img width="707" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/d98b951b-608d-4bdb-942b-0cf8a7dc3b5f">


### 객체 합성이 클래스 상속보다 더 좋은 방법이다
- 코드를 재사용하면서도 건전한 결합도를 유지할 수 있는 방법은 합성
  - 상속은 구현을 재사용하지만 합성은 객체의 인터페이스를 재사용한다.
- 상속의 종류
   - 구현 상속 -> 모든 단점은 이에 국한된다.
   - 인터페이스 상속

## 04. 믹스인
믹스인 : 객체를 생성할 때 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법으로, 컴파일 시점에 필요한 코드 조각을 조합하는 재사용 방법   

믹스인과 상속의 다른점   
- 상속은 is-a 관계를 만들기 위한 것이지만 믹스인은 말 그대로 코드를 다른 코드 안에 섞어 넣기 위함
- 상속이 클래스와 클래스 사이의 관계를 고정시키는 데 비해 믹스인은 유연하게 관계를 재구성할 수 있다.


### 기본 정책 구현하기

- 기본 정책을 구현하는 BasicRatePolicy는 기본 정책에 속하는 전체 요금제 클래스들이 확장할 수 있도록 추상 클래스로 구현   

```scala
abstract class BasicRatePolicy { 
  def calculateFee(phone: Phone): Money = phone.calls.map(calculateCallFee(_)).reduce(_ + _)

  protected def calculateCallFee(call: Call): Money;
}

```

표준 요금제를 구현하는 RegularPolicy는 BasicRatePolicy을 상속받아 개별 Call의 요금을 계산하는 calculateCallFee 메서드를 오버라이딩한다.   
```scala
class RegularPolicy(val amount: Money, val seconds: Duration) extends BasicRatePolicy {

  override protected def calculateCallFee(call: Call): Money = ...

}
```
심야 할인 요금제를 구현하기 위한 NightlyDiscountPolicy 역시 BasicRatePolicy를 상속받아 calculateCallFee메서드를 오버라이딩한다.
```scala
class NightlyDiscountPolicy(
    val nightlyAmount: Money,  
    val regularAmount: Money,
    val seconds: Duration) extends BasicRatePolicy {   

  override protected def calculateCallFee(call: Call): Money = ... 
}

object NightltDiscountPolicy {
  val LateNightHour: Integer = 22
}
```

### 트레이트로 부가 정책 구현하기

- 스칼라에서는 다른 코드와 조합해서 확장할 수 있는 기능을 트레이트로 구현할 수 있다.

세금 정책 트레이트
```scala
trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    return fee + fee * taxRate
  }
}
```

- TaxablePolicy 트레이트가 BasicRatePolicy를 확장한다.
    - 이는 상속의 개념이 아니다.   
      -> 단지 세금 정책이 BasicRatePolicy나 BasicRatePolicy 자손에 해당하는 경우에만 믹스인될 수 있다는 것을 의미
    - 기본 정책의 기능에 대해서만 부가 정책을 적용하기를 원하기 때문에 이 제약을 코드로 표현하는 것을 의미를 명확하게 전달할 수 있다.
- extends문은 단지 사용될 수 있는 문맥을 제한할 뿐
    - TaxablePolicy는 BasicRatePolicy를 상속받은 경우에만 믹스인될 수 있다.
    - RegularPolicy와 NightlyDiscountPOlicy에 믹스인될 수 있으며 추가될 새로운 자손에게도 믹스인될 수 있지만 다른 클래스나 트레이트에 믹스인될 수는 없다.
- 믹스인은 상속과 달리 동적이다.
    - 제약을 둘 뿐 어떤 코드에 믹스인될지 결정하지 않는다.
    - super 호출로 실행되는 메서드 또한 트레이트가 실제로 믹스인되는 시점에 결정된다.

### 부가 정책 트레이트 믹스인하기

- 스칼라에서 extends와 with 키워드로 믹스인할 수 있다.
    - extends: 믹스인하려는 대상 클래스의 부모 클래스가 존재하는 경우, extends를 이용해 상속받음
    - with: 트레이트는 with를 이용해 믹스인
    - 이를 트레이트 조합 (trait composition)이라 부른다.

```scala
class TaxableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val taxRate: Double) 
  extends RegularPolicy(amount, seconds) // 조합될 클래스
  with TaxablePolicy // 믹스인할 클래스
```
calculateFee 메시지를 전송했을 때 어떤 메서드가 실행될까?   
-> 스칼라는 특정 클래스에 믹스인한 클래스와 트레이트를 선형화에서 어떤 메서드를 호출할지 결정한다.

선형화 (linearization)
- 클래스의 인스턴스를 생성할 때 스칼라는 클래스 자신과 조상 클래스, 트레이트를 일렬로 나열해서 순서를 정한다.
- 실행 중인 메서드 내부에서 super 호출을 하면 다음 단계에 위치한 클래스나 트레이트의 메서드가 호출된다.
- 항상 맨 앞에는 구현한 클래스 자기 자신이 위치한다.

표준 요금제에 세금 정책을 적용한 후에 비율 할인 정책을 적용하는 경우   
```scala
class RateDiscountableAndTaxableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val discountAmount: Money,
    val taxRate: Double)
  extends RegularPolicy(amount, seconds) 
  with TaxablePolicy 
  with RateDiscountablePolicy 
```

반대로 표준 요금제에 비율 할인 정책을 적용한 후에 세금 정책을 적용하하는 경우   
```scala
class TaxableAndRateDiscountableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val discountAmount: Money,
    val taxRate: Double)
  extends RegularPolicy(amount, seconds)  
  with RateDiscountablePolicy 
  with TaxablePolicy 
```
- 믹스인이 상속에 비해 코드 재사용과 확장의 관점에서 편리하다.
- 믹스인을 사용하더라도 클래스 폭발 문제는 여전히 남아있다.
    - 클래스 폭발의 단점은 클래스 수의 증가가 아닌 중복 코드 수가 늘어난다는 점이지만, 믹스인에서는 이런 문제는 없음


만약, 클래스를 추가하는 것이 싫다면 아래처럼 인스턴스 생성 시 트레이트를 믹스인할 수도 있다.   
```scala
new RegularPlicy(Money(100), Duration.fSeconds(10))
    with RateDiscountPolicy
    with TaxablePolicy {
  val discountAmount = Money(100)
  val taxRate = 0.02
}
```
하지만, 코드 여러 곳에서 동일한 트레이트를 믹스인해서 사용해야 한다면 명시적으로 클래스를 정의하는 것이 좋다.   


### 쌓을 수 있는 변경
- 믹스인을 추상 서브클래스라 부르기도 한다.
    - 믹스인은 상속 계층 안에서 확장한 클래스보다 더 하위에 위치하게 됨
    - 믹스인은 대상 클래스의 자식 클래스처럼 사용될 용도로 만들어지는 것
- 믹스인의 특징: 쌓을 수 있는 변경 
    - 믹스인을 사용하면 특정 클래스에 대한 변경 또는 확장을 독립적으로 구현한 후 필요 시점에 차례대로 추가할 수 있다.
