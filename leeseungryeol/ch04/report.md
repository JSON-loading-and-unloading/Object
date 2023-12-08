<h1>설계 품질과 트레이드 오프</h1>
협력은 애플리케이션의 기능을 구현하기 위해 메시지를 주고받는 객체들 사이의 상호작용이다.</br>
책임은 객체가 다른 객체와 협력하기 위해 수행하는 행동이다.</br>
역할은 대체 가능한 책임의 집합이다.</br></br>

객체지향 설계란 올바른 객체에게 올바른 책임을 할당하면서 낮은 결함도와 높은 응집도를 가진 구조를 창조하는 활동이다.</br>
객체지향 설계 두 가지 관점</br>
1. 객체지향 설계의 핵심이 책임이다.
2. 책임을 할당하는 작업이 응집도와 결함도 같은 설계 품질과 깊이 연관돼 있다는 것이다.

</br></br>
결합도와 응집도를 합리적인 수준으로 유지할 수 있는 원칙</br>
- 객체 상태가 아니라 객체의 행동에 초점을 맞추는 것</br>

  <h2>데이터 중심의 영화 예매 시스템</h2>

  데이터 중심 관점에서 객체는 자신이 포함하고 있는 데이터를 조작하는 데 필요한 오퍼레이션을 정의한다.</br>
  책임 중심의 관점에서 객체는 다른 객체가 요청할 수 있는 오퍼레이션을 위해 필요한 상태를 보관</br>
  => 데이터 중심의 관점은 객체의 상태에 초점을 맞추고 책임 중심의 관점은 객체의 행동에 초점을 맞춘다.</br>

Movie 클래스
  ```
public class Movie {
    private String title;
    private Duration runningTime;

    private Money fee;
    private List<DiscountCondition> discountConditions;
    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public Money getFee() {
        return this.fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }

    public List<DiscountCondition> getDiscountConditions() {
        return this.discountConditions;
    }

    public void setDiscountConditions(List<DiscountCondition> discountConditions) {
        this.discountConditions = discountConditions;
    }

    public MovieType getMovieType() {
        return this.movieType;
    }

    public void setMovieType(MovieType movieType) {
        this.movieType = movieType;
    }

    public Money getDiscountAmount() {
        return this.discountAmount;
    }

    public void setDiscountAmount(Money discountAmount) {
        this.discountAmount = discountAmount;
    }

    public double getDiscountPercent() {
        return this.discountPercent;
    }

    public void setDiscountPercent(double discountPercent) {
        this.discountPercent = discountPercent;
    }

}
  ```

MovieType 
```
public enum MovieType {
    AMOUNT_DISCOUNT,
    PERCENT_DISCOUNT,
    NONE_DISCOUNT
}
```

DiscountConditionType
```
public enum DiscountConditionType {
    SEQEUNCE,
    PERIOD
}
```

DiscountCondition
```
public class DiscountCondition {
    private DiscountConditionType type;
    private int sequence;
    private DayOfWeek dayOfweek;
    private LocalTime startTime;
    private LocalTime endTime;

    public DiscountConditionType getType() {
        return this.type;
    }

    public void setType(DiscountConditionType type) {
        this.type = type;
    }

    public int getSequence() {
        return this.sequence;
    }

    public void setSequence(int sequence) {
        this.sequence = sequence;
    }

    public DayOfWeek getDayOfweek() {
        return this.dayOfweek;
    }

    public void setDayOfweek(DayOfWeek dayOfweek) {
        this.dayOfweek = dayOfweek;
    }

    public LocalTime getStartTime() {
        return this.startTime;
    }

    public void setStartTime(LocalTime startTime) {
        this.startTime = startTime;
    }

    public LocalTime getEndTime() {
        return this.endTime;
    }

    public void setEndTime(LocalTime endTime) {
        this.endTime = endTime;
    }

}

```

Screening
```
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Movie getMovie() {
        return this.movie;
    }

    public void setMovie(Movie movie) {
        this.movie = movie;
    }

    public int getSequence() {
        return this.sequence;
    }

    public void setSequence(int sequence) {
        this.sequence = sequence;
    }

    public LocalDateTime getWhenScreened() {
        return this.whenScreened;
    }

    public void setWhenScreened(LocalDateTime whenScreened) {
        this.whenScreened = whenScreened;
    }

}

```

Reservation
```
public class Reservation {
    private Customer customer;
    private Screening screening;
    private Money fee;
    private int audienceCount;

    public Customer getCustomer() {
        return this.customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    public Screening getScreening() {
        return this.screening;
    }

    public void setScreening(Screening screening) {
        this.screening = screening;
    }

    public Money getFee() {
        return this.fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }

    public int getAudienceCount() {
        return this.audienceCount;
    }

    public void setAudienceCount(int audienceCount) {
        this.audienceCount = audienceCount;
    }

    public Reservation(Customer customer, Screening screening, Money fee, int audienceCount) {
        this.customer = customer;
        this.screening = screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
    }
}

```


Customer
```
public class Customer {
    private String name;
    private String id;

    public Customer(String name, String id) {
        this.id = id;
        this.name = name;
    }

}
```

ReservationAgency
```
public class ReservationAgency{
    public Reservation reserve(Screening screening, Customer customer, int audienceCount){
        Movie movie = screening.getMovie();
    }

    boolean discountable = false;
    for(DiscountCondition condition : movie.getDiscountConditions()){
        if(condition.getType() == DiscountConditionType.PERIOD){
            discountable = screening.getWhenScreened().getDayOfweek().equals(condition.getDayOfweek())
            && condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 
            && condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0;
        }else{
            discountable = condition.getSequence() == screening.getSequence();
        }

        if(discountable){
            break;
        }
    }

    Money fee;
    if(discountable){
        Money discountAmount = Money.ZERO;
        switch (movie.getMovieType) {
            case AMOUNT_DISCOUNT:
                discountAmount = movie.getDiscountAmount();
                break;
            case PERCENT_DISCOUNT:
                discountAmount = movie.getFee().times(movie.getDiscountPercent());
                break;
            case NONE_DISCOUNT:
                discountAmount = Money.ZERO;
                break;    

        }

        fee = movie.getFee().minus(discountable);
    }else{
        fee = movie.getFee();
    }

    return new Reservation(customer, screening, fee, audienceCount);
}


```
위 클래스는 데이터 클래스들을 조합해서 영화 예매 절차를 구현하는 클래스이다.



<h2>설계 트레이드 오프</h2>

<h3>캡슐화</h3>
상태와 행동을 하나의 객체 안에 모으는 이유는 객체 내부 구현을 외부로부터 감추기 위해서다. 구현이란 나중에 변경될 가능성이 높은 어떤 것을 가리킨다.</br>
=> 객체를 사용하면 변경 가능성이 높은 부분은 내부에 숨기고 외부에는 상대적으로 안정적인 부분만 공개함으로써 변경의 여파를 통제할 수 있다.</br>
( 변경될 가능성이 높은 부분을 구현, 상대적으로 안정적인 부분을 인터페이스라고 한다.)</br></br>

<h3>응집도와 결합도</h3>

⏭응집도는 모듈에 포함된 내부 요소들이 연관돼 있는 정도를 나타낸다. (모듈 내의 요소들이 하나의 목적을 위해 긴밀하게 협력한다면 그 모듈은 높은 응집도를 가지고, 모듈 내의 요소들이 서로 다른 목적을 추구한다면 그 모듈은 낮은 응집도를 가진다.) </br>=> 객체지향의 관점에서 응집도는 객체 또는 클래스에 얼마나 관련 높은 책임들을 할당했는지를 나타낸다.</br>

⏭결합도 의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많은 지식을 갖고 있는지를 나타내는 척도다.(어떤 모듈이 다른 모듈에 대해 너무 자세한 부분까지 알고 있다면 두 모듈은 높은 결합도를 가지고 어떤 모듈이 다른 모듈에 비해 꼭 필요한 지식만 알고 있따면 두 모듈은 낮은 결합도를 가진다.)</br> => 객체지향의 관점에서 결합도는 객체 또는 클래스가 협력에 필요한 적절한 수준의 관계만을 유지하고 있는지를 나타낸다.</br>


<h2>데이터 중심의 영화 예매 시스템의 문제점</h2>

<h3>캡슐화 위반</h3>

 - Movie 클래스에서 getFee메서드와 setFee메서드는 Movie내부에 Money타입의 fee라는 이름의 인스턴스 변수가 존재한다는 사실을 퍼블릭 인터페이스에 노골적으로 드러냄
 - 객체가 수행할 책임이 아니라 내부에 저장할 데이터에 초점을 맞췄기 때문에 캡슐화의 원칙을 어기게 됨

<h3>높은 결합도</h3>

- ReservationAgency각 모든 객체에 의존하고 있다.
- DiscountCondition의 데이터가 변경되면 DiscountCondition뿐만 아닌라 ReservationAgency로 함께 수정한다..
- Screening의 데이터가 변경되면 Screening뿐만 아니라 ReservationAgency도 함께 수정해야한다.

  
<h3>낮은 응집도</h3>
수장사항이 발생하는 경우 ReservationAgency 코드 수정해야 경우

- 할인 정책이 추가될 경우
- 할인 정책별로 할인 요금을 계산하는 방법이 변경될 경우
- 할인 조건이 추가되는 경우
- 할인 조건별로 할인 여부를 판단하는 방법이 변경될 경우
- 예매 요금을 계산하는 방법이 변경될 경우


  <h2>자율적인 객체를 향해</h2>

<h3>캡슐화를 지켜라</h3>

캡슐화는 설계의 제1원리다. </br>
1. 객체는 자신이 어떤 데이터를 가지고 있는지를 내부에 캡슐화하고 외부에 공개해서는 안된다.
2. 객체는 스스로의 상태를 책임져야 하며 외부에서는 인터페이스에 정의된 메서드를 통해서만 상태에 접근할 수 있어야 한다.

❗여기서 말하는 메서드는 단순히 속상 하나의 값을 반환하거나 변경하는 접근자가 수정자를 의미하는 것이 아니다.</br>
=> 속성의 가시성을 private으로 설정했다고 해도 접근자와 수정자를 통해 속성을 외부로 제공하고 있다면 캠슐화를 위반하는 것이다.</br>

ex)
</br></br>
![KakaoTalk_20231204_225756463](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/5ac27a45-28cc-4032-80ab-27977d943d84)

위 코드에서 AnyClass에 사각형에 대한 책임이 있다.</br></br>

문제점 </br>
1. 코드 중복 발생
2. 변경에 취약
![KakaoTalk_20231204_225756463_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/af2dfcff-6a82-4173-9cda-a83211048129)

Rectangle에 책임을 주며 해결할 수 있다.</br>


<h3>스스로 자신의 데이터를 책임지는 객체</h3>

- 우리가 상태와 행동을 객체라는 하나의 단위로 묶는 이유는 객체의 스스로 자신의 상태를 처리할 수 있게 하기 위해서다.
- 객체는 단순한 데이터 제공자가 아니다. 객체 내부에 저장되는 데이터보다 객체가 협력에 참여하면서 수행할 책임을 정의하는 오퍼레이션이 더 중요하다.

  이 객체가 어떤 데이터를 포함해야 하는가?
  
  1. 이 객체가 어떤 데이터를 포함해야 하는가?
  2. 이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인가?
 

     
     DiscountCondition
     ```
public class DiscountCondition {

    public DiscountConditionType getType() {
        return type;
    }

    public boolean isDiscountable(DayOfWeek dayOfWeek, LocalDateTime time) {
        if (type != DiscountConditionType.PERIOD) {
            throw new IllegalArgumentException();
        }
        return this.dayOfWeek.equals(dayOfWeek) &&
                this.startTime.compareTo(time) <= 0 &&
                this.endTime.compareTo(time) >= 0;
    }

    public boolean isDiscountable(int sequence) {
        if (type != DiscountConditionType.SEQEUNCE) {
            throw new IllegalArgumentException();
        }
        return this.sequence == sequence;
    }

}

     ```
- 할인 조건에 맞는 함수 구현

   
Movie

```
public class Movie {

    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public MovieType getMovieType() {
        return movieType;
    }

    public Money calculateAmountDiscountedFee() {
        if (movieType != MovieType.AMOUNT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(discountAmount);
    }

    public Money calculatePercentDiscountdFee() {
        if (movieType != MovieType.PERCENT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(fee.times(discountPercent));
    }

    public Money calculateNoneDiscountedFee() {
        if (movieType != MovieType.NONE_DISCOUNT) {
            throw new IllegalArgumentException();
        }

        return fee;
    }

    public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
        for (DiscountCondition condition : discountConditions) {
            if (condition.getType() = DiscountConditionType.PERIOD) {
                if (condition.isDiscountable(whenScreened.getDayOfWeek(), whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }

        return false;
    }
}
```
 - 영화 요금을 계산하는 오퍼레이션
 - 할인 여부를 판단하는 오퍼레이션

   
Screening
```
public class Screening {

    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
    }

    public Money calculateFee(int audienceCount) {
        switch (movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculateAmountDiscountedFee().times(audienceCount);
                }
                break;

            case PERCENT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculatePercentDiscountedFee().times(audienceCount);
                }
                break;
            case NONE_DISCOUNT:
                return movie.calculateNoneDiscountedFee().times(audienceCount);

                return movie.calculateNoneDiscountedFee().times(audienceCount);
        }
    }
}
```

- Movie의 isDiscountable메서드를 호출해 할인 가능한지 여부를 판단 후 요금 계산


  ReservationAgency
  
~~~
  
    public class ReservationAgency {
        public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
            Money fee = screening.calculateFee(audienceCount);
            return new Reservation(customer, screening, fee, audienceCount);
        }
      }

~~~

- reserve가 Screening의 calculateFee함수를 호출

<h2>하지만 여전히 부족하다</h2>

두 번째 설계에서도 캡슐화를 위반한다.</br>

<h3>캡슐화 위반</h3>

1. DiscountCondition 클래스에서 isDiscountable함수 파라미터에서 해당 클래스의 속성을 인자로 받으며 속성을 노출한다.</br>
2. Movie에서는 calculateAmountDiscountedFee, calculatePercentDiscountedFee, calulateNoneDiscountedFee 이 세 메서드로 할인 정책에 대해 노출을 하고 있다.</br></br>

<h3>높은 결합도</h3>
Movie와 DiscountCondition 사이의 결합도</br>
 - DiscountCondition의 기간 할인 조건의 명칭이 PERIOD에서 다른 값으로 변경된다면 Movie를 수정해야 한다.</br>
 - DiscountCondition의 종류가 추가되거나 삭제된다면 Movie안의 if ~ else 구문을 수정해야 한다.</br>
 - 각 DiscountCondition의 만족 여부를 판단하는 데 필요한 정보가 변경된다면 movie의 isDiscountable메서드로 전달 된 파라미터를 변경해야한다.</br>


<h3>낮은 응집도</h3>

할인 조건의 종류를 변경하기 위해서는 DiscountCondition, Movie,그리고 Movie를 사용하는 Screening을 함께 수정해야 한다.</br>

<h2>데이터 중심 설계의 문제점</h2>

1. 데이터 중심 설계는 객체의 행동보다는 상태에 초점을 맞춘다.</br>
   => 데이터 중심 설계는 너무 이른 시기에 데이터에 대해 고민하기 때문에 캡슐화에 실패하게 된다.</br>
2. 데이터 중심 설계는 객체를 고립시킨 채 오퍼레이션을 정의하도록 만든다.</br>
   => 데이터 중심 설계의 초점은 객체의 외부가 아니라 내부로 향한다.</br>
   (협력이라는 문맥 안에서 필요한 책임을 결정하고 이를 수행할 적절한 객체를 결정하는 것이 가장 중요하다.</br>
    올바른 객체지향 설계의 무게 중심은 항상 객체의 내부가 아니라 외부에 맞춰져 있어야 한다.)</br>

