MessageSource
========================
**Spring MessageSource**는 국제화 (i18n) 기능을 제공하는 인터페이스이다.             
즉, 메세지 설정 파일을 모아놓고 각 국가마다 로컬라이징을 함으로서 각 지역에 맞춘 국제화 기능을 제공한다.         
       
# messages.properties
스프링 부트를 사용한다면 `messages.properties`를 통해 국제화 기능을 바로 적용할 수 있다.   
   
```
[파일이름]_[언어]_[국가].properties
```
      
메세지 설정 파일을 셋업하기 위해서는 `.properties` 확장자가 붙은 프로퍼티 파일에    
`[파일이름]_[언어]_[국가].properties` 형식으로 메세지 파일을 추가해야 한다.    

* message.properties : 기본 메시지. 시스템의 언어 및 지역에 맞는 프로퍼티 파일이 존재하지 않을 경우에 사용.
* message_en.properties : 영어 메시지.
* message_ko.properties : 한글 메시지.
* messages_ko_kr.properties : 한글 국가 메시지(한글 메시지만 사용해도 충분하다.)     
* message_en_UK.properties : 영국 메세지, 영국을 위해 `[언어]_[국가]`를 사용한다.      
   

리로딩 기능이 있는 메시지 소스 사용하기

```
@Bean
public MessageSource messageSource() {
    var messageSource = new ReloadableResourceBundleMessageSource();
    messageSource.setBasename("classpath:/messages");
    messageSource.setDefaultEncoding("UTF-8");
    messageSource.setCacheSeconds(3);
    return messageSource;
}
```
