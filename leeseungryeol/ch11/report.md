<h1>합성과 유연한 설계</h1>

상속이 부모 클래스와 자식 클래스를 연결해서 부모 클래스의 코드를 재사용하는 데 비해</br>
합성은 전체를 표현하는 객체가 부분을 표현하는 객체를 포함해서 부분 객체의 코드를 재사용한다.</br></br>

상속을 제대로 활용하기 위해서는 부모 클래스의 내부 구현에 대해 상세하게 알아야 하기 때문에 자식 클래스와 부모 클래스 사이의 결합도가 높아질 수 밖에 없다.</br>
합성은 구현에 의존하지 않는다는 점에서 상속과 다르다. 합성은 내부에 포함되는 객체의 구현이 아닌 퍼블릭 인터페이스에 의존하다.</br>
따라서 합성을 이용하면 객체의 내부 구현이 변경되더라도 영향을 최소화할 수 있기 때문에 변경에 더 안정적인 코드를 얻을 수 있게 된다.</br>
=> 상속 대신 합성을 사용하면 변경하기 쉽고 유연한 설계를 얻을 수 있다.</br>

<h2>상속을 합성으로 변경하기</h2>

합성을 사용하여 상속이 초래하는 세 가지 문제점 해결</br>


1. 불필용한 인터페이스 상속 문제</br>

~~~

public class Properties {
    private Hashtable<String, String> properties = new Hashtable <>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}

~~~


- 상속 관계를 제거하고 Hashtable을 properties의 인스턴스 변수로 포함시키면 합성 관계로 변경할 수 있다.
- 클라이언트는 오직 Properties에서 정의한 오퍼레이션만 사용할 수 있다.
- Properties클라이언트는 모든 타입의 키와 값을 저장할 수 있는 Hashtable의 오퍼레이션을 사용할 수 없기 때문에 String 타입의 키와 값만 허용하는 Properties의 규칙을 어길 위험성이 사라진다.


2. 메서드 오버라이딩의 오작용 문제</br>

~~~

public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    @Override public boolean remove(Object o) {
        return set.remove(o);
    }

    @Override public void clear() {
        set.clear();
    }

    @Override public boolean equals(Object o) {
        return set.equals(o);
    }

    @Override public int hashCode() {
        return set.hashCode();
    }

    @Override public Spliterator<E> spliterator() {
        return set.spliterator();
    }

    @Override public int size() {
        return set.size();
    }

    @Override public boolean isEmpty() {
        return set.isEmpty();
    }

    @Override public boolean contains(Object o) {
        return set.contains(o);
    }

    @Override public Iterator<E> iterator() {
        return set.iterator();
    }

    @Override public Object[] toArray() {
        return set.toArray();
    }

    @Override public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }

    @Override public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }

    @Override public boolean retainAll(Collection<?> c) {
        return set.retainAll(c);
    }

    @Override public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }
}

~~~

HashSet에 대한 구현 결합도는 제거하면서도 퍼블릭 인터페이스는 그대로 상속 받을 수 있는 방법은 자바의 인터페이스를 사용할 수 있다.</br>
HashSet은 Set 인터페이스를 실체화하는 구현체 중 하나이며, InstrumentedHashSet이 제공해야하는 모든 오퍼레이션들은 Set 인터페이스에 정의돼 있다.</br>


3. 부모 클래스와 자식 클래스의 동시 수정 문제</br>

~~~

public class PersonalPlaylist {
    private Playlist playlist = new Playlist();

    public void append(Song song) {
        playlist.append(song);
    }

    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}

~~~


Playlist와 PersonalPlaylist를 함께 수정해야 하는 문제가 해결되지는 않는다.
</br>
그렇다하더라도 합성을 이용하면 향후에 Playlist의 내부 구현을 변경하더라도 파급효과를 최대한 PersonalPlaylist 내부로 캡슐화할 수 있다.
</br>
