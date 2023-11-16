## 객체, 설계

### 01 티켓 판매 어플리케이션 구현하기

* 이벤트 당첨 여부를 확인한다.
    * 이벤트에 당첨된 관람객
        * 초대장을 티켓으로 교환 후 입장 가능하다.
    * 이벤트에 당첨되지 않은 관람객
        * 티켓을 구매해야 입장 가능하다.

### 티켓 판매 어플리케이션 클래스

* Invitation 
    * 이벤트 당첨자에게 발송되는 초대장
    * 초대일자(when) 인스턴스 변수를 가진다.

* Ticket 
    * 공연을 관람하기 위한 티켓

* Bag 
    * 관람객이 소지품을 보관할 용도
    * 초대장(Invitation), 현금(Amoun), 티켓(Ticket) 세 가지 인스턴스 변수를 포함한다.
    * hasInvitation(), hasTicket(), plusAmount(), minusAmount() 메서드를 구현하고 있다.
    * 생성자를 통해 Bag 인스턴스를 생성하는 시점에 상태를 제약한다.

* Audience
    * 관람객
    * 소지품을 보관하기 위해 가방을 소지할 수 있다.

* TicketOffice
    * 매표소
    * 판매하거나 교환해 줄 티켓의 목록(tickets)와 판매 금액(amount)를 인스턴스 변수로 가진다.
    * 티켓을 판매하는 getTicket(), 판매 금액을 더하거나 차감하는 plusAmount()와 minusAmount() 메서드를 구현한다.

* TicketSeller
    * 매표소에서 초대장을 티켓으로 교환해주거나 티켓을 판매하는 역할
    * 자신이 일하는 매표소를 인스턴스 변수로 가진다.

* Theater
    * 소극장을 구현하는 클래스

```
public class Theater {
    private TicketSeller ticketSeller;

    public Theather(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
``` 

### 02 무엇이 문제인가

### 소프트웨어 모듈의 세 가지 목적

* 실행 중에 제대로 동작하는 것
* 변경을 위해 존재하는 것
    * 변경하기 어려운 모듈은 제대로 동작하더라도 개선해야 한다.
* 코드를 읽는 사람과 의사소통하는 것

앞선 설계는 첫 번째 목적인 실행 중에 제대로 동작해야 한다는 제약은 만족시킨다. 하지만 나머지 두 가지 목적은 만족시키지 못한다.

### 예상을 빗나가는 코드

```audience.getBag().hasInvitation()```

* 관람객에게서 가방을 가져오고 초대장이 있는지 직접 확인한다. 관람객은 소극장의 통제를 받는 수동적인 존재가 된다.

```ticketSeller.getTicketOffice().getTicket();```

* 소극장이 판매원의 허락도 없이 매표소에 보관 중인 티켓과 현금에 마음대로 접근할 수 있다. 판매원이 수행하는게 아니라, 매표소가 이 일을 수행한다.

위의 예시들은 우리의 예상을 빗나간다. 현실에서는 관람객이 직접 자신의 가방에서 돈을 꺼내 건내고, 판매원이 매표소에 있는 티켓을 직접 꺼내 관람객에게 건넨다. 현재의 코드는 우리의 상식과는 너무 다르게 동작하기 때문에 코드를 읽는 사람과 제대로 의사소통하지 못한다.

또한 Theater의 enter 메서드 내부에서 너무 많은 정보들이 들어있다. 하나의 클래스나 메서드에 너무 많은 세부사항이 담겨있기 때문에 코드를 읽고 이해해야 하는 사람 모두에게 큰 부담을 준다.

가장 심각한 문제는 Audience와 TicketSeller의 코드를 변경하면 **Theater의 코드도 함께 변경**해야 한다는 점이다.

### 변경에 취약한 코드

만약 관람객이 가방을 들고 있다는 가정이 바뀌었다고 상상해보자.

Audience 클래스에서 Bag를 제거하고, Theater의 enter메서드에서 ```audience.getBag().hasInvitation()``` 해당 코드를 수정해야 한다. Theather는 관람객이 가방을 들고 있다는 세부적인 사실에 지나치게 의존하고 있다.

이러한 문제는 의존성(dependency)에 관련된 문제다. 의존성이라는 말 속에는 어떤 객체가 변경될 때 그 객체에게 의존하는 다른 객체도 함께 변경될 수 있다는 사실이 내포되어 있다.

객체 사이의 의존성을 완전히 없애는 것이 정답은 아니다. 객체지향 설계는 서로 의존하면서 협력하는 객체들의 공동체를 구축하는 것이다. 따라서 우리는 **최소한의 의존성**만 유지하고 불필요한 의존성은 제거해야 한다. 또한 객체 사이의 의존성이 과한 경우를 가리켜 결합도(coupling) 이라고 하는데, 이를 낮춰 변경이 용이한 설계를 만들어야 한다.

### 03 설계 개선하기

### 자율성을 높이자

설계를 변경하기 어려운 이유는 Theater가 Audience와 TicketSeller뿐만 아니라 Audience 소유의 Bag과 TicketSeller가 근무하는 TicketOffice까지 마음대로 접근할 수 있기 때문이다.

```audience.getBag().hasInvitation()```

```ticketSeller.getTicketOffice().getTicket();```

-> Audience와 TicketSeller가 직접 Bag과 TicketOffice를 처리하는 자율적인 존재가 되도록 설계를 변경하자.

```
public class TicketSeller {

    ...

    public void sellTo(Audience audience) {
         if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketOffice.getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketOffice.getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    } // 관객이 가지고 있는 입장권 종류에 따라 티켓 판매하는 책임을 TicketSeller 내부로 이동
}
```
ticketOffice는 private이어서 더 이상 외부에서 접근할 수 없다. 따라서 TicketSeller는 티켓을 꺼내거나 판매 요금을 적립하는 일을 스스로 처리한다.

다음과 같이 객체 내부의 세부적인 사항을 감추는 것을 캡슐화(encapsulation)이라고 한다. 캡슐화를 통해 객체 사이의 결합도를 낮출 수 있기 대문에 설계를 좀 더 쉽게 변경할 수 있다.

```
public class Audience {
    ...

    public Long buy(Ticket ticket) {
        if (bag.hasInvitation()) {
            bag.setTicket(ticket);
            return 0L;
        } else {
            bag.setTicket(ticket);
            bag.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

Audience는 자신의 가방 안에 초대장이 들어있는지를 스스로 확인한다. Audience클래스에서 getBag 메서드를 제거할 수 있고 결과적으로 Bag의 존재를 내부로 캡슐화할 수 있다. TicketSeller와 Audience 사이의 결합도가 낮아졌다.

### 절차지향과 객체지향

맨 처음 Theater의 enter 메소드는 프로세스(Process)이며, Audience, TicketSeller, Bag, TicketOffice는 데이터(Date)이다. 이처럼 프로세스와 데이터를 별도의 모듈에 위치시키는 방식은 절차적 프로그래밍이라고 부른다.

절차지향 프로그래밍 한계
* 객체들이 수동적인 존재이다. 코드를 읽는 사람과 원활하게 의사소통하지 못한다.
* 데이터의 변경으로 인한 영향을 지역적으로 고립시키기 어렵다.

변경하기 쉬운 설계는 한 번에 하나의 클래스만 변경할 수 있는 설계다. 이는 자신의 데이터를 스스로 처리하도록 함으로써 해결할 수 있다. 이처럼 데이터와 프로세스가 동일한 모듈 내부에 위치하도록 프로그래밍하는 방식을 객체지향 프로그래밍이라고 부른다.

훌륭한 객체지향 설계의 핵심은 캡슐화를 이용해 의존성을 적절히 관리함으로써 객체 사이의 결합도를 낮추는 것이다.

### 더 개선할 수 있다.

여전히 Bag은 Audience에 끌려다니는 수동적인 객체이다. 이를 자율적인 객체로 바꿔보자.

```
public class Bag {
    ...
    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        } else {
            setTicket(ticket);
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

또한 TicketSeller 역시 TicketOffice의 자율권을 침해한다.

```
public void sellTo(Audience audience) {
    ticketOffice.plusAmount(audinece.buy(ticketOffice.getTicket()));
}
```

TicketOffice의 자율성을 찾아주자.

```
public void sellTicketTo(Audience audience) {
    plusAmount(audience.buy(getTicket()));
}
```

물론 이렇게 설계함으로써 TicketOffice와 Audience 사이에 의존성이 추가됐다. 하지만 Audiencㄷ에 대한 결합도와 TicketOffice의 자율성 모두를 만족시키는 방법이 잘 떠오르지 않는다.

설계는 균형의 예술이다. 훌륭한 설계는 적절한 트레이드오프의 결과물이라는 사실을 명심해라.





