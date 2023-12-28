애플리케이션은 메시지를 통해 정의된다는 사실을 기억하라. 객체가 수신하는 메시지들이 객체의 퍼블릭 인터페이스를 구성한다. 훌륭한 퍼블릭 인터페이스를 얻기 위해서는 설계 원칙과 기법을 익히고 적용해야 한다. 이번 장에서는 이러한 기법들을 살펴보는 것이 주제이다.

## 01 협력과 메시지

먼저 혼란스러운 용어부터 정리하자

### 메시지와 메시지 전송

- 메시지 : 객체들이 협력하기 위해 사용하는 유일한 의사소통 수단
    - 오퍼레이션명과 인자로 구성된다.
- 메시지 전송 : 한 객체가 다른 객체에게 도움을 요청하는 것
- 메시지 전송자 : 메시지를 전송하는 객체
- 메시지 수신자 : 메시지를 수신하는 객체

### 메시지와 메서드

객체지향에서 메시지와 메서드는 명확하게 구분된다.

메서드는 메시지를 수신했을 때 실제로 실행되는 함수 또는 프로시저를 뜻한다. 코드 상에서 동일한 이름의 변수에게 동일한 메시지를 전송하더라도 객체의 타입에 따라 실행되는 메서드가 달라질 수 있다는 점이 중요하다.

즉 객체는 메시지와 메서드라는 두 가지 서로 다른 개념을 실행 시점에 연결해야 하기 때문에 컴파일 시점과 실행 시점의 의미가 달라질 수 있다.

### 퍼블릭 인터페이스와 오퍼레이션

- 오퍼레이션 : 수행 가능한 행동에 대한 **추상화**
    - 흔히 내부 구현 코드는 제외하고 단순히 메시지와 관련된 시그니처를 가리키는 경우가 대부분이다.
    - ex) DiscountConditon 인터페이스에 정의된 isSatisfiedBy
- 메서드 : 메시지를 수신했을 때 실제로 실행되는 **구체적인 코드**
    - ex) PeriodConditon에 정의된 각각의 isSatisfiedBy 메서드

다형성을 적용하면 하나의 오퍼레이션에 대해 다양한 메서드를 구현할 수 있다.

### 시그니처

오퍼레이션이나 메서드의 명세를 나타낸 것으로 이름과 인자의 목록을 포함한다.

## 02 인터페이스와 설계 품질

퍼블릭 인터페이스의 품질에 영향을 미치는 원칙과 기법들은 다음과 같다.

- 디미터 법칙
- 묻지 말고 시켜라
- 의도를 드러내는 인터페이스
- 명령-쿼리 분리

### 디미터 법칙

"낯선 자에게 말하지 말라" 또는 "오직 인접한 이웃하고만 말하라" 로 요약할 수 있다. "오직 하나의 **도트**만 사용하라"라고 요약되기도 한다.

다음은 디미터 법칙을 위반하는 코드의 전형적인 모습니다. 
```
screening.getMovie().getDiscountConditions();
```

Screening 이 Movie를 포함하지 않도록 변경되거나 Movie가 DiscountConditon를 포함하지 않도록 변경된다면 ReservationAgency의 코드가 이리저리 흔들리게 된다.

따라서 디미터 법칙에서는 모든 클래스 C와 C에 구현된 모든 메서드 M에 대해 M이 메시지를 전송할 수 있는 모든 객체는 다음의 서술된 클래스로 제한한다.

- M의 인자로 전달된 클래스(C자체를 포함)
- C의 인스턴스 변수의 클래스

위의 코드를 디미터 법칙을 따르도록 개선하면 다음과 같다.

```
screening.calculateFee(audienceCount);
```
ReservatonAgency는 오직 Screening에게만 메시지를 전송할 수 있으며 Screening 내부의 어떤 정보도 알지 못한다.

디미터 법칙은 클래스를 캡슐화하기 위해 따라야 하는 구체적인 지침을 제공한다는 점에서 가치가 있다.

### 묻지 말고 시켜라

묻지 말고 시켜라 원칙에 따르도록 메시지를 결정하다보면 자연스럽게 정보 전문가에게 책임을 할당하게 되고 높은 응집도를 가진 클래스를 얻을 확률이 높아진다.

### 의도를 드러내는 인터페이스

메서드의 이름을 지을 때 '어떻게'가 아니라 '무엇'을 하는지 드러내도록 한다.

'어떻게'를 먼저 고민하면 결과적으로 협력을 설계하기 시작하는 이른 시기부터 클래스의 내부 구현에 관해 고민하게 된다. 반면 '무엇'을 하는지를 드러내면 객체가 협력 안에서 수행해야 하는 책임에 관해 고민하게 된다. 

```
public class PeriodConditon {
    public boolean isSatisfiedByPeriod(Screening screeing) {...}
}

public class SequenceCondition  {
    public boolean isSatisfiedBySequence(Screening screening) {...}
}

-> 위의 메서드는 구현 방식으로 모두 드러내는, 캡슐화를 위반하는 코드이다. 아래와 같이 무엇을 하는지 드러내는 코드로 변경할 수 있다.

public class PeriodConditon {
    public boolean isSatisfiedBy(Screening screeing) {...}
}

public class SequenceCondition  {
    public boolean isSatisfiedBy(Screening screening) {...}
}
```

클라이언트가 두 메서드를 가진 객체를 동일 타입으로 간주할 수 있도록 동일한 타입 계층으로 묶어서 처리할 수 있다.

## 03 원칙과 함정

맹목적으로 원칙을 추종하지 말고 현재 상황에 부적합하다고 판다되면 과감하게 원칙을 무시하라.

### 디미터 법칙은 하나의 도트(.)를 강제하는 규칙이 아니다

```
IntStream.of(1, 15, 20, 3, 9).filter(x -> x > 10).distinct().count();
```

위의 코드는 도트를 계속해서 사용하지만 모두 IntStream이라는 동일한 클래스의 인스턴스를 반환한다. IntStream의 내부 구조가 외부로 유출되지않았기 때문에 캡슐은 그대로 유지된다.

기차 충돌처럼 보이는 코드라도 객체의 내부 구현에 대한 어떤 정보도 외부로 노출하지 않는다면 그것은 디미터 법칙을 준수한 것이다.

### 결합도와 응집도의 충돌

Theater는 Audience 내부에 포함된 Bag에 대해 질문한 후 반한된 결과를 이용해 Bag의 상태를 변경한다.

```
public class Theater {
    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            ...
        }
    }
}
```
이 코드는 Audience의 캡슐화를 위반하기 때문에 Theater는 Audience의 내부 구조에 강하게 결합된다.

다음과 같이 Audience에게 위임 메서드를 추가함으로써 객체의 응집도를 높일 수 있다.

```
public class Audience {
    public Long buy(TIcket ticket) {
        if (bag.hasInvitation()) {
            ...
        }
    }
}
```

하지만 디미터 법칙을 준수하는 것이 항상 긍정적인 결과로만 귀결되는 것은 아니다. 서로 상관없는 책임들이 함께 뭉쳐있는 클래스는 응집도가 낮으며 작은 변경으로도 쉽게 무너질 수 있다.

```
public class PeriodCondition implements DiscountCondition {
    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) && ...
    }
}
```

다음과 같이 PeriodConditon이 screening의 내부 상태를 가져와 사용하기 때문에 캡슐화를 위반한 것으로 생각할 수 있다. 따라서 할인 여부를 판단하는 로직을 Screening쪽으로 옮길 수 있는데 이렇게하면 Screeing의 본질적인 책임을 잃게 된다. Screeing의 본질적인 책임은 예매하는 것이다. 따라서 Screeing의 캡슐화를 향상시키는 것보다는 Screeing의 응집도를 높이고 Screeing과 PeriodCondition 사이의 결합도를 낮추는 것이 전체적인 관점에서 더 좋다.

## 04 명령-쿼리 분리 원칙

- 명령 : 상태를 변경할 수 있지만 상태를 반환해서는 안 된다.
- 쿼리 : 객체의 상태를 반환할 수 있지만 상태를 변경해서는 안 된다.

기계로서의 객체 메타포 관점에서 둥근 모양의 버튼은 쿼리에, 네모 모양은 명령에 해당한다. 둥근 모양 버튼을 누르면 기계의 현재 상태를 출력하지만 상태는 변경하지 않는다. 반면 네모 모양 버튼을 누르면 기계의 상태를 변경하지만 변경된 상태에 관한 어떤 정보도 외부로 공개하지 않는다. 

### 반복 일정의 명령과 쿼리 분리하기

Event 클래스는 현재 이벤트가 RecurringSchedule이 정의한 반복 일정 조건을 만족하는지를 검사하는 isSatisfied 메서드를 제공한다. 

하지만 이 isSatisfied 메서드 안에는 버그가 숨겨져 있다. 처음 이 메서드를 호출하면 false를 반환하는데 두 번째로 호출하면 true를 반환한다.

이는 isSatisfied 메서드가 명령과 쿼리의 두 가지 역할을 동시에 수행하고 있었기 때문이다.

```
public class Event {
    public boolean isSatisfied(RecurringSchedule schedule) {
        if (from.getDayOfWeek() != schedule.getDayOfWeek() || !from.toLocalTime().equals(schedule.getFrom()) ||
        !duration.equals(schedule.getDuration())) {
            reschedule(schedule);
            return false;
        }

        return true;
    }
}
```

- isSatisfied 메서드는 Event가 RecurringSchedule의 조건에 부합하는지를 판단한 후 부합 여부에 따라 boolean을 반환하므로 개념적으로 쿼리다.
- isSatisfied 메서드는 Event가 RecurringSchedule의 조건에 부합하지 않을 경우 Event를 조건에 부합하도록 변경하기에 실제로는 부수효과를 가지는 명령이다.

이와 같이 명령과 쿼리를 뒤섞으면 실행 결과를 예측하기 어려워질 수 있다. 명령과 쿼리를 명확하게 분리해야 한다.

### 명령-쿼리 분리와 참조 투명성

명령과 쿼리를 엄격하게 분류하면 객체의 부수효과를 제어하기 수월해진다. 즉 참조 투명성의 장점을 제한적으로나마 누릴 수 있다.

### 참조 투명성

참조 투명성은 "어떤 표현식 e가 있을 때 e의 값으로 나타나는 모든 위치를 교체하더라도 결과가 달라지지 않는 특성"이다.

수학에서 어떤 함수 f(n)이 존재할 때 f(1)=3 이라고 가정하자. 아래 식을 쉽게 계산할 수 있다.

f(1) + f(1) = 6
f(1) * 2 = 6
f(1) - 1 = 2

f(n)이 어디에 있든 n의 값이 1이라면 결과는 3인 것을 이용해 식을 계산할 수 있다. 함수는 동일한 입력에 대해 항상 동일한 값을 반환한기 때문에 수학의 함수는 참조 투명성을 만족시키는 이상적인 예이다.

반면 객체지향 패러다임에서는 객체의 상태 변경이라는 부수효과를 기반으로 하기 때문에 참조 투명성은 예외에 가깝다. 하지만 명령-쿼리 분리 원칙을 사용하면 이 균열을 조금이나마 줄일 수 있다.

최근 주목받고 있는 함수형 프로그래밍도 참조 투명성의 장점을 극대화할 수 있으며 명령형 프로그래밍에 비해 프로그램의 실행 결과를 더 이해하고 예측하기 쉽다.
