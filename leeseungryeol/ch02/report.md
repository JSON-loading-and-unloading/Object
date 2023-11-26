<h2>요구 사항</h2>

<h4>영화 예매 시스템</h4>

영화와 상영으로 나뉜다.</br>
=> 영화는 제목, 상영시간, 가격정보와  같이 영화가 가지고 있는 기본적인 정보를 가리킨다.</br>
=> 상영은 실제로 관객들이 영화를 관람하는 사건을 표현한다. 상영일자, 시간, 순번 등을 가리킨다.</br></br>

또한, 특정 조건을 만족하면 예매자는 요금을 할인 받을 수 있다.</br>
두 가지 규칙이 있는데 할인조건, 할인정책으로 나뉜다.</br>
=> 할인 조건은 가격의 할인 여부를 결정하며 '순서 조건'과 '기간 조건'으로 나뉜다.</br>
  ( 순서 조건은 상영 순번을 이용해 할인 여부를 결정하는 규칙이고, 기간 조건은 요일, 시작 시간, 종료시간의 세 부분으로 구성되며 영화 시작 시간이 해당 기간 안에 포함될 경우 요금을 할인한다.)</br>
=> 할인 정책은 할인 요금을 결정한다. 할인 정책에는 금액 할인 정책, 비율 할인 정책이 있다.</br>
  (금액 할인 정책은 일정 금액을 할인해주고, 비율 할인 정책은 일정 비율의 요금을 할인해주는 정책이다.)</br>


![KakaoTalk_20231117_185128632](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/06af1081-cd93-4cfa-af91-56d22c1fdabb)


  <h2>협력, 객체, 클래스</h2>

  ❗대부분 사람들은 클래스를 결정한 후에 클래스에 어떤 속성과 메서드가 필요한지 고민한다. 아지만 이것은 객체지향의 본질과는 거리가 멀다.❗</br>

 진정한 객체지향 페러다임으로서의 전환은 클래스가 아닌 객체에 초점을 맞출 때에만 얻을 수 있다.</br>
이를 위해 두 가지에 집중한다.</br>
1. 어떤 클래스가  필요핮니를 고민하기 전에 어떤 객체들이 필요한지 고민하라.
2. 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 봐야한다.


</br>
영화 시스템에 대한 도메인


![KakaoTalk_20231117_185128632_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/f77ddf16-199a-4c34-99e7-d4535594ba8e)


🔽상영 클래스
```
public class Screening{

    private Movie movie;
    private int sequence;
    private LocalDateTime whenSecreened;


    public Screening(Movie movie, int sequence, LocalDateTime whenScreened){

           this.movie = movie;
           this.sequence = sequence;
           this.whenScreened = whenScreened;
     }
    public LocalDateTime getStartTime(){
           return whenScreened;
     }

     public boolean isSequence(int sequence){
           return this.sequence == sequence;
     }

     public Money getMovieFee(){
           return movie.getFee();


      }
}
```

여기서 주목할 점은 인스턴스 변수의 가시성은 private이고 메서드의 가시성은 public이라는 것이다.</br>
클래스를 구현하거나 다른 개발자에 의해 개발된 클래스를 사용할 때 가장 중요한 것은 클래스의 경계를 구분 짓는 것이다.</br>
(Screening에서 알 수 있는 것처럼 외부에서는 객체의 속성에 직접 접근할 수 없도록 막고 적절한 public 메서드를 통해서만 내부 상태를 변경할 수 있게 해야한다.)</br>

<h4>자율적인 객체</h4>

</br>
알아야 할 것들</br>

1. 객체가 상태와 행동을 함께 가지는 복합적인 존재이다.
2. 객체가 스스로 판단하고 행동하는 자율적인 존재이다.

</br>
객체지향의 핵심은 스스로 상태를 관리하고, 판단하고 행동하는 자율적인 객체들의 공동체를 구성하는 것이다.</br>


객체지향은 객체라는 단위 안에 데이터와 기능을 한 덩어리로 묶으로써 문제 영역의 아이디어를 적절하게 표현할수 있게 한다. 이처럼 데이터와 기능을 객체 내부로 함께 묶는 것을 캡슐화라고 한다.</br>
더 나아가 public, protected, private와 같은 접근 수정다를 제공하여 외부의 접근을 통제할 수 있는 접근 제어 메커니즘도 제공한다.</br>

🔽추가 상영 클래스

```
public class Screening{
    private Reservation reserve(Customer customer, int audienceCount){

          return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
}
 
}
```

calculateFee라는 private 메서드를 호출하여 요금을 계산한 후 그 결과를 Reservation의 생성자에 전달하는 함.</br>



```
public class Screening{
    private Money claculateFee(int audienceCount){

          return movie.calculateMovieFee(this).times(audienceCount);
  }
 
}
```

🔽영화 클래스
```
public class Money{
 
   public static final Money ZERO = Money.wons(0);
   private final BigDecimal amount;
   private String title;
   private Duration runningTime;
   private Money fee;
   private DiscountPolicy discountPolicy;

   public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy){
        this.title = title;
        this.runningTime = runningTime;
         this.fee = fee;
        this.discountPolicy;
   }

   public Money getFee(){
      return fee;
}
   public Money calculateMovieFee(Screening screening){           // 할인 요금 반환
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));

}


   public static Money wons(long amount){
         return new Money(BigDecimal.valueOf(amount));
}
   public static Money wons(double amount){

       return new Money(BidDecimal.valueOf(amount));

}

  Money(BigDecimal amount){
      this.amount = amount
 }

  public Money plus(Money amount){
     return new Money(this.amount.add(amount.amount));
 }

  public Money minus(Money amount){
     return new Money(this.amount.subtract(amount.amount));
 }

  public Money times(double percent){
       return new Money(this.amount.multiply(
            BigDecimal.valueOf(percent));

  }

  public boolean isLessThan(Money other){
      return amount.compareTo(other.amount) < 0;


}

public boolean isGreaterThanOrEqual(Money other){
  return amount.compareTo(other.amount) >= 0;

}
}

```

🔽예약 클래스

```
public class Reservation {

   private Customer customer;
   private Screening screening;
   private Money fee;
   private int audienceCount;

   public Reservation( Customer customer, Screening screening, Money fee, int audienceCount){

        this.customer = customer;
        this.screening = screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
}
}

```

Reservation클래스는 고객, 상영 정보, 예매 요금 , 인원 수를 속성을 갖는다.</br>

위처럼 시스템의 어떤 기능을 구현하기 위해 객체들 사이에 이뤄지는 상호작용을 협력이라고 부른다.</br>


![KakaoTalk_20231117_185128632_02](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/48eeb9a8-590e-44bc-999e-316a0a9f0bf3)


🔥객체지향 프로그램을 작성할 때는 먼저 협력의 관점에서 어떤 객체가 필요한지를 결정하고, 객체들의 공통 상태와 행위를 구현하기 위해 클래스를 작성한다.</br>

🔽할인 클래스 
~~~
public abstract class DiscountPolicy{
  private List<DiscountCondition> conditions = new ArrayList<>();

  public DiscountPolicy( DiscountCondition ... conditions){
     this.conditions = Arrays.asList(conditions);
}

public Money calculateDiscountAmount(Screening screening){

   for(DiscountCondition each : conditions){
       if(each.isStatisfiedBy(screening)){
          return getDiscountAmount(screening);

}
}

return Money.ZERO;
}

abstract protected Money getDiscountAmount(Screening screening);


}
~~~
할인 정책이 두 가지인데 공통 코드를 줄이기 위해 추상 클래스를 통해 공통 코드를 작성함.

🔽할인 조건 인터페이스
~~~
public interface DiscountCondition {
    boolean isStisfiedBy(Screening screening);
}
~~~
인터페이스를 작성하여 오버라이딩을 통해 각 할인 조건의 클래스를 구현

![KakaoTalk_Photo_2023-11-20-20-33-31](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/35d35f8d-636d-4c8d-a480-d551f3c0195d)

플로우는 영화 예매 시 할인 정책, 조건 관계 다이어그램


<h2>상속과 다형성</h2>


![KakaoTalk_20231126_224146873](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/56a1c160-5fe3-4cf7-b7a2-a50a94b1738d)


코드를 보면 Movie가 AccountDiscountPolicy나 PercentDiscountPolicy에 의존하는 곳을 찾을 수 없다.</br>
그러나 실행 시점에서는 Movie의 인스턴스는 AmountDiscountPolicy나 PercentDiscountPolicy의 인스턴스에 의존하게 된다.</br>
=> 코드의 의존성과 실행시점의 의존성이 서로 다를 수 있다. 다시말해 클래스 사이의 의존성과 객체 사이의 의존성은 동일하지 않을 수 있다.</br>
📌코드의 의존성과 실행 시점의 의존성이 다르면 다를수록 코드를 이해하기 어려워진다는 것이다. 코드를 이해하기 위해서는 코드뿐만 아니라 객체를 생성하고 연결하는 부분을 찾아야하기 때문이다. 반면 코드의 의존성과 실행시점의 의존성이 다르면 다를 수록 코드는 더 유연해지고 확장 가능해진다. 이와 같은 의존성의 양면성은 설계가 트레이드오프의 산물이라는 사실을 잘 보여준다.</br>

📌설계가 유연해질수록 코드를 이해하고 디버깅하기는 점점 더 어려워진다는 사실을 기억하라. 반면 유연성을 역제하면 코드를 이해하고 디버깅하기는 쉬워지지만 재사용성과 확장 가능성은 낮아진다는 사실도 기억해라</br>


<h2>상속과 인터페이스</h2>


인터페이스는 객체가 이해할 수 있는 메세지의 목록을 정의한다는 것을 기억해라.
=> 상속을 통해 자식 클래스는 자신의 인터페이스에 부모 클래스의 인터페이스를 포함하게 된다.


![KakaoTalk_20231126_224146873_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/e38342dc-1554-498c-8ad1-7ea985d8d296)

위 함수에서 Movie는 협력 객체가 calculateDiscountAmount라는 메시지를 이해할 수만 있다면 그 객체가 어떤 클래스의 인스턴스인지는 상관하지 않는다는 것이다.
따라서, calculateDiscountAmount메시지를 수신할 수 있는 AmoountDiscountPolicy와 PercentDiscountPolicy 모두 DiscountPolicy를 대신해서 Movie와 협력할 수 있다.
=> 자식 클래스는 상속을 통해 부모 클래스의 인터페이스를 물려받기 때문에 부모 클래스 대신 사용될 수 있다.




<h2>다형성</h2>

다형성은 객체지향 프로그램의 컴파일 시간 의존성과 실행 시간 의존성이 다를 수 있다는 사실을 기반으로 한다.
프로그램을 작성할 때, movie클래스는 추상 클래스인 DiscountPolicy에 의존한다. 따라서, 컴파일 시간 의존성은 Movie에서 DiscountPolicy로 향한다.
반면 실행 시점에 Movie의 인스턴스와 실제로 상호작용하는 객체는 AmountDiscountPolicy 또는 PercentDiscountPolicy의 인스턴스이다.

다형성이란 동일한 메시지를 수신했을 때 객체의 타입에 따라 다르게 응답할 수 있는 능력을 의미한다.

다형성 구현하는 방법

메시지와 메서드를 실행 시점에 바인딩한다. 이를 지연 바인딩, 동적 바인딩이라고 한다.
컴파일 시점에 실행될 함수나 프로시저를 결정하는 것을 초기 바인딩, 정적 바인딩이라고 한다.








