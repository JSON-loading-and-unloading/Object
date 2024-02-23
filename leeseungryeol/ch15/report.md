![KakaoTalk_20240223_120506817_03](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/33773af6-553c-4cf6-aba1-3ebf2138106d)<h1>디자인 패턴과 프레임 워크</h1>

<h2>디자인 패턴과 설계 재사용</h2>

<h3>패턴 분류</h3>

1. 아키텍처 패턴 : 디자인 패턴 상위, 미리 정의된 서브 시스템들을 제공하고, 각 서브시스템들의 책임을 정의하며, 서브시스템들 사이의 관계를 조직화하는 규칙과 가이드라인을 포함
2. 분석 패턴 : 도메인 내의 개념적인 문제를 해결하는데 초점을 맞춘다. 업무 모델링 시에 발견되는 공통적인 구조를 표현하는 개념들의 집합
3. 디자인 패턴 : 특정한 설계 문제를 해결하는 목적으로 하며, 프로그래밍 언어나 프로그래밍 패러다임에 독립적이다.
4. 이디엄 : 주어진 언어의 기능을 사용해 컴포넌트, 혹은 컴포넌트 간의 특정 측면을 구현하는 방법을 서술



<h3>패턴과 책임-주도 설계</h3>


![KakaoTalk_20240223_120506817_03](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/96cd96f7-81d2-4bc5-93cb-6102dd96ac22)

Composite 패턴</br>
 패턴의 구성 요소는 클래스가 아니라 '역할'이다.</br>
 패턴 구성요소 : Component, Composite, Leaf</br></br>

 OverlappedDiscountPolicy는 Composite역할 수행</br>
 AmountDiscountPolicy, PercentDiscountPolicy는 Leaf역할 수행</br></br>

※폴더(디렉토리) 안에는 파일이 들어 있을수도 있고 파일을 담은 또 다른 폴더도 들어있을 수 있다. 이를 복합적으로 담을수 있다 해서 Composite 객체라고 불리운다. </br>
※반면 파일은 단일 객체 이기 때문에 이를 Leaf 객체라고 불리운다. 즉 Leaf는 자식이 없다.</br>


<h3>캡슐화와 디자인 패턴</h3>

Strategy 패턴</br>

![KakaoTalk_20240223_120506817_02](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/9ee1650b-58a1-42e4-9ba8-71b3f9627254)


Strategy 목적 : 알고리즘의 변경을 캡슐화하는 것이고 이를 구현하기 위해 객체 합성을 이용</br></br>

영화에 적용될 할인 정책의 종류는 Movie가 참조하는 DiscountPolicy의 서브클래스가 무엇이냐에 따라 결정된다.</br>
Strategy패턴을 이용하면 Movie와 DiscountPolicy 사이의 결합도를 낮게 유지할 수 있기 때문에 런타임에 알고리즘을 변경할 수 있다.</br>


Template Method 패턴</br>


![KakaoTalk_20240223_120506817_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/ae6c93ce-47bf-4cb4-a5df-99366f3250a3)


Template Method 패턴 : 알고리즘을 캡슐화하기 위해 합성 관계가 아닌 상속 관계를 사용하는 것 (변경하지 않는 부분은 부모 클래스로, 변하는 부분은 자식 클래스로 분리함으로써 변경을 캡슐화 한다.)</br></br>

※추상 메소드를 이용해 변경을 캡슐화 한다.</br>
※합성보다는 결합도가 높은 상속을 사용</br>
calculateDiscountAmount 메서드가 바로 서브 클래스의 변경을 캡슐화하기 위해 사용되는 추상 메서드다. 부모 클래스의 calculateFee 메서드 안에서 추상 메서드인 calculateDiscountAmount를 호출하고 자식 클래스들이 이 메서드를 오버라이딩해서
변하는 부분을 구현한다는 것이 중요하다.</br>


Decorator 패턴</br>


![KakaoTalk_20240223_120506817](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/6e785807-211c-4c06-9b7b-bbe848d062f7)


 - Decorator 패턴은 객체의 행동을 동적으로 추가할 수 있게 해주는 패턴으로서 기본적으로 객체의 행동을 결합하기 위해 객체 합성을 사용한다.
 - Decorator 패턴은 선택적인 행동의 개수와 순서에 대한 변경을 캡슐화할 수 있다.










