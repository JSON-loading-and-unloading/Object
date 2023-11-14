<h1>극장 클래스</h1>

🔽초대장 클래스
```
public class Invitation {
    private LocalDateTime when;
}
```
🔽티켓 클래스
```
public class Ticket {
    private Long fee;

    public Long getFee() {
        return fee;
    }
}
```

🔽가방 클래스 
```
public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    public Bag(long amount) { // 일반 손님일 경우 초대장 x, 돈 지불
        this(null, amount);
    }

    public Bag(Invitation invitation, long amount) { // 이벤트 당점 손님일 경우 초대장
        this.invitation = invitation;
        this.amount = amount;
    }

    public boolean hasInvitation() {
        return invitation != null; // 초대장 여부 확인
    }

    public boolean hasTicket() { // 티켓 여부 확인
        return ticket != null;

    }

    public void setTicket(Ticket ticket) { // 티켓을 얻음
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) { // 일반 손님이 티켓 구매 시 줄어듬
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```
 손님 가방 안에는 초대장, 돈, 티켓이 있을 것이다. 이벤트에 당첨된 손님은 초대장이 있고 </br>
 일반 손님은 없을 것이다. 돈은 모든 손님들에게 있을 것이고, </br>
 티켓은 모든 손님이 구매를 하거나 초대장을 티켓으로 교환 했을 때 있을 것이다.</br>

🔽손님 클래스  
```
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Bag getBag() {
        return bag;
    }
}
```
손님은 가방을 가지고 있을 것이다.</br>
🔽티켓 부스 클래스 
```
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }

    public Ticket gTicket() {
        return tickets.remove(0);
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

티켓 부스에서는 티켓들이 있고, 티켓이 없어질 때마다 돈이 들어온다.</br>

🔽티켓 판매원 클래스
```
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public TicketOffice gTicketOffice() {
        return ticketOffice;
    }
}
```

티켓 판매원은 티켓 부스에 있다.</br>

🔽극장 클래스
```
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
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
극장에 참가 시 가방에 초대장이 있다면,</br>
티켓 부스에 가서 티켓을 바꾼다.</br>
그 후, 가방에 티켓을 넣는다.</br>
만약 아니라면 티켓을 구매 한다.</br>
유저 가방의 돈은 줄어들고, 티켓 부스의 돈은 늘어난다.</br>
그 후, 가방에 티켓을 넣는다.</br>


<h3>자율성을 높이자</h3>

문제점 :  Theater가 관람객의 가방과 판매원의 매표소에 직접 접근함  </br>
원일 => 결합도가 높은  </br>
해결 => 캡슐화로 해결 </br>

<h5>1</h5>
🔽극장 클래스
 
```
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
/*
   위 부분은 TicketSeller에서 할 일이므로 로직을 옮겨준다.
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
*/
        ticketSeller.sellTo(audience);
    }
}
```
 

🔽티켓 판매원 클래스
```
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }
/*
    삭제
    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
*/
   public void sellTo(Audience audience){
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


이로 인해 ticketOffice에 대한 접근은 오직 TicketSeller안에만 존재하게 된다.  </br>
![KakaoTalk_20231114_122219512](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/b2e10f0d-db65-4ad6-bead-6ee716d50854)




※Theater에서 TicketOffice로의 의존성이 제거됐다. </br>

<h5>2</h5>



🔽티켓 판매원 클래스
```
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

   public void sellTo(Audience audience){
   /*
    수정 필요
         if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
        */

        /*
          수정 후
          ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));

        */
    }

}
```

현재 TicketSeller가 Audience의 가방을 직접 열어본다. => 결합도 제거 필요 </br>



🔽손님 클래스  
```
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Bag getBag() {
        return bag;
    }

    public Long buy(Ticket ticket){
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



Audience가 ticket과 bag에 대해서 주체로 다루게 한다.


![KakaoTalk_20231114_122219512_01 (1)](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/e10b0c6c-e4da-480e-bfd0-b5752efc9376)


TicketSeller와 Audience 사이의 결합도가 낮아졌다.


<h3>무엇이 개선됐는가?</h3>

Auduence와 TicketSeller는 자신이 가지고 있는 소지품을 스스로 관리한다. </br>

중요한 점은 결합도를 낮추므로써 Audience나 TicketSeller의 내부 구현을 변경하더라도 Theater를 함께 변경할 필요가 없어졌다. </br>



<h3>캡슐화와 응집도</h3>

핵심은 객체 내부의 상태를 캡슐화하고 객체 간에 오직 메시지를 통해서만 상호작용하도록 만드는 것이다. </br>
※Theater는 TocketSeller의 내부에 대해서는 전혀 알지 못한다. 단지 TocketSeller가 sellTo 메시지를 이해하고 응답할 수 있다는 사실만 알고 있을 뿐이다. </br>
✅밀집하게 연관된 작업만을 수행하고 연관성 없는 작업은 다른 객체에게 위임하는 객체를 가리켜 응집도가 높다고 말한다. </br>
=> 객체의 응집도를 높이기 위해서는 객체 스스로 자신의 데이터를 책임져야한다. </br>


<h3>절차지향과 객체지향</h3>

이전 코드인 Theater의 enter 메서드는 프로세스이며  Audience, TocketSeller, TicketOffice, Bag는 데이터다. </br>
이처럼 프로세스와 데이터를 별도의 모듈에 위치시키는 방식을 절차적 프로그래밍이라고 부른다. </br>
(Theater가 TicketSeller, TicketOffice, Audience, Bag에 모두 의존하고 있다.)</br>

수정한 후의 코드에서는 데이터를 사용하는 프로세스가 데이터를 소유하고 있는 Audience와 TicketSeller 내부로 옮겨졌다. </br>
이처럼 데이터와 프로세스가 동일한 모듈 내부에 위치하도록 프로그래밍하는 방식을 객체지향 프로그래밍이라고 한다.</br>

❗훌륭한 객체지향 설계의 핵심은 캡슐화를 이용해 의존성을 적절히 관리함으로써 객체 사이의 결합도를 낮추는 것이다.❗</br>


<h3>책임의 이동</h3>


![KakaoTalk_20231115_004027834_01 (1)](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/5eb4390e-a490-467c-8ecf-adcb4e844fc5)

이전 코드에는 Theater에 책임이 집중돼 있다.</br>
변경된 코드를 보면 각각 역할에 대해 책임이 분산돼있다.</br>


<h3>더 개선할 수 있다</h3>


🔽손님 클래스  
```
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Bag getBag() {
        return bag;
    }

    public Long buy(Ticket ticket){
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

여기서 bag에 대해서도 결합도를 낮출 수 있다.

🔽가방 클래스 
```
public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    public Bag(long amount) { // 일반 손님일 경우 초대장 x, 돈 지불
        this(null, amount);
    }

    pulbic Long hold(Ticket ticket){
         if(hasInvitation()){
              setTicket(ticket);
              return 0L;
         }else{
              setTicket(ticket);
              minusAmount(ticket.getFee());
              return ticket.getFee();
         }
    }

    public Bag(Invitation invitation, long amount) { // 이벤트 당점 손님일 경우 초대장
        this.invitation = invitation;
        this.amount = amount;
    }

    public boolean hasInvitation() {
        return invitation != null; // 초대장 여부 확인
    }

    public boolean hasTicket() { // 티켓 여부 확인
        return ticket != null;

    }

    public void setTicket(Ticket ticket) { // 티켓을 얻음
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) { // 일반 손님이 티켓 구매 시 줄어듬
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

---------------------------------------------------------------------------------------------------------------------------------</br>
변화된 Audience</br>


🔽손님 클래스  
```
public class Audience {
    private Bag bag;

    public Long buy(Ticket ticket){
       return bag.hold(ticket);
   }


}
```

----------------------------------------------------------------------------------------------------------------------------------</br>
🔽티켓 판매원 클래스
```
public class TicketSeller {
    private TicketOffice ticketOffice;

    

   public void sellTo(Audience audience){

          ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));  
    }

}
```

TicketSeller와 TicketOffice의 결합도를 낮출 수 있다.

---------------------------------------------------------------------------------------------------------------------------------</br>
변화된 TicketOffice</br>
🔽티켓 부스 클래스 
```
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public void sellTicketTo(Audience audience){
        plusAmount(audience.buy(getTicket()));
    }

    public Ticket geTicket() {
        return tickets.remove(0);
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}

```

🔽티켓 판매원 클래스
```
public class TicketSeller {
    private TicketOffice ticketOffice;

   public void sellTo(Audience audience){

          ticketOffice.sellTicketTo(audience);  
    }

}
```
---------------------------------------------------------------------------------------------------------------------------------</br>


![KakaoTalk_20231115_004027834](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/dd43f32e-02bf-42d2-aa88-8cba88139673)


⛔하지만, 이 과정에서 새로운 의존성이 추가됐다.( TicketOffice와 Audience 사이에 의존성이 추가됐다.)⛔</br>

이로 인해 자율성과 결합도를 선택할 것인지 선택해야 한다.</br>




 






