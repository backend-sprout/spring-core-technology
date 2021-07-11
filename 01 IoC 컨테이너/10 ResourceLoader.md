ResourceLoader    
================          
`ResourceLoader`는 리소스를 읽어오는 기능을 제공하는 인터페이스다.              
       
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:test.txt");   // resources/test.txt 파일을 읽는다.     
        System.out.println(resource.exists());                                  // 리소스가 존재하는지 여부 반환       
        System.out.println(resource.getDescription());                          // 리소스에 대한 풀패키지 경로 반환  
        System.out.println(Files.readString(Path.of(resource.getURI())));       // URI 경로를 이용해서 리소스에 대한 내부 값      
        System.out.println(resource.getURI().getPath());                        // 리소스에 대한 URI 경로 반환       
    }
}
```

   

