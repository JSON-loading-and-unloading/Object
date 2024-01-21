<h1>상속과 코드 재사용</h1>


<h2>상속과 중복 코드</h2>

<h3>DRY 원칙</h3>

중복 여부를 판단하는 기준은 변경이다.</br>
요구사항이 변경됐을 때 두 코드를 함께 수정해야 한다면 이 코드는 중복이다.</br></br>

DRY는 반복하지 마라라Don't Repeat Yourself는 뜻이다.</br>
DRY: 원칙의 이름이 무엇이건 핵심은 코드 안에 중복이 존재해서는 안 된다는 것이다.</br></br>


<h3>중복과 변경</h3>


```

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

public class Phone {
    private Money amount;
    private Duration seconds;
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


Call의 책임 : 개별 통화 기간을 저장함 ( 시작, 종료 시간)</br>
Phone의 책임 : 통화 요금 계산함. ( 단위시간, 단위시간당 값, 전체 통화 목록을 저장하는 리스트)</br></br>


여기서 심야 요금 할인이 추가될 시</br>


```

public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
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

Phone과 NightlyDiscountPhone은 중복 코드가 발생한다.</br></br>

중복코드가 발생할 시 요구사항을 아주 짧은 시간 안에 구현할 수 있게 해준다.</br>
하지만 구현 시간을 절약한 대가로 언제가 코드를 변경해야 할 때 문제가 발생할 수 있다.</br>


<h4>타입 코드 사용하기</h4>


```


public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType { REGULAR, NIGHTLY }

    private PhoneType type;

    private Money amount;
    private Money regularAmount;
    private Money nightlyAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this(PhoneType.REGULAR, amount, Money.ZERO, Money.ZERO, seconds);
    }

    public Phone(Money nightlyAmount, Money regularAmount,
                 Duration seconds) {
        this(PhoneType.NIGHTLY, Money.ZERO, nightlyAmount, regularAmount,
                seconds);
    }

    public Phone(PhoneType type, Money amount, Money nightlyAmount,
                 Money regularAmount, Duration seconds) {
        this.type = type;
        this.amount = amount;
        this.regularAmount = regularAmount;
        this.nightlyAmount = nightlyAmount;
        this.seconds = seconds;
    }

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




두 클래스를 하나로 합쳤을 경우</br>
하지만 타입 코드를 사용하는 클래스는 낮은 응집도와 높은 결합도 문제가 생김.</br>

=> 상속 사용</br></br>


<h3>상속을 이용해서 중복 코드 제거</h3>

```

public class Phone {
    private Money amount;
    private Duration seconds;
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





```

public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        super(regularAmount, seconds);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
        // 부모클래스의 calculateFee 호출
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

super 참조를 통해 부모 클래스인 Phone calculateFee메서드를 호출해서 일반 요금제에 따라 통화 요금을 계산 후 이 값에서 통화 시간 시간이 10시 이후인 통화 요금을 뺴준다.</br>

<h3>강하게 결합된 Phone과 NightDiscountPhone</h3>


하지만 여기서 세금을 부과하는 요구사항이 추가된다면 세율(taxRate)을 인스턴스 변수로 포함하고 calcualateFee 메서드에서 값을 반환할 때 taxRate를 이용해 세금을 부과해야 한다.</br>
=>NightDiscountPhone은 생성자에게 전달받은 taxRate를 부모 클래스인 Phone 생성자로 전달해야 한다.</br>
또한 Phone과 동일하게 값을 반환할 때 taxRate를 이용해 세금을 부과해야 한다.</br></br>


코드를 재사용하고 중복코드를 제거하기 위해 상속을 하였지만 세금을 부과하는 로직을 추가하기 위해 Phone을 수정할 때 유사한 코드를 NightDiscountPhone에도 추가해야 했다.</br></br>

❗상속을 위한 경고 1❗</br>

자식 클래스의 메서드 안에서 super참조를 이용해 부모 클래스의 메서드를 직접 호출하는 경우 두 클래스는 강하게 결합된다. super 호출을 제거할 수 있는 방법을 찾아 결합도를 제거하라.</br></br>


<h2>취약한 기반 클래스 문제</h2>

상속 관계를 추가할수록 전체 시스템의 결합도가 높아진다는 사실을 알고 있어야 한다.</br>
=> 상속은 코드의 재사용을 위해 캡슐화의 장점을 희석시키고 구현에 대한 겹합도를 높임으로써 객체지향이 가진 강력함을 반감시킨다.</br></br>

<h3>불필요한 인터페이스 상속 문제</h3>


Stack은 Vector로 상속을 받는다.</br>

```

Stack<String> stack = new Stack<>();

stack.push("1st");
stack.push("2st");
stack.push("3st");

stack.add(0, "4st");

assertEquals("4th", stack.pop());



```

Vector의 stack.add(0, "4st")를 사용하며 Stack이 규칙을 무너뜨릴 여지가 있는 위험한 Vector의 퍼블릭 인터페이스까지도 함께 사용받았다.</br></br>

이 예제는 퍼블릭 인터페이스에 대한 고려 없이 단순히 코드 재사용을 위해 상속을 이용하는 것이 얼마나 위험한지를 잘 보여준다.</br>

❗상속을 위한 경고2❗</br>
상속받은 부모 클래스의 메서드가 자식 클래스의 내부 구조에 대한 규칙을 깨트릴 수 있다.</br></br>


<h3>메서드 오버라이딩 오작용 문제</h3>

```


public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

//    @Override
//    public boolean addAll(Collection<? extends E> c) {
//        addCount += c.size();
//        return super.addAll(c);
//    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public int getAddCount() {
        return addCount;
    }
}


```


```

InstrumentHashSet<String> languages = new InstrumentedHashSet<>();
languages.addAll(Arrays.asList("Java", "Ruby", "Scala"));


```

InstrumentedHashSet의 addAll 메서드가 호출돼서 addCount에 3이 더해진다. 그 후 super.addAll 메서드가 호출되고 제어는 부모 클래스인 HashSet으로 이동한다.</br>
HashSet은 요소를 추가하기 위해 내부적으로 add 메서드를 호출하고 결과적으로 add 메서드가 세번 호출되서 3이 더해져 총 6이 된다.</br></br>


❗상속을 위한 경고3❗</br>
자식 클래스가 부모 클래스의 메서드를 오버라이딩할 경우 부모 클래스가 자신의 메서드를 사용하는 방법에 자식 클래스가 결합될 수 있다.</br></br>




<h3>부모 클래스와 자식 클래스의 동시 수정 문제</h3>


```

public class Song {
    private String singer;
    private String title;

    public Song(String singer, String title) {
        this.singer = singer;
        this.title = title;
    }

    public String getSinger() {
        return singer;
    }

    public String getTitle() {
        return title;
    }
}



```



```

public class Playlist {
    private List<Song> tracks = new ArrayList<>();

    public void append(Song song) {
        getTracks().add(song);
    }

    public List<Song> getTracks() {
        return tracks;
    }
}


```



```


public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
    }
}



```

위 코드에서 요구사항이 변경돼서 PlayList에서 노래의 목록뿐만 아니라 가수별 노래의 제목도 함께 관리해야 한다.</br>

```

public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();

    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());
    }

    public List<Song> getTracks() {
        return tracks;
    }

    public Map<String, String> getSingers() {
        return singers;
    }
}


```



```

public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSinger());
    }
}


```

이 예는 자식 클래스가 부모 클래스의 메서드를 오버라이딩하거나 불필요한 인터페이스를 상속받지 않았음에도 부모 클래스를 수정할 때 자식 클래스를 수정해야 할 수도 있다는 사실을 잘 보여준다.</br></br>

상속은 기본적으로 부모 클래스의 구현을 재사용한다는 기본 전제를 따르기 때문에 자식 클래스가 부모 클래스의 내부에 대해 속속들이 알도록 강요한다.</br>
따라서, 코드 재사용을 위한 상속은 부모 클래스와 자식 클래스를 강하게 결합시키기 때문에 함께 수정해야 하는 상황 역시 빈번하게 발생할 수밖에 없는 것이다.</br>



❗상속을 위한 경고4❗</br>
클래스를 상속하면 결합도로 인해 자식 클래스와 부모 클래스의 구현을 영원히 변경하지 않거나, 자식 클래스와 부모 클래스를 동시에 변경하거나 둘 중 하나를 선택할 수밖에 없다..</br></br>



