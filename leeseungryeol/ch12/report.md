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


