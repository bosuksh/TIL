2020-08-10

# Item16. public class에서는 public 필드가 아닌 접근자 메소드를 사용하라

public class에 public 필드를 사용하게 되면 데이터 필드에 직접 접근하게 되어, 캡슐화의 이점을 제공하지 못한다.([아이템15](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava15.md))

철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가한다.

```java
public class Point {
	private double x;
	private double y;

	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}

	public double getX() {
		return this.x;
	}

	public double getY() {
		return this.y;
	}

}
```

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다. 

**하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.** 

또한 public 클래스가 불변이라면 직접 노출할 때의 단점이 조금 줄어들지만, 여전히 결코 좋은 생각이 아니다. 

왜냐하면, API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 

단, 불변식은 보장할 수 있게 된다. 예컨대 다음 클래스는 각 인스턴스가 유효한 시간을 표현함을 보장한다. 

```java
public final class Time {
	private static final int HOURS_PER_DAY = 24;
	private static final int MINUTES_PER_HOUR = 60;

	public final int hour;
	public final int minute;
	
	public Time(int hour, int minute) {
		if(hour < 0 || hour >= HOURS_PER_DAY)
			throw new IllegalArgumentException("시간: " + hour);
		if(minute < 0 || minute >= MINUTES_PER_HOUR)
			throw new IllegalArgumentException("분: " + minute);
		this.hour = hour;
		this.minute = minute;
	}
}
```