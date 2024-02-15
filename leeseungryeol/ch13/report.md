<h1>서브클래싱과 서브타이핑</h1>

상속의 용도</br>

1. 타입 계층을 구현하는 것</br>
   - 타입 계층 안에서 부모 클래스는 일반적인 개념을 구현하고 자식 클래스는 특수한 개념을 구현한다.</br>
2. 코드 재사용</br>
   - 상속은 간단한 선언만으로 부모 클래스의 코드를 재사용할 수 있는 마법의 주문과도 같다.</br></br>

상속을 사용하는 일차적인 목표는 코드 재사용이 아니라 타입 계층을 구현하는 것이어야 한다.</br>
=> 동일한 메시지에 대해 서로 다르게 행동할 수 있는 다형적인 객체를 구현하기 위해서는 객체의 행동을 기반으로 타입 계층을 구성해야 한다.</br></br>

<h2>타입</h2>

1. 개념 관점의 타입 : 공통의 특징을 공유하는 대상들의 분류
    - 우리가 인지하는 세상의 사물의 종류
2. 프로그래밍 언어 관점에서 타입 : 동일한 오퍼레이션을 적용할 수 있는 인스턴스들의 집합
   - 프로그래밍 언어에서 타입의 두  가지 목적
   - 타입에 수행될 수 있는 유효한 오퍼레이션의 집합을 정의
   - 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공함 
3. 객체지향 페러다임 관점의 타입 : 객체 퍼블릭 인터페이스가 객체 타입을 결정한다. 따라서 동일한 퍼블릭 인터페이스를 제공하는 객체들은 동일한 타입으로 분류된다.
   - 객체에게 중요한 것은 속성이 아니라 행동이다.
   - 어떤 객체들이 동일한 상태를 가지고 있더라도 퍼블릭 인터페이스가 다르다면 이들은 서로 다른 타입으로 분류된다.
   - 반대로 어떤 객체들이 내부 상태는 다르지만 동일한 퍼블릭 인터페이스를 공유한다면 이들은 동일한 타입으로 분류된다.


<h2>타입 계층</h2>


![KakaoTalk_20240212_120739785](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/df95caaf-f5a4-4bdd-aaf9-ed9460560b6c)

타입 계층을 구성하는 두 타입 간의 관계에서 더 일반적인 타입을 슈퍼타입이라고 부르고 더 특수한 타입을 서브 타입이라고 부른다.</br>
프로그래밍 언어 타입은 객체지향 언어 타입과 절차적 언어 타입의 슈퍼 타입이고, 객체지향 언어 타입은 클래스 기반 언어 타입과 프로토 타입 기반 언어 타입의 슈퍼 타입이다.</br></br>

객체의 정의를 의미하는 내연 관점에서 일반화란 어떤 타입의 정의를 좀 더 보편적이고 추상적으로 만드는 과정을 의미한다.</br>
반대로 특수화란 어떤 타입의 정의를 좀 더 구체적이고 문맥 종속적으로 만드는 과정을 의미한다.</br>


내연과 외연의 관점에서 서브타입과 슈퍼타입 정의</br></br>

슈퍼타입은 다음과 같은 특징을 가지는 타입을 가리킨다.</br>
 - 집합이 다른 집합의 모든 멤버를 포함한다.
 - 타입 정의가 다른 타입보다 좀 더 일반적이다.

서브타입은 다음과 같은 특징을 가지는 타입을 가리킨다.</br>
 - 집합에 포함되는 인스턴스들이 더 큰 집합에 포함된다.
 - 타입 정의가 다른 타입보다 좀 더 구체적이다.


<h3>객체지향 프로그래밍과 타입 계층</h3>

객체의 타입을 결정하는 것은 퍼블릭 인터페이스다.</br>
일반적인 타입이란 비교하려는 타입에 속한 객체들의 퍼블릭 인터페이스보다 더 일반적인 퍼블릭 인터페이스를 가지는 객체들의 타입을 의미한다.</br>
특수한 타입이란 비교하려는 타입에 속한 객체들의 퍼블릭 인터페이스보다 더 특수한 퍼블릭 인터페이스를 갖지는 객체들의 타입을 의미한다.</br></br>

슈퍼타입이란 서브타입이 정의한 퍼블릭 인터페이스를 일반화시켜 상대적으로 범용적이고 넒은 의미로 정의한 것</br>
서브타입이란 슈퍼타입이 정의한 퍼블릭 인터페이스를 특수화시켜 상대적으로 구체적이고 좁은 의미로 정의한 것</br>


<h2>서브클래싱과 서브타이핑</h2>

타입 계층을 구현하는 일반적인 방법은 상속을 이용하는 것이다.</br>
상속을 이용해 타입 계층을 구현한다는 것은 부모 클래스가 슈퍼타입의 역할을, 자식 클래스가 서브 타입의 역할을 수행하도록 클래스 사이의 관계를 정의한다는 것을 의미한다.</br></br>


<h3>언제 상속을 사용해야 하는가?</h3>

위 두 질문에 대해 모두 예라고 답할 수 있을 때 상속을 사용</br>

1. 상속 관계가 is-a 관계를 모델링하는가?
   - [자식 클래스가]는 [부모 클래스]다 라고 말해도 이상하지 않다면 상속을 사용할 후보로 간주할 수 있다.
2. 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가?
   - 상속 계층을 사용하는 클라이언트의 입장에서 부모 클래스와 자식 클래스의 차이점을 몰라야한다. 이를 자식 클래스와 부모 클래스 사이의 행동 호환성이라고 부른다.


<h3>is-a 관계</h3>

- 펭귄은 새다
- 새는 날 수 있다

~~~
public class Bird{

   public void fly(){...}


}


public class Penguin extends Bird {


}

~~~

위 처럼 타입 계층의 의미는 행동이라는 문맥에 따라 달라질 수 있다.</br>
(펭귄은 새이긴 하지만 날 수는 없다.)</br></br>

따라서 어떤 두 대상을 언어적으로 is-a 라고 표현할 수 있더라도 일단은 상속을 사용할 예비 후보 정도로만 생각하라.</br>

<h3>행동 호환성</h3>

위 내용과 같이 두 타입 사이에 행동이 호환될 경우에만 타입 계층으로 묶어야 한다는 것을 알 수 있다.</br>
행동의 호환 여부를 판단하는 기준은 클라이언트의 관점이다.</br>
클라이언트가 두 타입이 동일하게 행동할 것이라고 기대한다면 두 타입을 타입 계층으로 묶을 수 있다.</br></br>

상속 관계를 유지하면서 문제 해결 방법</br>

1. Penguin의 fly 메서드를 오버라이딩해서 내부 구현을 비워둔다.</br>

```
public class Penguin extends Bird{

@override
public void fly(){

}

}

```

=> 어떤 행동도 수행하지 않기 때문에 모든 bird가 날 수 있다는 클라이언트의 기대를 만족시키지 못한다.
</br>

2. Penguin의 fly 메서드를 오버라이딩한 후 예외를 던진다.</br>

```
public class Penguin extends Bird{

@override
public void fly(){
   throw new UnsupportOperationException();
}

}

```

=> flyBird 메서드는 fly 메세지를 전송한 결과로  UnsupportOperationException 예외가 던져질 것이라고 기대하지 않을 것이다.</br>


3. flyBird 메서드를 수정해서 인자로 전달된 bird의 타입이 Penguin이 아닐 경우에만 fly 메시지를 전송하도록 하는 것이다.</br>

```
public void flyBird(){
  if(!bird instanceof Penguin){

    bird.fly();
}

}

```

=> new 연산자와 마찬가지로 구체적인 클래스에 대한 결합도를 높인다.</br>


<h3>클라이언트의 기대에 따라 계층 분리하기</h3>

이를 해결하기 위해 날 수 있는 새와 없는 새로 분리한다.</br>


```
public class Bird{

}

public class FlyingBird extends Bird{
   public void fly(){}
}

public class Penguin extends Bird{


}

```


```
public void flyBird(FlyingBird bird){

   bird.fly();
}

```

이와 같이 FlyingBird 타입의 인스턴스만이 fly 메시지를 수신할 수 있다.</br></br>


다른 방법!</br>

클라이언트에 따라 인터페이스를 분리한다.</br>

새가 날 수도 있고, 걸을 수도 있고 펭귄은 걸을 수 있다.</br>

이를 위해
</br>

![KakaoTalk_20240213_124602602](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/94b05a61-7c33-43ce-9695-e284f587ef02)

fly 오퍼레이션을 가진 Flyer 인터페이스와 walk 오퍼레이션을 가진 Walker 인터페이스로 분리하는 것이다.</br>


여기서 펭귄이 Bird 코드를 재사용하려면 합성을 이용한다.</br>


<h3>서브클래싱과 서브타이핑</h3>

상속은 두 두기 목적을 위해 사용된다
</br>
1. 코드 재사용
2. 타입 계층을 구성


서브 클래싱 : 다른 클래스의 코드를 재사용할 목적으로 상속을 사용하는 경우를 가리킨다. 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 없다.( 클래스 상속, 구현 상속)</br>
서브 타이핑 : 타입 계층을 구성하기 위해 상속을 사용하는 경우를 가리킨다. 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 있다.( 인터페이스 상속)</br>

자식 클래스가 부모 클래스의 코드를 재사용할 목적으로 상속을 사용했다면 서브 클래싱이고, 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용할 목적으로 상속을 사용했다면 서브 타이핑이다.</br></br>

<h2>리스코프 치환 원칙</h2>

리스코프 치환 원칙은 `서브 타입은 그것의 기반 타입에 대해 대체 가능해야 한다는 것으로 클라이언트가 차이점을 인식하지 못한 채 기반 클래스의 인터페이스를 통해 서브 클래스를 사용할 수 있어야 한다는 것이다.`</br>


리스코프 치환 원칙에 따르면 자식 클래스가 부모 클래스와 행동 호환성을 유지함으로써 부모 클래스를 대체할 수 있도록 구현된 상속 관계만을 서브타이핑이라고 불러야 한다.</br>


```
public class Rectangle {
    private int x, y, width, height;

    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }

    public int getY() {
        return y;
    }

    public void setY(int y) {
        this.y = y;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }
}

class Square extends Rectangle {

    public Square(int x, int y, int size) {
        super(x, y, size, size);
    }

    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setHeight(height);
        super.setWidth(height);
    }
}

```

위와 같이 코드가 구현됨.</br>



```
public void resize(Rectangle rectangle, int width, int height){
    rectangle.setWidth(width);
    rectangle.setHeight(height);
    assert rectangle.getWidth() == width && rectangle.getHeight() == height;
}

```

위에 따르면 Square의 너비와 높이는 항상 더 나중에 설정된 height의 값으로 설정된다. 따라서 다음과 같이 width와 height 값이 다르게 설정할 경우 메서드 실행이 실패하고 말 것이다.
</br>

```
Square square = new Squere(10, 10, 10);
resize(square, 50, 100);

```

resize 메서드의 관점에서 Rectangle 대신 Squeare를 사용할 수 없기 때문에 Square는 Rectangle이 아니다.</br>

Square는 Rectangle의 구현을 재사용하고 있을 뿐이다.</br>
 
두 클래스는 리스코프 치환 원칙을 위반하기 때문에 서브타이핑 관계가 아니라 서브 클래싱 관계다.</br>


<h3>클라이언트와 대체 가능성</h3>

 - 리스코프 치환 원칙은 자식 클래스가 부모 클래스를 대체하기 위해서 부모 클래스에 대한 클라이언트의 가정을 준수해야 한다는 것을 강조

<h3>리스코프 치환 원칙은 유연한 설계의 기반이다.</h3>

8장 DiscountPolicy 상속 계층을 보면 자식 클래스인 OVerlappedDiscountPolicy를 추가하더라도 클라이언트를 수정할 필요는 없었다.</br>

이 설계는 의존성 역전 원칙과 개방-폐쇄 원칙, 리스코프 치환 원칙이 어우러져 설계를 확장 가능하게 만든 대표적인 예다.</br>


![KakaoTalk_20240214_120655838](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/71235d48-76ed-459d-b25e-c62b224951f3)


자식 클래스가 클라이언트의 관점에서 부모 클래스를 대체할 수 있다면 가능 확장을 위해 자식 클래스를 추가하더라도 코드를 수정할 필요가 없어진다.</br>
따라서 리스코프 치환 원칙은 개방-폐쇄 원칙을 만족하는 설계를 위한 전제 조건이다.</br>


<h2>계약에 의한 설계와 서브타이핑</h2>

클라이언트와 서버 사이의 협력을 의무와 이익으로 구성된 계약의 관점에서 표현하는 것을 계약에 의한 설계라고 부른다.</br>

사전 조건 : 클라이언트가 정상적으로 메서드를 실행하기 위해 만족시켜야하는 조건</br>
사후 조건 : 메서드가 실행된 후에 서버가 클라이언트에게 보장해야 하는 조건</br>
클래스 불변식 : 메서드 실행 전과 실행 후에 인스턴스가 만족시켜야 하는 것</br></br>


리스코프 원칙과 계약에 의한 설계 사이의 관계</br>

 - 서브타입이 리스코프 치환 원칙을 만족시키기 위해서는 클라이언트와 슈퍼타입 간에 체결된 '계약을 준수해야 한다.


```
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition ... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        checkPrecondition(screening);

        Money amount = Money.ZERO;
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                amount = getDiscountAmount(screening);
                checkPostcondition(amount);
                return amount;
            }
        }

        amount = screening.getMovieFee();
        checkPostcondition(amount);
        return amount;
    }

    protected void checkPrecondition(Screening screening) {
        assert screening != null &&
                screening.getStartTime().isAfter(LocalDateTime.now());
    }

    protected void checkPostcondition(Money amount) {
        assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);
    }


    abstract protected Money getDiscountAmount(Screening Screening);
}

```


```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        if (screening == null ||
                screening.getStartTime().isBefore(LocalDateTime.now())) {
            throw new InvalidScreeningException();
        }

        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

```

사전 조건 : DiscountPolicy의 calculateDiscountAmount 메서드는 인자로 전달된 screening이 null인지 여부를 확인하지 않는다. 하지만 screening에 null이 전달된다면 screening.getMovieFee()가 실행될 때 NullPointerException 예외가 던져질 것이다.</br></br>

Screening에 null이 전달되는 것은 우리가 기대했던 것이 아니다.</br>
따라서 단정문을 사용해 사전 조건을 다음과 같이 표현한다.</br>

```
assert screening != null && screening.getStartTime().isAfter(LocalDateTime.now());

```


Movie의 calculateMovieFee 메서드를 보면 DiscountPolicy의 calculateDiscountAmount 메서드의 반환값에 어떤 처리도 하지 않고 fee에서 차감하고 있다.</br>
따라서 calculateDiscountAmount 메서드의 반환값은 항상 null이 아니어야 한다.</br></br>

추가로 반환되는 값은 청구되는 요금이기 때문에 최소 0원보다는 커야한다.</br>

사후 조건은</br>

```
assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);

```

DiscountPolicy 자식 클래스들은 DiscountPolicy의 calculateDiscountAmount메서드를 그대로 상속받기 때문에 계약을 변경하지 않는다.</br>
따라서 Movie의 입장에서 이 클래스들은 DiscountPolicy를 대체할 수 있기 때문에 서브타이핑 관계라고 할 수 있다.</br>


<h3>서브타입과 계약</h3>

자식 클래스가 부모 클래스의 메서드를 오버라이딩 할 수 있다.</br>

```
public class BrokenDiscountPolicy extends DiscountPolicy {

    public BrokenDiscountPolicy(DiscountCondition... conditions) {
        super(conditions);
    }

    @Override
    public Money calculateDiscountAmount(Screening screening) {
        checkPrecondition(screening);                 // 기존의 사전조건
        checkStrongerPrecondition(screening);         // 더 강력한 사전조건

        Money amount = screening.getMovieFee();
        checkPostcondition(amount);                   // 기존의 사후조건
        checkStrongerPostcondition(amount);           // 더 강력한 사후조건
        return amount;
    }

    private void checkStrongerPrecondition(Screening screening) {
        assert screening.getEndTime().toLocalTime()
                .isBefore(LocalTime.MIDNIGHT);
    }

    private void checkStrongerPostcondition(Money amount) {
        assert amount.isGreaterThanOrEqual(Money.wons(1000));
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;
    }
}

```

오버라이딩을 통해 강한 사후, 사전 조건이 추가되면 서브 타입의 조건이 깨지고 만다.</br></br>

계약에 의한 설계는 클라이언트 관점에서의 대체 가능성을 계약으로 설명할 수 있다는 사실을 잘 보여준다.</br>
따라서, 서브타이핑을 위해 상속을 사용하고 있다면 부모 클래스가 클라이언트와 맺고 있는 계약에 관해 깊이 고민해봐야한다.</br>


