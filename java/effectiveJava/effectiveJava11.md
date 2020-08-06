2020-08-05 



# 이펙티브 자바 Item11

# Item11. equals를 재정의하려거든 hashcode도 재정의해야한다.

**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다.**

그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

Object 명세에서 발췌한 규약이다.

- equals 비교에서 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야한다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 항상  같은 값을 반환해야한다.
- equals(Object)가 두 객체를 다르다고 판단헀더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다.  즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.** 

예를 들어, [아이템10](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava10.md) 에서 구현했던 PhoneNumber 클래스를 HashMap에 넣는다고 할 때,  

**m.put(new PhoneNumber(000,000,1111), SH)** 로 넣고 **m.get(new PhoneNumber(000,000,1111))**를 실행하면 "SH"가 나와야 할 것 같지만 실제로는 null이 반환되는 데 그 이유가 hashcode를 재정의 안했기 때문이다. 그래서 논리적 동치인 두 인스턴스가 다른 해시코드를 반환하여 값이 틀리게 되는 것이다. 

## 잘못된 hashCode 구현

```java
@Override public int hashcode() { return 42;}
```

***위의 작성법은 적법하지만 최악의 해시코드 구현이다.*** 

최악인 이유는 모든 객체에게 똑같은 값만 내주어 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트 처럼 동작하고 그 결과 평균 수행시간이 O(1)인 해시테이블이 O(n)으로 느려지게된다.

## 좋은 hashCode 구현 요령

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫번재 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다.(여기서 핵심 필드란 equals 비교에 사용되는 필드를 말한다.)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다. 

    a.  해당 필드의 해시코드 c를 계산한다.

        i. 기본 타입 필드라면, Type.hashcode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다. 

        ii. 참조 타입 필드면서 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출해 비교한다면 이 필드의 hashcode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.

        iii. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashcode를 사용한다.

    b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.

    result = 31 * result + c;

3. result를 반환한다. 

파생 필드는 해시코드 계산에서 제외해도 좋고, equals 비교에 사용되지 않은 필드는 '반드시' 제외해야 한다. 그렇지 않으면 hashCode 두번째 규약을 어기게 된다. 

곱셈은 필드를 곱하는 순서에 따라 result가 달라진다. 그 결과 클래스에 비슷한 필드가 여러개 있을 때 해시 효과를 크게 볼 수 있다. 그 예로, String의 hashCode를 곱셈 없이 구현한다면 모든 아나그램의 해시코드가 같아진다. 

곱할 숫자가 31인 이유는 홀수이면서 소수이기 때문이다. (짝수이고 오버플로우 발생대비 2를 곱하는 것은 시프트연산과 같다.) 결과적으로 31을 곱하는 것은 시프트연산과 뺄셈으로 최적화가 가능하다. (31 * i == ( i << 5) -i )

그렇다면 예로 [아이템10](https://github.com/bosuksh/TIL/blob/master/java/effectiveJava/effectiveJava10.md)에서 만든 PhoneNumber의 hashCode를 구현해보자. 

```java
@Override
public int hashCode(){
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
}
```

이 메소드는 PhoneNumber 인스턴스의 핵심 필드 3개만을 사용해 간단한 계산만 수행한다. 그리고 동치인 인스턴스는 같은 해시코드를 가질 것이다. 

---

Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 hash라는 정적 메소드를 제공한다. 

그러나 속도는 약간 느리다. 

```java
@Override
public int hashCode(){
    return Objects.hash(lineNum,prefix,areaCode
}
```

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식을 고려해야 한다. 

이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야한다. 

해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화(lazy Initialization) 전략을 어떨까? 필드를 지연 초기화하려면 그 클래스를 스레드 안전하게 만들도록 신경 써야 한다(아이템83)

---

지연 초기화하는 hashCode 메소드- 스레드 안정성까지 고려해야한다.

```java
private int hashCode; //자동으로 0 초기화

@Override int hashCode() {
    int result = hashCode;
    if(result == 0) {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	hashCode = result;
    }
    return result;
}
```

**성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.** 

속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다. 특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려 주는 효과가 있을지도 모른다. 

hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.
