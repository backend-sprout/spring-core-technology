# IoC 개요    
`IoC(inversion of control)`란, 이름 그대로 `제어의 역전`을 의미한다.    
             
**객체를 생성하고 관리하는 `제어`를 역전시키는 것**으로              
하나의 클래스에서 객체를 `생성`하고 `사용`하면서 관리하는 것이 아니라   
`객체의 생성`은 외부에 맡기고 `객체의 사용`만을 관리하는 것이다.      
그리고 외부에 맡긴다는 말은 **프레임워크에 빈을 생성하고 관리하는 것을 맡긴다는 것이다.**  
             
**`IoC`라는 개념이 도입되기 사용되기 전에는**           
* 애플리케이션 수행에 필요한 **객체의 생성이나 객체와 객체 사이의 의존 관계를 개발자가 직접 처리했다.**       
* 이런 상황에서 **의존 관계에 있는 객체를 변경할 때, 반드시 코드를 수정해야 한다는 문제가 발생했다.**           
* 결과적으로 `SOLID 원칙`을 위배한 결합도 높은 코드를 작성하고 있었다.             
    * `생성`과 `사용`이라는 2가지 책임이 있으므로 `SRP(단일 책임 원칙)` 위배     
    * 기능의 확장이나 변경을 꾀한다면 코드를 수정해야하기에 `OCP(개방폐쇄원칙)` 위배

**`IoC`가 적용되면서**
* **객체 생성을 컨테이너(빈 컨테이너)가 대신 처리한다.**    
* **객체와 객체 사이의 의존관계 역시 컨테이너가 처리한다.**        
* 결과적으로 **소스에 `의존 관계`가 명시되지 않으므로 결합도가 떨어져서 유지보수가 편리해진다.**      
   
# IoC 컨테이너와 빈      
`Spring framework` 에서 말하는 `IoC`는 `DI`와 동일하다고 말한다.(Spring 레퍼런스에서 직접 언급)        
즉, **어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.**   
  
스프링은 `스프링 설정`과 `애플리케이션 구현`과 관련된 `Bean`들을 `스프링 컨테이너`에 저장한다.       
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
   
# ApplicationContext 와 다양한 빈 설정 방법   
# Autowired 
[https://howtodoinjava.com/spring-core/spring-bean-life-cycle/](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/)   
