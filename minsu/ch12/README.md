# 12. 다형성
## 01. 다형성
  - 유니버셜
      - 매개변수 다형성 : 클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언 후 사용하는 시점에 구체적인 타입으로 지정
      - 포함 다형성 : 메시지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력, 보통 상속을 사용
  - 임시
      - 오버로딩 다형성 : 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우 
      - 강제 다형성 : 언어가 지원하는 자동적인 타입 변환이나 사용자가 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할 수 있는 방식

## 02. 상속의 양면성
- 객체지향 프로그래밍을 작성하기 위해서는 항상 **데이터**와 **행동**이라는 두 가지 관점을 고려해야 함
  - (상속) 데이터 : 부모 클래스에서 정의한 모든 데이터를 자식 클래스의 인스턴스에 자동으로 포함시킬 수 있다.
  - (상속) 행동 : 부모 클래스에서 정의한 일부 메서드 역시 자동으로 자식 클래스에 포함시킬 수 있다.   

<br>

> 상속의 목적은 코드 재사용이 아니다.   
> 상속은 프로그램을 구성하는 개념들을 기반으로 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것이다.


- 상속의 매커니즘을 이해하는데 필요한 몇 가지 개념
    - 업캐스팅
    - 동적 메서드 탐색
    - 동적 바인딩
    - self 참조
    - super 참조

### 상속을 사용한 강의 평가
수강생들의 성적을 계산하는 간단한 예제 프로그램을 구현 !   
`Pass:3 Fail:2, A:1 B:1 C:1 D:0 F:2`   

다음과 같은 형식으로 출력하는 Lecture 클래스가 이미 존재   
`Pass:3 Fail:2`

Lecture 클래스 : 
- 과목명-title-과 학생들의 성적을 보관할 리스트-scores-를 인스턴스 변수로 가진다.
- 강의 이수 기준 : 성적이 pass에 저장된 값보다 크거나 같을 경우
- average 메서드 : 전체 학생들의 평균 성적 계산
- evaluate 메서드 : 강의를 이수한 학생의 수와 낙제한 학생의 수를 반환
- getScores 메서드 : 전체 학생들의 성적을 반환

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

    public String evaluate() {
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }

    public long passCount(){
        return scores.stream().filter(score -> score >=pass).count();

    }

    public long failCount(){
        return score.size() - passCount();
    }

}
```

원하는 기능은 Lecture의 출력 결과에 등급별 통계를 추가: GradeLecture   
- Grade 클래스 : 등급의 이름과 각 등급 범위를 정의하는 최소 성적과 최대 성적을 인스턴스 변수로 포함
- evaluate 메서드 :
   - 예약어 super을 이용해 Lecture 클래스의 evaluate 메서드를 먼저 실행
   - 부모 클래스와 evaluate 메서드의 시그니처가 동일 - > 동일한 시그니처의 메서드를 재정의해 부모 클래스의 구현을 새로운 구현으로 대체하는 것을 **메서드 오버라이딩**이라고 한다.
   - 동일한 시그니처를 가진 메서드가 존재할 경우 자식 클래스의 메서드 우선순위가 더 높다.

- average 메서드 :
  - 자식 클래스에 부모 클래스에는 없는 새로운 메서드를 추가하는 것도 가능
  - 부모 클래스의 메서드와 이름은 같지만 시그니처가 다름 -> 부모 클래스에서 정의한 메서드와 이름은 동일하지만 시그니처는 다른 메서드를 자식 클래스에 추가하는 것을 **메서드 오버로딩**이라고 한다.

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

- 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함
- 자식 클래스의 인스턴스는 자동으로 부모 클래스에서 정의한 모든 인스턴스 변수를 내부에 포함하게 되는 것

![image](https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/9a53ff76-2dca-4b35-beb6-2df4d1bf35b1)

### 행동 관점의 상속
- 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것
- 부모 클래스의 모든 퍼블릭 메서드는 자식 클래스의 퍼블릭 인터페이스에 포함된다.
- 런타임에 시스템이 자식 클래스에 정의되지 않은 메서드가 있을 경우 이 메서드를 부모 클래스 안에서 탐색
- 객체의 경우에는 각 인스턴스별로 독립적인 메모리를 할당받음
- 메서드의 경우에는 동일한 클래스의 인스턴스끼리 공유가능
- 따라서 클래스는 한 번만 메모리에 로드하고 각 인스턴스별로 클래스를 가리키는 포인터를 갖게 함

<img width="632" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/5286e52b-a8e0-4f9b-aff1-7e373b82de71">
- 인스턴스는 두 개 생성됐지만, 클래스는 단 하나만 메모리에 로드됐다.

자식 클래스의 인스턴스를 통해 부모 클래스에서 정의된 메서드를 실행할 수 있는 방법    
- 메시지를 수신한 객체는 class 포인터로 연결된 자신의 클래스에서 적절한 메서드가 존재하는지를 찾는다.
- 만약 메서드가 존재하지 않으면 클래스의 parent 포인터를 따라 부모 클래스를 차례대로 훑어가면서 적절한 메서드가 존재하는지를 검색한다.

<img width="680" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/45301e88-f685-4d76-bb9c-ba74a57be6a6">


## 03 업캐스팅과 동적 바인딩

### 같은 메시지, 다른 메서드

성적 계산 프로그램에 각 교수별로 강의 대한 성적 통계를 계산하는 기능을 추가
- Professor 클래스 : 통계를 계산하는 책임
- compileStatistics 메서드 : 통계 정보를 생성하기 위해 Lecture의 evaluate 메서드와 average 메서드를 호출
  
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

- Professor가 생성자의 두 번째 인자로 Lecture 대신 GradeLecture를 받아도 아무 문제 없다.

```java
Professor professor = new Professor("다익스트라", 
                        new Lecture("알고리즘",
                            70,
                            List.of(81, 95, 75, 50, 45)));

String statistics = professor.compileStatistics();
// [다익스트라] Pass:3 Fail:2 - AVG: 69.2

Professor professor = new Professor("다익스트라", 
                        new GradeLecture("알고리즘",
                            70,
                            List.of(new Grade("A", 100, 95), ...),
                            List.of(81, 95, 75, 50, 45)));

String statistics = professor.compileStatistics();
// [다익스트라] Pass:3 Fail:2, A:1 B:1 C:1 D:1 F:1 - AVG: 69.2
```
-> 코드 안에서 선언된 참조 타입과 무관하게 실제로 메시지를 수신하는 객체의 타입에 따라 실행되는 메서드가 달라질 수 있는 것은 업캐스팅과 동적 바인딩이라는 메커니즘이 작용하기 때문

### 업캐스팅

- 상속을 이용하면 부모 클래스의 퍼블릭 인터페이스가 자식 클래스의 퍼블릭 인터페이스에 합쳐짐
- 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용하더라도 메시지를 처리하는데는 아무런 문제가 없음
- 컴파일러는 명시적인 타입 변환 없이도 자식 클래스가 부모 클래스를 대체할 수 있게 허용
- 부모 클래스 타입으로 선언된 파라미터에 자식 클래스의 인스턴스를 전달하는 것도 가능
- 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해서는 명시적인 타입 캐스팅이 필요 -> 다운 캐스팅

<img width="330" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/06aa0a2e-7cca-4960-aca4-52eee5bbe8de">


### 동적 바인딩

- 정적 바인딩
    - 호출될 함수를 컴파일타임에 결정한다.
    - 코드를 작성하는 시점에 호출될 코드가 결정된다.
- 동적 바인딩
    - 실행될 메서드가 런타임에 결정된다.

## 04 동적 메서드 탐색과 다형성

- 객체지향 시스템에서의 실행할 메서드를 선택하는 규칙
  - 메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사하고, 존재하면 메서드를 샐행하고 탐색을 종료한다.
  - 메서드를 찾지 못했다면 부모 클래스에서 메서드 탐색을 계속하며, 적합한 메서드를 찾을 때까지 상속 계층을 따라 올라가며 계속한다.
  - 상속 계층의 가장 최상위 클래스에 이르렀지만 메서드를 발견하지 못했다면 예외를 발생시키며 탐색을 중단한다.

- self 참조
  - 객체가 메시지를 수신하면 컴파일러는 self 참조라는 임시 변수를 자동으로 생성한 후 메시지를 수신한 객체를 가리키도록 설정
  - 동적 메서드 탐색은 self 가 가리키는 객체의 클래스에서 시작해서 상속 계층의 역방향으로 이뤄지며 메서드 탐색이 종료되는 순간 self 참조는 자동으로 소멸


### 자동적인 메시지 위임

- 상속을 이용할 경우 메시지가 상속 계층을 따라 부모 클래스에게 자동으로 위임된다.
  - 상속 계층을 정의하는 것은 메서드 탐색 경로를 정의하는 것과 동일하다.
  - 자식 클래스에서 부모 클래스의 방향으로 자동으로 메시지 처리가 위임되기 때문에 자식 클래스에서 어떤 메서드를 구현하고 있느냐에 따라 부모 클래스에 구현된 메서드의 운명이 결정됨
- 메서드 오버라이딩
  - 자식 클래스와 부모 클래스 양쪽 모두에 동일한 시그니처를 가진 메서드가 구현돼 있다면 자식 클래스의 메서드가 먼저 검색
  - 자식 클래스에서 부모 클래스로 향하는 메서드 탐색 순서 때문에 자식 클래스의 메서드가 부모 클래스의 메서드를 감추게 된다.
- 메서드 오버로딩
  - 자식 클래스가 부모 클래스에 존재하는 메서드가 동일한 시그니처를 가진 메서드를 재정의해서 부모 클래스의 메서드를 감추는 현상
  - 메서드의 이름은 같지만 시그니처가 다르므로 메서드가 공존한다.
  - C++ 언어에서는 상속 계층 사이의 메서드 오버로딩을 지원하지 않는다.

### 동적인 문맥

- 메시지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀐다.
  - 이 동적인 문맥을 결정하는 것이 self 참조
  - self 참조가 가리키는 객체의 타입을 변경함으로써 객체가 실행될 문맥을 동적으로 바꿀 수 있다.

- self 전송 : 자신에게 다시 메시지를 전송한다.

```java
public class Lecture {
    public String stats() {
        return String.format("Title: %s, Evaluation Method: %s", title, getEvaluationMethod());
    }

    public String getEvaluationMethod() {
        return "Pass or Fail";
    }
}
```

- stats() 메서드 안에서는 자신의 getEvaluationMethod()를 호출하고 있다.
  - 현재 클래스의 메서드를 호출하는 것이 아닌, 현재 객체에게 getEvaluationMethod 메시지를 전송하는 것
  - 현재 객체란 self가 가리키는 객체

```java
public class GradeLecture extends Lecture {
    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```

- GradeLecture에 stats 메시지를 전송하면
  - GradeLecture 클래스에는 stats 메시지를 처리할 수 있는 메서드가 존재하지 않으므로 부모 클래스인 Lecture의 stats 메서드를 실행
  - Lecture 클래스의 stats 메서드에서 self참조가 가리키는 객체가 GradeLecture이다.
  - GradeLecture 클래스에서 getEvaluationMethod메서드를 실행 후 동적 메서드 탐색이 종료

- self 전송은 자식 클래스에서 부모 클래스 방향으로 진행되는 동적 메서드 탐색 경로를 다시 self 참조가 가리키는 원래의 자식 클래스로 이동시킨다.
- 최악의 경우에는 실제로 실행될 메서드를 이해하기 위해 상속 게층 전체를 훑어가며 코드를 이해해야 한다.
<img width="479" alt="image" src="https://github.com/JSON-loading-and-unloading/Object-Study/assets/86006389/8c356aa5-72d3-4be1-8261-8ebea28b0b67">


### 이해할 수 없는 메시지

- 상속 계층의 정상에 오고 나서야 자신이 메시지를 처리할 수 없다는 사실을 알게 된다면 ? 
- 정적 타입 언어와 이해할 수 없는 메시지
  - 컴파일타임에 자신이 이해할 수 없는 메시지인지 파악해서 컴파일 에러를 발생시킨다.
  - 유연성이 부족하지만 안정적이다.
- 동적 타입 언어와 이해할 수 없는 메시지
  - 실제로 코드를 실행해보기 전에는 메시지 처리 가능 여부를 판단할 . 수없다.
  - 이해할 수 없는 메시지인 것을 발견하면 self 참조가 가리키는 현재 객체에게 이해할 수 없다는 메시지를 전송
  - 최상위 클래스에서 최종적으로 예외가 던져진다.
  - 동적 타입 언어에선 이해할 수 없는 메시지에 응답할 수 있는 메서드를 구현할 . 수있다/
  - 이러한 유연성은 코드를 이해하고 수정하기 어렵게하고 디버깅을 복잡하게 하기도 한다.

### self 대 super


- 자식 클래스에서 부모 클래스의 인스턴스 변수나 메서드에 접근하기 위해 super 참조를 사용
- super 참조의 용도는 부모 클래스에 정의된 메서드를 실행하기 위한 것이 아니다.
   - 지금 이클래스의 부모 클래스에서부터 메서드 탐색을 시작하는 것이다.
   - 반드시 부모 클래스에 위치하지 않아도 되고 그 메서드의 조상 클래스 중 어딘가에 있기만 하면 된다.
   - super 참조를 통해 메시지를 전송하는 것은 인스턴스에게 메시지를 전송하는 것처럼 보이기 때문에 super참조라고 부른다.
- self 전송은 메시지를 수신하는 객체의 클래스에 따라 메서드를 탐색할 시작 위치를 동적으로 결정한다.
  - self 전송은 어떤 클래스에서 메시지 탐색이 시작될지 알지 못한다.

## 05 상속 대 위임

### 위임과 self 참조

> 상속은 동적으로 메서드를 탐색하기 위해 현재의 실행 문맥을 가지고 있는 self 참조를 전달한다.   
> 이 객체들 사이에서 메서드를 전달하는 과정은 자동으로 이루어지며, 이를 자동적인 메시지 위임이라고 한다.

- 상속
  - 상속 계층 내의 객체들은 self 참조를 공유한다.
  - 메서드를 탐색할 때 자식과 부모 클래스 인스턴스가 동일한 self 참조를 사용한다.
- 위임
  - 객체가 받은 메시지를 다른 객체에게 전달해 처리하는 것을 의미한다.
  - 위임을 할 때는 현재 실행 문맥을 나타내는 self 참조를 인자로 함께 전달한다.

-> 위임은 객체 사이의 동적인 연결 관계를 이용해 구현하는 방법



### 프로토타입 기반의 객체지향 언어

- 클래스가 존재하지 않고 오직 객체만 존재하는 프로토타입 기반의 객체지향 언어에서 상속을 구현하는 유일한 방법은 객체 사이의 위임을 이용
- 위임을 통한 상속: 다른 객체에게 메시지를 위임해 self참조를 자동으로 전달
- 자바스크립트의 Prototype:
  - 자바스크립트는 객체들이 프로토타입을 통해 연결되어 있으며, 이 체인을 통해 메시지(메서드 호출 등)를 위임하여 상속과 유사한 특성을 나타낸다.
  - prototype으로 연결된 객체들의 체인을 거슬러 올라가며 자동적으로 메시지에 대한 위임을 처리한다.
- 클래스 기반 언어:
  - 상속과 동일하게 객체 사이에 self 참조가 전달된다.

- 객체지향은 객체를 지향하는 것
  - 클래스가 없이도 객체 사이의 협력 관계를 구축하는 것은 가능
  - 상속 없이도 다형성을 구현하는 것이 가능
  - 중요한 것은 클래스 기반의 상속과 객체 기반의 위임 사이에 기본 개념과 메커니점을 공유한다는 것
