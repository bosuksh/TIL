2020-08-14

# Item18. 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘 못 사용하면 오히려 위험해질 수 있다.

*이번 아이템에서 논하는 문제는 인터페이스 상속이 아닌 구현 상속(다른 클래스가 다른 클래스를 상속)을 의미한다.*

## 1. 메소드 호출과 달리 상속은 캡슐화를 깨뜨린다.

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.  상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며 이것이 하위 클래스를 건드리지 않았음에도 문제가 발생할 수 있다. 

잘못된 예를 보자. 

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	//추가된 원소의 수
	private int addCount = 0;

	public InstrumentedHashSet() {
	}

	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap,loadFactor);
	}

	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}

}
```

위의 클래스는 HashSet을 상속 받았고 addCount라는 필드를 통해 몇 개의 원소가 지금까지 추가되었는지를 셀 수 있도록 클래스를 만들었다. 

여기서 아래의 코드를 실행시키면 3을 예상하지만 실제 값은 6이 나온다.

```java
// expected 3, real 6
InstrumentedHashSet<String> s = new InstrumentedHashSet();
s.addAll(List.of("1","2","3"));
```

6이 나오는 이유는 HashSet의 addAll은 add함수를 사용하기 때문이다. 즉, 우리가 만든 클래스에서도 역시 addAll은 상속받은 add를 사용하고 그 결과 값이 두 번씩 더해지는 것이다. 

그러면 addAll 메소드를 재정의하지 않으면 문제를 고칠 수 있을리라고 생각되지만, 지금 당장은 고칠 수 있지 몰라도 다음 릴리스때 또 그렇게 사용하라는 법은 없다. 

하위 클래스가 깨지기 쉬운 이유는 더 있다. 다음 릴리스때 상위 클래스에 새로운 메소드를 추가한다 했을 때 하위 클래스는 재정의하지 못하게 될 것이고 그로 인해 문제가 발생할 수도 있다.

클래스를 확장하더라도 메소드 재정의 대신 새로운 메소드를 추가하면 된다고 생각하지만, 이 것 역시 완전히 안전하다고 할 수 없다. 

왜냐하면 하위 클래스에 추가한 메소드가 다음 릴리스 상위버전에서 추가한 메소드와 완전히 같을 수도 있기 때문이다. 

## 해결법(composition)

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다. 

이러한 설계를 `컴포지션`이라고 한다.

새 클래스의 인스턴스 메소드들은 기존 클래스의 대응하는 메소드를 호출해 결과를 반환한다. 이 방식을 forwarding이라고 하며, 새 클래스의 메소드들을 forwarding method라고 부른다.

이 결과로 새로운 클래스는 새로운 메소드가 추가되더라도 영향을 받지 않는다. 

구체적인 예시는 위에서 제시했던 InstrumentedSet이다.

```java
public class InstrumentedHashSet<E> extends ForwardingSet<E> {
	//추가된 원소의 수
	private int addCount = 0;

	public InstrumentedHashSet(Set<E> s) {
		super(s);
	}

	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}

}
```

아래 클래스는 전달 클래스(Forwarding Class)이다. 

```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;

  public ForwardingSet(Set<E> s) { this.s = s; }

  @Override
  public int size() { return s.size(); }

  @Override
  public boolean isEmpty() { return s.isEmpty(); }

  @Override
  public boolean contains(Object o) { return s.contains(o); }

  @Override
  public Iterator<E> iterator() { return s.iterator(); }

  @Override
  public Object[] toArray() { return s.toArray(); }

  @Override
  public <T> T[] toArray(T[] ts) { return s.toArray(ts); }

  @Override
  public boolean add(E e) { return s.add(e); }

  @Override
  public boolean remove(Object o) { return s.remove(o); }

  @Override
  public boolean containsAll(Collection<?> collection) { return s.containsAll(collection); }

  @Override
  public boolean addAll(Collection<? extends E> collection) { return s.addAll(collection); }

  @Override
  public boolean retainAll(Collection<?> collection) { return s.retainAll(collection); }

  @Override
  public boolean removeAll(Collection<?> collection) { return s.removeAll(collection); }

  @Override
  public void clear() { s.clear(); }

  @Override
  public int hashCode() { return s.hashCode(); }

  @Override
  public boolean equals(Object obj) { return s.equals(obj); }

  @Override
  public String toString() { return s.toString(); }
}
```

InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다. 구체적으로 Set 인터페이스를 구현했고 Set 인스턴스를 인수로 받는 생성자를 하나 제공한다. 

**핵심은 Set의 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.** 

상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해야 한다. 

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

**다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet같은 클래스를 래퍼 클래스라고 한다. 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다.** 

**컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다.**



------



### 래퍼 클래스(Wrapper Class)

래퍼클래스는 단점이 거의 없다. 한 가지, 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는다. 

콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 콜백 때 사용한다.

내부 객체는 자기 자신을 감싸고 있는 래퍼의 존재를 모르니 자기 자신의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다. 

전달 메소드를 작성하는 게 지루하겠지만, 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수 있다. 

상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다. 

다르게 말하면 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야한다. *`(B is a A → A를 상속)`*

컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야할 지문을 소개한다. 

확장하려는 클래스의 API의 아무러 결함이 없는가?

결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파돼도 괜찮은가?