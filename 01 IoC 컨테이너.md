# 📗 IoC 컨테이너와 빈      
`Spring framework` 에서 말하는 `IoC`는 `DI`와 동일하다고 말한다.(Spring 레퍼런스에서 직접 언급)          
즉, **어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.**      
  
스프링은 `스프링 설정`과 `애플리케이션 구현`과 관련된 `Bean`들을 `스프링 컨테이너`에 저장한다.       

## 스프링 IoC 컨테이너
스프링 IoC 컨테이너는 **✔BeanFactory**를 기반으로 구현된 구현체이다.         
애플리케이션 컴포넌트의 중앙 저장소의 역할을 맡고 있으며            
**✔빈 설정 소스**로 부터 **✔빈 정의**를 읽어들이고, 빈을 구성하고 제공한다.           

### 빈 설정 소스
#### XML 기반
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="memberService" class="hello.core.member.MemberServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository"/>
    </bean>

    <bean id="memberRepository" class="hello.core.member.MemoryMemberRepository"/>

    <bean id="orderService" class="hello.core.order.OrderServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository"/>
        <constructor-arg name="discountPolicy" ref="discountPolicy"/>
    </bean>

    <bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy"/>
</beans>
```

#### 자바 @Configuration 기반
```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        System.out.println("call AppConfig.discountPolicy");
        return new RateDiscountPolicy();
    }

}
```

#### 자바 @Component 기반
* @Controller
* @RestController
* @Service
* @Repository
* 등등..  

```java
@Service
public class SampleService {
    ... // 생략 
}
```
   
이게 가능한 이유는 해당 어노테이션 내부에 `@Component` 어노테이션이 존재하기 때문이다.      
이렇듯 내부에서 `해당 어노테이션을 설명하는 어노테이션`을 **메타 어노테이션**이라고도한다.      

### 빈 정의  



빈
● 스프링 IoC 컨테이너가 관리 하는 객체.
● 장점
○ 의존성 관리
○ 스코프
■ 싱글톤: 하나
■ 프로포토타입: 매번 다른 객체
○ 라이프사이클 인터페이스


이러한, 스프링 컨테이너는 `Servlet 컨테이너`와 비슷하며 2가지 종류가 존재한다.        


**BeanFactory**   
* 스프링 설정 파일에 등록된 `Bean`을 생성하고 관리하는 가장 기본적인 컨테이너 기능만 제공     
* 처음부터 객체를 생성하지 않고,        
  클라이언트의 요청(Lookup)에 의해서만 `Bean`이 생성되는 **지연로딩 방식을 사용한다.**        
* 일반적인 스프링 프로젝트에서 `BeanFactroy`를 사용할 일은 거의 없다.          
   
**ApplicationContext**  
* `BeanFactory`를 상속받고 있다. (인터페이스 상속, `HierarchicalBeanFactory`)       
* 컨테이너식으로 동작하며 `트랜잭션 관리`나 `리소스 로딩 기능` 그리고 `메시지 기반의 다국어 처리` 등 다양한 기능을 지원한다.    
* 대부분 스프링 프로젝트는 `ApplicationContext` 유형의 스프링 컨테이너를 이용한다.          
* `BeanFactory`, `ApplicationEventPublisher`, `EnvironmentCapable`,   `HierarchicalBeanFactory`, `ListableBeanFactory`,   
  `MessageSource`, `ResourceLoader`, `ResourcePatternResolver`등의 인터페이스를 구현하고 있다.   
   
`BeanFactory` 같은 경우, 빈을 관리하는 기본적인 역할만 수행하기에        
우리가 스프링에서 사용하는 컨테이너는 대부분 `ApplicationContext`를 상속받은 구현체들이다.   
      
* ClassPathXmlApplicationContext (XML)
* AnnotationConfigApplicationContext (Java)

**빈 설정**    
* 빈 명세서  
* 빈에 대한 정의를 담고 있다.     
     
|Property|Explained in…|
|---|---|
|Class|Instantiating Beans|
|Name|Naming Beans|
|Scope|Bean Scopes|
|Constructor arguments|Dependency Injection|
|Properties|Dependency Injection|
|Autowiring mode|Autowiring Collaborators|
|Lazy initialization mode|Lazy-initialized Beans|
|Initialization method|Initialization Callbacks|
|Destruction method|Destruction Callbacks|
     
[빈팩토리와 라이프사이클](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/)   

# 📕 ApplicationContext 와 다양한 빈 설정 


# 📒 @Autowired 
빈 라이프 사이클에서 사용되는 `BeanPostProcessor` 인터페이스 구현체 AutoWiredAnnotationBeanPost 에 의해 가능한것이다.     

`BeanPostProcessor`는 빈을 만든 다음에, 빈의 초기화 과정 전후로 부가적인 작업을 할 수 있게 해주는 또다른 라이프 사이클 콜백이다.   
즉, 처음 빈이 만들어지고 init 메서드를 실행하기 전후로 콜백을 실행해서 의존 관계를 맺는것이다.      
 
ApplicationContext의 빈팩토리가, `BeanPostProcessor`를 찾는다.      
등록된 빈들 중에는 `AutoWiredAnnotationBeanPost`를 찾게 될 것이다.      
다른 일반적인 빈들에게 `AutoWiredAnnotationBeanPost`를 적용하는 것이다.   






[https://howtodoinjava.com/spring-core/spring-bean-life-cycle/](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/)   
