## CH04 설계 품질과 트레이드오프

좋은 설계는 **변경에 잘 대응할 수 있는 설계**이다.

좋은 객체 지향 설계는 객체에게 올바른 책임을 할당하면서 낮은 결합도와 높은 응집도를 가진 구조를 창조하는 활동이다. 이는 합리적인 비용 안에서 변경을 수용할 수 있는 구조를 만든다.

데이터 중심의 설계와 객체 지향적 설계를 비교해보면서 훌륭한 설계를 위한 책임 할당 원칙을 이해해보자.

## 01 데이터 중심의 영화 예매 시스템 

p. 98

단순히 get/set 이용한 코드

## 02 설계 트레이드오프

좋은 설계의 특징을 판단할 수 있는 품질 척도인 캡슐화, 응집도와 결합도의 의미를 살펴보자.

### 캡슐화

캡슐화는 변경 가능성이 높은 부분을 내부에 숨기고 외부에는 상대적으로 안정적인 부분만 공개하는 것이다. 캡슐화는 불안정한 부분과 안정적인 부분을 분리함으로써 변경의 영향을 최소화할 수 있다.

### 응집도와 결합도

응집도는 모듈에 포함된 내부 요소들이 연관돼 있는 정도를 나타낸다. 결합도는 의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많은 지식을 갖고 있는지를 나타내는 척도이다.

좋은 설계를 위해서는 **높은 응집도와 낮은 결합도**가 요구된다. 하나의 변경을 위해 하나의 모듈만 변경된다면 응집도가 높지만 다수의 모듈이 함께 변경된다면 응집도가 낮다고 할 수 있다. 응집도가 높을수록 코드를 변경하기 위해 여러 모듈의 코드를 구석구석 헤매고 다닐 필요가 없어진다. 결합도는 높을수록 함께 변경해야 하는 모듈의 수가 늘어나기 때문에 변경하기 어려워진다.

### 캡슐화, 응집도, 결합도

응집도와 결합도를 고려하기 전에 먼저 캡슐화를 향상시키기 위해 노력하자. 캡슐화를 지키면 모듈 안의 응집도는 높아지고 모듈 사이의 결합도는 낮아진다.

## 03 데이터 중심의 영화 예매 시스템의 문제점

- 캡슐화 위반
- 높은 결합도
- 낮은 응집도

### 캡슐화 위반

```java
public class Movie {
    private Money fee

    public Money getFee() {
        return fee;
    }

    public void setFee(Money fee) {
        this.fee = fee;
    }
}
```

언뜻보면 직접 객체 내부에 접근할 수 없기 때문에 캡슐화의 원칙을 잘 지킨 것처럼 보인다. 하지만 get, set을 통해 fee 변수를 퍼블릭 인터페이스에 노골적으로 드러내고 있다. 

캡슐화를 위해서는 내부에 저장할 데이터 자체(fee)에 초점을 맞추는 것이 아니라, 협력이라는 문맥을 고려하여 책임을 부여해야 한다.

### 높은 결합도

```java
public class ReservationAgency {
    ...

    Money fee;
    if (discountable) {
        ...
        fee = movie.getFee().minus(discountedAmount).times(audienceCount);
    } else {
        fee = movie.getFee();
    }
}
```

여러 데이터 객체들을 사용하는 제어 로직이 특정 객체 안에 집중되어 있다. 이 결합도로 인해 어떤 데이터 객체를 변경하더라도 제어 객체를 함께 변경할 수밖에 없다.

### 낮은 응집도

```java
public class ReservationAgency {
    public Reservation reserve(...) {
        Movie movie = screening.getMovie();

        ...

        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;

            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT :
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }
        }

        ...
    }
}

```

하나의 변경에 대해 여러 모듈을 수정해야 한다. 예를 들어, 새로운 할인 정책이 추가되면 아래의 3가지 클래스도 함께 변경해야 한다.
- ReservationAgency 의 switch 문에 조건이 추가된다.
- MovieType 에 새로운 할인 정책이 추가된다.
- Movie 에는 새로운 할인 정책의 조건이 추가된다.

하나를 변경했을 때 아무 상관 없는 코드들이 영향을 받을 수 있다. 새로운 할인 정책이 추가되면, 할인 조건 코드에도 영향을 미칠 수 있다.

### 단일 책임 원칙(SRP)

클래스는 단 한가자의 변경 이유만 가져야 한다. 응집도를 높일 수 있는 설계 원칙이다.

## 04 자율적인 객체를 향해

### 스스로 자신의 데이터를 책임지는 객체

객체 설계시 다음과 같은 질문을 할 수 있다.

- 이 객체가 어떤 데이터를 포함해야 하는가?
- 이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인가?

DiscountCondition, Movie , Screening 클래스를 위의 질문들에 답하며 변경했다. 그 결과 ReservationAgency 클래스는 Screening과만 의존성을 갖게 되어 결합도를 낮출 수 있다. 또한 클래스들은 자신들의 데이터를 이용해 스스로 행동을 결정한다.

### 05 하지만 여전히 부족하다.

하지만 여전히 **진정한 의미의 캡슐화**를 지키고 있지 않다.

![](https://github.com/JINU-CHANG/TIL/assets/98975580/073628c4-a2cd-4d6f-9555-79e2fe420cea)

파라미터를 통해 인스턴스 정보를 외부에 노출하고 있다.또한 set 은 없어졌지만 여전히 get 메서드를 통해 정보를 노출시키고 있다.

![](https://github.com/JINU-CHANG/TIL/assets/98975580/ae96c38c-698b-47d0-9d1a-fe841509b6fd)

직접적으로 파라미터나 반환값을 통해 인스턴스 변수의 값을 노출시키지는 않지만, 간접적으로 3개의 할인 정책에 대해 노출시키고 있다.

### 높은 결합도과 낮은 응집도

![KakaoTalk_Photo_2023-12-06-20-03-09](https://github.com/JINU-CHANG/TIL/assets/98975580/365e5029-0a5c-473a-ad50-66987f67a251)
![KakaoTalk_Photo_2023-12-06-20-03-03](https://github.com/JINU-CHANG/TIL/assets/98975580/4275d168-05a6-4d71-bd2e-a5d5f7ba7df9)


캡슐화 위반으로 인해 DiscountCondition의 내부 구현이 외부로 노출되면 높은 결합도, 낮은 응집도의 결과를 가져왔다.

- DiscountCondition -> Movie 
    - DiscountCondition 기간 할인 조건의 명칭이 PERIOD에서 바뀌면 Movie를 수정해야 한다.

- DiscountCondition -> Movie 
    - DiscountCondition의 종류가 추가되거나 삭제되면 if/else 문이 추가된다.

- DiscountCondition -> Movie -> Screening
    - 파라미터가 변경된다면 Movie의 isDiscountable 메서드로 전달된 파라미터를 변경해야 한다. 이로 인해 이 메서드에 의존하는 Screening도 변경해야 한다.

DiscountConditon의 구현을 변경하는 경우에도 이에 의존하는 Movie를 변경해야 한다. 두 클래스 간의 결합도가 높다. 또한 이 영향이 Screening 까지 미치는데 이는 하나의 변경을 수용하기 위해 여러 곳을 동시에 변경해야 하는 상황으로 응집도가 낮다는 증거이다.

### 객체 지향 설계와 비교

![KakaoTalk_Photo_2023-12-06-20-11-56 001](https://github.com/JINU-CHANG/TIL/assets/98975580/02977d69-78ce-495d-92cb-eb6936122654)
![KakaoTalk_Photo_2023-12-06-20-11-56 002](https://github.com/JINU-CHANG/TIL/assets/98975580/f33b7a8b-00fd-4b79-891d-cd767a3ca385)
![KakaoTalk_Photo_2023-12-06-20-11-56 003](https://github.com/JINU-CHANG/TIL/assets/98975580/6032f1b0-2701-435c-b163-d07e9c97e63a)


- DiscountPolicy를 외부에 노출시키지 않는다. -> 캡슐화, Screening과의 결합도 ⬇
- DiscountPolicy라는 객체를 새로 생성했다. 해당 객체는 할인 금액의 양을 계산하는 책임을 갖는다. Movie와 DiscountPolicy의 책임을 분리했다. -> 응집도 ⬆
- 인스턴스 변수를 외부에 노출시키지 않는다. -> 캡슐화, 결합도 ⬇ 



 