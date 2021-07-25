AOP
=======
# 📙 AOP 개요 
## 📖 시나리오

```java
public void join(JoinRequest joinRequest) {
  StopWatch stopWatch = new StopWatch();
  stopWatch.strat();
  try {
    memberRepository.save(joinRequest.toMember());
  } finally {
    stopWatch.stop();
    log.info("join spen {} ms", stopWatch.getLastTaskTimeMillis());
  }
}
```         
회원가입을 진행하는 Service 코드다.                      
성능 부하가 의심되어 해당 메서드에 시간을 측정하는 코드를 작성했다.                 
그러나, 코드를 보면 알 수 있듯이 **가독성이 떨어지고 타 클래스에 대한 결합도가 높아져 응집성이 떨어진다.**                     
그리고 무엇보다도 **많은 클래스에서 코드의 변경이 필요할 때 이를 한 번에 관리할 수 있을지 의문이다.**           
                     
* 각각의 메서드는 1가지 기능만 수행해야 한다.       
* 클래스와 그 메서드의 본질적인 역할은 **비즈니스 로직을 수행하는 것이다.**          
* 인프라 로직이 삽입되면 비즈니스 로직에 대한 가독성은 줄어들고 관리해야 할 코드는 많아진다.         
  * **핵심 기능 :** 비지니스 로직  
  * **부가 기능 :** 로깅 로직      
       
위 요소들을 고려해보면, 핵심 기능 로직에서 부가 기능 로직을 제거해야한다.     
하지만 성능측정, 로깅에 대한 작업은 프로그래밍 개발에 있어서 필수적인 부분이다.         
그렇다면 **이런 부가 기능 로직들을 분리해서 사용할 수 있는 방법은 없을까? 있다. 바로 `AOP`이다.**       
        
## 📖 부가 기능   
> 인프라 로직  
   
<img width="1282" alt="스크린샷 2020-11-16 오후 1 05 46" src="https://user-images.githubusercontent.com/50267433/99211839-a4879680-280c-11eb-897d-c2970351f3bb.png">   

위와 같은 부가적인 기능들이 각각의 핵심 기능위에 블록을 쌓듯이 중복되어 존재하는 것을 알 수 있다.         
또한 같은 블록을 동일 선상에 두어보면 횡단하는 듯한 모습 또한 볼 수 있다.       
      
<img width="1285" alt="스크린샷 2020-11-16 오후 1 06 43" src="https://user-images.githubusercontent.com/50267433/99211873-b9fcc080-280c-11eb-8c39-d98b7dd63e80.png">
     
* **cross-cutting concern :** 부가적인 기능인 인프라 로직이 중복되어 횡단(가로)과 같은 모습을 나타내는 로직      
* **core concern :** 사용자의 요청에 따라 실제로 수행되는 핵심 비즈니스 로직        
   
**인프라 로직**          
인프라 로직은 애플리케이션의 전 영역에서 나타날 수 있다.          
그러나 `cross-cutting concern`에서 알 수 있듯이 각각의 영역에서 중복된 코드가 발생하고 있다.             
이로 인해 유지보수가 힘들어지고 비즈니스 로직을 이해하기 어렵게 한다는 문제를 가지고 있다.        

# 📕 AOP  
> Aspect-Oriented Programming | 관점 지향 프로그래밍   
   
* OOP 객체지향 프로그램   
* AOP 관점지향 프로그램    
       
AOP는 OOP를 더욱 OOP답게 프로그래밍 할 수 있게 도와주는 것으로      
**애플리케이션의 `핵심적인 기능`과 `부가적인 기능`을 `분리`해**    
**`Aspect`라는 모듈로 만들어 설계하고 개발하는 방법이다.**        

실제로 Spring doc document 에서는 아래와 같이 말하고 있다.  
```   
Aspect-oriented Programming (AOP) complements    
Object-oriented Programming (OOP) by providing   
another way of thinking about program structure   

AOP는 프로그램 구조에 대한 다른 생각의 방향을 제공해주면서 OOP를 보완하고 있다.       
```      
  
예를 들면 어떠한 클래스를 대상으로 `핵심 기능`과 `부가적인 기능`의 관점으로         
**공통 사용 부가 기능들을 외부의 독립된 클래스로 분리하고**                
**이를 모듈화하여 재사용할 수 있게끔 하는 프로그래밍 기법이다.**                 
               
독립된 클래스/모듈을 Aspect라고 부르며           
Aspect는 여러 기능들이 복합적으로 모여 있는 것이 아닌,   
`성능 측정`, `로깅`과 같이 **한 가지 특정 기능에 대해 관심사를 분리한 클래스다.**     
   
# AOP 용어 

|용어|설명|
|----|----|
|**Aspect(Advisor)**|**✔ 특정 한 가지 기능에 대해 흩어진 관심사를 모듈화하여 묶은 클래스**<br>- `PointCut + Advice`를 묶은 메서드를 정의 및 관리한다.|
|Target|**✔ 비즈니스(핵심) 로직을 구현하는 AOP의 대상 '클래스'**|  
|Advice|**✔ AOP 메서드 : 부가 기능을 적용시키는 메서드(부가 기능 로직 메서드)**<br>**적용 시점도 지정할 수 있다.**<br>- before : 비즈니스 메소드 실행 전에 동작<br>- after : 비즈니스 메소드 실행 후에 동작<br>- after-returning : 비즈니스 메소드 실행 중 리턴 되는 순간<br>- after-throwing : 비즈니스 메소드 실행 중 에러 발생 순간<br>- around	: 비즈니스 메소드 실행 전/후에 동작| 
|JoinPoint|**✔ AOP가 적용될 수 있는 `메소드`, 필드, 객체, 생성자 등의 요소**<br>- 타겟 클래스의 요소들을 의미하며 **PointCut의 후보**<br>- **Spring AOP에서는 메서드만 지정 가능**|
|PointCut|**✔ AOP 적용 가능 요소들 중에서 '실제 적용될 요소들'을 의미한다.**<br>- `JoinPoint`에서 실제 `Advice`가 적용될 지점<br>- **Spring AOP 에서는 advice가 적용될 메서드를 의미**|    
|Weaving|**✔  PointCut이 호출될 때 Advice가 호출되는 과정을 의미한다.**<br>즉, **PointCut에 Advice 메서드가 삽입되는 과정을 의미한다.**<br>- 컴파일 전에는 핵심 로직과 인프라 로직이 분리되어있지만<br>- 컴파일타임에 바이트 코드를 넣는다던지 런타임에 프록시를 만들던지<br>- 관심사를 분리한 코드를 비즈니스 코드와 병합하는 작업을 의미한다.<br>**Weaving 처리 방식 (AOP 구현 방법)**<br>- 컴파일 타임 위빙 : `a.java -> a.clss` 컴파일 될 때<br>- 로딩 타임 위빙 : 클래스파일을 클래스 로더가 메모리에 로드할 때<br>- 런타임/프록시 위빙 : 타겟 클래스에 부가 기능을 추가하도록 Proxy 객체를 만들어(감싸서) 실행<br>- 스프링 AOP는 **런타임 위빙**만을 지원한다.(IOC/DI를 이용한 방법)|    
|proxy|**✔ Target 객체에 Advice를 적용시키는 래퍼 클래스**|    

![AOP](https://user-images.githubusercontent.com/50267433/126900960-e0ed4a26-9521-49ac-bfd3-c9a915c12772.png)    
        
1. 애플리케이션은 자연스럽게 여러 메서드(Join Point)를 호출한다.      
2. 이때 특정 **PointCut으로 지정한 메소드가 호출되는 순간**       
3. **Aspect 의 Advice 메소드가 호출된다.**             
4. Advice 메소드의 **동작 시점은 5가지로 정할 수 있다.**               
5. 애스팩트 설정에 따라 **위빙 처리되어 프록시 객체가 생성된다.(동적 프록시 생성)**                 
6. 프록시 객체를 통해 부가기능이 포함된 비즈니스 로직을 수행한다.              

## 📖 AOP 구현 
```gradle
implementation 'org.springframework.boot:spring-boot-starter-aop'
```
```java
// @EnableAspectJAutoProxy 생략 가능 
@SpringBootApplication
public class Application {
    public static void main(String[] args) { SpringApplication.run(Application.class,args); }
}
```
```java
@Service
public class AuthServiceImpl {
    public void businessLogicMethod(){
        System.out.println("businessLogicMethod process!");
    }
}
```

### 📄 일반적인 구현      
```java
@Aspect
@Configuration
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("execution(* me.kwj1270.study.service.AuthServiceImpl.*(..))")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("execution(* me.kwj1270.study.service.AuthServiceImpl.*(..))")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```

### 📄 어노테이션으로 구현   
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerformanceCheck {
}
```
```java
@Service
public class AuthServiceImpl {
    @PerformanceCheck
    public void businessLogicMethod(){
        System.out.println("businessLogicMethod process!");
    }
}
```  
```java
@Aspect
@Component
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("@annotation(me.kwj1270.study.annotation.PerformanceCheck)")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("@annotation(me.kwj1270.study.annotation.PerformanceCheck)")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```
* `어노테이션`과 `Aspect`가 동일 위치면 어노테이션만 적어도 된다.   
* 패키지가 다르면 FQCN(Fully qualified Class Name)을 다 입력해주어야 한다.    

### 📄 빈으로 구현   
```java
@Aspect
@Component
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("bean(me.kwj1270.study.service.AuthServiceImpl)")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("bean(me.kwj1270.study.service.AuthServiceImpl)")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```
* `빈`과 `Aspect`가 동일 위치면 어노테이션만 적어도 된다.   
* 패키지가 다르면 FQCN(Fully qualified Class Name)을 다 입력해주어야 한다.    
   
## 📖 AOP PointCut 설정  
```java
execution(* com.springbook.biz..*Impl.get*(..))"
```   

* `*` : 리턴값
* `com.springbook.biz..` : 패키지
* `*Impl`: 클래스 이름
* `get*` : 메서드 이름
* `(..)` : 매개변수 
* 중간마다의 `.` : 구분점   

### 📄 execution 포인트 컷 리턴

|표현식|설명| 
|----|---|   
|`*`|모든 리턴타입 허용|   
|`void`|리턴타입이 void인 메서드 선택|   
|`!void`|리턴타입이 void가 아닌 메서드 선택|   

### 📄 execution 포인트 컷 패키지 

|표현식|설명|  
|----|---|      
|`com.springbook.biz`|정학하게 해당 패키지만 선택|      
|`com.springbook.biz..`|해당 패키지 및 모든 하위 패키지 선택|      
|`com.springbook..impl`|`..`앞 패키지로 시작하면서 마지막 패키지 이름이 `..`뒤로 끝나는 패키지 선택|          
   
### 📄 execution 포인트 컷 클래스
|표현식|설명|  
|----|---|      
|`BoardServiceImpl`|정학하게 해당 클래스만 선택|      
|`*Impl`|클래스 이름이 `*`뒷 글자로 끝나는 클래스만 선택|      
|`BoardService+`|해당 클래스는 물론 파생된 모든 자식 클래스도 선택 가능|
|`variable+`|해당 인터페이스를 구현한 모든 클래스 선택 가능 |    

### 📄 execution execution 포인트 컷 메서드

|표현식|설명|
|----|---|
|`*(..)`|가장 기본 설정으로 모든 메서드 선택|
|`get*(..)`|메서드 이름이 get으로 시작하는 모든 메서드 선택|
|`매서드이름(..)`|특정 메서드 이름을 가진 메서드 선택|

### 📄 execution 포인트 컷 매개변수 
   
|표현식|설명|
|----|---|
|`(..)`|가장 기본 설정으로서 매개변수 타입의 제한이 없음을 의미|
|`(*)`|반드시 1개의 매개변수를 가지는 메서드만 허용한다는 의미|
|`(com.spring.user.User)`|해당 패키지의 클래스를 가지는 메서드만 허용한다는 의미|
|`(!com.spring.user.User)`|해당 패키지의 클래스를 가지지 않는 메서드만 허용한다는 의미|
|`(Integer, ..)`|한 개 이상의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
|`(Integer, *)`|반드시 두 개의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
        
   
# 🔍 런타임/프록시 위빙   
* Proxy를 생성하여 실제 타깃(Target) 오브젝트의 변형없이 위빙을 수행한다.    
* 실제 런타임 상, Method 호출 시에 위빙이 이루어 지는 방식이다.     
* 소스파일, 클래스 파일에 대한 변형이 없다는 장점이 있지만, 포인트 컷에 대한 어드바이스 적용 갯수가 늘어 날수록 성능이 떨어진다는 단점이 있다.    
* 또한 메소드 호출에 대해서만 어드바이스를 적용 할 수 있다.       
     
**org.woowacourse.aoppractice.controller.AopController**     
```java
package org.woowacourse.aoppractice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.woowacourse.aoppractice.service.AuthServiceImpl;

@RestController
public class AopController {
    
    private final AuthServiceImpl authService;
    
    public AopController(AuthServiceImpl authService){
        System.out.println(authService.getClass().getName());
        this.authService = authService;
    }

    @GetMapping("/")
    public void logTest(){
        authService.businessLogicMethod();
    }

}
```

**결과**
```
org.woowacourse.aoppractice.service.AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d
```

* 메서드를 감싸는 것이 아닌 Target 클래스를 프록시로 감싸는 것을 알 수 있다.   
* 물론 AOP를 사용하지 않으면 `org.woowacourse.aoppractice.service.AuthServiceImpl`가 출력된다.   
* 기존 객체 : AuthServiceImpl      
* 프록시 객체 : AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d    
* CGLIB란? : https://www.youtube.com/watch?v=RHxTV7qFV7M    
  * 간략히 말하면 클래스 상속을 이용해서 만든 프록시 객체를 의미    
  * 추후에 정리할 예정        
    
<img width="1285" alt="스크린샷 2020-11-17 오전 11 06 34" src="https://user-images.githubusercontent.com/50267433/99337118-0b688680-28c5-11eb-9c99-b0992130f269.png">   

* 기존에는 `AuthController(AopController)` 가 `AuthService`를 의존(참조함)  
* 프록시 위빙을 적용하면 `AuthService`를 상속받은 `AuthService$$블라블라`를 의존  
* 상속을 이용한 방식이기에 다형성을 이용하여 하위 클래스인 `AuthService$$블라블라`를 의존 참조할 수 있는 것이다.
  * `private final AuthServiceImpl authService;`
  * `public AopController(AuthServiceImpl authService){this.authService = authService;}`
* `AuthService$$블라블라`는 `AuthService`를 상속받은 클래스인데 **private는 어떻게 될까?** - 좋은 의문점   
  * 결과 : 상속을 통한 구현이기 때문에 `private`에 관한 메서드는 프록시로 감싸지지 않음  
    * 이는 final 도 마찬가지 : 상수로 오버라이딩을 지원하지 않으므로    
  * 그렇다면 `protected`는? : 아마 될 것 같은데 실험해보자  

## private 메서드를 사용 가능한지 확인해보기  
**org.woowacourse.aoppractice.service.AuthServiceImpl**
```java
package org.woowacourse.aoppractice.service;

import org.springframework.stereotype.Service;
import org.woowacourse.aoppractice.annotation.PerformanceCheck;

@Service
public class AuthServiceImpl {

    public void businessLogicMethod(){
        initMethod();
        System.out.println("businessLogicMethod process!");
    }

    @PerformanceCheck
    private void initMethod(){
        System.out.println("initMethod process!");
    }

}
```
* 기존 `businessLogicMethod()` 의 `@PerformanceCheck` 어노테이션을 제거
* `initMethod()` 를 생성하고 `@PerformanceCheck` 어노테이션을 추가  
  * 단, **`initMethod()`는 private** 접근 제어자를 사용 
  
**org.woowacourse.aoppractice.util.UselessAdvisor**  
```java
package org.woowacourse.aoppractice.util;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;


@Aspect
@Component
public class UselessAdvisor {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("@annotation(org.woowacourse.aoppractice.annotation.PerformanceCheck)")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }
/*
    @Before("execution(* org.woowacourse.aoppractice.service.AuthServiceImpl.*(..))")
    public void Before() throws Throwable {
        log.info("이것은 before 어드바이스이다.");
    }

    @AfterReturning("execution(* org.woowacourse.aoppractice.service.AuthServiceImpl.*(..))")
    public void AfterReturning() throws Throwable {
        log.info("이것은 AfterReturning 어드바이스이다.");
    }

    @AfterThrowing("execution(* org.woowacourse.aoppractice.service.AuthServiceImpl.*(..))")
    public void AfterThrowing() throws Throwable {
        log.info("이것은 AfterThrowing 어드바이스이다.");
    }

    @After("execution(* org.woowacourse.aoppractice.service.AuthServiceImpl.*(..))")
    public void After() throws Throwable {
        log.info("이것은 After 어드바이스이다.");
    }
*/
}

```
* 필자는 여러 어드바이스를 적용시켰기에 불필요한 것들은 주석으로 처리
* 만약 private 메서드에 AOP가 적용된다면 `request spent {} ms", stopWatch.getLastTaskTimeMillis()` 출력 될 것

**결과**
```
initMethod process!
businessLogicMethod process!
```
* private 메서드에는 AOP가 적용되지 않는다.

## 하지만 위 실행 방법으로는 절대 로그가 나올 수 없다!!!!   
![놀라는 짤](https://user-images.githubusercontent.com/50267433/99340805-14a92180-28cc-11eb-9ac4-bdde9449ed2e.png)    
    
|JoinPoint|SpringAOP|AspectJ|
|---------|---------|-------|
|메서드 호출|X|O|
|메서드 실행|O|O|
|생성자 호출|X|O|  
|생성자 실행|X|O|
|Static 초기화 실행|X|O|
|객체 초기화|X|O|
|필드 참조|X|O|
|핸들러 실행|X|O|
|Advice 실행|X|O|

* 스프링 AOP에서는 메서드 실행에 대해서만 적용되지 호출에 대해서는 적용이 되지 않는다.   
* 그래서 이를 확인하기 위해서는 AspectJ를 적용해야 할 것 같다.

# AspectJ를 이용한 테스트

## 이를 통해 깨달은점  
### @Trancsactional    
private 메서드에 `@Trancsactional` 어노테이션을 붙였을 때 작동을 하지 않는다는 말이 있다.   
이유는 바로, `@Trancsactional` 어노테이션도 스프링 AOP 메서드이기 때문이다.   

* **@Trancsactional :**    
  * 로직 시작시 트랜잭션을 열어줌
  * 로직 끝날시 commit하고 트랜잭션을 닫아줌  
  * 트랜잭션에 관련된 인프라 로직을 지원하기에 우리는 비즈니스 로직에 집중할 수 있게 

그렇기 때문에 `@Trancsactional` 어노테이션을 적용한 메서드는 private를 적용하지 않는 것이 좋습니다.   
또한 `@Trancsactional`뿐만 아니라 Interceptor 나 Filter 같은 개념들도 마찬가지입니다.     
     
# Spring AOP vs AspectJ        
        
||SpringAOP|AspectJ|
|-|---------|-------|
|목표|간단한 AOP 제공|완벽한 AOP 제공|
|join point|메서들 레벨만 지원|생성자, 필드, 메서드 등 다양하게 지원|
|weaving|런타임 시에만 가능|런타임은 제공하지 않음, compile-time, post-compile, load-time 제공|   
|대상|Spring Container가 관리하는 Bean에만 가능|모든 JAVA Object에 가능|  
        
        
# 참고        
블로그       
https://devbox.tistory.com/entry/spring-AOP-%EC%9A%A9%EC%96%B4-%EC%84%A4%EB%AA%85        
https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#joinpoint-%EA%B4%80%EB%A0%A8-annotations        
https://galid1.tistory.com/498 - 사용자 지정 어노테이션 만들기      
https://jojoldu.tistory.com/69 - jojoldu 님의 AOP 시리즈       
https://onlyformylittlefox.tistory.com/16 - 어노테이션 실행 안되는 이유 (같은 경로 아니면 패키지 다 입력)
https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#aspectj%EB%9E%80 - 위빙 관련 내용  
https://logical-code.tistory.com/118 - 전체적인 요약본, 가장 좋다.   
   
실행 참고 블로그     
https://medium.com/msolo021015/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8%EB%A1%9C-aop-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0-43022d22d091    
   
   
