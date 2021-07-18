데이터 바인딩 추상화: PropertyEditor  
=====================================   
Spring 을 사용하다보면 간혹 **마법 같은 일**이 일어난다.       
`URL의 쿼리 스트링`이나, `HttpMessage Body`에 담겨져오는 데이터들은 사실 문자열이다.         
그런데 `@PathVariable`, `@RequestParam`, `@RequetBody`, `@ModelAttribute`을 사용해보면         
`int`와 같은 정수형은 물론 사용자가 직접 정의한 클래스 타입으로도 변환해서 사용하는 것도 가능하다.           
                     
**과연 이러한 동작이 어떻게 가능한 것일까? 🤔**                  
눈치챘겠지만 `데이터 바인딩 추상화` 덕분이다.               
그래서 이번에는 `데이터 바인딩 추상화`에 대한 설명과 코드를 통해 어떻게 돌아가는지 살펴보려 한다.       
      
# Data Binding     
데이터 바인딩 추상화는 `org.springframework.validation.DataBinder`를 의미한다.(validation에 속해 있다.)            
`org.springframework.validation.DataBinder`는 스프링 초기부터 있던 기능으로                
주로, 웹 애플리케이션을 구현할 때 **사용자의 요청 데이터를 알맞는 형태로 변환할 때 사용한다.**               
             
* 기술적인 관점: 프로퍼티 값을 타겟 객체에 설정하는 기능       
* 사용자 관점: 사용자 입력값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능.            
          
쉽게 말하자면, 사용자 요청 데이터는 대부분 `“문자열”`인데            
그 값을 `int`, `long`, `Boolean`, `Date`와 같은 타입으로 변환은 물론      
심지어 `Book`과 같은 사용자가 정의 객체로도 변환이 가능하다.        
               
이외에도 **`xml 설정 파일`에 있던 문자열을 빈이 가지고 있는 적절한 타입으로 변환 및**              
**`SpEL`에서 선언된 문자열을 숫자형으로 바꾸어 연산을 가능하도록 한다.**          
       
# PropertyEditor - 고전적인 데이터 바인딩  

```java
public class Event {
    private Integer id;
    private String title;
    public Event(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    @Override
    public String toString() {
        return "Event{" + "id=" + id + '}';
    }
}
```
```java
@RestController
public class EventController {

    private Logger logger = LoggerFactory.getLogger(EventController.class);

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        logger.info(event.toString());
        return event.getId().toString();
    }
}
```
```java
@WebMvcTest
class EventControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    void getTest() throws Exception {
        mockMvc.perform(get("/event/1"))
                .andExpect(status().isOk())
                .andExpect(content().string("1"));
    }
}
```
단순히 위와 같이 코드를 작성할 경우 데이터 바인딩은 원활히 이루어지지 않을 것이다.       
즉, 테스트 실패가 뜨는데 바인딩 작업을 따로 지정해주지 않았기 때문이다.                     
필자는 이미 해결했지만, 동작 원리를 알기 위해서 `PropertyEditor`를 구현한 클래스를 만들어보자               
           
참고로, `PropertyEditor`는 단순한 인터페이스로 구현을 하기 위해서 많은 추상 메서드를 정의해줘야한다.      
그렇기에 `PropertyEditor` 보다는 `PropertyEditorSupport`를 상속받는 형태로 작성해보자     
       
```java
public class EventEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        Event event = (Event) getValue();
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.valueOf(text)));
    }

}

```      
```shell
2021-07-18 20:03:23.085  INFO 11312 --- [           main] com.example.core.EventController         : Event{id=1}
```  
           
* **getAsText :** 애플리케이션에서 사용자로 보낼 때 매핑                
* **setAsText :** 클라이언트 요청에서 애플리케이션으로 들어올 때 매핑             
   
그런데 여기서 주의할 점은 코드를 보면 알겠지만 `setValue()/getValue()`를 통해 값을 저장/반환한다.        
즉, `PropertyEditor`는 내부적으로 `value`를 저장하는데 여기서 단점이 `thread-safe`하게 설계되어 있지 않다.        
그렇기에 **만약 싱글톤으로 선언을 하게 된다면 여러 스레드에서 `PropertyEditor`의 value를 수정할 것이다.**        
빈으로 만들어 사용하려면 `스레드 스코프`로 활동 영역을 제한하여 사용하면 된다.       

```java
@RestController
public class EventController {

    private Logger logger = LoggerFactory.getLogger(EventController.class);

    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        logger.info(event.toString());
        return event.getId().toString();
    }
}
```
```shell
2021-07-18 22:24:23.085  INFO 11312 --- [           main] com.example.core.EventController         : Event{id=1}
```  
아니면 `@InitBinder`를 통해 `WebDataBinder`객체를 의존 주입받고       
`webDataBinder.registerCustomEditor()`를 이용하여 `editor 대상`과 `editor`를 등록한다.         
하지만, 앞서 봤듯이 해당 작업을 하지 않더라도 동작은 하므로 크게 신경 쓸 필요는 없을 것 같다.       

**@InitBinder란? 🤔**    

      
      
**PropertyEditor 정리**
- Spring이 제공하는 DataBinder 인터페이스를 통해 사용됨
- Spring 3 이전까지 DataBinder가 변환 작업에 사용한 인터페이스
- 값(상태 정보)을 저장하고 있어 thread-safe하지 않음
- 일반적인 싱글톤 scope 빈으로 등록해서 사용할 수 없음
- Object - String간의 변환만 할 수 있어 사용 범위가 제한적
           
PropertyEditor를 사용하기에는 신경쓸게 많고 제한적이기에      
이러한 문제를 해결하고자 Spring 3에서는 Converter와 Formatter를 지원하기 시작했다.        



