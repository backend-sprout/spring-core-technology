AOP
=======
# ๐ AOP ๊ฐ์ 
## ๐ ์๋๋ฆฌ์ค

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
ํ์๊ฐ์์ ์งํํ๋ Service ์ฝ๋๋ค.                      
์ฑ๋ฅ ๋ถํ๊ฐ ์์ฌ๋์ด ํด๋น ๋ฉ์๋์ ์๊ฐ์ ์ธก์ ํ๋ ์ฝ๋๋ฅผ ์์ฑํ๋ค.                 
๊ทธ๋ฌ๋, ์ฝ๋๋ฅผ ๋ณด๋ฉด ์ ์ ์๋ฏ์ด **๊ฐ๋์ฑ์ด ๋จ์ด์ง๊ณ  ํ ํด๋์ค์ ๋ํ ๊ฒฐํฉ๋๊ฐ ๋์์ ธ ์์ง์ฑ์ด ๋จ์ด์ง๋ค.**                     
๊ทธ๋ฆฌ๊ณ  ๋ฌด์๋ณด๋ค๋ **๋ง์ ํด๋์ค์์ ์ฝ๋์ ๋ณ๊ฒฝ์ด ํ์ํ  ๋ ์ด๋ฅผ ํ ๋ฒ์ ๊ด๋ฆฌํ  ์ ์์์ง ์๋ฌธ์ด๋ค.**           
                     
* ๊ฐ๊ฐ์ ๋ฉ์๋๋ 1๊ฐ์ง ๊ธฐ๋ฅ๋ง ์ํํด์ผ ํ๋ค.       
* ํด๋์ค์ ๊ทธ ๋ฉ์๋์ ๋ณธ์ง์ ์ธ ์ญํ ์ **๋น์ฆ๋์ค ๋ก์ง์ ์ํํ๋ ๊ฒ์ด๋ค.**          
* ์ธํ๋ผ ๋ก์ง์ด ์ฝ์๋๋ฉด ๋น์ฆ๋์ค ๋ก์ง์ ๋ํ ๊ฐ๋์ฑ์ ์ค์ด๋ค๊ณ  ๊ด๋ฆฌํด์ผ ํ  ์ฝ๋๋ ๋ง์์ง๋ค.         
  * **ํต์ฌ ๊ธฐ๋ฅ :** ๋น์ง๋์ค ๋ก์ง  
  * **๋ถ๊ฐ ๊ธฐ๋ฅ :** ๋ก๊น ๋ก์ง      
       
์ ์์๋ค์ ๊ณ ๋ คํด๋ณด๋ฉด, ํต์ฌ ๊ธฐ๋ฅ ๋ก์ง์์ ๋ถ๊ฐ ๊ธฐ๋ฅ ๋ก์ง์ ์ ๊ฑฐํด์ผํ๋ค.     
ํ์ง๋ง ์ฑ๋ฅ์ธก์ , ๋ก๊น์ ๋ํ ์์์ ํ๋ก๊ทธ๋๋ฐ ๊ฐ๋ฐ์ ์์ด์ ํ์์ ์ธ ๋ถ๋ถ์ด๋ค.         
๊ทธ๋ ๋ค๋ฉด **์ด๋ฐ ๋ถ๊ฐ ๊ธฐ๋ฅ ๋ก์ง๋ค์ ๋ถ๋ฆฌํด์ ์ฌ์ฉํ  ์ ์๋ ๋ฐฉ๋ฒ์ ์์๊น? ์๋ค. ๋ฐ๋ก `AOP`์ด๋ค.**       
        
## ๐ ๋ถ๊ฐ ๊ธฐ๋ฅ   
> ์ธํ๋ผ ๋ก์ง  
   
<img width="1282" alt="แแณแแณแแตแซแแฃแบ 2020-11-16 แแฉแแฎ 1 05 46" src="https://user-images.githubusercontent.com/50267433/99211839-a4879680-280c-11eb-897d-c2970351f3bb.png">   

์์ ๊ฐ์ ๋ถ๊ฐ์ ์ธ ๊ธฐ๋ฅ๋ค์ด ๊ฐ๊ฐ์ ํต์ฌ ๊ธฐ๋ฅ์์ ๋ธ๋ก์ ์๋ฏ์ด ์ค๋ณต๋์ด ์กด์ฌํ๋ ๊ฒ์ ์ ์ ์๋ค.         
๋ํ ๊ฐ์ ๋ธ๋ก์ ๋์ผ ์ ์์ ๋์ด๋ณด๋ฉด ํก๋จํ๋ ๋ฏํ ๋ชจ์ต ๋ํ ๋ณผ ์ ์๋ค.       
      
<img width="1285" alt="แแณแแณแแตแซแแฃแบ 2020-11-16 แแฉแแฎ 1 06 43" src="https://user-images.githubusercontent.com/50267433/99211873-b9fcc080-280c-11eb-8c39-d98b7dd63e80.png">
     
* **cross-cutting concern :** ๋ถ๊ฐ์ ์ธ ๊ธฐ๋ฅ์ธ ์ธํ๋ผ ๋ก์ง์ด ์ค๋ณต๋์ด ํก๋จ(๊ฐ๋ก)๊ณผ ๊ฐ์ ๋ชจ์ต์ ๋ํ๋ด๋ ๋ก์ง      
* **core concern :** ์ฌ์ฉ์์ ์์ฒญ์ ๋ฐ๋ผ ์ค์ ๋ก ์ํ๋๋ ํต์ฌ ๋น์ฆ๋์ค ๋ก์ง        
   
**์ธํ๋ผ ๋ก์ง**          
์ธํ๋ผ ๋ก์ง์ ์ ํ๋ฆฌ์ผ์ด์์ ์  ์์ญ์์ ๋ํ๋  ์ ์๋ค.          
๊ทธ๋ฌ๋ `cross-cutting concern`์์ ์ ์ ์๋ฏ์ด ๊ฐ๊ฐ์ ์์ญ์์ ์ค๋ณต๋ ์ฝ๋๊ฐ ๋ฐ์ํ๊ณ  ์๋ค.             
์ด๋ก ์ธํด ์ ์ง๋ณด์๊ฐ ํ๋ค์ด์ง๊ณ  ๋น์ฆ๋์ค ๋ก์ง์ ์ดํดํ๊ธฐ ์ด๋ ต๊ฒ ํ๋ค๋ ๋ฌธ์ ๋ฅผ ๊ฐ์ง๊ณ  ์๋ค.        

# ๐ AOP  
> Aspect-Oriented Programming | ๊ด์  ์งํฅ ํ๋ก๊ทธ๋๋ฐ   
   
* OOP ๊ฐ์ฒด์งํฅ ํ๋ก๊ทธ๋จ   
* AOP ๊ด์ ์งํฅ ํ๋ก๊ทธ๋จ    
       
AOP๋ OOP๋ฅผ ๋์ฑ OOP๋ต๊ฒ ํ๋ก๊ทธ๋๋ฐ ํ  ์ ์๊ฒ ๋์์ฃผ๋ ๊ฒ์ผ๋ก      
**์ ํ๋ฆฌ์ผ์ด์์ `ํต์ฌ์ ์ธ ๊ธฐ๋ฅ`๊ณผ `๋ถ๊ฐ์ ์ธ ๊ธฐ๋ฅ`์ `๋ถ๋ฆฌ`ํด**    
**`Aspect`๋ผ๋ ๋ชจ๋๋ก ๋ง๋ค์ด ์ค๊ณํ๊ณ  ๊ฐ๋ฐํ๋ ๋ฐฉ๋ฒ์ด๋ค.**        

์ค์ ๋ก Spring doc document ์์๋ ์๋์ ๊ฐ์ด ๋งํ๊ณ  ์๋ค.  
```   
Aspect-oriented Programming (AOP) complements    
Object-oriented Programming (OOP) by providing   
another way of thinking about program structure   

AOP๋ ํ๋ก๊ทธ๋จ ๊ตฌ์กฐ์ ๋ํ ๋ค๋ฅธ ์๊ฐ์ ๋ฐฉํฅ์ ์ ๊ณตํด์ฃผ๋ฉด์ OOP๋ฅผ ๋ณด์ํ๊ณ  ์๋ค.       
```      
  
์๋ฅผ ๋ค๋ฉด ์ด๋ ํ ํด๋์ค๋ฅผ ๋์์ผ๋ก `ํต์ฌ ๊ธฐ๋ฅ`๊ณผ `๋ถ๊ฐ์ ์ธ ๊ธฐ๋ฅ`์ ๊ด์ ์ผ๋ก         
**๊ณตํต ์ฌ์ฉ ๋ถ๊ฐ ๊ธฐ๋ฅ๋ค์ ์ธ๋ถ์ ๋๋ฆฝ๋ ํด๋์ค๋ก ๋ถ๋ฆฌํ๊ณ **                
**์ด๋ฅผ ๋ชจ๋ํํ์ฌ ์ฌ์ฌ์ฉํ  ์ ์๊ฒ๋ ํ๋ ํ๋ก๊ทธ๋๋ฐ ๊ธฐ๋ฒ์ด๋ค.**                 
                 
**AOP ๊ตฌํ์ฒด**    
* AspectJ : ๋ค์ํ ํฌ์ธํธ ์ปท ์ ๊ณต       
* ์คํ๋ง AOP : ์ ํ์ ์ด์ง๋ง ํจ์จ์ ์ธ ๊ธฐ๋ฅ ์ ๊ณต   

||SpringAOP|AspectJ|
|-|---------|-------|
|๋ชฉํ|๊ฐ๋จํ AOP ์ ๊ณต|์๋ฒฝํ AOP ์ ๊ณต|
|join point|๋ฉ์๋ค ๋ ๋ฒจ๋ง ์ง์|์์ฑ์, ํ๋, ๋ฉ์๋ ๋ฑ ๋ค์ํ๊ฒ ์ง์|
|weaving|๋ฐํ์ ์์๋ง ๊ฐ๋ฅ|๋ฐํ์์ ์ ๊ณตํ์ง ์์, compile-time, post-compile, load-time ์ ๊ณต|   
|๋์|Spring Container๊ฐ ๊ด๋ฆฌํ๋ Bean์๋ง ๊ฐ๋ฅ|๋ชจ๋  JAVA Object์ ๊ฐ๋ฅ|  

# ๐ AOP ์ฉ์ด 

|์ฉ์ด|์ค๋ช|
|----|----|
|**Aspect(Advisor)**|**โ ํน์  ํ ๊ฐ์ง ๊ธฐ๋ฅ์ ๋ํด ํฉ์ด์ง ๊ด์ฌ์ฌ๋ฅผ ๋ชจ๋ํํ์ฌ ๋ฌถ์ ํด๋์ค**<br>- `PointCut + Advice`๋ฅผ ๋ฌถ์ ๋ฉ์๋๋ฅผ ์ ์ ๋ฐ ๊ด๋ฆฌํ๋ค.|
|Target|**โ ๋น์ฆ๋์ค(ํต์ฌ) ๋ก์ง์ ๊ตฌํํ๋ AOP์ ๋์ 'ํด๋์ค'**|  
|Advice|**โ AOP ๋ฉ์๋ : ๋ถ๊ฐ ๊ธฐ๋ฅ์ ์ ์ฉ์ํค๋ ๋ฉ์๋(๋ถ๊ฐ ๊ธฐ๋ฅ ๋ก์ง ๋ฉ์๋)**<br>**์ ์ฉ ์์ ๋ ์ง์ ํ  ์ ์๋ค.**<br>- before : ๋น์ฆ๋์ค ๋ฉ์๋ ์คํ ์ ์ ๋์<br>- after : ๋น์ฆ๋์ค ๋ฉ์๋ ์คํ ํ์ ๋์<br>- after-returning : ๋น์ฆ๋์ค ๋ฉ์๋ ์คํ ์ค ๋ฆฌํด ๋๋ ์๊ฐ<br>- after-throwing : ๋น์ฆ๋์ค ๋ฉ์๋ ์คํ ์ค ์๋ฌ ๋ฐ์ ์๊ฐ<br>- around	: ๋น์ฆ๋์ค ๋ฉ์๋ ์คํ ์ /ํ์ ๋์| 
|JoinPoint|**โ AOP๊ฐ ์ ์ฉ๋  ์ ์๋ `๋ฉ์๋`, ํ๋, ๊ฐ์ฒด, ์์ฑ์ ๋ฑ์ ์์**<br>- ํ๊ฒ ํด๋์ค์ ์์๋ค์ ์๋ฏธํ๋ฉฐ **PointCut์ ํ๋ณด**<br>- **Spring AOP์์๋ ๋ฉ์๋๋ง ์ง์  ๊ฐ๋ฅ**|
|PointCut|**โ AOP ์ ์ฉ ๊ฐ๋ฅ ์์๋ค ์ค์์ '์ค์  ์ ์ฉ๋  ์์๋ค'์ ์๋ฏธํ๋ค.**<br>- `JoinPoint`์์ ์ค์  `Advice`๊ฐ ์ ์ฉ๋  ์ง์ <br>- **Spring AOP ์์๋ advice๊ฐ ์ ์ฉ๋  ๋ฉ์๋๋ฅผ ์๋ฏธ**|    
|Weaving|**โ  PointCut์ด ํธ์ถ๋  ๋ Advice๊ฐ ํธ์ถ๋๋ ๊ณผ์ ์ ์๋ฏธํ๋ค.**<br>์ฆ, **PointCut์ Advice ๋ฉ์๋๊ฐ ์ฝ์๋๋ ๊ณผ์ ์ ์๋ฏธํ๋ค.**<br>- ์ปดํ์ผ ์ ์๋ ํต์ฌ ๋ก์ง๊ณผ ์ธํ๋ผ ๋ก์ง์ด ๋ถ๋ฆฌ๋์ด์์ง๋ง<br>- ์ปดํ์ผํ์์ ๋ฐ์ดํธ ์ฝ๋๋ฅผ ๋ฃ๋๋ค๋์ง ๋ฐํ์์ ํ๋ก์๋ฅผ ๋ง๋ค๋์ง<br>- ๊ด์ฌ์ฌ๋ฅผ ๋ถ๋ฆฌํ ์ฝ๋๋ฅผ ๋น์ฆ๋์ค ์ฝ๋์ ๋ณํฉํ๋ ์์์ ์๋ฏธํ๋ค.<br>**Weaving ์ฒ๋ฆฌ ๋ฐฉ์ (AOP ๊ตฌํ ๋ฐฉ๋ฒ)**<br>- ์ปดํ์ผ ํ์ ์๋น : `a.java -> a.clss` ์ปดํ์ผ ๋  ๋<br>- ๋ก๋ฉ ํ์ ์๋น : ํด๋์คํ์ผ์ ํด๋์ค ๋ก๋๊ฐ ๋ฉ๋ชจ๋ฆฌ์ ๋ก๋ํ  ๋<br>- ๋ฐํ์/ํ๋ก์ ์๋น : ํ๊ฒ ํด๋์ค์ ๋ถ๊ฐ ๊ธฐ๋ฅ์ ์ถ๊ฐํ๋๋ก Proxy ๊ฐ์ฒด๋ฅผ ๋ง๋ค์ด(๊ฐ์ธ์) ์คํ<br>- ์คํ๋ง AOP๋ **๋ฐํ์ ์๋น**๋ง์ ์ง์ํ๋ค.(IOC/DI๋ฅผ ์ด์ฉํ ๋ฐฉ๋ฒ)|    
|proxy|**โ Target ๊ฐ์ฒด์ Advice๋ฅผ ์ ์ฉ์ํค๋ ๋ํผ ํด๋์ค**|    

![AOP](https://user-images.githubusercontent.com/50267433/126900960-e0ed4a26-9521-49ac-bfd3-c9a915c12772.png)    
        
1. ์ ํ๋ฆฌ์ผ์ด์์ ์์ฐ์ค๋ฝ๊ฒ ์ฌ๋ฌ ๋ฉ์๋(Join Point)๋ฅผ ํธ์ถํ๋ค.      
2. ์ด๋ ํน์  **PointCut์ผ๋ก ์ง์ ํ ๋ฉ์๋๊ฐ ํธ์ถ๋๋ ์๊ฐ**       
3. **Aspect ์ Advice ๋ฉ์๋๊ฐ ํธ์ถ๋๋ค.**             
4. Advice ๋ฉ์๋์ **๋์ ์์ ์ 5๊ฐ์ง๋ก ์ ํ  ์ ์๋ค.**               
5. ์ ์คํฉํธ ์ค์ ์ ๋ฐ๋ผ **์๋น ์ฒ๋ฆฌ๋์ด ํ๋ก์ ๊ฐ์ฒด๊ฐ ์์ฑ๋๋ค.(๋์  ํ๋ก์ ์์ฑ)**                 
6. ํ๋ก์ ๊ฐ์ฒด๋ฅผ ํตํด ๋ถ๊ฐ๊ธฐ๋ฅ์ด ํฌํจ๋ ๋น์ฆ๋์ค ๋ก์ง์ ์ํํ๋ค.              

## ๐ AOP ๊ตฌํ 
```gradle
implementation 'org.springframework.boot:spring-boot-starter-aop'
```
```java
// @EnableAspectJAutoProxy ์๋ต ๊ฐ๋ฅ 
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

### ๐ ์ผ๋ฐ์ ์ธ ๊ตฌํ      
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
        log.info("After ์ด๋๋ฐ์ด์ค");
    }

}
```

### ๐ ์ด๋ธํ์ด์์ผ๋ก ๊ตฌํ   
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
        log.info("After ์ด๋๋ฐ์ด์ค");
    }

}
```
* `์ด๋ธํ์ด์`๊ณผ `Aspect`๊ฐ ๋์ผ ์์น๋ฉด ์ด๋ธํ์ด์๋ง ์ ์ด๋ ๋๋ค.   
* ํจํค์ง๊ฐ ๋ค๋ฅด๋ฉด FQCN(Fully qualified Class Name)์ ๋ค ์๋ ฅํด์ฃผ์ด์ผ ํ๋ค.    

### ๐ ๋น์ผ๋ก ๊ตฌํ   
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
        log.info("After ์ด๋๋ฐ์ด์ค");
    }

}
```
* `๋น`๊ณผ `Aspect`๊ฐ ๋์ผ ์์น๋ฉด ์ด๋ธํ์ด์๋ง ์ ์ด๋ ๋๋ค.   
* ํจํค์ง๊ฐ ๋ค๋ฅด๋ฉด FQCN(Fully qualified Class Name)์ ๋ค ์๋ ฅํด์ฃผ์ด์ผ ํ๋ค.    
   
## ๐ AOP PointCut ์ค์   
```java
execution(* com.springbook.biz..*Impl.get*(..))"
```   

* `*` : ๋ฆฌํด๊ฐ
* `com.springbook.biz..` : ํจํค์ง
* `*Impl`: ํด๋์ค ์ด๋ฆ
* `get*` : ๋ฉ์๋ ์ด๋ฆ
* `(..)` : ๋งค๊ฐ๋ณ์ 
* ์ค๊ฐ๋ง๋ค์ `.` : ๊ตฌ๋ถ์    

### ๐ execution ํฌ์ธํธ ์ปท ๋ฆฌํด

|ํํ์|์ค๋ช| 
|----|---|   
|`*`|๋ชจ๋  ๋ฆฌํดํ์ ํ์ฉ|   
|`void`|๋ฆฌํดํ์์ด void์ธ ๋ฉ์๋ ์ ํ|   
|`!void`|๋ฆฌํดํ์์ด void๊ฐ ์๋ ๋ฉ์๋ ์ ํ|   

### ๐ execution ํฌ์ธํธ ์ปท ํจํค์ง 

|ํํ์|์ค๋ช|  
|----|---|      
|`com.springbook.biz`|์ ํํ๊ฒ ํด๋น ํจํค์ง๋ง ์ ํ|      
|`com.springbook.biz..`|ํด๋น ํจํค์ง ๋ฐ ๋ชจ๋  ํ์ ํจํค์ง ์ ํ|      
|`com.springbook..impl`|`..`์ ํจํค์ง๋ก ์์ํ๋ฉด์ ๋ง์ง๋ง ํจํค์ง ์ด๋ฆ์ด `..`๋ค๋ก ๋๋๋ ํจํค์ง ์ ํ|          
   
### ๐ execution ํฌ์ธํธ ์ปท ํด๋์ค
|ํํ์|์ค๋ช|  
|----|---|      
|`BoardServiceImpl`|์ ํํ๊ฒ ํด๋น ํด๋์ค๋ง ์ ํ|      
|`*Impl`|ํด๋์ค ์ด๋ฆ์ด `*`๋ท ๊ธ์๋ก ๋๋๋ ํด๋์ค๋ง ์ ํ|      
|`BoardService+`|ํด๋น ํด๋์ค๋ ๋ฌผ๋ก  ํ์๋ ๋ชจ๋  ์์ ํด๋์ค๋ ์ ํ ๊ฐ๋ฅ|
|`variable+`|ํด๋น ์ธํฐํ์ด์ค๋ฅผ ๊ตฌํํ ๋ชจ๋  ํด๋์ค ์ ํ ๊ฐ๋ฅ |    

### ๐ execution execution ํฌ์ธํธ ์ปท ๋ฉ์๋

|ํํ์|์ค๋ช|
|----|---|
|`*(..)`|๊ฐ์ฅ ๊ธฐ๋ณธ ์ค์ ์ผ๋ก ๋ชจ๋  ๋ฉ์๋ ์ ํ|
|`get*(..)`|๋ฉ์๋ ์ด๋ฆ์ด get์ผ๋ก ์์ํ๋ ๋ชจ๋  ๋ฉ์๋ ์ ํ|
|`๋งค์๋์ด๋ฆ(..)`|ํน์  ๋ฉ์๋ ์ด๋ฆ์ ๊ฐ์ง ๋ฉ์๋ ์ ํ|

### ๐ execution ํฌ์ธํธ ์ปท ๋งค๊ฐ๋ณ์ 
   
|ํํ์|์ค๋ช|
|----|---|
|`(..)`|๊ฐ์ฅ ๊ธฐ๋ณธ ์ค์ ์ผ๋ก์ ๋งค๊ฐ๋ณ์ ํ์์ ์ ํ์ด ์์์ ์๋ฏธ|
|`(*)`|๋ฐ๋์ 1๊ฐ์ ๋งค๊ฐ๋ณ์๋ฅผ ๊ฐ์ง๋ ๋ฉ์๋๋ง ํ์ฉํ๋ค๋ ์๋ฏธ|
|`(com.spring.user.User)`|ํด๋น ํจํค์ง์ ํด๋์ค๋ฅผ ๊ฐ์ง๋ ๋ฉ์๋๋ง ํ์ฉํ๋ค๋ ์๋ฏธ|
|`(!com.spring.user.User)`|ํด๋น ํจํค์ง์ ํด๋์ค๋ฅผ ๊ฐ์ง์ง ์๋ ๋ฉ์๋๋ง ํ์ฉํ๋ค๋ ์๋ฏธ|
|`(Integer, ..)`|ํ ๊ฐ ์ด์์ ๋งค๊ฐ๋ณ์๋ฅผ ๊ฐ์ง๋, ์ฒซ ๋ฒ์งธ ๋งค๊ฐ๋ณ์์ ํ์์ด integer์ธ ๋ฉ์๋๋ง ํ์ฉ|
|`(Integer, *)`|๋ฐ๋์ ๋ ๊ฐ์ ๋งค๊ฐ๋ณ์๋ฅผ ๊ฐ์ง๋, ์ฒซ ๋ฒ์งธ ๋งค๊ฐ๋ณ์์ ํ์์ด integer์ธ ๋ฉ์๋๋ง ํ์ฉ|
        

# ๐ ์คํ๋ง AOP  
์คํ๋ง AOP ๋ **๋ฐํ์ ์๋น์ ์ง์ํ๊ธฐ์ Proxy ๊ธฐ๋ฐ์ AOP ๊ตฌํ ๊ธฐ๋ฅ์ ์ง์ํ๋ค.**             
์ด๋ฌํ **Proxy ๊ธฐ๋ฐ์ AOP๋ ์คํ๋ง ๋น์๋ง ์ ์ฉ๊ฐ๋ฅํ๊ธฐ์ ๋น์ผ๋ก ๋ฑ๋ก๋ ๋์๋ง AOP ๋์์ด ๋๋ค.**         
๋ํ, **์คํ๋ง AOP ํน์ง์ ๋ฉ์๋๋ง์ ์ง์ํ  ์ ์๋ค.**        
         
## ์คํ๋ง AOP ๊ฐ์    
**ํ๋ก์ ํจํด**   
```java
@Primary
@Service
public class ProxySimpleEventService implements EventService{

    private final EventService simpleEventService;

    ProxySimpleEventService(EventService simpleEventService) {
        this.simpleEventService = simpleEventService;
    }

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println("createEvent ์คํ์๊ฐ: "+(System.currentTimeMillis()-begin));
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println("createEvent ์คํ์๊ฐ: "+(System.currentTimeMillis()-begin));
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
    }
}
```

ํ๋ก์ ํจํด์ **AOP์์ ๊ธฐ์กด ์ฝ๋์ ๋ณ๊ฒฝ ์์ด ์ ๊ทผ ์ ์ด ๋๋ ๋ถ๊ฐ ๊ธฐ๋ฅ ์ถ๊ฐํ๊ธฐ ์ํด ์ฌ์ฉ๋๋ค.**              
๊ทธ๋ฌ๋ ์ผ๋ฐ์ ์ธ ๋ฐฉ๋ฒ์ผ๋ก ํ๋ก์ ํจํด์ ๊ตฌํํ๋ฉด ์๋์ ๊ฐ์ ๋ฌธ์ ์ ์ด ๋ฐ์ํ๋ค.         
                
**๋ฌธ์ ์ **  
* ๋งค๋ฒ Proxy ํด๋์ค๋ฅผ ์์ฑํด์ผ ํ๋ค.    
* ํ๋์ ํด๋์ค๊ฐ ์๋ ์ฌ๋ฌ ํด๋์ค๋ฅผ ๋์์ผ๋ก ์ ์ฉํ๊ธฐ ํ๋ค๋ค.    
* ๊ฐ์ฒด ๊ด๊ณ๊ฐ ๋ณต์กํด์ง๊ณ  ๊ด๋ฆฌํ๊ธฐ ์ด๋ ต๋ค.     
         
์ด๋ฌํ ๋ฌธ์ ์ ์ ํด๊ฒฐํ๊ธฐ ์ํด ๋ฑ์ฅํ ๊ฒ์ด ๋ฐ๋ก `Spring AOP`์ `Dynamic Proxy`๋ค.  
**์คํ๋ง IoC ์ปจํ์ด๋๊ฐ ์ ๊ณตํ๋ ๊ธฐ๋ฐ ์์ค๊ณผ Dynamic Proxy๋ฅผ ์ฌ์ฉํ์ฌ ์ฌ๋ฌ ๋ณต์กํ ๋ฌธ์  ํด๊ฒฐํ๋ค.**      
  
* **Dynamic Proxy :** ๋์ ์ผ๋ก ํ๋ก์ ๊ฐ์ฒด ์์ฑํ๋ ๋ฐฉ๋ฒ     
    * **JDK Dynamic Proxy :** ์๋ฐ๊ฐ ์ ๊ณตํ๋ ์ธํฐํ์ด์ค ๊ธฐ๋ฐ ํ๋ก์ ์์ฑ ๋ผ์ด๋ธ๋ฌ๋ฆฌ      
    * **CGlib :** ์๋ฐ๊ฐ ์ ๊ณตํ๋ ํด๋์ค ๊ธฐ๋ฐ ํ๋ก์ ์์ฑ ๋ผ์ด๋ธ๋ฌ๋ฆฌ    
* **Spring IoC :** **๊ธฐ์กด ๋น์ ๋์ฒดํ๋ '๋์  ํ๋ก์ ๋น'์ ๋ง๋ค์ด ๋ฑ๋กํ๋ค.**      
    * ํด๋ผ์ด์ธํธ ์ฝ๋ ๋ณ๊ฒฝ์ด ์๋ค.      
    * `AbstractAutoProxyCreator implements BeanPostProcessor`๋ฅผ ๊ธฐ๋ฐ์ผ๋ก ๋ง๋ ๋ค.    
    * ์ฆ ๋น์ด ์์ฑ๋ ํ, ํ๋ก์๋ฅผ ์์ฑํ๋ ๋ก์ง์ ์ํํ๋ค.       
  
์ด๋ ๋ฏ **๋ฐํ์์์ ๋์ ์ผ๋ก ํ๋ก์ ๋น์ ์์ฑํ๊ณ  AOP๋ฅผ ์ ์ฉ์ํค๋ ๊ฒ์ ๋ฐํ์ ์๋น์ด๋ผ ๋ถ๋ฅธ๋ค.**       
   
## ๐ ๋ฐํ์ ์๋น(ํ๋ก์ ์๋น)      
๋ฐํ์ ์๋น์ **๋ฐํ์์ ๋์ ์ผ๋ก Proxy๋ฅผ ์์ฑํ์ฌ ์ค์  Target ๊ฐ์ฒด์ ๋ณํ์์ด AOP๋ฅผ ์ํํ๋ ๊ฒ์ ์๋ฏธํ๋ค.**      
์คํ๋ง์ ๊ธฐ์ค์ผ๋ก ๋งํ์๋ฉด ๋ฉ์๋๋ง์ด JoinPoint๊ฐ ๋  ์ ์๊ธฐ์ Method ํธ์ถ ์์ ์๋น์ด ์ด๋ฃจ์ด ์ง๋ ๋ฐฉ์์ด๋ค.        
   
๋ฐํ์ ์๋น ๋ํ, ์ผ๋ฐ์ ์ธ Proxy ํจํด์ ๊ฐ์ ํ Dynamic Proxy๋ฅผ ์ฌ์ฉํ๊ณ  ์์ง๋ง ์๋์ ๊ฐ์ ๋ฌธ์ ๊ฐ ์๋ค.     
* ํฌ์ธํธ ์ปท์ ๋ํ ์ด๋๋ฐ์ด์ค ์ ์ฉ ๊ฐฏ์๊ฐ ๋์ด ๋ ์๋ก ์ฑ๋ฅ์ด ๋จ์ด์ง๋ค.            
* ๋ฉ์๋ ํธ์ถ์ ๋ํด์๋ง ์ด๋๋ฐ์ด์ค๋ฅผ ์ ์ฉ ํ  ์ ์๋ค.(ํ๋ก์์ด๊ธฐ์)                 
   
**AOP ํ๋ก์ ๊ฐ์ฒด**
```
org.woowacourse.aoppractice.service.AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d
```

* ๊ธฐ์กด ๊ฐ์ฒด : org.woowacourse.aoppractice.service.AuthServiceImpl      
* ํ๋ก์ ๊ฐ์ฒด : org.woowacourse.aoppractice.service.AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d    

์ ์ฝ๋๋ฅผ ๋ณด๋ฉด, Target ํด๋์ค ์์ฒด๋ฅผ ํ๋ก์๋ก ๊ฐ์ธ๋ ๊ฒ์ ์ ์ ์๋ค.   
      
<img width="842" alt="aop-proxy" src="https://user-images.githubusercontent.com/50267433/126906051-ec8d6b7c-0f34-44c7-8a8a-51523a4c6a84.png">

ํ๋ก์ ์๋น์ด ๊ฐ๋ฅํ ์ด์ ๋ **์์์ ์ด์ฉํ ๋ฐฉ์์ด๊ธฐ์ ๋คํ์ฑ์ ์ ์ฉ์ํฌ ์ ์๋ค.**    
์ฆ, ์ ๊ทธ๋ฆผ์์ `AuthController`๊ฐ `AuthService`์ ํ๋ก์ ํด๋์ค์ธ `AuthService$$๋ธ๋ผ๋ธ๋ผ`๋ฅผ ์ฐธ์กฐํ  ์ ์๋ ๊ฒ์ด๋ค.      
            
๊ทธ๋ฐ๋ฐ ํ๊ฐ์ง ์๊ฐํด๋ณผ ์ ์ด ์๋ค.           
**DynamicProxy๋ ์์/๊ตฌํ์ ์ ์ฉํ๋ ์๋์ ๊ฐ์ ์์๋ค์ ๊ฐ๋ฅํ ๊น? ๐ค**          
     
* final ํด๋์ค   
* final ๋ฉ์๋   
* private ๋ฉ์๋   
   
๋น์ฐํ๊ฒ๋, **์๋ฐ ๋ฌธ๋ฒ์ ์ผ๋ก ์์ ๋ฐ ์ค๋ฒ๋ผ์ด๋ฉ์ ์ง์ํ์ง ๋ชปํ๋ฏ๋ก ์ ์ฉ์ด ๋์ง ์๋๋ค.**                   
์ด์ ๋น์ทํ๊ฒ Spring์์ ์ง์ํด์ฃผ๋ `@Trancsactional` ์ด๋ธํ์ด์์ด ์๋๋ฐ               
`@Trancsactional` ์ด๋ธํ์ด์ ๋ํ AOP๊ธฐ๋ฐ์ด๊ธฐ์ private ๋ฉ์๋๋ฅผ ๋ถ์ด๋ฉด ์๋์ ํ์ง ์๋๋ค.         
     
**@Trancsactional**       
* ๋ก์ง ์์์ ํธ๋์ญ์์ ์ด์ด์ค    
* ๋ก์ง ๋๋ ์ commitํ๊ณ  ํธ๋์ญ์์ ๋ซ์์ค        
* ํธ๋์ญ์์ ๊ด๋ จ๋ ์ธํ๋ผ ๋ก์ง์ ์ง์ํ๊ธฐ์ ์ฐ๋ฆฌ๋ ๋น์ฆ๋์ค ๋ก์ง์ ์ง์คํ  ์ ์๊ฒํด์ค๋ค.                  
            
# ์ฐธ๊ณ             
[๋ฐฑ๊ธฐ์ -์คํ๋ง ํ๋ ์์ํฌ ํต์ฌ ๊ธฐ์ ](https://www.inflearn.com/course/spring-framework_core)      
[์คํ๋ง ํต ์คํํธ](http://www.yes24.com/Product/Goods/29173715)      
[ํ์ฝํก-์คํ๋ง AOP](https://www.youtube.com/watch?v=Hm0w_9ngDpM)      
[์ฅ์ธ ๊ฐ๋ฐ์๋ฅผ ๊ฟ๊พธ๋ : ๊ธฐ๋กํ๋ ๊ณต๊ฐ](https://devbox.tistory.com/entry/spring-AOP-%EC%9A%A9%EC%96%B4-%EC%84%A4%EB%AA%85)           
[Carrey's ๊ธฐ์ ๋ธ๋ก๊ทธ](https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#joinpoint-%EA%B4%80%EB%A0%A8-annotations)          
[์ง์ง ๊ฐ๋ฐ์](https://galid1.tistory.com/498)       
[๊ธฐ์ต๋ณด๋จ ๊ธฐ๋ก์](https://jojoldu.tistory.com/69)         
[๋ฆฌ๋ค์์ ๊ฐ๋ฐ ์ธ์](https://onlyformylittlefox.tistory.com/16)   
[Carrey's ๊ธฐ์ ๋ธ๋ก๊ทธ-AspectJ ๋์ค์ ์ฐธ๊ณ ํ๋ฉด ์ข์](https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#aspectj%EB%9E%80)    
[๋ผ๋ฆฌ์  ์ฝ๋ฉ](https://logical-code.tistory.com/118)             
     
