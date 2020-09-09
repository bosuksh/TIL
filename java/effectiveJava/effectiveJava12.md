2020-08-06


# Item12. toString을 항상 재정의하라

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다. 단순히 **클래스_이름@16진수로_표시한_해시코드**를 반환한다. 

toString의 일반 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
그리고 모든 하위 클래스에서 이 메소드를 재정의하라.

**toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.**

toString메소드는 우리가 직접 호출하지 않아도 다른 어딘가에서 쓰일 수 있다(printf, println, 문자열 연결(+), assert 등)

**실전 toString은 그 객체가 가진 주요 정보를 모두 반환하는 게 좋다.** 

하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기 적합하지 않다면 무리가 있다. 

toString을 구현할 때면 반환값의 포맷을 문서화 할지 정해야 한다. 전화번호나 행렬 같은 값 클래스라면 문서화하기를 권한다. 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호전환할 수 있는 정적 팩터리 메소드나 생성자를 제공해주면 좋다. (BigDecimal, BigInteger)

단점은 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다. 향후 릴리스에서 포맷을 바꾸면 이를 사용하던 코드들과 데이터가 엉망이 될 것이다. 

**포맷을 명시하든 아니든 아무튼 의도는 명확히 밝혀야한다.**

[아이템10](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava10.md), [아이템11](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava11.md)에서 봤던 PhoneNumber 클래스로 toString을 재정의 해보자

```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ"형태의 12글자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면
* 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override public String toString() {
	return String.format("%03d-%03d-%04d",areacode,prefix,lineNum);	
}
```

**포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.**

예컨대 PhoneNumber 클래스는 지역 코드, 프리픽스, 가입자 번호용 접근자를 제공해야 한다. 그렇지 않으면 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱할 수 밖에 없다. 

정적 유틸리티 클래스([아이템4](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava4.md))는 toString을 제공할 이유가 없다. 또한 대부분의 열거 타입(아이템34)도 자바가 이미 완벽한 toString 클래스를 제공하니 따로 재정의하지 않아도 된다. 

하지만 하위 클래스들이 공유해야할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해야한다.

아이템10에서 소개한 구글의 AutoValue 프레임워크 역시 toString을 생성해준다.

그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.
