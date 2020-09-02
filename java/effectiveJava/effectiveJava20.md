2020-09-03

# Item20. 추상 클래스보다는 인터페이스를 우선하라.



### 추상 클래스는 추상 메소드가 들어있는 클래스를 말하고, 인터페이스는 추상 메소드로만 이루어진 클래스를 말한다.

그러면 왜 인터페이스를 사용하면 좋은지 알아보자. 

## 1. 인터페이스는 기존 클래스에도 손쉽게 구현해 넣을 수 있다.

추상 클래스를 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 하고, 자바는 단일 상속만 지원하니 추상 클래스는 새로운 타입을 정의하는 데 큰 제약을 받는다는 것이다.

그러나, 인터페이스는 몇 개가 되어도 implements로 구현해 넣을 수 있다. 

그래서 자바 플랫폼에 `Comparable`, `Iterable`, `AutoCloseable` 이 추가되었을 때 수많은 기존 클래스가 이 인터페이스를 구현한 채 릴리스됐다. 

## 2. 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.

믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입'외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.  

예컨대, `Comparable`은 자신을 구현하는 클래스의 인서튼서끼리는 순서를 정할 수 있다고 선언한 믹스인 인터페이스다. 

## 3. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다. 예를 들어, 가수(Singer) 인터페이스와 작곡가(Songwriter) 인터페이스가 있다고 해보자. 

```java
public interface Singer {
	AudioClip sing(Song s);   
}

public interface Songwriter {
	Song compose(int charPosition);
}
```

인터페이스를 이용하면 가수 클래스와 작곡 클래스를 모두 구현하는 클래스를 만들 수 있다. 거기에 덧붙여 새로운 메소드까지 추가하는 제 3의 인터페이스도 만들 수 있다.  

```java
public interface SingerSongwriter implements Singer, Songwriter {
	AudioClip strum();
	void actSenstive();
}
```

## 4. 래퍼 클래스 관용구([아이템18](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava18.md))와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해서 만든 클래스는 래퍼클래스보다 활용도가 떨어지고 깨지기는 더 쉽다. 

## 5. 인터페이스의 메소드 중 구현 방법이 명백한 것이 있다면, 디폴트 메소드로 제공해 프로그래머의 일감을 덜어줄 수 있다.

그러나 디폴트 메소드에도 제약이 있다. equals나 hashcode를 디폴트 메소드로 제공하면 안된다. 

또한 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다. 

---

한편 인터페이스와 추상 골격 구현(skeleton implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점 모두 취하는 방법도 있다. 

## 템플릿 메소드 패턴

인터페이스로 타입을 정의하고, 필요하면 디폴트 메소드도 몇 개 제공한다. 그리고 골격 구현 클래스는 나머지 메소드들을 구현한다. 

관례상 인터페이스 이름이 *Interface* 라면 그 골격 구현 클래스는 *AbstractInterface* 로 이름짓는다.

예를 보자. 다음은 List 구현체를 반환하는 정적 팩토리 메소드로, AbstractList를 골격 구현으로 활용했다. 

```java
static List<Integer> intArrayAsList(int[] a) {
	Objects.requireNonNull(a);
	
	//다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능
	//더 낮은 버저능ㄹ 사용한다면 <Integer>로 수정하자. 
	return new AbstractList<>() {
		@Override public Integer get(int i) {
			return a[i]; 오토박싱(아이템6)
		}
		@Override public Integer set(int i, Integer val) {
			int oldVal = a[i];
			a[i] = val;    // 오토언박싱
			return oldVal; // 오토박싱
		}
		@Override public int size() {
			return a.length;
		}
	}
}
```

## 골격 구현 작성 방법

1. 인터페이스를 잘 살펴 다른 메소드들의 구현에 사용되는 기반 메서드들을 선정한다. 
이 기반 메소드들은 골격 구현에서는 추상 메소드가 될 것이다. 
2. 기반 메소드들을 사용해 직접 구현할 수 있는 메소드는 모두 디폴트 메소드로 제공한다. 
3. 기반 메소드나 디폴트 메소드로 만들지 못한 메소드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메소드들을 작성해 넣는다. 

간단한 예를 살펴보자. 

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

  //변경 가능한 엔트리는 이 메소드를 반드시 재정의해야한다. 
  @Override
  public V setValue(V v) {
    throw new UnsupportedOperationException();
  }
  // Map.Entry.hashCode의 일반 규약 
  @Override
  public int hashCode() {
    return Objects.hashCode(getKey()) 
            ^ Objects.hashCode(getValue());
  }
  
  // Map.Entry.equals의 일반 규약 
  @Override
  public boolean equals(Object obj) {
   if (obj == this)
     return true;
   if (!(obj instanceof Map.Entry))
     return false;
   Map.Entry<?,?> e = (Map.Entry) obj;
   return Objects.equals(e.getKey(), getKey()) 
           && Objects.equals(e.getValue(), getValue());
  }

  @Override
  public String toString() {
    return getKey() + "=" + getValue() ;
  }
}
```

골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 [아이템19](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava19.md)에서 이야기한 설계와 문서화 지침을 모두 따라야 한다. 

**단순 구현은 골격 구현의 작은 변종으로, AbstractMap.SimpleEntry가 좋은 예다. 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다.**