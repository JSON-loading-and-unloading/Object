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
