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
Converter를 구현하여 데이터바인딩 로직을 알맞게 정의했다면 ConverterRegistry에 등록해서 사용해야한다.     

```java
public class StringToEventConverter implements Converter<String, Event> {
    @Override
    public Event convert(String source) {
        Event event = new Event();
        event.setId(Integer.parseInt(source));
        return event;
    }
}
```

**특징**
* S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기.
* 상태 정보 없음 == Stateless == 쓰레드세이프
* ConverterRegistry에 등록해서 사용해야 한다.  


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
