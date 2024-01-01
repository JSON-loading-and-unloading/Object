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

최소한의 인터페이스는 꼭 필요한 오퍼레이션만을 인터페이스에 포함함.
추상적인 인터페이스는 어떻게 수행했는지가 아니라 무엇을 하느지를 표현함.


<h3>디미터 법칙</h3>

디미터 법칙은 객체의 내부 구현에 강하기 결합되지 않도록 협력 경로를 제한하는 것이다.
("낯선 자에게 말하지 말라")
=> "오직 하나의 도트만 사용하라"

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

디미터 법칙을 따르기 위해서는 클래스가 특정한 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그래밍 해야함.
모든 클래스 c와 c에 구현된 모든 메서드 m에 대해서, m이 메시지를 전송할 수 있는 모든 객체는 다름에 서술된 클래스의 인스턴스여야 한다.

 - m의 인자로 전달된 클래스(c자체를 포함)
 - c의 인스턴스 변수의 클래스

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

위 코드에서는 reserve메소드의 인자 screening와만 결합했다.


<h3>묻지 말고 시켜라</h3>

메시지 전송자는 메시지 수신자의 상태를 기반으로 결정을 내린 후 메시지 수신자의 상태를 바꿔서는 안 된다.
묻지 말고 시켜라 원칙을 따르면 객체의 정보를 이용하는 행동을 객체의 외부가 아닌 내부에 위치시키기 때문에 자연스럽게 정보와 행동을 동일한 클래스 안에 두게 된다.
=> 정보 전문가에게 책임을 할당하게 되고 높은 응집도를 가진 클래스를 얻을 확률이 높아진다.











