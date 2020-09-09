2020-09-09

# Item25. 톱레벨 클래스는 한 파일에 하나만 담으라.



자바 한 파일에 여러 톱 레벨 클래스를 담으면 어느 소스 파일을 먼저 컴파일 하냐에 따라 달라진다. 

예를 통해서 확인해보자. 

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);	
	}
}
```

*Utensil* 과 *Dessert*  클래스가 모두 Utensil.java라는 한 파일에 정의되어 있다고 가정해보자. 

```java
// Utensil.java
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

그리고 [Dessert.java](http://dessert.java) 파일에 똑같은 형태로 클래스가 정의되어 있다고 가정해보자. 

```java
// Dessert.java
class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	static final String NAME = "pie";
}
```

맨 처음에 javac [Utensil.java](http://utensil.java) [Main.java](http://main.java) 를 통해 컴파일하면 pancake가 출력될 것이다. 

다시 javac [Main.java](http://main.java) [Dessert.java](http://dessert.java) 로 컴파일 하면 *Utensil , Dessert* 클래스가 중복 정의 되었다고 컴파일 에러가 날 것이다.

그러나 만약 javac [Main.java](http://main.java)나 javac Main.java Utensil.java로 컴파일 하면 [Dessert.java](http://dessert.java) 파일을 작성하기 전처럼 pancake를 출력한다. 

그러나 javac Dessert.java Main.java로 컴파일 하면 potpie가 출력된다.

이렇게 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라질 수 있다. 

다행히 해결책은 매우 간단하다. 두 클래스를 각각 파일에 분리해서 담거나 한 클래스 내부의 정적 멤버 클래스([아이템24](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava24.md))로 사용할 수 있다. 

다음 방법은 정적 멤버 클래스로 선언한 방법이다. 

private로 관리하면 접근 범위도 최소로 관리할 수 있다. 

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);	
	}
	private static class Utensil {
		static final String NAME = "pan";
	}
	
	private static class Dessert {
		static final String NAME = "cake";
	}
}
```

> 그러나 사실 IntelliJ 같은 IDE가 한 파일내에 톱 레벨 클래스 2개 이상 선언하지 못하도록 막아준다.