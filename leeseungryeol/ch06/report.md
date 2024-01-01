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

















