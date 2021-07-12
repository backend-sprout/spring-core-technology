ApplicationEventPublisher
================================      

옵저버 패턴 구현체로 이벤트 기반의 프로그래밍에 필요한 기능들을 제공해준다.   
  
**이벤트 만들기**  
* `ApplicationEvent` 상속  
  * 이 과정에서 이벤트를 발생시킨 Object 소스를 `ApplicationEvent` 클래스 즉, 상위 생성자에 넣어야했다.       
* 스프링 4.2 부터는 이 클래스를 상속받지 않아도 이벤트로 사용할 수 있다.       
  * 이 말은 이전과 달리 이벤트 소스를 넣을 수 있고 안 넣을 수 있으며 상위 생성자에 안 넣어줘도 되는식으로 바뀌었다.     
             
**이벤트 발생 시키는 방법**       
* ApplicationEventPublisher.publishEvent();           
* `.publishEvent()`에 이벤트 객체를 넣는다. -> 이벤트 발생시킨다.       
     
**이벤트 처리하는 방법**   
* `ApplicationListener<이벤트>` 구현한 `Handler 클래스`를 만들어서 빈으로 등록한다. -   
    * 이 과정에서 `onApplicationEvent()`메서드를 오버라이딩해서 이벤트 발생시 실행할 동작을 구현한다.         
* 스프링 4.2 부터는 상속을 받지 않고, `@EventListener`를 사용해서 메소드에 사용하여 동작시킬 수 있다.         
* 기본적으로는 synchronized.       
* 순서를 정하고 싶다면 @Order와 함께 사용.  
* 비동기적으로 실행하고 싶다면 @Async와 함께 사용.   
    
**스프링이 제공하는 기본 이벤트**         
* ContextRefreshedEvent: ApplicationContext를 초기화 했더나 리프래시 했을 때 발생.      
* ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생.     
* ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생.        
* ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생     
* RequestHandledEvent: HTTP 요청을 처리했을 때 발생.     
