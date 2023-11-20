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









