2020-07-08

# Item3. private 생성자나 열거 타입으로 싱글턴임을 보장해라

## 싱글턴

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 

싱글턴의 예로는 함수(아이템24)와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다. 

## 싱글턴의 문제점

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다.** 

타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 mock 구현으로 대체할 수 없기 때문이다. 

## 싱글턴 생성 방식

싱글턴을 생성하는 방식은 두가지이다. 두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다. 

### 1.  public static 멤버가 final인 필드

```java
public class Elvis {
	public static final Elvis INSTANCE = new ELVIS();
	private Elvis() { ... }

	public void leaveTheBuilding() { ... }

}
```

 private 생성자는 public static final 필드를 초기화할 때 딱 한 번만 호출된다. public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화 될 때 만들어진 인스턴스가 전체 시스템에 하나뿐임이 보장된다. 

*예외는 단 한 가지, 권한이 있는 클라이언트는 리플렉션 API(아이템 65)인 AccessibleObject.setAccessible을 사용해 private한 생성자를 호출할 수 있다.* 

→ **이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.** 

장점1 : 해당 클래스가 싱글턴임이 API에 명백히 드러난다. 

장점2: 간결함이다.



### 2. 정적 팩토리 메서드를 public static 멤버로 제공한다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE;}

    public void leaveTheBuilding() { ... }
}
```

Elvis.getInstance()는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스란 결코 만들어 질 수 없다.
(리플렉션을 통한 예외는 똑같이 이뤄진다.)

장점1: API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.

장점2: 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다는 점이다(아이템 30)

장점3: 정적 팩토리의 메서드 참조를 공급자로 사용할 수 있다. 
           Elvis::getInstance를 Supplier<Elvis>로 사용하는 방식이다. (아이템 43, 아이템 44)

둘 중 하나로 만든 싱글턴 클래스를 직렬화하려면 Serializable을 구현한다고 선언하는 것만으로 부족하다. 
모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야한다.(아이템 89)

→ 그 이유는 직렬화된 인스턴스를 역직렬화 할 때마다 새로운 인스턴스가 만들어진다.

```java
//싱글턴임을 보장해주는 readResolve 메소드
private Object readResolve(){
    // '진짜' Elvis를 반환하고, '가짜' Elvis는 GC에 맡긴다. 		
    return INSTANCE;
}
```



### 3. 원소가 하나인 열거(Enum) 타입을 만드는 것이다.

```java
public enum Elvis {
    INSTANCE;

    public void leaveBuilding() { ... }
}
```

public 필드 방식과 아주 비슷하지만, 더 간결하고, 추가 노력없이 직렬화 할 수 있고 리플렉션 공격에서도 제 2의 인스턴스가 생기는일을 완벽히 막아준다. 

> 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
