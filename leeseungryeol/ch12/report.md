<h1>다형성</h1>


<h2>다형성</h2>

간단하게 말해서 다형성은 여러 타입을 대상으로 동작할 수 있는 코드를 작성할 수 있는 방법</br>

다형성 종류</br>

![KakaoTalk_Photo_2024-02-02-22-09-57](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/053e4f90-58c6-4f26-80b3-57a612daa0f9)

오버로딩 다형성 : 일반적으로 하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우를 가리킴</br>
강제 다형성 : 언어가 지원하는 자동적인 타입 변환이나 사용자가 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할 수 있는 방식</br>
매개변후 다형성 : 제네릭 프로그래밍과 관련이 높은데 클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식을 가리킨다.</br>
포함 다형성 : 메시지가 동일하더라도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력 </br></br>

포함 다형성을 구현하는 가장 일반적인 방법은 상속을 사용하는 것이다.</br>
=> 서브 타입 다형성이라고 부른다는 사실에서 예상할 수 있겠지만 포함 다형성을 위한 전제조건은 자식 클래스가 부모 클래스의 서브타입이어야 한다는 것이다.</br>
또한 상속의 진정한 목적은 코드 재사용이 아니라 다형성을 위한 서브타입 계층을 구축하는 것이다.</br></br>

포함 다형성을 위해 상속을 사용하는 가장 큰 이유는 상속이 클래스들을 계층으로 쌓아 올린 후 상황에 따라 적절한 메서드를 선택할 수 있는 메커니즘을 제공하기 때문이다.</br>

<h2>상속의 양면성</h2>

객체지향 프로그램을 작성하기 위해서는 항상 데이터와 행동이라는 두 가지 관점을 함께 고려해야 한다.</br></br>

상속을 이용하면 부모 클래스에서 정의한 모든 데이터를 자식 클래스의 인스턴스에 자동으로 포함시킬 수 있다. 이것이 데이터 관점의 상속이다.</br>
부모 클래스에서 정의한 일부 메서드 역시 자동으로 자식 클래스에 포함시킬 수 있다. 이것은이 행동 관점의 상속이다.</br>

```

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


```
Lecture lecture = new Lecture("객체지향 프로그래밍",
                                70,
                                Arrays.asList(81, 95, 75, 50, 45));
String evaluration = lecture.evaluate();

```

Lecture 클래스를 상속받으면 새로운 기능을 쉽고 빠르게 추가할 수 있다.</br>


```
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

```
public class Grade {
    private String name;
    private int upper,lower;

    private Grade(String name, int upper, int lower) {
        this.name = name;
        this.upper = upper;
        this.lower = lower;
    }

    public String getName() {
        return name;
    }

    public boolean isName(String name) {
        return this.name.equals(name);
    }

    public boolean include(int score) {
        return score >= lower && score <= upper;
    }
}

```

GradeLecture의 evaluate 메서드에서는 예약어 super를 이용해 Lecture 클래스의 evaluate 메서드를 먼저 실행한다.</br></br>

여기서 GradeLecture와 Lecture에 구현된 두 evaluate 메서드의 시그니처가 완전히 동일하다.</br>
부모 클래스와 자식 클래스에 동일한 시그니처를 가진 메서드가 존재할 경우 자식 클래스의 메서드 우선순위가 높다.</br>
이로 인해 자식 클래스의 메서드가 실행된다.</br></br>

average 메서드는 부모 클래스인 Lecture에 정의된 메소드와 시그니처가 달라 average 메서드를 대처하지 않는다.</br></br>

이처럼 상속을 사용하면 새로운 기능을 쉽고 빠르게 추가할 수 있다.</br>
(상속을 사용하는 일차적인 목표는 코드 재사용이 아니다.)</br>


<h3>데이터 관점의 상속</h3>

```
Lecture lecture = new Lecture("객체지향 프로그래밍",
                                70,
                                Arrays.asList(81, 95, 75, 50, 45));
String evaluration = lecture.evaluate();

```



```

Lecture lecture = new GradeLecture("객체지향 프로그래밍",
                                70,
                                Arrays.asList( new Grade("A", 100, 95),
                                              new Grade("B", 94, 80),
                                              new Grade("C", 79, 70),
                                              new Grade("D", 69, 50),
                                              new Grade("F", 49, 0),

                                ),
                                Arrays.asList(81, 95, 75, 50, 45));


```

데이터 관점은 상속을 인스턴스 관점으로 바라볼 때는 개념적으로 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스가 포함되는 것으로 생각하는 것이 유용한다.</br>
(상속은 자식 클래스의 인스턴스 안에 부모 클래스의 인스턴스를 포함하는 것으로 볼 수 있다.)</br></br>

<h3>행동 관점의 상속</h3>

행동 관점의 상속은 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것을 의미한다.</br></br>


메서드의 경우에는 동일한 클래스의 인스턴스끼리 공유가 가능하기 때문에 클래스에 한 번만 메모리에 로드하고 각 인스턴스별로 클래스를 가리키는 포인터를 갖게 하는 것이 경제적이다.</br>
실제로도 두 개의 인스턴스가 있지만 클래스는 단 하나만 메모리에 로드된다.</br>


![KakaoTalk_20240203_002801041](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/ff4dfb68-aa1f-446d-af15-c22fda1fc693)


객체로부터 메시지를 수신한 객체는 class 포인터로 연결된 자신의 클래스에서 적절한 메서드가 존재하는지를 찾는다.</br>
만약 존재하지 않는다면 클래스의 parent 포인터를 따라 부모 클래스를 차례대로 훑어가면서 적절한 메서드가 존재하는지를 검색한다.</br></br>

이처럼 부모클래스로부터 상속을 받았을 경우 런타임에 시스템이 자식 클래스에 정의되지 않은 메서드가 있을 경우 이 메서드를 부모 클래스 안에서 탐색한다.</br>


<h2>업캐스팅과 동적 바인딩</h2>

```

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

```

Professor professor = new Professor( "다익스트라",
                                "객체지향 프로그래밍",
                                70,
                                Arrays.asList(81, 95, 75, 50, 45));
String evaluration = lecture.evaluate();

```



```

Professor professor = new Professor( "다익스트라",
                                "객체지향 프로그래밍",
                                70,
                                Arrays.asList( new Grade("A", 100, 95),
                                              new Grade("B", 94, 80),
                                              new Grade("C", 79, 70),
                                              new Grade("D", 69, 50),
                                              new Grade("F", 49, 0),

                                ),
                                Arrays.asList(81, 95, 75, 50, 45));                 // 업캐스팅


String statistics = professor.compileStatistics();                                  // 동적 바인딩

```

<h3>업캐스팅</h3>

부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용하더라도 메시지를 처리하는 데에 아무런 문제가 없으며, 컴파일러는 명시적인 타입 변환 없이도 자식 클래스가 부모 클래스를 대체할 수 있게 허용한다.</br>
이를 업캐스팅이라고 한다.</br></br>

반대로 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해서는 명시적인 타입 캐스팅이 필요한데 이를 다운 캐스팅이라고 부른다.</br>

```

Lecture lecture = new GradeLecture();
GradeLecture GradeLecture = (GradeLecture)lecture;

```


<h3>동적 바인딩</h3>

bar 함수를 호출하는 구문이 나타난다면 실제로 실행되는 코드는 바로 bar함수다.</br>
다시말해 코드를 작성하는 시점에 호출될 코드가 결정된다.</br>
이처럼 컴파일타임에 호출할 함수를 결정하는 방식을 정적 바인딩, 초기 바인딩, 컴파일타임 바인딩이라고 부른다.</br></br>

객체지향 언어에서는 메시지를 수신했을 때 실행될 메서드가 런타임에 결정된다.</br>
foo.bar()를 읽는 것으로는 실행되는 bar가 어떤 클래스의 메서드인지를 판단하기 어렵다.</br>
이처럼 실행될 메서드를 런타임에 결정하는 방식을 동적 바인딩 또는 지연 바인딩이라고 부른다.</br></br>

<h2>동적 메서드 탐색과 다형성</h2>

객체지향 시스템은 다음 규칙에 따라 실행할 메서드를 선택한다.</br></br>

- 메시지를 수신한 객체는 먼저 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사한다. 존재하면 메서드를 실행하고 탐색을 종료한다.</br>
- 메서드를 찾기 못했다면 부모 클래스에서 메서드 탐색을 계속한다. 이 과정은 적합한 메서드를 찾을 때까지 상속 계층을 따라 올라가며 계속된다.</br>
- 상속 계층의 가장 최상위 클래스에 이르렀지만 메서드를 발견하지 못한 경우 예외를 발생시키며 탐색을 중단한다.</br>

<h3>자동적인 메시지 위임</h3>

<h4>메서드 오버라이딩</h4>
![KakaoTalk_20240203_155537765](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/218d1fe1-47f4-46e4-a97f-01f60385735d)


위 그림은 Lecture 인스턴스에게 evalaute 메시지를 전송한 시점의 메모리 상태를 나타냈다.</br>
런타임에 자동으로 self 참조가 메시지 수신 객체를 가리키도록 설정된다.</br></br>

![KakaoTalk_20240203_155537765_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/882ddfe1-078d-4ca7-9291-7e4ec3d2f45b)

Lecture클래스의 evaluate메서드와 시그니처가 동일한 메서드를 자식 클래스인 GradeLecture에서 재정의하고 있다.</br>
동적 메서드 탐색은 self 참조가 가리키는 객체의 클래스인 GradeLecture에서 시작되고 GradeLecture 클래스 안에 evaluate 메서드가 구현돼 있기 때문에 먼저 발결된 메서드가 실행되는 것이다.</br></br>


<h4>메서드 오버로딩</h4>

Lecture클래스의 average 메서드와 시그니처가 다른 메서드를 자식 클래스인 GradeLecture에서 재정의하고 있다.</br>
이 경우 self는 GradeLecture에서 해당 함수를 못찾을 경우 Lecture로 올라간다.</br>


<h3>동적인 문맥</h3>

위처럼 
동일한 코드라고 하더라도 self 참조가 가리키는 객체가 무엇인지에 따라 메서드 탐색을 위한 상속 계측의 범위가 동적으로 변한다.
따라서, self 참조가 가리키는 객체의 타입을 변경함으로써 객체가 실행될 문맥을 동적으로 바꿀 수 있다.

```
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

    public String stats() {
        return String.format("Title: %s, Evaluation Method: %s",
                title, getEvaluationMethod());
    }

    public String getEvaluationMethod() {
        return "Pass or Fail";
    }
}

```

Lecture 클래스에서 stats 메서드 안에서 자신의 getEvaluationMethod 메서드를 호출한다.
</br>
다시 한 번, 현재 클래스의 메서드를 호출하는 것이 아니라 현재 객체에서 메시지를 전송하는 것이다.</br></br>

현재 객체란? self 참조가 가리키는 객체다.</br>
이 객체는 처음에 stats 메시지를 수신했던 그 객체다.</br>
이처럼 self 참조가 가리키는 자기 자신에게 메시지를 전송하는 것을 self 전송이라고 부른다.</br>
=> self 전송을 이해하기 위해서는 self 참조가 가리키는 바로 그 객체에서부터 메시지를 탐색을 다시 시작한다는 사실을 기억하자!</br>



![KakaoTalk_20240206_175854799](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/c50e7c72-8f5d-4b61-adc3-564fc14f94da)


위 이미지를 보면 GradeLecture에서 stats 메서드를 찾고 Lecture 클래스에서 getEvaluatetionMethod메서드를 찾는데 Lecture 클래스가 아닌 self 가 가리키는 GradeLecture 클래스에서 먼저 찾는다.</br>
하지만 이로 인해 최악의 경우네는 실제로 실행될 메서드를 이해하기 위해 상속 계층 전체를 훑어가며 코드를 이해해야하는 상황이 발생할 수도 있다.</br>

<h3>이해할 수 없는 메시지</h3>

이해할 수 없는 메시지를 처리하는 방법은 프로그래밍 언어가 정적 타입 언어에 속하는지, 동적 타입 언어에 속하는지에 따라 달라진다.</br></br>

<h4>정적 타입 언어와 이해할 수 없는 메시지</h4>

코드를 컴파일할 때 상속 계층 안의 클래스들이 메시지를 이해할 수 있는지 여부를 판단</br>
따라서 상속 계층 전체를 탐색한 후에도 메시지를 처리할 수 있는 메서드를 발견하지 못했다면 컴파일 에러를 발생시킨다.</br>

<h4>동적 타입 언어와 이해할 수 없는 메시지</h4>

동적 타입 언어 역시 메시지를 수신한 객체의 클래스부터 부모 클래스의 방향으로 메서드를 탐색한다.</br>
차이점이라면 동적 타입 언어에서는 컴파일 단계가 존재하지 않기 때문에 실제로 코드를 실행해보기 전에는 메시지 처리 가능 여부를 판단할 수 없다.</br>

![KakaoTalk_20240206_175854799_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/5006a470-0232-453b-9f44-70ae48b1a7f2)


이 또한 NomethodError 예외를 던진다.</br></br>

이해할 수 없는 메시지를 처리할 수 있는 동적 타입 언어는 좀 더 순수한 관점에서 객체지향 패러다임을 구현한다고 볼 수 있다.</br>
협력을 위해 메시지를 전송하는 객체를 메시지를 전송하는 객체는 메시지를 수신한 객체의 내부 구현에 대해서는 알지 못한다.</br>
단지 객체가 메시지를 처리할 수 있다고 믿고 메시지를 전송할 뿐이다.</br>

그러나 동적 타입 언어의 이러한 동적인 특성과 유연성은 코드를 이해하고 수정하기 어렵게 만들뿐만 아니라 디버깅 과정을 복잡하게 만들기도 한다.</br>
정적 타입 언어에서는 이런 유연성이 부족하지만 좀 더 안정적이다. 모든 메시지는 컴파일 타임에 확인되고 이해할 수 없는 메시지는 컴파일 에러로 이어진다.</br>
=> 실행 시점에 오류가 발생할 가능성을 줄임으로써 프로그램이 좀 더 안정적으로 실행될 수 있다.
</br>

<h3>self 대 super</h3>

self 참조의 가장 큰 특징은 동적이다.</br>
자식 클래스에서 부모 클래스의 인스턴스 변수나 메서드에 접근하기 위해 사용할 수 있는 super 참조라는 내부 변수를 제공한다.</br>

```
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

    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}

```

대부분의 사람들은 super.evaluate()라는 문장이 단순히 부모 클래스의 evaluate()를 호출한다고 생각한다.</br>
하지만 super.evaluate()에 의해 호출되는 메서드는 부모 클래스의 메서드가 아니라 더 상위에 위치한 조상 클래스의 메서드일 수도 있다.</br>


```
public class FormattedGradeLecture extends GradeLecture {
    public FormattedGradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, grades, scores);
    }

    public String formatAverage() {
        return String.format("Avg: %1.1f", super.average());
    }
}

```

사실 super 참조의 용도는 부모 클래스에 정의된 메서드를 실행하기 위한 것이 아니다.</br>
super 참조의 정확한 의도는 '지금 이 클래스의 부모 클래스에서부터 메서드 탐색 시작하세요' 다.</br>
=> super 참조를 통해 실행하고자 하는 메서드가 반드시 부모 클래스에 위치하지 않아도 되는 유연성을 제공한다.</br></br>

부모 클래스의 메서드를 호출하는 것과 부모 클래스에서 메서드 탐색을 시작하는 것은 의미가 매우 다르다.</br>
부모 클래스의 메서드를 호출한다는 것은 그 메서드가 반드시 부모 클래스 안에 정의돼 있어야 한다는 것을 의미한다.</br>
그에 비해 부모 클래스에서 메서드 탐색을 시작한다는 것은 그 클래스의 조상 어딘가에 그 메서드가 정의돼 있기만 하면 실행할 수 있다는 것을 의미한다.</br>

<h2>상속 대 위임</h2>

```
class Lecture
  def initialize(name, scores)
    @name = name
    @scores = scores
  end

  def stats(this)
    "Name: #{@name}, Evaluation Method: #{this.evaluationMethod(this)}"
  end

  def getEvaluationMethod()
    "Pass or Fail"
  end
end

class GradeLecture
  def initialize(name, canceled, scores)
    @parent = Lecture.new(name, scores)
    @canceled = canceled
  end

  def stats(this)
    @parent.stats(this)
  end

  def getEvaluationMethod()
    "Grade"
  end
end

lecture = Lecture.new("OOP", [1,2,3])
puts lecture.stats(lecture)

grade_lecture = GradeLecture.new("OOP", false, [1,2,3])
puts grade_lecture.stats(grade_lecture)

```

```
grade_lecture = GradeLecture.new("OOP", false, [1,2,3])
puts grade_lecture.stats(grade_lecture)

```

1. GradeLecture는 인스턴스 변수인 @parent에 Lecture의 인스턴스를 생성해서 저장한다.
2. GradeLecture의 stats 메서드는 추가적인 작업 없이 @parent에게 요청을 그대로 전달한다.
3. GradeLecture의 getEvaluationMethod메서드는 stats 메소드처럼 요청을 @parent에 전달하지 않고 자신만의 방법으로 메서드를 구현하고 있다.
4. GradeLecture의 stats 메서드는 인자로 전달된 this를 그대로 Lecture의 stats메서드에 전달한다.

GradeLecture의 stars메서드는 메시지를 직접 처리하지 않고 Lecture의 stats 메서드에게 요청을 전달한다는 것에 주목해보자.</br>
이처럼 자신이 수신한 메시지를 다른 객체에게 동일하게 전달해서 처리를 요청하는 것을 위임이라고 한다.</br></br>

위임은 본질적으로는 자신이 정의하지 않거나 처리할 수 없는 속성 또는 메서드의 탐색 과정을 다른 객체로 이동시키기 위해 사용한다.</br>
이를 위해 위임은 항상 현재의 실행 문맥을 기리키는 self 참조를 인자로 전달한다.</br>
이것이 self 참조를 전달하지 않은 포워딩과 위임의 차이점이다.</br>





