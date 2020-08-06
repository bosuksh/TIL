2020-07-07

# Item2. 생성자에 매개변수가 많다면 빌더를 고려하라.

## 문제점

정적 팩토리 메서드와 생성자에는 선택적 매개변수가 많을 때 적절하게 대응하기 어렵다는 점이있다. 

예를 들어 식품 포장의 영양정보를 표현하는 클래스를 생각해보면 1회 내용량, 총 n회 제공량, 1회 제공당 칼로리 등몇개의 필수 항목과  총 지방, 트랜스 지방,  포화지방 등 20가지 넘는 선택 항목으로 이뤄진다. 그런데 대부분 이 선택 항목은 0 이다. 

 

## 해결책 1. 점층적 생성자 패턴

필수 매개변수만 받는 생성자, 필수 매개 변수와 선택 매개변수 1개를 받는 생성자, 필수 매개 변수와 선택 매개변수 2개까지 받는 생성자 이렇게 점층적으로 늘려가는 방식이다. 그래서 이중에서 필요한 가장 짧은 생성자를 사용하면 된다. 

```java
NutritionFacts cocaCola = new NutritionFactor(240, 8, 100, 0, 35, 27);
```

**그러나 점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.** 

매개변수가 뭘 의미하는지 헷갈릴 것이고, 매개변수가 몇 개인지도 주의해서 세어보아야 한다. 
또한 클라이언트가 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다. (아이템51)

## 해결책 2. 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.
자바빈즈 패턴을 이용하면 코드가 길어지긴 했지만 인스턴스를 만들기 쉽고, 더 읽기 쉬운 코드 가 되었다. 

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServing(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbonhydrate(27);
```

**자바빈즈 패턴에서는 객체 하나를 만들려면 메서드 여러개를 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.
자바빈즈 패턴에서는 클래스를 불변(아이템17)으로 만들 수 없다.**

## 해결책3 빌더 패턴

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다.
그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수를 설정한다. 

```java
public class NutritoinFacts{
	private final int servingSize;     
	private final int servings;       
	private final int calories;        
	private final int fat;            
	private final int sodium;          
	private final int carbonhydrate;   

	public static class Builder{
		//필수 매개변수
		private final int servingSize;     
		private final int servings;        
		//선택 매개변수 - 기본값으로 초기화한다. 
		private int calories      = 0;
		private int fat           = 0;
		private int sodium        = 0;
		private int carbonhydrate = 0;

		public Builder(int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings    = servings;
		}
		public Builder calories(int val){
				calories = val;
				return this;
		}
		public Builder fat(int val){
				fat = val;
				return this;
		}
		public Builder sodium(int val){
				sodium = val;
				return this;
		}
		public Builder carbonhydrate(int val){
				carbonhydrate = val;
				return this;
		}
		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
	private NutritionFacts(Builder builder) {
		servingSize   = builder.servingSize;
		servings      = builder.servings;
		calories      = builder.calories;
		fat           = builder.fat;			
		sodium        = builder.sodium;
		carbonhydrate = builder.carbonhydrate;
	}
}
```

NutritionFacts 클래스는 불변이며, 모든 매개변수의 기본값들을 한곳에 모아뒀다. 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 

이런 방식을 fluent API 혹은 method chaining이라고 한다. 

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
				.calories(100).sodium(35).carbonhydrate(27).build();
```

**빌더패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것이다.** 

불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야한다. (아이템50)
검사해서 잘못된 점을 발견하면 어떤 매개변수가 잘못되었는지 알려주는 IllegalArgumentException을 던지면 된다.(아이템 75)

**빌더패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.**

각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza<set> {
    public enum Topping{ HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    final Set<Topping> toppings;

    abstract static class Builder<T extends  Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();
				
				//하위클래스는 이 메서드를 overriding하여
				//"this"를 반환하도록 해야한다.
        protected abstract T self();
    }
    Pizza(Builder<?> builder){
        toppings = builder.toppings.clone();    //아이템50 참조
    }
}
```

Pizza.Builder 클래스는 재귀적 타입 한정(아이템30)을 이용하는 제네릭 타입이다. 여기에 추상메서드인 self()를 더해 하위클래스에서 형변환하지 않고도 method chaining을 지원할 수 있다. 

여기 Pizza의 하위클래스 2개가 있다. 

하나는 일반적인 뉴욕피자이고, 다른 하나는 칼초네 피자이다. 뉴욕피자는 크기를 매개변수로 필수로 받고, 칼초네 피자는 소스를 안에 넣을지 선택하는 매개변수를 필수로 받는다. 

```java
public class NyPizza extends Pizza {

    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false;

        public Builder sauceInside(boolean sauceInside) {
            sauceInside = true;
            return this;
        }
        @Override
        Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
   private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
				.addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone.Builder()
				.addTopping(HAM).sauceInside().build();
```

생성자로는 누릴 수 없는 사소한 이점으로, 빌더를 이용하면 가변인수 매개변수를 여러개 사용할 수 있다. 

## 빌더패턴의 단점

빌더패턴의 단점은 객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다. 또한, 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 

> 생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다.

매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
