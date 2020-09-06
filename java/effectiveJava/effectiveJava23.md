2020-09-06

# Item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라.

두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 본 적이 있을 것이다. 다음 코드는 원과 사각형을 표현할 수 있는 클래스다. 

```java
class Figure {
	enum Shape { RECTANGLE, CIRCLE };
	
	//태그 필드 - 현재 모양을 나타낸다.
	final Shape shape;
		
	//다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
	double length;
	double width;
	
	// 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다. 
	double radius;
	
	// 원용 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// 사각형용 생성자
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}
	
	double area() {
		switch(shape) {
			case RECTANGLE:
				return length * width;
			case CIRCLE:
				return Math.PI * (radius * radius);
			default:
				throw new AssertionError(shape);
	}	
}
```

**태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다**. 

우선 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다. 

여러 구현이 한 코드에 혼합되어 있어 가독성도 나쁘다. 

다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다. 

엉뚱한 필드를 초기화해도 런타임에서야 문제가 드러난다.

또 다른 의미를 추가하려면 코드를 수정해야 한다. 

하지만 자바 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공한다. 

**바로 클래스 계층 구조를 이용한 서브 타이핑이다.**

그렇다면 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법을 알아보자. 

1. 계층구조의 루트가 될 추상클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메소드를 루트 클래스의 추상 메소드로 선언한다. 
*위 코드의 area가 그에 해당된다.*
2. 태그 값에 상관없이 동작이 일정한 메소드를 루트 클래스에 일반 메소드로 추가한다. 
3. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드도 전부 루트 클래스로 올린다. 
*위의 코드에서는 태그 값에 상관없는 메소드가 하나도 없고 공통으로 사용하는 데이터 필드도 없다.* 
*즉,  루트클래스에는 추상메소드 area만 남게 된다.*
4. 다음으로 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. 
*우리 예에서는 Circle 클래스와 Rectangle 클래스를 만들면 된다.* 

그렇게 바뀐 코드를 보자. 

```java
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;
	
	Circle(double radius) { this.radius = radius; }

	@Override double area() { return Math.PI * (radius * radius);
}

class Rectangle extends Figure {
	final double width;
	final double length
	
	Rectangle(double width, double length) {
		this.width = width;
		this.length = length; 
	}

	@Override double area() { return width * length);
}
```

위의 코드는 태그 달린 클래스의 단점을 모두 날려버렸다. 

간결하고 명확하며, 쓸데 없는 코드가 모두 사라졌다.

관련 없던 데이터 필드도 모두 사라졌다. 

살아 남은 필드도 모두 final이다. 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메소드를 모두 구현했는지 컴파일러가 확인해준다. 

실수로 빼먹은 case문 때문에 런타임 오류가 발생할 일도 없다. 

루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 

또한 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력도 높여준다. 

아래처럼 정사각형을 구현할 때, 태그 타입으로 구현하면 무엇 무엇을 고쳐야 할지 생각만 해도 머리아프다.

```java
class Square extends Rectangle {
	Square(double side) {
		super(side, side);
	}
}
```