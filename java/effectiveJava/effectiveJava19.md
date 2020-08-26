# Item19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

---

## 1. 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야한다.

클래스의 API로 공개된 메소드에서 클래스 자신의 또 다른 메소드를 호출할 수 도 있다. 그런데 마침 호출되는 메소드가 재정의 가능 메소드(*public과 protected 중 final이 아닌 메소드*)라면 그 사실을 호출하는 메소드의 API 설명에 적시해야 한다. 

덧붙여 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다. 

API 문서의 메소드 설명 끝에 "Implementation Requirements"로 시작하는 절을 볼 수 있는데, 그 메소드의 내부동작 방식을 설명한 것이다. 메소드에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다. 

그 예로 java.util.AbstractCollection을 볼 수 있다. 

![https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/img/Untitled.png](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/img/Untitled.png)

@implSpec은 자바8에서 처음 도입됐고 자바9에서부터 본격적으로 사용됐다.

이 태그를 활성화 하려면 매개변수로 -tag "implSpec: a :Implementation Requiremets:"를 지정해주면 된다.

## 2. 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메소드 형태로 공개해야 할 수도 있다.

java.util.AbstractList의 removeRange 메소드를 예로 보자. 

![https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/img/Untitled.png](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/img/Untitled 1.png)

List구현체의 최종 사용자는 removeRange 메소드에 관심이 없다. 그럼에도 이 메소드를 제공한 이유는 단지 하위 클래스에서 부분리스트의 clear 메소드를 고성능으로 만들기 위해서이다. 

## 3. **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다.**

꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때 까지 전혀 쓰이지 않는 protected 멤버는 사실 private였어야 할 가능성이 크다. 

상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.

## 4. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메소드를 호출해서는 안 된다.

이 규칙을 어기면 프로그램은 오작동할 것이다. 그 이유는 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 이때 그 재정의한 메소드가 하위 클래스의 생성자에서 초기화한 값에 의존한다면 의도대로 동작하지 않을 것이다. 

```java
//상위클래스
public class Super {
	//잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다. 
	public Super() {
		overrideMe();
	}
	
	public void overrideMe() {
	}
}
//Super을 상속한 클래스
public final class Sub extends Super {
	//초기화 되지 않은 final 필드. 생성자에서 초기화 
  private final Instant instant;

  Sub(){
    instant = Instant.now();
  }
	
	//재정의 가능 메소드. 상위 클래스의 생성자가 호출된다.
  @Override
  public void overrideMe() {
    System.out.println(instant);
  }

  public static void main(String[] args) {
    Sub sub = new Sub();
    sub.overrideMe();
  }
}

```

이 프로그램이 instant를 두번 출력 하리라 기대했겠지만, 첫 번째는 null을 출력한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴트 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다. final 필드의 상태가 이 프로그램에서는 두 가지 임에 주목하자. 

## 4. Cloneable이나 Serializable 인터페이스는 상속용 설계의 어려움을 한층 더해준다.

clone과 readObject는 생성자와 비슷한 효과를 낸다. 따라서 상속용 클래스에서 Cloneable이나 Serializable을 구현할지 정해야 한다면, 이들을 구현할 때 따르는 제약도 생성자와 비슷하다는 점에 주의하자.

즉, **clone과 readObject 모두 직접적이든 간접적이든 재정의 가능 메소드를 호출해서는 안된다.**

readObject의 경우 하위 클래스의 상태가 역직렬화되기 전에 재정의한 메소드부터 호출한다. 

clone의 경우 하위 클래스의 clone 메소드가 복제본의 상태를 수정하기 전에 재정의한 메소드를 호출한다. 

Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메소드를 갖는다면 이 메소드들은 private이 아닌 protected로 선언해야 한다. private로 선언하면 하위 클래스에서 무시되기 때문이다.

## 클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당함을 알았다.

## 구체 클래스(상속용으로 설계하지 않은 클래스)는 상속을 금지한다.

상속을 금지하는 방법은 두 가지이다. 

### 1. 클래스를 final로 선언한다.

### 2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩토리를 만들어 주는 방법이다.

정적 팩토리는 내부에서 다양한 하위 클래스를 쓸 수 있는 유연성을 주며, 이와 관련해서는 [아이템17](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava17.md)에서 잘 다뤘다.  

**이러한 구체 클래스를 상속하는 방법이 하나 있다.**

클래스 내부에서는 재정의 가능 메소드를 사용하지 않게 만들고 이 사실을 문서로 남기는 것이다 .

즉, 재정의 가능 메소드를 호출하는 자기 사용 코드를 완벽히 제거하라는 말이다. 

이렇게 되면 재정의해도 다른 메소드에 아무런 영향을 주지 않기 때문이다. 

### 클래스의 동작을 유지하면서 재정의 가능 메소드를 사용하는 코드를 제거할 수 있는 기계적인 방법을 소개한다.

먼저 각각의 재정의 가능 메소드는 자신의 본문 코드를 private '도우미 메소드'로 옮기고, 이 도우미 메소드를 호출하도록 수정한다. 

그런 다음 재정의 가능 메소드를 호출하는 다른 코드들도 모두 이 도우미 메소드를 직접 호출하도록 수정하면 된다 .
