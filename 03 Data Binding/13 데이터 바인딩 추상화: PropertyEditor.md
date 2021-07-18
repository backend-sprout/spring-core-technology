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
   
**그런데...**  
```shell
2021-07-18 20:03:23.085  INFO 11312 --- [           main] com.example.core.EventController         : Event{id=1}
```  
스프링 버전이 올라가면서 바인딩 기능이 더 좋아진 것 같다.         
추측컨데 생성자가 `Integer` 한 개만 받으므로 이를 주입해준 것 같다.            
              
더 자세히 말하면,    
**쿼리 스트링으로 온 값**일 경우 이름이 같거나 따로 매칭해줬어야 되었던걸로 기억하는데                  
**단순히 REST API의 URL Path 값**은 이름이 없기 때문에 바로 생성자로 주입해준 것 같다.            
          
아무튼 백기선님 강의에서는 테스트 실패가 뜨는데 바인딩 작업을 따로 지정해주지 않았기 때문이다.                   
필자는 이미 해결했지만, 동작 원리를 알기 위해서 `PropertyEditor`를 구현한 클래스를 만들어보자             
    
참고로, `PropertyEditor`는 단순한 인터페이스로 구현을 하기 위해서 많은 추상 메서드를 정의해줘야한다.   
그렇기에 `PropertyEditor` 보다는 `PropertyEditorSupport`를 상속받는 형태로 작성해보자   

```java

```






● 스프링 3.0 이전까지 DataBinder가 변환 작업 사용하던 인터페이스
● 쓰레드-세이프 하지 않음 (상태 정보 저장 하고 있음, 따라서 싱글톤 빈으로 등록해서
쓰다가는...)
● Object와 String 간의 변환만 할 수 있어, 사용 범위가 제한적 임. (그래도 그런 경우가
대부분이기 때문에 잘 사용해 왔음. 조심해서..)

