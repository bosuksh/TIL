2020-08-08

# 이펙티브 자바 Item13

# Item13.clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(아이템20)지만, 그 목적을 제대로 이루지 못했다. 

Cloneable의 정의를 보자. 보다시피 아무것도 정의되지 않았다.

```java
package java.lang;

/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * {@code Object.clone} (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the {@code clone} method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   1.0
 */
public interface Cloneable {
}
```

메서드 하나 없는 Cloneable 인터페이스는 무슨 일을 할까?

이 인터페이스는 Object의 protected 메소드인 **clone**의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드를 하나하나 복사한 객체를 반환하며, 구현하지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다. 

**실무에서는  Cloneable을 구현한 클래스는 clone 메소드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.**

그러나 이 기대를 만족시키다가 깨지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생할 수도 있다. 

다음은 Object 명세에서 가져온 clone 메소드의 일반 규약이다. 쫌 허술하다.

```markdown
이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라다를 수 있다. 
일반적인 의도는 다음과 같다. 어떤 객체 x에 대해서 다음 식은 참이다.

x.clone() != x

또한 다음 식도 참이다.

x.clone().getClass() = x.getClass()

하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.

x.clone().equals(x)

관례상, 이 메소드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한)
모든 상위 클래스가 이 관례를 따른다면 다음 식은 항상 참이다.

x.clone().getClass() = x.getClass()

관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중
하나 이상을 반환전에 수정해야 할 수도 있다. 
```

---

제대로 동작하는 clone메소드를 가진 상위 클래스를 상속해 Cloneable을 구현해보자.

### 1. 불변 클래스나 일반 타입을 참조할 때 clone

---

먼저 super.clone을 호출한다. 그렇게 얻은 객체는 원본의 완벽한 복제본일 것이다. 

모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽하다. 그러나 쓸데없는 복사를 지양한다는 관점에서 불변 클래스는 굳이 clone 메소드를 제공하지 않는 게 좋다. 

[아이템10](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava10.md)에서 구현한 PhoneNumber 클래스의 clone메소드를 다음처럼 구현할 수 있다. 

```java
@Override
public PhoneNumber clone() {
    try {
	return (PhoneNumber) super.clone();
    } catch(CloneNotSupportedException e) {
	throw new AssertionError(); //일어날 수 없는 일이다.
    }
}
```

이 메소드가 동작하게 하려면 PhoneNumber의 클래스 선언에 Cloneable을 구현한다고 추가해야 한다. 

super.clone 호출을 `try-catch`으로 감싼 이유는 Object의 clone 메소드가 검사 예외인 CloneNotSupportedException을 던지도록 선언되었기 때문이다. PhoneNumber가 Cloneable을 구현하니, 우리는 super.clone이 성공할 것임을 안다. 이 거추장스러운 코드는 CloneNotSuportedException이 사실 비검사 예외였어야 했다는 신호다.(아이템71)

### 2-1 가변 클래스를 참조할때 clone

---

만약 clone의 구현이 가변 클래스를 참조하면 문제가 생긴다. [아이템7](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava7.md)에서 구현한 **Stack** 클래스를 예로 들어보자.

clone 메소드가 단순히 super.clone의 결과를 그대로 반환하면 어떻게 될까?

반환된 size필드는 올바른 값을 갖겠지만, element 필드는 원본 stack인스턴스와 똑같은 배열을 참조할 것이다. 

**다시말해, 원본을 수정하면 복제본도 수정되고 복제본을 수정하면 원본도 수정되어 불변식을 해친다.**

생성자를 호출한다면 이런 상황을 발생하지 않는다. 

**즉, clone는 사실상 생성자와 같은 효과를 내야한다. clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.** 

이 사실을 알고 Stack 클래스의 clone메소드를 작성해보면 아래와 같다.

 

```java
@Override
public Stack clone() {
    try {
	Stack result = (Stack)super.clone();
	result.elements = elements.clone();
	return result;
    } catch (CloneNotSupportedException e) {
	throw new AssertionError();
    }
}
```

> 배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 따라서, 배열을 복사할때 clone 메소드를 권장한다. 
사실 clone의 기능을 제대로 사용할 수 있는 유일한 예다.

한편, elements 필드가 final로 선언되었다면 clone을 사용할 수 없다. 

**Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.**

그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final한정자를 제거해야 할 수도 있다. 

### 2-2 해시테이블용 clone

간단한 해시테이블을 구현한다고 할 때, 재귀적으로 참조한 clone을 구현한다고 해도 문제가 발생한다. 

```java
public class HashTable implements Cloneable{
  private Entry[] buckets = ...;
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }
  @Override
  protected Object clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    }catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

이렇게 구현하게 되면  복제본은 자신만의 버킷 배열을 갖는다. 그러나 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 둘다 문제가 발생할 수 있다. 

이걸 해결하기 위해서는 각 버킷을 구성하는 연결 리스트를 복사해야한다.

```java
public class HashTable implements Cloneable{
  private Entry[] buckets = ...;

  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
    //엔트리가 가리키는 연결리스트를 재귀적으로 복
    Entry deepCopy() {
      return new Entry(key,value,next == null? null: next.deepCopy());
    }
  }
  

  @Override
  protected Object clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for(int i = 0; i< buckets.length; i++) {
          if(buckets[i] != null) {
              result.buckets[i] = buckets[i].deepCopy(); 
          }
      }
      return result;
    }catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

HashTable의 clone 메소드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순환하면서 비지 않은 각 버킷에 대해 깊은 복사를 수행한다.

이 때, Entry의 deepCopy 메소드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 

그러나 이 방법은 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로우를 일으킬 위험이 있다. 대신, 반복문을 이용해서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for(Entry p = result; p.next != null, p = p.next) {
	p.next = new Entry(p.next.key, p.next.value, p.next.next);
	return result;
    }
}
```

### 2-3. 가변 객체를 복제하는 마지막 clone

먼저 super.clone으로 얻은 객체의 모든 필드를 초기화한다. 

그리고 원본 객체의 상태를 다시 생성하는 고수준 메소드를 호출한다. 
예를 들어, HashTable에서 버킷을 새로운 버킷으로 초기화 한 후 원래 있던 값을 똑같이 put(key, value)를 해준다. 

그러나 문제점은 속도가 느려지게 된다.

---

이외에도 생각해야할게 몇 개 더 있다.

public인 clone 메소드에서는 throws절을 없애야한다. 검사 예외를 던지지 않아야 그 메소드를 사용하기 편하기 때문이다.(아이템71)

또한 상속용 클래스에서는 Cloneable을 구현해서는 안된다. 다음과 같은 방법으로 퇴화시켜둬야한다. 

```java
@Override
public final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메소드 역시 적절히 동기화 해줘야한다(아이템78)

요약하면, Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다. 

이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다. 

이 메소드는 전부 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다. 

내부의 숨어있는 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체를 가리키게 해야 한다. 

그런데 이 모든 작업이 꼭 필요할까? 

Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야한다. 

**그렇지 않은 상황에서는 복사 생성자와 복사 팩토리라는 더 나은 객체 복사 방식을 제공할 수 있다.**

복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인스턴스 인수로 받는 생성자를 말한다. 

```java
public Yum(Yum yum) { ... };
```

복사 팩토리는 복사 생성자를 모방한 정적 팩토리이다.([아이템1](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava1.md))

```java
public static Yum newInstance(Yum yum) {... };
```

복사 생성자와 복사 팩토리는 해당 클래스가 구현한 '인터페이스'타입의 인스턴스를 인수로 받을 수 있다. 예컨대 관례성 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다. 

이들을 이용하면 원본의 구현타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

예컨대 HashSet 객체 s를 TreeSet 타입으로 복제할 수 있다.
