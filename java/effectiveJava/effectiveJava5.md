2020-07-11

# 이펙티브 자바 Item5

# Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다. 

예를 들어 맞춤법 검사기는 사전에 의존하는데, 이 클래스는 정적 유틸리티 클래스([아이템4](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava4.md))나 싱글턴([아이템3](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava3.md))으로 구현하는 경우가 있다.

```java
public interface Lexicon { ... }

// Lexicon의 구현체 
public class EnglishDictionary implements Lexicon { ... }

// 정적 유틸리티 클래스
public class SpellChecker {
		private static final Lexicon dictionary = new EnglishDictionary();

		private SpellChecker() { ... }
		
		public static boolean inValid(String word) { ... }
		public static List<String> suggestions(String typo) { ... }
}

// Singleton 클래스
public class SpellChecker {
		private static final Lexicon dictionary = new EnglishDictionary();
		
		private SpellChecker() { ... }
		public static SpellChecker INSTANCE = new SpellChecker();
		
		public boolean inValid(String word) { ... }
		public List<String> suggestions(String typo) { ... }
}
```

그러나 두 방식 모두 사전을 단 하나만 사용한다고 가정하게 되는데, 사실 사전은 언어별로 따로 있고, 특수 어휘용 사전이 별도로 있기도 한다. 

**즉, 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적절하지 않다.** 

```java
public class SpellChecker {
		private final Lexicon dictionary;

		public SpellChecker(Lexicon dictionary) { 
					this.dictionary = Object.requireNonNull(dictionary);
		}

		public boolean inValid(String word) { ... }
		public List<String> suggestions(String typo) { ... }

}
```

**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다.**

불변(아이템17)을 보장하여 같은 자원을 사용하려는 여러 클라이언트가 의존 객체를 안심하고 공유할 수 있기도 한다. 의존 객체 주입은  생성자, 정적팩터리(아이템1), 빌더([아이템2](https://github.com/bosuksh/TIL/blob/java/java/effectiveJava/effectiveJava2.md)) 모두에 똑같이 응용할 수 있다. 

이 패턴의 변형으로는 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 

팩토리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.
자바 8에서 소개한 **Supplier<T>**  인터페이스가 팩토리를 표현한 완벽한 예다. 

의존 객체 주입이 유연성과 테스트 용이성을 개선해주기는 하지만, 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들 수 있다. 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용하면 이런 어질러짐을 해소할 수 있다.


> 클래스가 내부적으로 하나 이상 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면  필요한 자원을 생성자에 넘겨주자.
