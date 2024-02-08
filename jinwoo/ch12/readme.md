## CH12 다형성

이번 장에서는 상속의 관점에서 다형성이 구현되는 기술적인 메커니즘을 살펴본다. 다형성이 런타임에 메시지를 처리하기에 적합한 메시지를 동적으로 탐색하는 과정을 통해 구현되며, 상속이 이런 메서드를 찾기 위한 일종의 탐색 경로를 클래스 계층의 형태로 구현하기 위한 방법이라는 사실을 이해하게 될 것이다. 즉 상속의 목적은 코드 재사용이 아닌, 다형성의 기반인 타입 계층을 구조화하기 위해 사용해야 한다.

## 01 다형성

> 여러 타입을 대상으로 동작할 수 있는 코드를 작성할 수 있는 방법

다형성의 종류는 아래와 같다.

- 유니버설 다형성
    - 매개변수 다형성
    - 포함 다형성
- 임시 다형성
    - 오버로딩 다형성
    - 강제 다형성

**오버로딩 다형성**

하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우

**강제 다형성**

언어가 지원하는 자동적인 타입 변환이나 사용자가 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할 수 있는 방식

- 1 + 2 -> 덧셈 연산자
- "예시" + 1 -> 연결 연산자

**매개변수 다형성**

클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식

- 제네릭 프로그래밍

**포함 다형성(서브타입 다형성)**

메시지가 동일하더라도 수신한 객체의 타입에 따라 실제 수행되는 행동이 달라지는 능력

- 우리가 일반적으로 말하는 다형성

포함 다형성을 구현하는 가장 일반적인 방법은 상속이며 이번장에서는 포함 다형성에 관해 중점적으로 다룬다.

## 02 상속의 양면성

상속은 재사용 메커니즘으로 보이기 쉽다. 상속을 이용하면 부모 클래스에서 정의한 모든 데이터와 메서드를 자식 클래스의 인스턴스에 자동으로 포함시키기 때문이다. 하지만 재사용을 목적으로한 상속 코드는 이해하기 어렵고 유지보수가 어려운 코드가 될 확률이 높다. 이런 문제를 피하기 위해 상속이 무엇이고 언제 사용해야 하는지 이해하고자 한다. 

다음은 상속을 이해하기 위해 필요한 몇 가지 개념이다.

- 업캐스팅
- 동적 메서드 탐색
- 동적 바인딩
- self 참조
- super 참조

### 상속을 사용한 강의 평가

- Lecture 클래스

```java
public class Lecture {
    private int pass;
    private String title;
    private List<Integer> scores = new ArrayList<>();

    public Lecture(String title, int pass, List<Integer> scores) {
        this.title = title;
        this.pass = pass;
        this.scores = scores;
    }

    public double average() {
        return scores.stream().mapToInt(Integer::intValue).average().orElse(0);
    }

    public List<Integer> getScores() {
        return Collections.unmodifiableList(scores);
    }

    public String evaluate() {
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }

    private long passCount() {
        return scores.stream().filter(score -> score >= pass).count();
    }

    private long failCount() {
        return scores.size() - passCount();
    }
}
```

- GradeLecture 클래스
    - 상속을 이용해 Lecture 클래스를 재사용

```java
public class GradeLecture extends Lecture {
    private List<Grade> grades;

    public GradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }

    @Override
    public String evaluate() {
        return super.evaluate() + ", " + gradesStatistics();
    }

    private String gradesStatistics() {
        return grades.stream().map(grade -> format(grade)).collect(joining(" "));
    }

    private String format(Grade grade) {
        return String.format("%s:%d", grade.getName(), gradeCount(grade));
    }

    private long gradeCount(Grade grade) {
        return getScores().stream().filter(grade::include).count();
    }

    public double average(String gradeName) {
        return grades.stream()
                .filter(each -> each.isName(gradeName))
                .findFirst()
                .map(this::gradeAverage)
                .orElse(0d);
    }

    private double gradeAverage(Grade grade) {
        return getScores().stream()
                .filter(grade::include)
                .mapToInt(Integer::intValue)
                .average()
                .orElse(0);
    }
}
```

### 데이터 관점의 상속

데이터 관점에서 상속은 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함하는 것으로 볼 수 있다. 따라서 자식 클래스의 인스턴스는 자동으로 부모 클래스에서 정의한 모든 인스턴스 변수를 내부에 포함한다.

Lecture를 상속받는 GradeLecture 인스턴스를 생성했다. 
```
Lecture lecture = new GradeLecture("객체지향 프로그래밍",
                    70,
                    Arrays.asList(new Grade("A", 100, 95),
                                ...)),
                    Arrays.asList(81, 95, 75, 50, 45));
```

GradeLecture 클래스의 인스턴스는 직접 정의한 인스턴스 변수뿐만 아니라 부모 클래스인 Lecture가 정의한 인스턴스 변수도 함께 포함한다.

### 행동 관점의 상속

행동 관점의 상속은 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것을 의미한다.

부모 클래스에서 구현한 메서드를 자식 클래스의 인스턴스에서 수행할 수 있는데, 이는 런타임에 시스템이 자식 클래스에 정의되지 않은 메서드가 있는 경우 이 메서드를 부모 클래스 안에서 탐색하기 때문이다.

상속 관계로 연결된 클래스 사이의 메서드 탐색 과정을 살펴보기 전 행동 관점의 상속을 이해하는데 유용한 객체와 클래스 사이의 관계에 초점을 맞춰보자.

GradeLecture 클래스의 인스턴스를 생성했을 때의 메모리 구조를 살펴보자. GradeLecture 인스턴스는 자신의 클래스의 위치를 가리키는 class라는 이름의 포인터를 가지며 이 포인터를 이용해 자신의 클래스 정보에 접근할 수 있다. 또한 부모 클래스의 위치를 가리키는 parent 포인터를 가진다. 

<사진>

## 03 업캐스팅과 다운캐스팅

지금까지 작성한 성적 계산 프로그램에 각 교수별로 강의에 대한 성적 통계를 계산하는 기능을 추가해보자.

```java
public class Professor {
    private String name;
    private Lecture lecture;

    public Professor(String name, Lecture lecture) {
        this.name = name;
        this.lecture = lecture;
    }

    public String compileStatistics() {
        return String.format("[%s] %s - Avg: %.1f", name,
                lecture.evaluate(), lecture.average());
    }
}
```

```java

//Lecture 클래스의 인스턴스 전달
Professor professor = new Professor("다익스트라",
                    new Lecture("알고리즘",
                    ...));

//결과 -> "[다익스트라] Pass:3 Fail:2 - Avg: 69.2"
String statistics = professor.compileStatistics();

//GradeLecture 클래스의 인스턴스 전달
Professor professor = new Professor("다익스트라",
                    new GradeLecture("알고리즘",
                    ...));

//결과 -> "[다익스트라] Pass:3 Fail:2, A:1, B:1, C:1, D:1, F:1 - Avg: 69.2"
String statistics = professor.compileStatistics();
```

Professor 생성자의 인자 타입은 Lecture이지만 GradeLecture 인스턴스를 전달하더라도 문제없이 실행되는 것을 볼 수 있다. 이는 Professor 클래스 내부에 구현된 `lecture.evaluate(), lecture.average()`를 서로 다른 클래스에서 실행할 수 있음을 의미한다. 이를 가능하게 하는 것은 **업캐스팅과 동적 바인딩**이다.

### 업캐스팅

> 부모 클래스 타입으로 선언된 변수에 자식 클래스의 인스턴스를 할당하는 것

```java
//대입문
Lecture lecture = new GradeLecture();

//메서드의 파라미터 타입
public class Professor {
    public Professor(String name, Lecture lecture) {...}
}

Professor professor = new Professor("다익스트라", new GradeLecture(...));
```
이러한 특성을 이용하면 자식 클래스는 부모 클래스를 대체할 수 있기 때문에 부모 클래스와 협력하는 클라이언트는 다양한 자식 클래스의 인스턴스와 협력하는 것이 가능하다. 이는 현재의 자식 클래스뿐만 아니라 미래의 자식 클래스들도 포함하기 때문에 무한한 확장 가능성을 가진다.

### 동적 바인딩

> 선언된 변수의 타입이 아니라 메시지를 수신하는 객체의 타입에 따라 실행되는 메서드가 결정된다.

객체지향 언어에서는 실행될 메서드가 런타임에 결정된다.

Professor의 compileStatistics 메서드가 호출하는 `lecture.evaluate()`는 어떤 클래스에 정의되어있는가? 이는 런타임에 결정되기 때문에 클래스 정의를 살펴보는 것만으로는 정확한 메서드를 알 수 없다.

## 04 동적 메서드 탐색과 다형성

객체지향 시스템은 다음의 규칙에 따라 실행할 메서드를 선택한다.

- 메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사한다. 존재하면 메서드를 실행하고 탐색을 종료한다.
- 메서드를 찾지 못하면 부모 클래스에서 메서드 탐색을 계속한다. 이 과정은 적합한 메서드를 찾을 때까지 상속 계층을 따라 올라가며 계속된다.
- 상속 계층의 가장 최상위 클래스에 이르렀지만 메서드를 발견하지 못한 경우 예외를 발생시키며 탐색을 중단한다.
- 이때 중요한 역할을 하는 것이 바로 self 참조이다.

동적 메서드 탐색은 **자동적 메시지 위임** 그리고 **동적인 문맥**이라는 두 가지 원리로 구성된다는 것을 알 수 있다.

### 자동적 메시지 위임

> 메시지는 상속 계층을 따라 부모 클래스에게 자동으로 위임된다.

이와 관련된 개념으로 메서드 오버라이딩과 오버로딩을 살펴본다.

**오버라이딩**

> 상속 관계에서 부모 클래스에 정의된 메서드를 자식 클래스에서 동일한 시그니처를 갖는 메서드로 재정의하는 것

```java
Lecture lecture = new GradeLecture(...);
lecture.evaluate();
```

Lecture에서 정의한 `evaluate()`을 자식 클래스인 GradeLecture에서 재정의하고 있다. 실행 결과 Lecture에 정의된 메서드가 아닌 실제 객체를 생성한 클래스의 GradeLecture에 정의된 메서드가 살행된다.

자식 클래스가 부모 클래스의 메서드를 오버라이딩하면 자식 클래스에서 부모 클래스로 향하는 탐색 순서 때문에 자식 클래스의 메서드가 부모 클래스의 메서드를 감추게 된다.

**오버로딩**

> 이름은 같지만 시그니처는 다른 메서드를 재정의하는 것

보통 **하나의 클래스** 안에서 같은 이름을 가진 메서드들을 정의하는 것을 오버로딩이라고 하지만, 상속 계층 사이에서 같은 이름을 가진 메서드를 정의하는 것도 메서드 오버로딩이다.

일부 언어에서는 (ex) C++) 메서드 오버로딩을 지원하지 않는다.

### 동적인 문맥

> 메시지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀐다. 이 동적 문맥을 결정하는 것은 바로 메시지를 수신한 객체를 가리키는 self 참조다.

Lecture 클래스의 stats() 메서드에서 getEvaluationMethod()를 호출하는 부분을 주의깊게 살펴보길 바란다.

```java
public class Lecture {
    public String stats() {
        return String.format(..., ..., getEvaluationMethod());
    }

    public String getEvaluationMethod() {
        return "Pass or Fail";
    }
}
```

`getEvaluationMethod()` 라는 구문은 현재 클래스의 메서드를 호출하는 것이 아니라 **현재 객체**에게 해당 메시지를 전송한다. **현재 객체란 self 참조가 가리키는 객체다.**

Lecture 클래스를 상속한 GradeLecture 클래스를 예제로 self 전송을 이해해보자.

```java
public class GradeLecture extends Lecture {
    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```

1. GradeLecture에 stats 메시지를 전송하면 self참조는 GradeLecture 인스턴스를 가리키도록 설정되고 메서드 탐색은 GradeLecture 부터 시작된다.
2. GradeLecture에는 stats 처리를 위한 메서드가 존재하지 않기 때문에 부모 클래스인 Lecture에서 메서드를 탐색하고 이를 실행한다.
3. 부모 클래스의 stats 메서드를 실행하는도중 self 참조가 가리키는 객체에게 getEvaluationMethod() 메시지를 전송하는 구문과 마주치게 된다. 
4. self 참조가 가리키는 객체는 GradeLecture의 인스턴스이므로 메서드 탐색은 GradeLecture에서 다시 시작된다.

이런 원리로 인해 깊은 상속 계층과 계층 중간중간에 함정처럼 숨겨져 있는 메서드 오버라이딩과 만나면 극단적으로 이해하기 어려운 코드가 만들어질 수 있다.

### self 대 super

self 의 특징과 대비해 언급할 만하 가치가 있는 것이 super 참조다. super 참조는 자식 클래스에서 부모 클래스의 인스턴스 변수나 메서드에 접근하기 위해 사용된다. 이때 부모 클래스에서 원하는 메서드를 찾지 못한다면 더 상위의 부모 클래스로 이동하면서 메서드가 존재하는지 검사한다.

self 전송이 메시지를 수신하는 객체의 클래스에 다라 메서드를 탐색할 시작 위치를 동적으로 결정하는데 비해, super 전송은 항상 메시지를 전송하는 클래스의 부모 클래스에서부터 시작된다. 즉 동적인 self 전송에 비해 super 전송은 메서드 탐색을 싲ㄱ할 클래스를 컴파일 시점에 결정해놓을 수 있다.

## 05 상속 대 위임

### 위임과 self 참조

위임은 자신이 수신한 메시지를 다른 객체에게 동일하게 전달해서 처리를 요청한다. 이를 위해 위임은 항상 현재의 실행 문맥을 가리키는 self 참조를 인자로 전달한다. 

**포워딩과 위임**

단순히 코드를 재사용하고 처리를 요청할 때 self 참조를 전달하지 않는 경우를 포워딩이라고 한다. 이와 달리 self 참조를 전달하는 것은 위임이다.

위임은 상속을 구현하는 방법이며 상속이 매력적인 이유는 우리가 구현하지 않아도 자동으로 self 참조를 전달하기 때문이다. self 참조의 전달은 결과적으로 자식 클래스의 인스턴스와 부모 클래스의 인스턴스 사이에 동일한 실행 문맥을 공유할 수 있게 한다.








