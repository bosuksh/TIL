2020-07-30



# 이펙티브 자바 Item10

# Item10. equals는 일반 규약을 지켜 재정의하라

equals 메소드를 재정의 하지 않으면 인스턴스는 오직 자기 자신과 같게된다. 

equals 메소드는 아래 열거한 상황에 해당되면  재정의 하지 않는 것이 좋다. 

## Equals 재 정의를 피해야 하는 상황

1. **각 인스턴스가 본질적으로 고유하다.**
- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다. 예) Thread

2. **인스턴스의 논리적 동치성을 검사할 일이 없다.**
- java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는, 즉 논리적 동치성을 검사하는 방법도 있다. (즉, 다른 정규표현식이더라도 논리적으로 동치인지 검사) 
3. **상위클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.** 
- 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다. 
4. **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.** 
- equals가 실수로라도 호출되는 걸 막고 싶다면 아래처럼 구현하자. 

```java
@Override public boolean equals(Object o) {
    throw new AssertionError();  // 호출금지
}
```

그렇다면 equals를 재정의해야 할 때는 언제일까? 객체 식별성(두 객체가 물리적으로 같은가)가 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.  
**주로 값 클래스가 해당된다.**

값 클래스란 Integer와 String 처럼 값을 표현하는 클래스를 말한다. 두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어할 것이다. 이렇게 논리적 동치성을 확인하도록 재정의하면 Map의 key와 Set으로 사용할 수 있다. 

값 클래스라 해도, 값이 같은 클래스가 둘 이상 만들어진다는 것이 보장되면 equals를 재정의 안해도 된다. 
ex) 인스턴스 통제 클래스(아이템 1), Enum(아이템 34)

equals 메소드를 재정의할 때는 반드시 일반 규약을 따라야한다. 다음은 Object 명세에 적힌 규약이다. 

## Equals 메소드는 동치관계를 구현하며 다음을 만족한다.

### 1. 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.

다시 말해 객체는 자기 자신과 같아야 한다.

이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메소드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다. 

### 2. 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true 이면 y.equals(x)도 true이다.

두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다. 

대칭성은 잘못하면 어길 수 있다. 

```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
       if(o instanceof CaseInsensitiveString)
           return s.equalsIgnoreCase(((CaseInsensitiveString)o).s);
       **if(o instanceof String)   //한 방향으로만 작용한다 문제점
           return s.equalsIgnoreCase((String) o);**
       return false;
    }
}
```

여기서 문제점은 CaseInsensitiveString이 String과 비교할 때 문제가 생긴다. 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("HELLO");
String s = "hello";

cis.equals(s);     // true
s.equals(cis);     // false
```

즉, 대칭성을 위반하게 된다.  다음은 Collection에 넣어보자. 

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

이때, list.contains(s)를 호출하면 어떤 결과가 나올 지 알 수 없다. JDK 버전에 따라 다르게 반응 할 것이다. 

**즉, equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.** 

이 문제를 해결하려면 CaseInsensitiveString가 String과 연동하겠다는 생각을 버려야 한다. 

```java
/**
* 간단해진 equals
*/
@Override
public boolean equals(Object o) {
   return o instanceof CaseInsensitiveString &&
					 ((CaseInsensitiveString)o).s.equalsIgnoreCase(s);
}
```

### 3. 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)가 true이면 x.equals(z)도 true이다.

첫번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫번째 객체와 세 번째도 같아야 한다는 뜻이다. 

예를 들어 확인해보자. Point라는 2차원 좌표를 나타내는 클래스가 있고, 그걸 상속받은 ColorPoint라는 클래스가 있다.

```java
public class Point{
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}

// Point를 상속받은 ColorPoint 클래스
public class ColorPoint extends Point{
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
		
    @Override
    public boolean equals(Object o) {    //잘못된 코드 대칭성 위배
        if(!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

이런식으로 코드를 짜게 되면 대칭성이 위배돼서 문제가 생긴다. 

```java
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2,Color.RED);

p.equals(cp); //true
cp.equals(p); //false
```

그렇다면 Point와 ColorPoint를 비교할 땐, Color를 무시하면 될까? 

미리 답을 말하자면 그것은 추이성 위배가 된다. 

```java
@Override
public boolean equals(Object o) {    //잘못된 코드 추이성 위배
    if(!(o instanceof Point))
        return false;
    //o가 일반 Point면 색상 무시 
    if(!(o instanceOf ColorPoint))
	return o.equals(this);
    //o가 Color Point면 색상 비교
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

추이성이 위배되는 예시를 알아보자

```java
ColorPoint p1 = new ColorPoint(1,2,Color.Blue);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2,Color.RED);

p1.equals(p2); //true
p2.equals(p3); //true
p1.equals(p3); //false
```

p1과 p2, p2와 p3를 비교할 때는 Point로 Color가 무시되었지만, p1과 p3를 비교할 때는 Color가 고려되었기 때문이다. 

그렇다면 해법은 무엇일까?

> **구체 클래스를 확장해 새로운 값을 추가하면서 equals규약을 만족시킬 방법은 존재하지 않는다.**

객체 지향의 추상화를 포기하지 않는 한은 말이다. 

```java
@Override
public boolean equals(Object o) {    //리스코프 치환 원칙 위배
    if(o == null || o.getClass() != this.getClass())
        return false;
    Point p = (Point)o;
    return p.x == x && p.y == y;
}
```

잘 작동해 보이지만 실제로 사용할 수 없다. 왜냐하면 구현 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 한다.

**리스코프 치환 원칙에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.** 

**따라서, 그 타입의 모든 메소드가 하위 타입에서도 똑같이 잘 작동해야 한다.**

구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. 

"상속 대신 컴포지션을 사용하라"(아이템 18)

```java
public class ColorPoint extends Point{
		private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
	point = new Point(x,y);
        this.color = Objects.requiresNonNull(color);
    }
    public Point asPoint(){
	return point;
    }
		
    @Override
    public boolean equals(Object o) {    //잘못된 코드 대칭성 위배
        if(!(o instanceof ColorPoint))
            return false;
	ColorPoint cp = (ColorPoint)o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

자바 라이브러리 중에서도 java.sql.Timestamp는 java.util.Date를 확장한 후 nanoseconds를 추가했다. 그 결과 Timestamp는 대칭성을 위배하며, Date 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있다.

### 4. 일관성(consistency): null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

두 객체가 같다면 수정되지 않는 한 앞으로 영원히 같아야 한다는 뜻이다. 

불변 클래스로 만들기로 했다면 equals가 한번 같다고 한 객체와는 영원히 같다고 답하고, 다르다고 한 객체와는 영원히 다르다고 답하도록 만들어야 한다.

**클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.**

그걸 위반한 자바의 API인 java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다. 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다. 

### 5. null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.

모든 객체가 null과 같지 않아야 한다는 뜻이다. 실수로라도 NullPointException을 던지게 해서는 안된다. 

```java
@Override
public boolean equals(Object o) {    
    if(o == null)
        return false;
		... 
}

@Override
public boolean equals(Object o) {    //위의 코드 보다는 묵시적 null검사 - 이쪽이 낫다. 
    if(!(o instanceof MyType))
        return false;
		...
}
```

## Equals 사용 포인트

지금까지의 내용을 종합해서 양질의 equal 메소드 구현 방법을 단계별로 정리해보겠다. 

1. **== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.** 
2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.** (그렇지 않다면 false를 반환한다.)
3. **입력을 올바른 타입으로 형변환한다.** (2번에서 instanceof 검사를 했기 때문에 100% 성공이다.)
4. **입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.** (모든 필드가 일치하면 true, 하나라도 다르면 false를 반환한다.)

- float과 double을 제외한 기본 타입은 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals 메소드로, float과 double은 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.
- 배열의 모든 원소가 핵심필드라면 Arrays.equals 메소드들 중 하나를 사용하자.
- 때론 null도 정상 값으로 가져가는 참조 타입 필드도 있다. 이런 필드는 정적 메소드인 Objects.equals(Object, Object)로 비교해 NullPointerException을 예방할 수 있다.
- 또 비교하기 아주 복잡한 필드를 가진 클래스는 그 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적이다. 이 기법은 불변 클래스(아이템17)에 제격이다.
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최성의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.

**Equals를 다 구현했다면 세 가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가? 단위테스트를 돌려서 확인해보자** 

전형적인 equals 메소드 사용 예이다.

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(short areaCode, short prefix, short lineNum) {
        this.areaCode = rangeCheck(areaCode,999,"지역코드"); 
        this.prefix =  rangeCheck(prefix,999,"프리픽스");
        this.lineNum =  rangeCheck(lineNum,9999,"가입자 번호");
    }
    
    private static short rangeCheck(int val, int max, String args) {
        if(val < 0 || val > max)
            throw new IllegalArgumentException(args+": "+ val);
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this)
            return true;
        if(!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```

## 마지막 주의사항

- equals를 재정의할 땐 hashcode도 반드시 재정의하자(아이템11)
- 너무 복잡하게 해결하려 들지 말자
- Object 외의 타입을 매개변수로 받는 equals 메소드는 선언하지 말자.

예시

```java
//잘못된 예
public boolean equals(MyClass o) {
		...
}
```

이건 재정의가 아니라 다중정의이다(아이템 52)
@Override를 이용하면 실수를 예방할 수 있다. (아이템 40)

- AutoValue 프레임워크를 이용하면 손쉽게 메소드를 작성할 수 있다..
