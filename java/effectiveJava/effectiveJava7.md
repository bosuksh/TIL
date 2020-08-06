2020-07-19



# 이펙티브 자바 Item7

# Item7. 다 쓴 객체 참조를 해제하라

C, C++ 처럼 메모리를 직접 관리해야 하는 언어를 쓰다가 자바처럼  가비지 컬렉터를 갖춘 언어로 넘어오면서 프로그래머의 삶이 훨씬 평안해진다. 그래서 자칫 메모리 관리에 더 이상 신경 쓰지 않아도 된다고 오해할 수 있는데, 절대 사실이 아니다. 

### 메모리를 직접 관리하는 클래스

스택을 간단히 구현한 다음 코드를 보자. 

```java
public class Stack{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public stack() {
	elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
	ensureCapacity();
	element[size++] = e;
    }

    public Object pop() {
	if(size == 0)
	    throw new EmptyStackException();
	return element[--size];

    }

    /**
    * 원소를 위한 공간을 저거도 하나 이상을 확보한다.
    * 배열 크기를 늘려할 때마다 대략 두배씩 늘린다. 
    */
    private void ensureCapacity() {
	if(elements.length == size)
	    elements = Arrays.copyOf(elements, 2 * size + 1);
    }

}
```

특별한 문제는 없어 보인다. 하지만 꼭꼭 숨어있는 문제가 있다. 이는 바로 '메모리 누수'로, 이 스택을 사용하는 프로그램을 오래 실행하다 보면 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하 될 것이다. 상대적으로 드문 경우긴 하지만 심할 때는 디스크 페이징이나 OOM을 일으켜 프로그램이 종료되기도 한다. 

그러면 앞 코드의 메모리 누수는 어디서 일어날까? 이 코드에서는 스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 프로그램에서 그 객체들을  더 이상 사용하지 않더라도 말이다. 

앞의 코드에서의 elements 배열의 '활성 영역' 밖의 참조들이 모두 여기에 해당한다. 활성 영역은 인덱스가 size보다 작은 원소들로 구성된다. 

해법은 간단하다.  해당 참조를 다 썼을 때 null처리를 하면 된다. 

```java
public Object pop() {
    if(size == 0)
	throw new EmptyStackException();
    Object result = element[--size];
    element[size] = null;   //다 쓴 참조 해제
    return result;
}
```

다 쓴 참조를 null처리하면 다른 이점도 따라온다. 만약 null처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 NullPointerException을 던지며 종료한다. 

그렇다고 모든 객체를 쓰자마자 null처리 할 필요는 없다. **객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.**

다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.  변수의 범위를 최소가 되게 정의했다면(아이템 57) 이 일은 자연스럽게 이뤄진다. 

일반적으로 **자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야한다.**  원소를 다 사용한 즉시 그 원소가 참조한 객체을 다 null 처리 해줘야 한다. 

### **캐시 역시 메모리 누수를 일으키는 주범이다.**

해법은 여러 가지다. 운 좋게 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 **WeakHashMap**을 사용해 캐시를 만들자. 다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다. 

(**Scheduled ThreadPoolExecutor** 같은) 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수작업으로 수행하는 방법이 있다. 

LinkedHashMap은 removeEldestEntry 메소드를 써서 후자의 방식으로 처리한다. 

### 메모리 누수의 세번째 주범은 리스너 혹은 콜백이다.

클라이언트가 콜백을 등록만하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이럴 때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. 

예를 들면 WeakHashMap에 키로 저장하면 된다.

`메모리 누수는 겉으로 잘 드러나지 않아 시스템이 수년간 잠복하는 사례도 있다. 이런 누수는 예방법을 미리 익혀두는 것이 매우 중요하다.`
