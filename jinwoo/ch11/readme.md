## 합성과 유연한 설계

> 객체 합성이 클래스 상속보다 더 좋은 방법이다

## 01 상속을 합성으로 변경하기

상속을 남용했을 때 아래의 3가지 문제점이 있다.

- 불필요한 인터페이스 상속 문제
- 메서드 오버라이딩 오작용 문제
- 부모 클래스와 자식 클래스의 동시 수정 문제

**불필요한 인터페이스 상속 문제: java.util.Properties와 java.util.Stack**

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

Hashtable과 Vector를 합성 관계로 변경함으로써 더 이상 불필요한 인터페이스들이 Properties와 Stack 외부로 노출되지 않는다.

**🤔 메서드 오버라이딩 오작용 문제: InstrumentedHashSet**

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

HashSet에 대한 구현 결합도는 제거하면서도 퍼블릭 인터페이스는 그대로 상속받을 수 있다.

**부모 클래스와 자식 클래스의 동시 수정 문제: PersonalPlaylist**

```java
public class PersonalPlaylist {
    private Playlist playlist = new Playlist();

    public void append(Song song) {
        playlist.append(song);
    }

    public void remote(Song song) {
        playlist.getTracks().remote(song);
        playlist.getSingers().remote(song.getSinger());
    }
}
```

Playlist와 PersonalPlaylist를 함께 수정해야 하는 문제는 해결되지 않는다. 하지만 향후 Playlist의 내부 구현을 변경하더라도 파급 효과를 최대한 Personallist 내부로 캡슐화할 수 있다.

## 02 상속으로 인한 조합의 폭발적인 증가

작은 기능들을 조합해서 더 큰 기능을 수행하는 객체를 만들어야 하는 경우 합성을 사용하면 상속으로 인해 발생하는 클래스의 증가와 중복 코드 문제를 해결할 수 있다.

### 기본 정책과 부가 정책 조합하기

<핸드폰 과금 시스템>

- 기본 정책
    - 일반 요금제
    - 심야 할인 요금제

- 부가 정책
    - 세금 정책
    - 기본 요금 할인 정책

- 특성
    - 기본 정책 적용 후 부가 정책을 적용한다.
    - 부가 정책은 선택적으로 적용 가능하며 조합도 가능하다.
    - 부가 정책은 임의의 순서로 적용 가능하다.

이에 따라 조합 가능한 수는 10가지이다.

### 상속을 이용해서 기본 정책 구현하기

추상 클래스인 Phone을 상속받아 `RegularPhone`, `NightlyDiscountPhone` 클래스를 생성한다. Phone의 `추상 메서드 calculateCallFee()`를 오버라이딩하여 각 정책의 계산법을 적용하여 요금을 계산한다.

### 기본 정책에 부가 정책 조합하기

- 일반 요금(심야할인 요금제) + 세금 정책
- 일반 요금(심야할인 요금제) + 기본 요금 할인 정책
- 일반 요금(심야할인 요금제) + 세금 정책 + 기본 요금 할인 정책
- 일반 요금(심야할인 요금제) + 기본 요금 할인 정책 + 세금 정책

가능한 조합 8개에 맞는 클래스를 생성한다. 이때 **기본 정책 적용 후 부가 정책을 적용할 수 있다**는 규칙에 따라 Phone 클래스의 코드를 수정한다.

```java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return afterCalculated(result); //기본 정책 적용 후 부가 정책 적용할 수 있도록 코드 수정
    }

    protected abstract Money calculateCallFee(Call call);

    protected Money afterCalculated(Money fee) {
        return fee;
    }
}
```

```java

//일반 요금 + 세금 정책
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;

    public TaxableRegularPhone(Money amount, Duration seconds, double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}

//심야할인 요금제 + 세금 정책
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

### 중복 코드의 덫에 걸리다

상속을 이용한 해결 방법은 모든 가능합 조합별로 자식 클래스를 하나씩 추가하는 것이다. 이의 문제점은 새로운 정책을 추가하기 위해서는 불필요하게 많은 수의 클래스를 상속 계층 안에 추가해야 한다는 것이다.

`고정요금제(FixedRatePhone)` 이라는 기본 정책을 추가하기 위해서는 이와 조합가능한 부가 정책들을 모두 자식 클래스로 추가해주어야 한다.

- 고정요금제 + 세금 정책
- 고정요금제 + 기본 요금 할인 정책
- 고정요금제 + 세금 정책 + 기본 요금 할인 정책
- 고정요금제 + 기본 요금 할인 정책 + 세금 정책

이처럼 상속의 남용으로 하나의 기능을 추가하기 위해 필요 이상으로 많은 수의 클래스를 추가하는 경우를 클래스 폭발(class explosion) 혹은 조합의 폭발(combinational explosion)이라고 부른다.

또한 새로운 기능을 수정할 때도 문제가 있다. 세금 정책을 변경해야 한다면 이와 관련해 많은 클래스들의 코드를 수정해야 한다.

## 03 합성 관계로 변경하기

합성은 조합을 구성하는 요소들을 개별 클래스로 구현한 후 **실행 시점에 인스턴스를 조립함**으로써 상속의 문제를 해결한다.

### 기본 정책 합성하기

각 기본 정책을 별도의 클래스로 구현한다.

```java
// 기본 정책 추상 클래스
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

// 일반 요금제
public class RegularPolicy extends BasicRatePolicy { ... }

// 심야 할인 요금제
public class NightlyDiscountPolicy extends BasicRatePolicy { ... }

```

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

Phone 내부에 RatePlicy을 합성했다. Phone이 다양한 요금 정책과 협력할 수 있도록 요금 정책의 타입을 RatePolicy라는 인터페이스로 정의했다. 

Phone은 컴파일타임 의존성을 구체적인 런타임 의존성으로 대체하기 위해 생성자를 통해 RatePolicy의 인스턴스에 대한 의존성을 주입받는다.

```java
Phone phone = new Phone(new RegularPolicy(Money.wons(10), Duration.ofSeconds(10)));
```

### 부가 정책 적용하기

여기에 부가 정책을 적용하면 합성의 강력함을 더욱 실감할 수 있을 것이다.

다음의 제약 조건을 고려하여 부가 정책을 구현할 수 있다.

- 부가정책은 기본 정책 적용후 적용된다.
- 부가정책은 어떤 종류의 정책과도 합성될 수 있어야 한다.
- Phone 입장에서는 자신이 기본 정책의 인스턴스에 메시지를 전송하고 있는지, 부가 정책의 인스턴스에게 메시지를 전송하고 있는지 몰라야 한다.

```java
// 부가 정책
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

// 세금 정책
public class TaxablePolicy extends AdditionalRatePolicy {...}

// 기본 할인 정책
public class RateDiscountablePolicy extends AdditionalRatePolicy {...}
```

다음과 같이 기본 정책과 부가 정책을 합성할 수 있다.

```java
// 일반 요금 + 세금 정책
Phone phone = new Phone(
    new Taxable(0.05,
    new RegularPolicy(...));
)

// 일반 요금 + 세금 정책 + 기본 요금 할인정책

Phone phone = new Phone(
    new RateDiscountablePolicy(Money.wons(1000),
    new TaxablePolicy(0.05, 
    new RegularPolicy(...)));
)
```

객체를 조합하고 사용하는 방식이 상속을 사용한 방식보다 더 일관성있고 예측 가능하다는 사실을 알 수 있다. 그리고 합성의 진가는 새로운 클래스를 추가하거나 수정하는 시점에서 발휘된다.

상속을 사용할 경우 `고정 요금제(FixedRatePolicy)`를 추가하는 경우 불필요한 정도로 많은 클래스를 추가해야 했다. 하지만 합성을 기반으로 한 설곙;서는 단 '하나'의 클래스만 추가하면 된다. 또한 요구사항이 변경될 때도 하나의 클래스만 변경하면 된다.

## 04 믹스인

믹스인은 객체 생성시 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법이다. 

합성이 실행 시점에 객체를 조합한다면 믹스인은 컴파일 시점에 코드 조각을 조합해서 코드를 재사용한다. 상속이 클래스와 클래스 사이의 관계를 고정시키는 데 비해 믹스인은 유연하게 관계를 재구성할 수 있다. 믹스인은 합성처럼 유연하면서도 상속처럼 쉽게 코드를 재사용할 수 있다.

스칼라 언어에서 제공하는 트레이트(trait)를 이용해 믹스인을 구현해보자.

### 기본 정책 구현하기

이부분은 자바와 거의 유사하다.

```
// 기본 정책
abstract class BasicRatePolicy {
  def calculateFee(phone: Phone): Money = phone.calls.map(calculateCallFee(_)).reduce(_ + _)

  protected def calculateCallFee(call: Call): Money;
}

// 표준 요금제
class RegularPolicy(val amount: Money, val seconds: Duration) extends BasicRatePolicy {
  override protected def calculateCallFee(call: Call): Money = ...
}

// 심야 할인 요금제
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

스칼라에서는 다른 코드와 조합해서 확장할 수 있는 기능을 트레이트로 구현할 수 있다.

```
trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    return fee + fee * taxRate
  }
}
```

`extends BasicRatePolicy`

여기서 이 확장의 개념은 상속의 개념이 아니다. 단지 세금 정책이 BasicRatePolicy나 BasicRatePolicy의 자손에 해당하는 경우에만 믹스인될 수 있음을 의미한다. 믹스인은 상속과 달리 동적으로, 제약만 둘뿐 어떤 코드에 믹스인될지 결정하지 않는다. 따라서 super()로 참조되는 코드 역시 고정되지 않는다. 

### 부가 정책 트레이트 믹스인하기

```
// 표준 요금제 + 세금 정책
class TaxableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val taxRate: Double) 
  extends RegularPolicy(amount, seconds) // 조합될 클래스
  with TaxablePolicy // 믹스인할 트레이트
```

스칼라는 특정 클래스에 믹스인한 클래스와 트레이트를 선형화(linearization)해서 어떤 메서드를 호출할지 결정한다.

믹스인을 사용하더라도 여전히 클래스 폭발 문제는 남아있다. 하지만 클래스 폭발의 진짜 문제점은 클래스 수의 증가가 아닌 중복 코드 수가 늘어난다는 점이다. 믹스인에서는 중복 코드로 인한 단점이 없다.

