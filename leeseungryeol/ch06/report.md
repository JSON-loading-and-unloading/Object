<h1>메시지와 인터페이스</h1>

❗️객체지향의 애플리케이션의 가장 중요한 재료는 클래스가 아니라 객체들이 주고받는 메시지다.

훌률한 퍼블릭 인터페이스를 얻기 위해서는 책임 주도 설계 방법을 따르는 것만으로는 부족.
유연하고 재사용 가능한 퍼블릭 인터페이스를 만드는 데 도움이 되는 설계 원칙과 기법을 익히고 적용해야함.

<h2>협력과 메시지</h2>

<h6>용어 정리</h6>

메시지 : 객체가 다른 객체와 협력하기 위해 사용하는 의사소통 메커니즘</br>
(객체의 오퍼레이션이 실행되도록 요청하는 것을 '메시지 전송'이라고 부름.</br></br>
오퍼레이션 : 객체가 다른 객체에게 제공하는 추상적인 서비스</br>
(메시지 전송자는 고려하지 않은채 메시지 수신자의 관점만을 다른다.)</br></br>
메서드 : 메시지에 응답하기 위해 실행되는 코드 블록</br>
(오퍼레이션의 구현)</br></br>
퍼블릭 인터페이스 : 객체가 협력에 참여하기 위해 외부에서 수신할 수 있는 메시지의 묶음</br>
(클래스의 퍼블릭 메서드들의 집합이나 메시지의 집합을 가르키는 데 사용)</br></br>
시그니처 : 오퍼레이션이나 메서드의 명세를 나타낸 것으로 이름과 인자의 목록을 포함</br></br>


<h2>인터페이스와 설계 품질</h2>

최소한의 인터페이스는 꼭 필요한 오퍼레이션만을 인터페이스에 포함함.</br>
추상적인 인터페이스는 어떻게 수행했는지가 아니라 무엇을 하느지를 표현함.</br>


<h3>디미터 법칙</h3>

디미터 법칙은 객체의 내부 구현에 강하기 결합되지 않도록 협력 경로를 제한하는 것이다.</br>
("낯선 자에게 말하지 말라")</br>
=> "오직 하나의 도트만 사용하라"</br>

~~~
public class ReservationAgency{

public Reservation reserve(Screening screening, Customer customer, int audienceCount){

     Movie movie = screening.getMovie();

     boolean discountable = false;
     for(DiscountCondition condition : movie.getDiscountConditions()){
           if(condition.getType() == DiscountConditionType.PERIOD){

                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek())&&
                  condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                  condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;

}else {
    doscountable = condition.getSequence() == screening.getSequence();
}

  if(discountable){
     break;
  }
 }
}
}

~~~

디미터 법칙을 따르기 위해서는 클래스가 특정한 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그래밍 해야함.</br>
모든 클래스 c와 c에 구현된 모든 메서드 m에 대해서, m이 메시지를 전송할 수 있는 모든 객체는 다름에 서술된 클래스의 인스턴스여야 한다.</br></br>

 - m의 인자로 전달된 클래스(c자체를 포함)
 - c의 인스턴스 변수의 클래스
</br>
코드적
 - this 객체
 - 메서드의 매개변수
 - this의 속성
 - this의 속성인 컬렉션의 요소
 - 메서드 내에서 생성된 지역 객체

~~~

public class ReservationAgency{

   public Reservation reserve(Screening screening, Customer customer, int audienceCount){

       Money fee = screening.calculateFee(audienceCount);
       return new Reservation(customer, screening, fee, audienceCount);
  }
}


~~~

위 코드에서는 reserve메소드의 인자 screening와만 결합했다.</br>


<h3>묻지 말고 시켜라</h3>

메시지 전송자는 메시지 수신자의 상태를 기반으로 결정을 내린 후 메시지 수신자의 상태를 바꿔서는 안 된다.</br>
묻지 말고 시켜라 원칙을 따르면 객체의 정보를 이용하는 행동을 객체의 외부가 아닌 내부에 위치시키기 때문에 자연스럽게 정보와 행동을 동일한 클래스 안에 두게 된다.</br>
=> 정보 전문가에게 책임을 할당하게 되고 높은 응집도를 가진 클래스를 얻을 확률이 높아진다.</br></br>




<h3>의도를 드러내는 인터페이스</h3>

메서드를 명명하는 두 가지 방법</br>
 1. 메서드가 작업을 어떻게 수행하는지를 나타내도록 이름 짓는 것
 2. '어떻게'가 아니라 '무엇'을 하는지를 드러내는 것

```

public class PeriodCondition {
    public boolean isStatisfiedByPeriod(Screening screening){...}
}

public class SequenceCondition {
  public boolean isSatisfiedBySequence(Screening screening){..}

}

```

위 메서드의 문제점</br>

1. 메서드에 대해 제대로 커뮤니케이션하지 못한다.
2. 메서드 수준에서 캡슐화를 위반한다.


이를 해결

~~~

public class PeriodCondition {
    public boolean isStatisfiedBy(Screening screening){...}
}

public class SequenceCondition {
  public boolean isSatisfiedBy(Screening screening){..}

}

public interface DiscountCondition {

   boolean inSatisfiedBy(Screening screening);
}

~~~

인터페이스를 통해 이를 분리</br>
어떻게 하느냐가 아니라 무엇을 하느냐에 따라 메서드의 이름을 짓는 패턴을 의도를 드러내는 선택자라고 부름.</br>

<h3>함께 모으다</h3>


1장 Theater코드 참고


~~~

audience.getBag().minusAmount(ticket.getFee());

~~~

위 코드는 Audience 내부에 포함된 Bag에게도 메시지를 전송</br></br>

❗️인퍼페이스와 구현의 분리 원칙 위반</br></br>

요청 순서</br>

Theater -> TickerSeller -> Audience -> Bag</br></br>


Theater 객체의 결합도를 낮추기 위해 각 객체의 책임을 분리</br>

메소드 이름 또한 setTicket이 아닌</br>

Theater -> TickerSeller</br>

sellTo()</br></br>

TickerSeller -> Audience</br>

buy()</br></br>

Audience -> Bag</br>

hold()</br></br>

각 책임에 맞는 메소드 이름을 사용</br>


<h2>원칙의 함정</h2>

원칙이 현재 상황에 부적합하다고 판단된다면 과감하게 원칙을 무시하라.

<h5>디미터 법칙은 하나의 도트를 강제하는 규칙이 아니다</h5>

```
InstScream.of(1,15,20,3,9)filter(x -> x > 10).disctinct().count();

```

이 코드는 미미터 법칙을 위반하지 않는다. 디미터 법칙은 결합도와 관련된 것이며, 이 결합도가 문제가 되는 것은 객체의 내부 구조가 외부로 노출되는 경우로 한정된다.</br></br>

디미터 법칙을 적용하였을 경우</br>
(묻는 것 외에는 다른 방법이 존재하지 않은 경우도 존재한다.)</br>
컬렉션에 포함된 객체들은 처리하는 유일한 방법은 객체에게 물어보는것이다.</br>

```
for( Movie each : movies ){
   total += each.getFee();

}

```

물으려는 객체가 정말로 데이터인 경우도 있다.</br>
객체는 내부 구조를 숨겨야 하므로 디미터 법칙을 따르는 것이 좋지만, 자료 구조라면 당연히 내부를 노출해야 하므로 디미터 법칙을 적용할 필요가 없다.</br></br>


‼소프트웨어 설계에 법칙이란 존재하지 않는다는 것‼</br>


<h2>명령-쿼리 분리 원칙</h2>

루틴 : 어떤 절차를 묶어 호출 가능하도록 이름을 부여한 기능 모듈</br>
프로시저 : 정해진 절차에 따라 내부의 상태를 변경하는 루틴의 한 종류</br>
함수 : 어떤 절차에 따라 필요한 값을 계산해서 반환하는 루틴의 한 종류</br></br>

 - 프로시저는 부수효과를 발생시킬 수 있지만 값을 반환할 수 없다.
 - 함수는 값을 반환할 수 있지만 부수효과를 발생시킬 수 없다.


명령 : 객체의 상태를 수정하는 오퍼레이션</br>
쿼리 : 객체와 관련된 정보를 반환하는 오퍼레이션</br></br>

개념적으로 명령은 프로시저와 동일하고 쿼리는 함수와 동일하다.</br>
❗ 어떤 오퍼레이션도 명령인 동시에 쿼리여서는 안 된다.</br>


Event 클래스</br>

```
public class Event {

   private String subject;
   private LocalDate from;
   private Duration duration;

   public Event(String subject, LocalDateTime from, Duration duration){

           this.subject = subject;
           this.from = from;
           this.duration = duration;
     }
}


```

```

public class RecurringSchedule{

   private String subject;
   private DayOfWeek dayOfWeek;
   private LocalTime from;
   private Duration duration;

   public RecurringSchedule(String subject, DayOfWeek dayOfWeek, LocalTime from, Duration duration){

           this.subject = subject;
           this.dayOfWeek = dayOfWeek;
           this.from = from;
           this.duration = duration;
     }
}


```


```

RecurringSchedule schedule = new RecurringSchedule("회의", DayOfWeek,WEDNESDAY, LocalTime.of(10, 30), Duration.ofMinutes(30));
Event meeting = new Event("회의", LocalDateTime.of(2019, 5, 9, 10, 30), Duration.of(Minutes(30));

assert meeting.isSatisfied(schedule) == false;
assert meeting.isSatisfied(schedule) == true;

```
위 코드에서 매주 수요일 10시 30분부터 30분 동안 진행되는 회의에 대한 반복 일정을 표현하는 RecurringSchedule인스턴스를 생성한다.</br>
다음으로 2019년 5월 9일 10시 30분부터 30분 동안 진행되는 회의를 위한 Event인스턴스인 meeting을 생성한다.</br>
2019년 5월 9일은 목요일이므로 수요일이라는 반복 일정의 조건을 만족시키지 못한다.</br>
따라서 false를 반환</br></br>

하지만 여기서 RecurringSchedule을 이용해 isSatisfied를 두 번 호출했을 때 결과가 다르다.</br>

```

public class Event{
  public boolean isSatisfied(RecurringSchedule schedule){
      if(from.getDayOfWeek() != schedule.getDayOfWeek() ||
           !from.toLocalTime().equals(schedule.getFrom() ||
           !duration.equals(schedule.getDuartion()) {
        reschedule(schedule);
        return false;
          }
     return true;
     }
}


```

reschedule메서드를 호출하여 오류가 뜨는데,</br>
이를 검출하기 어렵다.</br></br>

사실 isSatisfied함수는 명령, 쿼리를 둘 다 적용한 상태이기 때문</br>

따라서,</br>

```
public class Event{

    private void reschedule(RecurringSchedule schedule){
       from = LocalDateTime.of(from.toLocalDate().plusDays(daysDisctance(schedule)),
                 schedule,getFrom());
       duration = schedule.getDuration();

     }

  private long daysDisctance(RecurringSchedule schedule){

       return schedule.getDayOfWeek().getValue() - from.getDayOfWeek().getValue();
}
}

```

위와 같이 수정할 수 있다.</br>
=> 명령과 쿼리를 분리함으로써 명령형 언어의 틀 안에서 참조 투명성의 장점을 제한적이나마 누릴 수 있게 된다.</br>

<h4>책임에 초점을 맞춰라</h4>

디미터 법칙 :  협력이라는 컨텍스트 안에서 객체보다 메시지를 먼저 결정하면 두 객체 사이의 구조적인 결합도를 낮출 수 있다. 수신할 객체를 알지 못한 상태에서 메시지를 먼저 선택하기 때문에 객체의 내부 구조에 대해 고민할 필요가 없어진다.</br></br>

묻지 말고 시켜라 : 메시지를 먼저 선택하면 묻지 말고 시켜라 스타일에 따라 협력을 구조화하게 된다. 클라이언트의 관점에서 메시지를 선택하기 때문에 필요한 정보를 물을 필요 없이 원하는 것을 표현한 메시지를 전송하면 된다.</br></br>



의도를 드러내는 인터페이스 : 메시지를 먼저 선택한다는 것은 메시지를 전송하는 클라이언트의 관점에서 메시지의 이름을 정한다는 것이다. 당연히 그 이름에는 클라이언트가 무엇을 원하는지, 그 의도가 드러날 수밖에 없다.</br></br>



명령-쿼리 분리 원칙 : 메시지를 먼저 선택한다는 것은 협력이라는 문맥 안에서 객체의 인터페이스에 관해 고민한다는 것을 의미한다. 객체가 단순히 어떤 일을 해야하는지 뿐만 아니라 협력 속에서 객체의 상태를 예측하고 이해하기 쉽게 만들기 위한 방법에 관해 고민하게 된다.</br></br>

















