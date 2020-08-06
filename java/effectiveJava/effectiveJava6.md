2020-07-19



# 이펙티브 자바 Item6

# Item6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.  재사용은 빠르고 세련됐다. 특히 불변객체(아이템17)는 언제든지 재사용할 수 있다. 

생성자 대신 정적 팩토리 메소드 ([아이템 1](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava1.md))를 제공하는 불변 클래스에서는 정적 팩토리 메소드를 사용해 불필요한 객체 생성을 피할 수 있다. 

### 비싼 객체

---

예를 들어, Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩토리 메서드를 사용하는 것이 좋다. 

생성 비용이 아주 비싼 객체가 있다. 이런 '비싼 객체'가 반복해서  필요하다면 캐싱하여 재사용하길 권한다. 

예를 들어, 주어진 문자열이 유효한 로마 숫자인지를 확인하는 메소드를 작성한다고 해보자. 

```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3}"
	    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3}$");
}
```

이 방식의 문제는 String.matches 메소드를 사용한다는 데 있다. 
**String.matches는 정규 표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복하기엔 적합하지 않다.** 

이 메소드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 인스턴스 생성 비용이 높다. 

성능을 개선하려면 필요한 정규표현식을 표현하는 (불변인) **Pattern** 인스턴스 클래스 초기화 과정에서 직접 캐싱해두고, 나중에 isRomanNumeral 메소드가 호출될 때마다 이 인스턴스를 재사용한다.  

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
	"^(?=.)M*(C[MD]|D?C{0,3}"
	+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3}$");

    static boolean isRomanNumeral(String s) {
	return ROMAN.matcher(s).matches();
    }
}
```

개선된 isRomanNumeral 방식의 클래스가 초기화 된 후 이 메소드를 한 번도 호출하지 않는다면 ROMAN 필드는 쓸데없이 초기화 된 꼴이다. isRomanNumeral 메소드가 처음 호출될 때 필드를 초기화하는 지연 초기화(lazy initialization, 아이템83)로 불필요한 초기화를 없앨 수 는 있지만, 권하지 않는다. 지연 초기화는 코드를 복잡하게 만드는데, 성능은 크게 개선되지 않는다.(아이템67)

### 어댑터

---

객체가 불변이라면 재사용해도 안전함이 명백하다. 하지만 그렇지 않은 경우도 있다. 어댑터는 실제 작업은 뒷단에 위임하고 자신은 제2의 인터페이스 역할을 해주는 객체이다. 어댑터는 뒷단 객체만 관리하면 된다. 즉, 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 하나씩만 만들면 충분하다. 

예를들어, Map 인터페이스의 keySet 메소드는 Map 객체 안의 키 전부를 담은 Set 어댑터를 반환한다. keySet을 호출할 때마다 새로운 Set 인스턴스가 만들어지리라고 생각할 수도 있으나, 사실은 매번 같은 Set 인스턴스를 반환할지도 모른다. 

```java
public class UsingKeySet {

    public static void main(String[] args) {
        Map<String, Integer> menu = new HashMap<>();
        menu.put("Burger", 8);
        menu.put("Pizza", 9);

        Set<String> names1 = menu.keySet();
        Set<String> names2 = menu.keySet();

        names1.remove("Burger");
        System.out.println(names2.size()); // 1
        System.out.println(menu.size()); // 1
    }
}
```

출처 [keesun/study](https://github.com/keesun/study/blob/master/effective-java/item6.md)

### 오토박싱

---

오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 

**오토박싱은 기본 타입과 그에 대응하는 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.**

의미상 별다를 것이 없지만 성능상 그렇지 않다 (아이템61)

```java
private static long sum() {
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++)
	sum += i;
    return sum;
}
```

이 프로그램은 정확한 답을 내기는 하지만, 제대로 구현했을 때보다 훨씬 느리다. 

sum 변수를 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 약 231개나 만들어진 것이다.(long 타입인 i가 Long 타입인 sum으로 더해질 때마다)

> 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

이번 아이템을 "객체 생성이 비싸니 피해야 한다"로 오해하면 안된다. 특히나 요즘의 JVM에서는 별다른 일을 하지 않은 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다. 

 거꾸로, 아주 무거운 객체가 아니고서야 단순히 객체 생성을 피하고자 여러분만의 객체 풀을 만들지는 말자.
일반적으로 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. (DB connection 풀 말고)

이번 아이템은 방어적 복사를 다루는 아이템50과 대조적이다. 이번 아이템이 "기존 객체를 재사용해야 한다면 새로운 객체를 만들지 말자" 라면 아이템50은 "새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라"다. 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자.
