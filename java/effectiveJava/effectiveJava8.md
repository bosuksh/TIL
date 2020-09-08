2020-07-28



# 이펙티브 자바 Item8

# Item8. finalizer와 cleaner 사용을 피하라

자바는 두가지 객체 소멸자를 제공한다. **finalizer과 cleaner**이다.

`finalizer와 cleaner는 GC될때 호출된다.`

**finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.** (자바9에서는 finalizer을 deprecated 시켰고 그 대안으로 cleaner을 제시했다.)

**cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.** 

자바의 finalizer와 cleaner는 c++의 destructor의 개념과 다르다.

c++의 destructor는 특정 객체와 관련된 자원을 회수하는 보편적인 방법이다. 그러나 자바는 GC가 그 역할을 대신하고 프로그래머에게는 아무런 작업도 요구하지 않는다. 

c++의 destructor는 비메모리 자원을 회수하는 용도로 쓰이는데, 자바에서는 `try-with-resources`나 `try-finally`를 사용한다.

finalizer와 cleaner의 단점에 대해서 알아보자. 

### 1. finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

---

객체에 접근을 할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼머나 걸리는지 알수 없다. 

**즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.** 

예컨대, 파일 닫기를 finalizer나 cleaner에게 맡기면 중대한 finalizer가 cleaner가 파일 닫기를 게을리해서 새로운 파일을 열지 못해 문제가 발생할 수 있다.

### 2. finalizer는 인스턴스의 자원 회수가 제멋대로 지연될 수 있다.

---

finalizer쓰레드는 다른 애플리케이션 쓰레드보다 우선순위가 낮아서 제대로 실행될 기회를 얻지 못한다. 

이는 곧 OOM을 발생시키는 원인이 될 수도 있다. 

cleaner은  자신을 수행할 쓰레드를 제어할 수 있다는 점이 조금 나을순 있으나, 여전히 백그라운드에서 수행되며 GC의 통제하에 있으니 즉각수행되리라 보장은 없다. 

### 3. finalizer와 cleaner는 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다.

---

**상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.**

예를 들어, DB같은 공유 자원의 영구 락 해제를 finalizer나 cleaner에 맡겨 놓으면 분산시스템 전체가 서서히 멈출 것이다. 

System.gc나 System.runFinalization 메서드에 현혹되지 말자. finalizer와 cleaner가 실행될 가능성을 높여줄 수는 있으나, 보장해주지 않는다. 이 것을 보장해주는 메서드가 있긴 하다. System.runFinalizerOnExit과 그 쌍둥이인 Runtime.runFinalizersOnExit이다. 하지만 이 두 메서드는 심각한 결함 때문에 수 십년간 지탄받았다.(Thread Stop)

### 4. finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.

---

잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다. 

그리고 다른 쓰레드가 이 훼손된 객체를 사용하려 한다면 무슨 일이 일어날지 알 수 없다.

보통의 경우에는 잡지 못한 예외가 스레드를 중단하고 스택 추적 내역을 출력하겠지만, 같은 일이 finalizer에서 일어난다면 경고조차 출력하지 않는다. 

그나마 cleaner를 사용하는 라이브러리는 자신의 쓰레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

### 5. finalizer와 cleaner는 심각한 성능 문제도 동반한다.

---

AutoCloseable 객체를 생성하고 GC가 수거하기까지 12ns가 걸린 반면(try-with-resources로 자신을 닫도록 했다), finalizer를 사용하면 550ns가 걸렸다. 

그 이유는 finalizer가 GC의 효율을 떨어뜨리기 때문이다.

### 6. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

---

finalizer의 공격 원리는 A라는 클래스가 있을 때, 이것을 상속받은 B라는 클래스가 악의적으로 예외를 던지고 finalize()를 overriding하게 되면 죽을 때 finalize가 실행되는데 이때, 인스턴스의 static field나 메소드에 접근할 수 있다. 그래서 죽어야하는 인스턴스인데 좀비처럼 살아 있게 된다.

**객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지도 않다.** 

**그래서 final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메소드를 만들고 final로 선언하면 된다.** 

### 자원을 반납하는 방법

### # AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메소드를 호출한다.

일반적으로 예외가 발생해도 제대로 종료되도록 `try-with-resources` 를 사용해야 한다 (아이템9)

close 메소드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메소드는 이 필드를 검사해서 객체가 닫힌 후 에 불렸다면 IllegalStateException을 던지는것이다 .

```java
public class SampleClass implements AutoCloseable{
    private boolean isClosed;

    @Override
    public void close() throws RuntimeException {
	if(this.isClosed) {
	    throw new IllegalStateException;
	}
	isClosed = true;
    }

    /**
    *  다른 메소드 생략
    */
    }

public Main() {

    public static void main(String[] args) {
	SampleClass sampleClass = new SampleClass();
 	sampleClass.close();
    }
}
```

### finalizer와 cleaner을 도대체 어떻게 사용할까?

### 1. client가 close 메소드를 호출하지 않는 것에 대비한 안전망 역할이다.

cleaner나 finalizer가 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안 하는 것보다는 낫기 때문이다.  **즉, finalizer나 cleaner에서 close를 호출하는 것이다.**

그 예로, `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`가 있다. 

### 2. 네이티브 피어와 연결된 객체이다.

- 네이티브 피어란 일반 자바 객체가 네이티브 메소드를 통해 기능을 위임한 네이티브 객체를 말한다.

네이티브 피어는 자바 객체가 아니니 GC는 그 존재를 알 지 못한다. 그 결과 자바 피어를 회수 할때 네이티브 객체까지 회수하지 못한다. 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 해당된다. 

- **cleaner 예제**

```java
public class CleanerSample implements AutoCloseable {

    private static final Cleaner CLEANER = Cleaner.create();

    private final CleanerRunner cleanerRunner;

    private final Cleaner.Cleanable cleanable;

    public CleanerSample() {
        cleanerRunner = new CleanerRunner();
        cleanable = CLEANER.register(this, cleanerRunner);
    }

    @Override
    public void close() {
        cleanable.clean();
    }

    public void doSomething() {

        System.out.println("do it");
    }

    private static class CleanerRunner implements Runnable {

        // TODO 여기에 정리할 리소스 전달

        @Override
        public void run() {
            // 여기서 정리
            System.out.printf("close");
        }
    }

}
```

CleanerRunner 인스턴스는 절대로 CleanerSample 인스턴스를 참조해서는 안된다. 왜냐하면, 순환참조가 생겨 GC가 CleanerRunner을 회수해 갈 기회가 오지 않는다. 

소스 출처: [https://github.com/keesun/study/blob/master/effective-java/item8.md](https://github.com/keesun/study/blob/master/effective-java/item8.md)

