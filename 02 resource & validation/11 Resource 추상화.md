Resource 추상화
================= 
`애플리케이션 컨텍스트`는 `IOC 컨테이너`로 빈 팩토리 기능뿐만 아니라 **다양한 부가 기능을 제공해준다.**      
대표적으로 `ResourceLoader`, `EvenetPublisher`, `MessageSource`, `Environment` 등등이 있었다.     
     
지금까지는 `IOC 컨테이너`에 대해서 살펴보았다면     
지금부터는 스프링 레퍼런스의 아주 많은 양을 차지하는 `추상화`에 대해서 알아보고자 한다.      
    
해당 클래스 내에서 프로젝트 내 존재하는 resource를 읽어들이는 코드를 실행했을 때    
기본적으로는 'WebApplicationContext'가 동작해서 resource를 읽어들이는 것이고(이때 resource는 ServletContextResource 입니다.)   
만약, 접두어(classpath 혹은 file)가 붙는 경우에는 그에 맞는 resource 타입으로 리소스를 찾아준다    
   
# Resource         
`Resource`란, 기본적으로 `자원`이라는 말이다.                
웹 애플리케이션을 기준으로 **프로젝트내에 존재**하며 **컴파일 대상이되지 않는 자원**들이 이에 해당하며             
알기 쉽게 표현하자면 **`.html`, `.css`, `.js` 같은 것들이 대표적인 예다.**            
                      
이러한 Resource는 주로 **HTTP 통신을 통해 데이터를 수신할 수 있으며 이 과정에서 URI이 사용된다.**          
URI 사용되는 이유는 **Resource를 고유한 문자열을 통해 식별하여 가져올 수 있기 때문이다.**                
현대에는 **대부분 URL을 기반으로 리소스를 찾으며(REST API) URN은 거의 사용하지 않는다.**       
  
* URI : Uniform Resource Inteface
  * URL : Uniform resource Location
  * URN : Uniform resource Name
   
참고로, HTTP/HTTPS/FTP 등을 이용할 수 있다.     
   
# Resource 추상화        
**Resource 추상화**는 `org.springframework.core.io.Resource` 라이브러리를 의미한다.   
          
자바의 표준 라이브러리인 `java.net.URL 클래스`와 `다양한 URL 접두어`들에 대한       
**표준 핸들러(인터셉터)** 는 안타깝게도 `저수준 리소스`에 모두 접근하기에는 전혀 충분하지 않다.       
   
예를 들면, URL을 기준으로 `HTTP/HTTPS/FTP` 같은 프로토콜을 이용하여 데이터 통신을 하기에    
`HTTP/HTTPS/FTP`에 맞는 접두어는 준비 되어있지만   
`클래스패스`를 기준으로 리소를 가져오는 기능이 없으며             
`ServletContext`를 기준으로 상대 경로로 읽어오는 기능 또한 부재하다.       
         
물론, **새로운 핸들러(인터셉터)**를 등록하여             
특별한 `URL 접두사`를 만들어 사용할 수는 있지만 **구현이 복잡하고 편의성 메소드가 부족하다.**          
        
이를 해결하기 위해 스프링 코어에서         
디폴트로 사용할 수 있는 `org.springframework.core.io.Resource`를 개발하여 리소스 자체를 추상화했다.        
(리소스를 가져오는 기능은 이전 `ApplicationContext`를 활용하여 가져오는 방법들을 통해 학습했다.)           

**특징**  
* java.net.URL를 추상화    
* 스프링 내부에서 많이 사용하는 인터페이스   
  
**추상화 이유**
* 기존, java.net.URL은 클래스패스 기준으로 리소를 가져오는 기능이 없었다.         
* ServletContext를 기준으로 상대 경로로 읽어오는 기능 부재         
* 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만   
  구현이 복잡하고 편의성 메소드가 부족하다.    
