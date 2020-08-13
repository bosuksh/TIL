2020-08-12

# Item17. 변경 가능성을 최소화하라.

불변 클래스란 인스턴스의 내부 값을 수정할 수 없는 클래스다. 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다. 
예를 들어 BigInteger, BigDecimal, String, 기본 타입이 박싱된 클래스가 있다. 

클래스를 불변으로 만들려면 다음 다섯 가지 규칙을 따르면 된다.

- 객체의 상태를 변경하는 메소드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private으로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

다음 불변 클래스의 예제를 보자. 복소수 클래스이다. 

```java
public final class Complex {
  private final double re;
  private final double im;

  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }
  
  public double realPart() {
    return re;
  }
  
  public double imaginaryPart() {
    return im;
  }
  
  public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }
  
  public Complex minus(Complex c) {
    return new Complex(re - c.re, im - c.im);
  }
  
  public Complex times(Complex c){
    return new Complex(re * c.re - im * c.re,
                       re * c.im + im - c.re); 
  }
  
  public Complex dividedBy(Complex c) {
    double tmp = c.re * c.re + c.im * c.im;
    return new Complex((re * c.re + im * c.im)/tmp,
                       (im * c.re - re * c.im)/tmp);
  }

  @Override
  public int hashCode() {
    return 31 * Double.hashCode(re) + Double.hashCode(im);
  }

  @Override
  public boolean equals(Object o) {
    if(o == this)
      return true;
    if(!(o instanceof Complex))
     return false;
    Complex c = (Complex) o;
    
    return Double.compare(re, c.re) == 0 
            && Double.compare(im, c.re) == 0; 
  }

  @Override
  public String toString() {
    return "(" + re + " + " + im + ")";
  }
}
```

여기서 주목해야 할 점은 두 가지이다.

1. **사칙연산 메소드들이 새로운 Complex 인스턴스를 생성한다.** 
→ 이런 방식을 함수형 프로그램이라고 하는데 함수의 피연산자 자체는 그대로인 방식이다.
2. **메서드 이름이 동사(add)가 아닌 전치사(plus)이다** 
이는 해당 메소드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도이다. 

## 불변 객체의 특징

1. **불변식은 단순하다.**
객체의 생성 시점부터 파괴 될 때까지 그대로 유지하면 된다. 
2. **불변 객체는 근본적으로 쓰레드 안전하여 따로 동기화 할 필요가 없다. 
→ 불변 객체는 안심하고 공유할 수 있다.** 

그렇기 때문에 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리를 제공할 수 있다.  
public static final Complex ZERO = new Complex(0, 0);
3. **불변 객체는 clone메소드나 복사 생성자를 제공하지 않는게 좋다.**
어차피 복사해봤자 같은 인스턴스이기 때문이다. 
4. **불변 객체를 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**
5. **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.** 
6. **불변 객체는 그 자체로 실패 원자성을 제공한다.(아이템 76)** 
실패 원자성이란 메소드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다는 성질이다. 
불변 객체의 메소드는 내부 상태를 바꾸지 않으니 이 성질을 만족한다. 
7. **불변 클래스의 단점은 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.** 
이는 곧 성능 문제를 야기할 수 있다.

## 불변 클래스를 만드는 또 다른 설계 방법

### 1. 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩토리를 제공한다.

```java
public class Complex {
  private final double re;
  private final double im;

  private Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

	public static Complex valueOf(double re, double im) {
		return new Complex(re, im);
	}
}
```

패키지 바깥에서 클라이언트가 바라본 이 불변 객체는 사실상 final이다. public이나 protected 생성자가 없기 때문이다. 

정적 팩토리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고, 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다. 

## 정리

### getter가 있다고 해서 무조건 setter을 만들지는 말자.

**클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.** 

단순한 값 객체는 불변으로 만들자.

### 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

**다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.** 

### 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.

java.util.concurrent 패키지의 CountDownLatch 클래스가 잘 방증한다.