# 📗 IoC 컨테이너와 빈      
`Spring framework` 에서 말하는 `IoC`는 `DI`와 동일하다고 말한다.(Spring 레퍼런스에서 직접 언급)          
즉, **어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.**      
  
스프링은 `스프링 설정`과 `애플리케이션 구현`과 관련된 `Bean`들을 `스프링 컨테이너`에 저장한다.       

## 스프링 IoC 컨테이너
스프링 IoC 컨테이너는 **✔BeanFactory**를 기반으로 구현된 구현체이다.         
애플리케이션 컴포넌트의 중앙 저장소의 역할을 맡고 있으며            
**✔빈 설정 소스**로 부터 **✔빈 정의**를 읽어들이고, 빈을 구성하고 제공한다.           

## 빈 설정 소스
### XML 기반
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

### 자바 @Configuration 기반
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

### 자바 @Component 기반
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
   
참고로 빈 등록이 가능한 이유는 해당 어노테이션 내부에 `@Component` 어노테이션이 존재하기 때문이다.         
`@ComponentScan`이 `클래스 패스`를 기준으로 타고 내려오면서 `@Component`이 붙은 클래스를 자동으로 빈 등록을한다.     
사실, `@Configuration`도 `@Component`가 붙어있어 빈으로 등록되고,      
빈은 실행할 수 있는 객체이기에 `@Bean`이 붙은 메서드를 호출하면서 반환된 값들을 빈 등록 했던 것이다.         
참고로 이렇게, 내부에서 `어노테이션을 보조하는 어노테이션`을 **메타 어노테이션**이라고도한다.       
     
```java
@Configuration // 1. AppConfig를 빈 등록한다. 
public class AppConfig {

    @Bean // 2. IoC Container에서 @Bean이 붙은 메서드를 찾아 호출하고 결과값을 빈으로 등록한다.  
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }
```   
   
### 빈 정의(스프링 빈 설정 메타 정보 - BeanDefinition)      
**스프링은 어떻게 `XML`, `JAVA` 방식과 같은 다양한 설정 형식을 지원하는 것일까? 🤔**    
그 중심에는 `BeanDefinition`이라는 인터페이스가 있다. (빈 정보 자체를 추상화)     
     
쉽게 이야기해서 **`역할`과 `구현`을 개념적으로 나눈 것이다.**        
* `XML`을 읽어서 `BeanDefinition`을 만들면 된다.   
* 자바코드를 읽어서 `BeanDefinition`을 만들면 된다.   
* 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 오로지 `BeanDefinition`만 알면 된다.   
                
**`BeanDefinition`을 빈 메타 정보라고 말한다.**         
* **`@Bean`, `<bean>`당 각각 하나씩 메타 정보가 생성된다.**         
* 스프링 컨테이너는 이 메타 정보를 기반으로 스프링 빈을 생성한다.         
    
[사진]()    
[사진]()    
           
`BeanDefinition` 인터페이스를 구현한 클래스들은        
`BeanDefinitionReader`인터페이스를 구현한 구현체를 의존하고 있다.        
        
`BeanDefinitionReader` 인터페이스는 설정파일(자바,xml,..등등)에서 정보를 읽는 역할을 한다.            
그리고 설정파일을 기반으로 `BeanDefinition`을 생성하는 역할을 한다.         
정확히 말하면 `Bean`의 메타정보를 읽어오는 것이다.    
    
* `AnnotationConfigApplicationContext`는 `AnnotatedBeanDefinitionReader`를 사용해서     
  `AppConfig.class`를 읽고, `BeanDefinition`을 생성한다.      
* `GenericXmlConfigApplicationContext`는 `XmlBeanDefinitionReader`를 사용해서     
  `AppConfig.xml`을 읽고, `BeanDefinition`을 생성한다.       
* 새로운 형식의 설정 정보가 추가되면, `XxxBeanDefinitionReader`를 만들어서 `BeanDefinition`을 생성하면 된다.  
       
**`XxxBeanDefinitionReader` 만드는 방법은? 🤔**        
구글링을 해도 방법이 나오지 않는다!! (오히려 만들면 최초이기에 흥미가 돋는다.)         
지금은 집중해야할 것이 있으니 나중에 만드는 방법을 직접 찾아봐야겠다.       
`JSONBeanDefinition`이라는 주제로 만들어보자  

## BeanDefinition 살펴보기 
* **BeanClassName:** 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)
* **factoryBeanName:** 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
* **factoryMethodName:** 빈을 생성할 팩토리 메서드 지정, 예) memberService
* **Scope:** 싱글톤(기본값)
* **lazyInit:** 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연처리 하는지 여부  
* **InitMethodName:** 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명    
* **DestroyMethodName:** 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명    
* **Constructor arguments, Properties:** 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)     

**코드로 확인해보기**
```java
package hello.core.beandefinition;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class BeanDefinitionTest {

    AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    GenericXmlApplicationContext xmlApplicationContext = new GenericXmlApplicationContext("appConfig.xml");

    @Test
    @DisplayName("빈 설정 메타정보 확인")
    void findAnnotationApplicationBean() {
        String[] beanDefinitionNames = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = annotationConfigApplicationContext.getBeanDefinition(beanDefinitionName);

            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                System.out.println("beanDefinitionName = " + beanDefinitionName +
                        " beanDefinition = " + beanDefinition);
            }
        }
    }

    @Test
    @DisplayName("빈 설정 메타정보 확인")
    void findXmlApplicationBean() {
        String[] beanDefinitionNames = xmlApplicationContext.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = xmlApplicationContext.getBeanDefinition(beanDefinitionName);

            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                System.out.println("beanDefinitionName = " + beanDefinitionName +
                        " beanDefinition = " + beanDefinition);
            }
        }
    }

}
```
   
**정리**     
* `BeanDefinition`을 직접 생성해서 스프링 컨테이너에 등록할 수 도 있다.         
  하지만 실무에서 `BeanDefinition`을 직접 정의하거나 사용할 일은 거의 없다.           
  그래도 나중에 직접 구현을 해보는 편이 개발자 다운 삶이지 않을까 싶다.        
* `BeanDefinition`에 대해서는 너무 깊이있게 이해하기 보다는,     
  스프링이 다양한 형태의 설정 정보를 `BeanDefinition`으로 추상화해서 사용하는 것 정도만 이해하자      
* 가끔 스프링 코드나 스프링 관련 오픈 소스의 코드를 볼 때,`BeanDefinition` 이라는 것이 보일 때가 있다.    
  이때 이러한 메커니즘을 떠올리면 된다.   
      
**필자가 생각하기론**      
인스턴스를 만들어 반환하는 것은 단순히 자바 코드를 수행하기 위한 작업일 뿐이고      
`BeanDefinitionReader` 구현체가 해당 클래스를 통해 정보를 읽어들어와          
`BeanDefinition`이라는 메타데이터를 만들고, 이를 통해 다시 객체를 새로 생성한다고 생각한다.      
       
**참고로 `ApplicationContext`로 의존하지 않은 이유는? 🤔**   
* `ApplicationContext`에서는 `getBeanDefinition()`이라는 메서드가 존재하지 않는다.    

**그리고**   
`GenericXmlApplicationContext` 인스턴스는 
**factoryBeanName**, **factoryMethodName** 요소가 null로 빠져있다.   
           
사실 이러한 이유에 대해서 아래 내용을 참고하자   
스프링에 빈을 등록하는 방법이 크게 2가지로 나뉜다.   
          
1. 직접 스프링빈을 등록하는 방법 (xml)     
2. 팩터리 메서드를 이용해서 등록하는 방법(class)  
      
예전, XML로 설정하는 경우 직접 스프링빈을 등록하는 방법이기에 빈에 대한 클래스가 밖에 드러났는데           
`JAVAConfig`로 바뀌면서 팩토리 메서드(AppConfig와 메서드)를 이용하는 방법을 주로 사용하게 되었고      
`BeanDefinition` 또한 팩터리 클래스와 메서드를 사용하는지를 출력하도록 설정이 변경되었다.      
그렇기에 `xml`의 직접 스프링빈을 등록하는 방법은 팩터리 클래스와 메서드에 대해 `null`값을 출력한다.    





### 빈
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
