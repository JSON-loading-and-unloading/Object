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











