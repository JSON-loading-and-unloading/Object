<h1>합성과 유연한 설계</h1>

상속이 부모 클래스와 자식 클래스를 연결해서 부모 클래스의 코드를 재사용하는 데 비해</br>
합성은 전체를 표현하는 객체가 부분을 표현하는 객체를 포함해서 부분 객체의 코드를 재사용한다.</br></br>

상속을 제대로 활용하기 위해서는 부모 클래스의 내부 구현에 대해 상세하게 알아야 하기 때문에 자식 클래스와 부모 클래스 사이의 결합도가 높아질 수 밖에 없다.</br>
합성은 구현에 의존하지 않는다는 점에서 상속과 다르다. 합성은 내부에 포함되는 객체의 구현이 아닌 퍼블릭 인터페이스에 의존하다.</br>
따라서 합성을 이용하면 객체의 내부 구현이 변경되더라도 영향을 최소화할 수 있기 때문에 변경에 더 안정적인 코드를 얻을 수 있게 된다.</br>
=> 상속 대신 합성을 사용하면 변경하기 쉽고 유연한 설계를 얻을 수 있다.</br>

<h2>상속을 합성으로 변경하기</h2>

합성을 사용하여 상속이 초래하는 세 가지 문제점 해결</br>


1. 불필용한 인터페이스 상속 문제</br>

~~~

public class Properties {
    private Hashtable<String, String> properties = new Hashtable <>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}

~~~


- 상속 관계를 제거하고 Hashtable을 properties의 인스턴스 변수로 포함시키면 합성 관계로 변경할 수 있다.
- 클라이언트는 오직 Properties에서 정의한 오퍼레이션만 사용할 수 있다.
- Properties클라이언트는 모든 타입의 키와 값을 저장할 수 있는 Hashtable의 오퍼레이션을 사용할 수 없기 때문에 String 타입의 키와 값만 허용하는 Properties의 규칙을 어길 위험성이 사라진다.


2. 메서드 오버라이딩의 오작용 문제</br>

~~~

public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

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

    public int getAddCount() {
        return addCount;
    }

    @Override public boolean remove(Object o) {
        return set.remove(o);
    }

    @Override public void clear() {
        set.clear();
    }

    @Override public boolean equals(Object o) {
        return set.equals(o);
    }

    @Override public int hashCode() {
        return set.hashCode();
    }

    @Override public Spliterator<E> spliterator() {
        return set.spliterator();
    }

    @Override public int size() {
        return set.size();
    }

    @Override public boolean isEmpty() {
        return set.isEmpty();
    }

    @Override public boolean contains(Object o) {
        return set.contains(o);
    }

    @Override public Iterator<E> iterator() {
        return set.iterator();
    }

    @Override public Object[] toArray() {
        return set.toArray();
    }

    @Override public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }

    @Override public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }

    @Override public boolean retainAll(Collection<?> c) {
        return set.retainAll(c);
    }

    @Override public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }
}

~~~

HashSet에 대한 구현 결합도는 제거하면서도 퍼블릭 인터페이스는 그대로 상속 받을 수 있는 방법은 자바의 인터페이스를 사용할 수 있다.</br>
HashSet은 Set 인터페이스를 실체화하는 구현체 중 하나이며, InstrumentedHashSet이 제공해야하는 모든 오퍼레이션들은 Set 인터페이스에 정의돼 있다.</br>


3. 부모 클래스와 자식 클래스의 동시 수정 문제</br>

~~~

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

~~~


Playlist와 PersonalPlaylist를 함께 수정해야 하는 문제가 해결되지는 않는다.
</br>
그렇다하더라도 합성을 이용하면 향후에 Playlist의 내부 구현을 변경하더라도 파급효과를 최대한 PersonalPlaylist 내부로 캡슐화할 수 있다.
</br>



<h2>상속으로 인한 조합의 폭발적인 증가</h2>


상속으로 인한 문제점

1. 하나의 기능을 추가하거나 수정하기 위해 불필요하게 많은 수의 클래스를 추가하거나 수정해야 한다.
2. 단일 상속만 지원하는 언어에서는 상속으로 인해 오히려 중복 코드의 양이 늘어날 수 있다.


<h3>기본 정책에 세금 정책 조합하기</h3>

~~~

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

~~~

부모 클래스의 메서드를 재사용하기 위해 super 호출을 사용하면 원하는 겨로가를 쉽게 얻을 수는 있지만 자식 클래스와 부모 클래스 사이의 결합도가 높아지고 만다.</br>

~~~

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
~~~



~~~

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

~~~


Phone 클래스에 새로운 추상 메서드인 afterCalculated를 추가하고 RegularPhone은ㅇ 요금을 수정할 필요가 없기 때문에 afterCalculated메서드에서 파라미터로 전달된 요금을 그대로 반환하도록 구현한다.</br>

=> 부모 클래스에 추상 메서드를 추가하면 모든 자식 클래스들이 추상 메서드를 오버라이딩해야 하는 문제가 발생한다.</br>
   (자식 클래스의 수가 많을 경우에는 꽤나 번거로운 일이 될 수밖에 없다.)</br>


~~~

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

~~~



~~~

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

~~~

부가 정책에 따른 클래스 생성</br>

![KakaoTalk_Photo_2024-01-28-21-19-42](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/f479e461-8c43-4954-b99d-7da1b78510fa)


위 클래스들을 다이어그램을 나타냈다.</br></br>

TaxableNightlyDiscountPhone과 TaxableRegularPhone 사이에 코드를 중복했다.</br></br>

다른 조건의 부가 정책을 경우의 수에 따라 상속을 하여도 계속하여 코드가 중복된다.</br>
=> 이를 해결하는 방법은 상속을 포기하는 것이다.</br>




<h3>합성 관계로 변경하기</h3>

상속 관계는 컴파일타임에 결정되고 고정되기 때문에 코드를 실행하는 도중에는 변경할 수 없다.</br>
따라서 여러 기능을 조합해야 하는 설계에 상속을 이용하면 모든 조합 가능한 경우별로 클래스르 추가해야한다.</br></br>

합성은 컴파일 타임 관계를 런타임 관계로 변경함으로써 이 문제를 해결한다.</br>
합성을 사용하면 구현이 아닌 퍼블릭 인터페이스에 대해서만 의존할 수 있기 때문에 런타임에 객체의 관계를 변경할 수 있다.</br>

<h3>기본 정책 합성하기</h3>

```

public interface RatePolicy {
    Money calculateFee(Phone phone);
}

```


```

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


```
public class RegularPolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;

    public RegularPolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

```


```
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

RatePolicy ratePolicy가 Phone 클래스 내부에 포함돼있다.</br>
=> 컴파일 의존성을 구체적인 런타임 의존성으로 대체하기 위해 생성자를 통해 RatePolicy의 인스턴스에 대한 의존성을 주입받는다.</br>




![KakaoTalk_20240129_152053779](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/e5cb4f99-bc98-4199-914f-f452b97301da)


일반 요금제의 규칙에 따라 통화 요금을 계산하고 싶다면 </br>

```
Phone  phone = new Phone( new RegularPolicy( Money.wons(10), Duration.ofSeconds(10)));

```


<h3>부가 정책 적용하기</h3>


![KakaoTalk_20240129_151846644_02](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/b08861c0-bcb4-4817-afc9-36eac3f43efa)

부가 정책 AdditionalRatePolicy

```
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

부가 정책 AdditionalRatePolicysms RatePolicy의 역할을 수행하기 때문에 RatePolicy 인터페이스를 구현한다.</br>


```

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


```

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



![KakaoTalk_20240129_151846644_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/90d7e40e-55f9-4d18-b029-fd3de3902a7f)


<h3>기본 정책과 부가 정책 합성하기</h3>

```

Phone phone = new Phone(
                      new TaxablePolicy(0.05,
                             new RegularPolicy(...));

Phone phone = new Phone(
                      new TaxablePolicy(0.05,
                             new RateDiscountablePolicy(Money.wons(1000),
                                    new RegularPolicy(...)));

```

합성의 장점은 여기서 끝나지 않는다~ㅎ


<h3>새로운 정책 추가하기</h3>


![KakaoTalk_20240129_151846644](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/d0fe6403-12af-4ad7-8283-18602dc97d63)


고정 요금제가 필요하다면 고정 요금제를 구현한 클래스 '하나만' 추가한 후 원하는 방식으로 조합한다.</br>


<h2>믹스인</h2>

믹스인은 객체를 생성할 때 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법을 가리킨다.
합성이 실행 시점에 객체를 조합하는 재사용 방법이라면 믹스인은 컴파일 시점에 필요한 코드 조각을 조합하는 재사용 방법이다.

상속과 차이점

상속의 목적은 자식 클래스를 부모 클래스와 동일한 개념적인 범주로 묶어 is-a관계를 만들기 위한 것이다.
반면 믹스인은 말 그대로 코드를 다른 코드 안에 섞어 넣기 위한 방법이다.
하지만 상속이 클래스와 클래스 사이의 관계를 고정시키는 데 비해 믹스인은 유연하게 관계를 재구성 할 수 있다.


<h3>기본정책 구현하기</h3>


```
abstract class BasicRatePolicy {
  def calculateFee(phone: Phone): Money = phone.calls.map(calculateCallFee(_)).reduce(_ + _)
  
  protected def calculateCallFee(call: Call): Money;
}

```


```

class RegularPolicy(val amount: Money, val seconds: Duration) extends BasicRatePolicy {
  override protected def calculateCallFee(call: Call): Money = amount * (call.duration.getSeconds / seconds.getSeconds)
}

```

표준 요금제를 구현하는 RegularPolicy는 BasicRatePoilicy를 상속받아 개별 call의 요금을 계산하는 calculateCallFee메서드를 오버라이딩한다.

<h3>트레이트로 부가 정책 구현하기</h3>

```

trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    return fee + fee * taxRate
  }
}

```

TaxablePolicy 트레이트가 BasicRatePolicy를 확장한다.</br>
이것은 상속의 개념이 아니라 TaxablePolicy가 BasicRatePolicy나 BasicRatePolicy의 자손에 해당하는 경우에만 믹스인될 수 있다는 것이다.</br>

TaxablePolicy는 BasicRatePolicy의 요금 계산이 끝난 후 결과로 반환된 요금에 세금을 부과해야 한다.</br>
따라서 BasicRatePolicy의 calculateFee메서드를 오버라이딩한 후 super 호출을 통해 먼저 BasicRatePolicy의 calculateFee메서드를 실행한 후 자신의 처리를 수행한다.</br></br>

super를 쓰지 말라더니 이러면 결합도가 커지지 않나?</br>

TaxablePolicy 트레이트가 BasicRatePolicy를 상속하도록 구현했지만 실제로 TaxablePolicy가 BasicRatePolicy의 자식 트레이트가 되는 것이 아니다.</br>
위 코드에서 extends 문은 단지 TaxablePolicy가 사용될 수 있는 문맥을 제한할 뿐이다.</br></br>

super로 참조되는 코드는 고정되지 않는다.</br>
RegularPolicy에 믹스인되는 경우에 RegularPolicy의 calculateFee 메서드가 호출될 것이다.</br>

<h3>부가 정책 트레이트 믹스인하기</h3>

스칼라는 트레이트를 클래스나 다른 트레이트에 믹스인할 수 있도록 extends와 with 키워드를 제공한다.</br>

```

trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    return fee + fee * taxRate
  }
}

```

extends를 이용해 RegularPolicy 클래스를 상속 받고 with를 이용해 TaxablePolicy 트레이트를 믹스인한 새로운 클래스를 만들 수 있다.</br></br>

위 코드에서 TaxableRegularPolicy의 인스턴스가 calcualteFee 메시지를 수신했다고 하자. 먼저 TaxableRegularPolicy 클래스에서 메시지를 처리할 메서드를 찾는다.</br>
이 경우 메서드를 발견할 수 없어. TaxablePolicy에서 메서드를 찾는다.</br>
TaxablePolicy에 calcualteFee메서드가 구현돼 있기 때문에 해당 메서드를 실행한다.</br>
메서드 구현 안에 super 호출이 있기 때문에 상속 계층의 다음 단계에 위치한 RegularPolicy에 calculateFee 메서드가 존재하는지 검색한다.</br>
RegularPolicy에 calculateFee 메서드를 발견할 수 없기 때문에 다음 단계인 BasicRatePolicy의 calculateFee메서드가 호출되고 표준 요금제에 따라 요금이 계산된다.</br>

위 예제를 통해 트레이트 믹스인이 상속에 비해 코드 재사용과 확장의 관점에서 얼마나 편리한지를 알 수 있다.</br>

믹스인을 사용하더라도 상속에서 클래스의 숫자가 기하급수적을 늘어나는 클래스 폭발 문제가 있나라는 말을 할 수 있다.</br>
클래스를 만들어야 하는 것이 불만이라면 클래스를 만들지 않고 다음과 같이 인스턴스를 생성할 때 트레이트 믹스인할 수도 있다.</br>

```
new RegularPolicy( Money(100), Duration.ofSeconds(10))
   with RateDiscountablePolicy
   with TaxablePolicy {
 val discounstAmount = Money(100)     
 val taxRate = 0.2

}


```

※이 방법은 RateDiscountablePolicy와 TaxablePolicy를 RegularPolicy에 믹스인한 인스턴스가 오직 한 군데에서만 필요한 경우에 사용할 수 있다.</br>














