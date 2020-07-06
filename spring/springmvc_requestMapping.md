2020-07-06



Spring MVC를 보다가 @RequestMapping의 우선순위에 대해서 궁금함이 생겼다. 

1. interface에 @Controller 어노테이션을 붙여도 작동하지 않는다. 

```java
@Controller
public interface SampleController {
    @GetMapping("/hello")
    public String hello(Model model);
}



public class SampleControllerImpl implements SampleController{

    public String hello(Model model) {
        model.addAttribute("name","doflamingo" );
        return "hello";
    }

}
```

아래와 같이 실행을 했을 때는 실행되지 않음을 확인했다. 따라서 Interface로 Controller를 구현하더라도 

interface의 구현체에 @Controller 어노테이션을 붙여야한다. 



2. 추상클래스, 인터페이스, 구현클래스의 @RequestMapping 우선순위는 

   구현클래스 > 인터페이스 > 추상클래스 순서이다. 

```java

public interface SampleController {
  
  	@RequestMapping("/foo")
    public String foo(Model model);
}

public abstract class AbsSampleController implements SampleController{

    @Override
    @RequestMapping("/foo2")
    public abstract String foo(Model model);
}

@Controller
public class SampleControllerImpl implements SampleController{

 	 	@Override
    public String foo(Model model) {
        return "foo";
    }

}
```





![image-20200706204038912](/Users/leesanghyup/Desktop/dev/til/spring/img/img1.png)



![image-20200706204122401](/Users/leesanghyup/Desktop/dev/til/spring/img/img2.png)

물론 구현체의 우선순위가 제일 높다.(이건 확인 안해도 알겠지)