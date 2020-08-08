2020-08-08

# Item14. Comparable을 구현할 지 고민하라.



이번에는 Comparable의 유일무이한 메소드인 compareTo를 알아보자. 

compareTo는 사실 Object의 메소드가 아니다. 그러나 성격은 두가지만 빼면 Object의 equals와 같다. 

다른 점은 compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다. 

Comparable을 구현한 클래스는 클래스의 인스턴스에 natural order가 있다는 뜻이다. 

그렇기 때문에 `Arrays.sort(a);` 로 정렬이 가능하다. 

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable을 구현하자. 

```java
public interface Comparable<T> {
	int compareTo(T t);
}
```

**compareTo 메서드의 일반규약은 다음과 같다.** 

```markdown
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 
같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면
ClassCastException을 던진다. 

 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호함수를 뜻하며, 표현식의 값이 음수,
0, 양수일 때, -1,0,1을 반환하도록 정의했다.

 * Comparable을 구현한 클래스는 모든 x,y에 대해서 
sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
(따라서 x.compareTo(y)가 예외를 던지면 y.compareTo(x)도 예외를 던져야한다.)

 * Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && 
y.compareTo(z) > 0) 이면 x.compareTo(z) > 0 이다.
 
 * Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면 
sgn(x.compareTo(z)) == sgn(y.compareTo(z))다.

 * 이번 권고가 필수는 아니지만 꼭 지키는게 좋다. (x.compareTo(y) == 0) == (x.equals(y))여야 한다.
Comparable을 구현하고 이 권고를 지키지 않은 모든 클래스는 사실을 명시해야 한다. 
다음과 같이 명시하면 적당할 것이다.

"주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."

```

compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 

비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 `TreeSet`과 `TreeMap`, 검색과 정렬 알고리즘을 활용한 `Collections`와 `Arrays`가 있다.

위의 규약은 compareTo 메소드도 equals와 마찬가지로 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. 

그래서 주의사항도 equals와 같다.

기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다. 

객체 지향적 추상화의 이점을 포기할 생각이 아니라면 말이다([아이템10](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava10.md))

우회법도 같다. 

Comparable을 구현한 클래스를 확정해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키도록 필드를 둔다.
그런 다음 내부 인스턴스를 반환하는 '뷰' 메소드를 제공한다. 
이렇게 하면 바깥 클래스에 우리가 원하는 compareTo 메소드를 구현해 넣을 수 있다. 

정렬된 클래스는 동치성을 비교할 때 equals 대신 compareTo 를 사용한다. 그 예로 TreeSet과 HashSet을 들 수 있는데 TreeSet은 정렬되어 있기 때문에 동치성을 compareTo로 비교하고, HashSet은 equals로 비교한다. 
이 경우가 equals가 compareTo가 일관되지 않은 경우이다. ( 규약 4)

```java
HashSet<BigDecimal> hashSet = new HashSet<>();
hashSet.add(new BigDecimal("1.0"));
hashSet.add(new BigDecimal("1.00"));
System.out.println(hashSet.size());     // 2

TreeSet<BigDecimal> treeSet = new TreeSet<>();
treeSet.add(new BigDecimal("1.0"));
treeSet.add(new BigDecimal("1.00"));
System.out.println(treeSet.size());    // 1
```

Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메소드의 인수 타입은 컴파일 타임에 정해진다. 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다는 뜻이다.

compareTo 메소드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다. 

### 1. 객체 참조 필드가 하나인 비교자

일단 객체 참조 필드가 한 개든 여러 개든 Comparable<T> 을 구현 해야한다.

이것은 T라는 클래스를 만들 때 T라는 클래스와 비교할 수 있다는 뜻으로 일반적으로 따르는 패턴이다. 

즉, 내가 만든 클래스와 Comparable안에 제네릭으로 들어가야 하는 타입은 같아야 한다. 

```java
public final class CaseInsensitiveString 
		implements Comparable<CaseInsensitiveString> {

	public int compareTo(CaseInsensitiveString cis) {
		return String.CASE_INSENSTIVE_ORDER.compare(s,cis.s);
	}
}

```

**그리고 compareTo 메소드에서 관계 연산자 < 와 > 를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다.** 

대신 compare 이라는 메소드를 사용해서 비교를 한다. 

### 2. 객체 참조 필드가 여러 개일 때 비교자

객체 참조 필드가 여러 개라면 어떤 것을 먼저 비교하느냐가 중요해진다.

가장 핵심적인 필드부터 비교하고, 비교 결과가 0이 아니라면 결과를 곧장 반환한다. 

즉, 핵심적인 필드부터 똑같지 않은 필드가 나올 때까지 비교해 나간다.

이전에 계속 구현했던 PhoneNumber 클래스를 예를 들어 설명해보자.

```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if(result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if(result == 0) {
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}
```

### Comparator을 이용한 비교

comparator을 이용해서 비교를 하면 간결하게 비교를 할 수 있지만, 약간의 성능 저하가 뒤따른다. 

```java
private static Comparator<PhoneNumber> COMPARATOR = 
		comparingInt((PhoneNumber pn) -> pn.areaCode)
			.thenComparingInt(pn -> pn.prefix)
			.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this,pn);
}
```

### HashCode 값의 차를 기준으로 하는 비교자 - 추이성 위배

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object2 o2) {
		return o1.hashCode() - o2.hashCode();
	}
}
```

이 방식은 값의 차이를 이용해서 첫 번째 값이 두 번째 값보다 작으면 음수를, 두 값이 같으면 0을 , 첫 번째 값이 크면 양수를 반환하게 하는 메소드이지만 **이 방식은 사용하면 안된다.** 

이 방식은 정수 오버플로우나 부동 소수점 계산 방식에서 오류를 일으킬 수 있다. 

대신 다음의 방법을 사용하라 

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object2 o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
}

static Comparator<Object> hashCodeOrder = 
		Comparator.comparingInt(o -> o.hashCode());
```