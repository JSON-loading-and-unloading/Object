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




