데이터 바인딩 추상화: Converter와 Formatter
=================================================  
  
# Converter   
기존의 `PropertyEditor` 는 `String <-> Object` 간의 매핑을 지원한다.         
반대로 생각해보면, **String 이 아닌 다른 타입으로는 매핑을 할 수 없다는 뜻이기도하다.**      
      
사실 데이터 바인딩은 주로 클라이언트의 요청 데이터를 가공할 때 쓰이기에 String 만 있어도 충분하다.            
하지만, 조금 더 일반적으로 `특정 타입 <-> 특정 타입`으로도 변환이 가능한 환경도 존재해야하기에         
**`Converter`라는 인터페이스가 등장했다.**       

```java
package org.springframework.core.convert.converter;

@FunctionalInterface
public interface Converter<S, T> {
    @Nullable
    T convert(S source);
}
``` 
`Converter 인터페이스`는 제네릭을 지원하며          
제네릭의 첫번째로 선언된 타입을 두번째로 선언된 타입으로 반환하는 메서드를 제공한다.          
또한, `@FunctionalInterface`로 람다를 이용한 선언까지 가능하다.       

```java
public class EventConverter {
    
    public static class StringToEventConverter implements Converter<String, Event> {
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }
    
    public static class EventToStringConverter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
    }
}
```   
```java
public class EventConverter {

    public static class StringToEventConverter implements Converter<String, Event> {
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConverter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
    }
}
```
위와 같이 S타입을 T타입으로 변환하는 코드를 작성하면 된다.     
`Converter`는 **상태를 가지지 않기에(Stateless) `Thread-safe`하고 빈을 등록해도 된다.**          
**`스프링 레거시의 MVC`를 사용하는 경우 직접 `ConverterRegistry`에 등록해서 사용해야한다.**          
 
**특징**
* S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기.
* 상태 정보 없음 == Stateless == 쓰레드세이프
* ConverterRegistry에 등록해서 사용해야 한다.  
    
##  Converter VS PropertyEditor    
`Converter` 와 `PropertyEditor` 두 클래스 모두 데이터 바인딩에 사용되는 인터페이스다.    
만약 두 인터페이스를 구현한 구현체가 동시에 존재하다면 우선순위는 어떻게 될까?     

```java
public class EventConverter {

    private static final Logger logger = LoggerFactory.getLogger(EventConverter.class);

    public static class StringToEventConverter implements Converter<String, Event> {
        @Override
        public Event convert(String source) {
            logger.info("EventConverter : " + source);
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConverter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            logger.info("EventConverter : " + source.toString());
            return source.getId().toString();
        }
    }
}
```
```java
public class EventEditor extends PropertyEditorSupport {

    private static final Logger logger = LoggerFactory.getLogger(EventEditor.class);

    @Override
    public String getAsText() {
        Event event = (Event) getValue();
        logger.info("EventEditor :" + event.toString());
        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        logger.info("EventEditor :" + text);
        setValue(new Event(Integer.valueOf(text)));
    }

}
```
```java
@RestController
public class EventController {

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        return event.getId().toString();
    }
}
```
```shell
2021-07-19 20:24:45.658  INFO 10640 --- [           main] com.example.core.EventConverter          : EventConverter : 1
```
별다른 설정없이 동작을 하면 기본적으로 `Converter`가 우선순위가 높다.   


```java
@RestController
public class EventController {

    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event) {
        return event.getId().toString();
    }
}
```
```java
2021-07-19 20:25:01.533  INFO 15048 --- [           main] com.example.core.EventEditor             : EventEditor :1
```
단, `@InitBinder`를 사용하여 `webDataBinder`에 `PropertyEditor`를 등록하면         
해당 `Controller` 클래스에 한정하여 `PropertyEditor`가 우선으로 동작한다.             
  

Formatter
● PropertyEditor 대체제
● Object와 String 간의 변환을 담당한다.
● 문자열을 Locale에 따라 다국화하는 기능도 제공한다. (optional)
● FormatterRegistry에 등록해서 사용
public class EventFormatter implements Formatter<Event> {
@Override
public Event parse(String text, Locale locale) throws ParseException {
Event event = new Event();
int id = Integer.parseInt(text);
event.setId(id);
return event;
}
@Override
public String print(Event object, Locale locale) {
return object.getId().toString();
}
}
ConversionService
● 실제 변환 작업은 이 인터페이스를 통해서 쓰레드-세이프하게 사용할 수 있음.
● 스프링 MVC, 빈 (value) 설정, SpEL에서 사용한다.
● DefaultFormattingConversionService
● FormatterRegistry
● ConversionService
● 여러 기본 컴버터와 포매터 등록 해 줌.

스프링 부트
● 웹 애플리케이션인 경우에 DefaultFormattingConversionSerivce를 상속하여 만든
WebConversionService를 빈으로 등록해 준다.
● Formatter와 Converter 빈을 찾아 자동으로 등록해 준다.
