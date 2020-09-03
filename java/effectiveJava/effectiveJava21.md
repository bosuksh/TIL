2020-09-03

# Item21. 인터페이스는 구현하는 쪽을 생각해 설계하라.



자바8이 생기고 기존 구현체를 깨뜨리지 않고 인터페이스에 디폴트 메소드를 추가할 수 있다. 

그러나 그렇다고 위험이 완전히 사라진 것이 아니다. 

디폴트 메소드를 선언하면 인터페이스를 구현한 후 디폴트 메소드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다. 이처럼 자바에도 기존 인터페이스에 메소드를 추가하는 길이 열렸지만 모든 기존 구현체들과 매끄럽게 연동되리라는 보장은 없다. 

## **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메소드를 작성하기란 어려운 법이다.**

자바 8 `Collection`인터페이스에 추가된 `removeIf` 메소드를 예로 생각해보자. 

```java
/*
* 이 메서드는 주어진 boolean 함수(predicate)가 true를 반환하는 모든 원소를 제거한다. 
*/
default boolean removeIf(Predicate< ? super E> filter {
	Objects.requireNonNull(filter);
	boolean result = false;
	for(Iterator<E> it = iterator(); it = iterator(); it.hasNext();) {
		if(filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}
	return result;
}
```

이 코드가 모든 `Collection`구현체와 잘 어우러지는 것은 아니다. 대표적은 예가 **org.apache.common.commons.collections4.collection.SynchronizedCollection**이다.

이 클래스는 java.util의 **Collections.synchronizedCollection** 정적 팩토리 메소드가 반환하는 클래스와 비슷하다. 

아파치 버전은 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 즉, 모든 메소드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스([아이템18](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava18.md))이다.

removeIf의 구현은 동기화에 관해 아무것도 모르므로 락 개체를 사용할 수 없다. 따라서 SynchronizedCollection 인스턴스를 여러 쓰레드가 공유하는 환경에서 한 쓰레드가 removeIf를 호출하면 ConcurretnModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

이걸 막기 위해서 자바 플랫폼 라이브러리에서는 구현한 디폴트 메소드를 재정의하고, 다른 메소드에서 디폴트 메소드를 호출하기 전에 필요한 작업을 수행하도록 했다. 

## 디폴트 메소드는 기존 구현체에 런타임 오류를 일으킬 수 있다.

기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다. 

**디폴트 메소드라는 도구가 생겼더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.** 

또한 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 한다. 수많은 개발자가 그 인터페이스를 나름의 방식으로 구현할 것이니, 서로 다른 방식으로 최소한 세 가지는 구현해봐야 한다. 

각 인터페이스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어 봐야한다. 

**인터페이스를 릴리스한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안된다.**