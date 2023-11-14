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

문제점 :  Theater가 관람객의 가방과 판매원의 매표소에 직접 접근함 
원일 => 결합도가 높은 
해결 => 캡슐화로 해결

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


이로 인해 ticketOffice에 대한 접근은 오직 TicketSeller안에만 존재하게 된다. 
![KakaoTalk_20231114_122219512](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/b2e10f0d-db65-4ad6-bead-6ee716d50854)




※Theater에서 TicketOffice로의 의존성이 제거됐다.

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

현재 TicketSeller가 Audience의 가방을 직접 열어본다. => 결합도 제거 필요



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

Auduence와 TicketSeller는 자신이 가지고 있는 소지품을 스스로 관리한다.

중요한 점은 결합도를 낮추므로써 Audience나 TicketSeller의 내부 구현을 변경하더라도 Theater를 함께 변경할 필요가 없어졌다.


 






