
# AutoConfiguration 자동 환경 설정 이해하기         
자동 환경 설정은 스프링 부트의 장점이며 매우 중요한 역할을 수행합니다.               
스프링 부트의 자동 설정 (Spring Boot auto-configuration)은 Web, H2, JDBC를 비롯해 약 100여 개의 자동 설정을 제공합니다.            
그리고 새로 추가되는 라이브러리(JAR)는 스프링 부트 자동-설정 의존성에 따라서 설정이 자동 적용됩니다.            
          
만약 H2 의존성이 클래스 경로에 존재한다면 자동으로 인메모리 데이터베이스에 접근합니다.           
이런 마법같은 자동 설정은 ```@EnableAutoConfiguration```또는 이를 포함한 ```@SpringBootApplication```중 하나를 사용하면 됩니다.        
(```@EnableAutoConfiguration```은 항상 ```@Configuration```과 함께 사용해야 합니다.)           
     
여기서 ```@SpringBootApplication```은 자동 설정뿐만 아니라 부트 실행에 있어서 필수적인 어노테이션이기도 합니다. (```Application.java```)            
그럼 ```@SpringBootApplication```의 내부는 어떻게 구성되어있는지 살펴보며 자동 환경 설정의 원리를 차근차근 파악해보겠습니다.       
     
## 5.1. 자동 환경 설정 어노테이션   
기존의 스프링 프레임워크를 사용했다면 의존성을 일일이 빈 ```<bean>```으로 설정했을 겁니다.        
스프링 부트는 관련 의존성을 스타터(starter)라는 묶음으로 제공하며 **수동 설정을 지양**합니다.      
그렇다면 어떻게 스타터에 있는 자동 설정이 적용되는지 원리를 알아봅시다.       
   
먼저 코드상에서 분석을 해보겠습니다.      
```@SpringBootApplication```의 내부 코드는 다음과 같습니다.      
    
**@SpringBootApplication**       
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration // (1)
@EnableAutoConfiguration // (2)
@ComponentScan(excludeFilters = { // (3)
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```
**소스 코드 해석**    
```java     
@SpringBootConfiguration :   

* 스프링 부트의 설정을 나타내는 어노테이션입니다.   
* 스프링의 @Configuration을 대체하며 스프링 부트 전용으로 사용합니다.   
* 예를 들어 스프링 부트의 테스트 어노테이션 (@SpringbootTest)을 사용할 때 찾기 알고리즘을 사용하여   
  계속 @SpringBootConfiguration 어노테이션을 찾기 때문에 스프링 부트에서는 필수 어노테이션 중 하나입니다. 

_____________________________________________________________________________________________________
@EnableAutoConfiguration :   

* 자동 설정의 핵심 어노테이션입니다.   
* 클래스 경로에 지정된 내용을 기반으로 영리하게 설정 자동화를 수행합니다.   
* 특별한 설정값을 추가하지 않으면 기본값으로 작동합니다.     
   
_____________________________________________________________________________________________________
@ComponentScan( basePackages-경로 ) :   

* 특정 패키지 경로를 기반으로 @Configuration에서 사용할 @Component 설정 클래스를 찾습니다.       
* @ComponentScan의 basePackages 프로퍼티값에 별도의 경로를 설정하지 않으면    
  @ComponentScan이 위치한 패키지가 루트 경로로 설정됩니다.   
```    
```@SpringBootApplication``` 어노테이션은 ```@SpringBootConfiguration``` + ```@EnableAutoConfiguration``` + ```@ComponentScan```의 조합이다.  
이 중에서 ```@EnableAutoConfiguration```이 우리가 살펴볼 자동 환경 설정의 핵심 어노테이션이다.   

## 5.2. ```@EnableAutoConfiguration``` 살펴보기 
자동 설정을 관장하는 ```@EnableAutoConfiguration``` 내부를 살펴보자    

**@EnableAutoConfiguration**
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class) // ★
public @interface EnableAutoConfiguration {
```
위 코드중 자동 설정을 지원해주는 어노테이션은 ```@Import(AutoConfigurationImportSelector.class)``` 입니다.          
클래스명을 통해 알 수 있는 점은 ```임포트할 자동 설정을 선택한다```는 의미로 해석됩니다.         
       
```AutoConfigurationImportSelector```클래스도 자세히 살펴보겠습니다.    
    
**AutoConfigurationImportSelector**   
```java 
public class AutoConfigurationImportSelector
		implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
		BeanFactoryAware, EnvironmentAware, Ordered {
		
	...............
	...............
	...............
	
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata,
				attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return StringUtils.toStringArray(configurations);
	}
	
	...............
	...............
	...............
}	
```
내부 코드가 좀 복잡해 보이지만 핵심과 과정 위주로 살펴보겠습니다.     
```AutoConfigurationImportSelector``` 클래스는 ```DeferredImportSelector```인터페이스를 구현한 클래스로    
**오버라이드 받은 ```selectImports()```메서드가 자동 설정할 빈을 결정합니다.**      
      
```java
	AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
	AnnotationAttributes attributes = getAttributes(annotationMetadata);
	List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);   
```
모든 후보 빈을 ```getCandidateConfigurations()``` 메서드를 사용해 불러옵니다.           
더 자세히 설명하자면 ```META-INF/spring.factories```에 정의된 자동 설정할 클래스들을 먼저 불러옵니다.      
대략 100여 개 정도의 설정이 미리 정의되어 있습니다.           
   
즉, 자동 설정할 클래스 목록들을 리스트로 저장하는 것입니다.      
  
```java

	configurations = removeDuplicates(configurations);
	Set<String> exclusions = getExclusions(annotationMetadata, attributes);
```

스프링 부트 스타터를 여러 개 등록하여 사용할 경우 내부에 중복된 빈이 설정될 경우가 빈번합니다.          
**이러한 경우를 위해 중복된 설정 ```removeDuplicates()``` 과 제외할 설정 ```getExclusions()```을 제외시켜줍니다.**       

```java
	checkExcludedClasses(configurations, exclusions);
	configurations.removeAll(exclusions);
	configurations = filter(configurations, autoConfigurationMetadata);
	fireAutoConfigurationImportEvents(configurations, exclusions);
	return StringUtils.toStringArray(configurations);
```
마지막으로 이 중에서 프로젝트에서 사용하는 빈만 임포트할 자동 설정 대상으로 선택합니다.      
     
그렇다면 빈의 등록과 자동 설정에 필요한 파일은 무엇일까요?          
아래와 같은 파일이 빈 등록과 자동 설정에 사용됩니다.          
     
* META-INF/spring.factories :           
자동 설정 타깃 클래스 목록입니다.        
즉, 이곳에서 선언되어 있는 클래스들이 ```@EnableAutoConfiguration``` 사용 시 자동 설정 타깃이 됩니다.        
       
* META-INF/spring-configuration-metadata.json :      
자동 설정에 사용할 프로퍼티 정의 파일입니다.         
미리 구현되어 있는 자동 설정에 프로퍼티만 주입시켜주면 됩니다.       
따라서 별도의 환경 설정은 필요 없습니다.       
         
* org/springframework/boot/autoconfigure :      
미리 구현해놓은 자동 설정 리스트입니다.       
이름은 ```[특정 설정의 이름]Auto Configuration``` 형식으로 지정되어 있으며 모두 자바 설정 방식을 따르고 있습니다.       
   
위 파일 모두 ```spring-boot-autoconfiguration```에 미리 정의되어 있으며      
지정된 프로퍼티값을 사용하여 설정 클래스 내부의 값들을 변경할 수 있습니다.      
___
예를 들어 H2를 자동 설정한다고 가정합니다.   
   
1. 먼저 ```spring.factories```에서 자동 설정 대상에 해당되는지 확인합니다.           
      
**spring.factories**    
```
~ 생략 ~
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\    
~ 생략 ~
```  
   
2. ```spring-configuration-metadata.json```에 주요 프로퍼티값들은 무엇이고 어떤 타입으로 설정할 수 있는지도 확인합니다.     
     
**spring-configuration-metadata.json**     
```javascript
    ...
    {
      "sourceType": "org.springframework.boot.autoconfigure.h2.H2ConsoleProperties",
      "defaultValue": "\/h2-console",
      "name": "spring.h2.console.path",
      "description": "Path at which the console is available.",
      "type": "java.lang.String"
    },
    ...
```
H2 경로의 기본값은 ```/h2-console```이고 ```String 형```인 것을 확인할 수 있습니다.         
다른 경로로 변경하기 위해서는 ```application.properties```나 ```application.yml```에 프로퍼티값을 추가합니다.           
       
**application.yml에서 H2 PATH 변경**           
```yml
spring:
  h2:
    console:
      path: /h2-test
```
    
**application.yml**   
```yml  
server:
  port: 80
---
spring:
  profiles: local
server:
  port: 8080
---
spring:
  profiles: dev
server:
  port: 8081
---
spring:
  profiles: real
server:
  port: 8082
---
property:
  test:
    name: property depth test
propertyTest: test
propertyTestList: a,b,c

fruit:
  list:
    - name: banana
      color: yellow
    - name: apple
      color: red
    - name: water melon
      color: green
      
spring:      
  h2:
    console:
      path: /h2-test      
```  
위와 같이 프로퍼티값을 추가하는 것만으로도 앞서 살펴본 자동 환경 설정에 자동으로 적용되어 애플리케이션이 실행됩니다.      
    
지금까지 자동 환경 설정의 원리와 설정 파일을 수정하여 설정값을 수정하는 방법에 대해 알아보았습니다.     
사실 스프링 프로퍼티 문서를 사용하면 더 쉽게 프로퍼티값을 확인할 수 있습니다.            
아래 페이지에 접속한 후 ```A. Common application properties``` 카테고리를 클릭하여 이동하면 됩니다.          
https://docs.spring.io/spring-boot/docs/current/reference/html/   
   
하지만 위 작업을 통해 동작 원리가 어떻게 추상화되었는지 내부 코드를 통해 파악하면 이 과정을 좀 더 깊게 이해할 수 있기 때문입니다.      
스프링 부트의 자동 설정을 정확하게 파악했으니 이제 스프링 부트를 부트답게 설정해 더 효율적으로 사용하는 방법을 알아보겠습니다.      
   
## 5.3. 자동 설정 어노테이션 살펴보기   
스프링 부트는 자동 설정이 적용되는 **조건, 시점** 등에 다라 다양한 어노테이션을 지원합니다.        
이를 잘 알아두면 설정 관리 능력을 향상시킬 수 있습니다.       
       
또한 나만의 스타터를 생성하여 최적화된 자동 설정 관리 능력을 향상시킬 수 있습니다.      
물론 이러한 경우는 흔치 않지만 팀원들에게 공통된 스타터를 제공하여       
프로젝트의 설정을 간소화하고 싶을 때 혹은 오픈 소스로 사용하고 싶을때도 사용 가능합니다.       

자동 설정 관련 어노테이션을 먼저 살펴보겠습니다.   
다음은 자동 설정을 위한 조건 어노테이션 입니다.   

[그림]
  
```java
@ConditionalOnBean : 해당하는 빈 클래스나 이름이 미리 빈 팩토리에 포함되어 있을 경우  

@ConditionalOnClass : 해당하는 클래스가 클래스 경로에 있을 경우 

@ConditionalOnCloudPlatform : 해당하는 클라우드 플랫폼이 활용 상태일 경우   

@ConditionalOnExpression : SpEL에 의존하는 조건일 경우 

@ConditionalOnJava : JVM 버전이 일치하는 경우  

@ConditionalOnJndi : JNDI가 사용 가능하고 특정 위치에 있는 경우

@ConditionalOnMissingBean : 해당하는 빈 클래스나 이름이 미리 빈 팩토리에 포함되지 않은 경우   

@ConditionalOnMissingClass : 해당하는 클래스가 클래스 경로에 없을 경우   

@ConditionalOnNotWebApplication : 웹 애플리케이션이 아닌 경우  

@ConditionalOnProperty : 특정한 프로퍼티가 지정한 값을 갖는 경우    

@ConditionalOnResource : 특정한 리소스가 클래스 경로에 있는 경우 

@ConditionalOnSingleCandidate : 지정한 빈 클래스가 이미 빈 팩토리에 포함되어 있고 단일 후보자로 지정 가능한 경우      

@ConditionalOnWebApplication : 웹 애플리케이션인 경우     
```
다음은 자동 설정을 위한 순서 어노테이션 입니다.  

[사진]  

```java
@AutoConfigureAfter : 지정한 특정 자동 설정 클래스들이 적용된 이후에 해당 자동 설정 적용   

@AutoConfigureBefore : 지정한 특정 자동 설정 클래스들이 적용되기 이전에 해당 자동 설정 적용  

@AutoConfigureOrder : 자동 설정 순서 지정을 위한   
스프링 프레임워크의 @Order 변형 어노테이션 기준의 설정 클래스에는 영향을 주지 않고 자동 설정 클래스들 간의 순서만 지정    
```

자동 설정 관련 어노테이션을 살펴보았으니 이제, 어떻게 쓰이는지 알아보겠습니다.    
     
**H2ConsoleAutoConfiguration**       
```java
/*
 * Copyright 2012-2017 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.boot.autoconfigure.h2;

import org.h2.server.web.WebServlet;

import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication.Type;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * {@link EnableAutoConfiguration Auto-configuration} for H2's web console.
 *
 * @author Andy Wilkinson
 * @author Marten Deinum
 * @author Stephane Nicoll
 * @since 1.3.0
 */
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass(WebServlet.class)
@ConditionalOnProperty(prefix = "spring.h2.console", name = "enabled", havingValue = "true", matchIfMissing = false)
@EnableConfigurationProperties(H2ConsoleProperties.class)
public class H2ConsoleAutoConfiguration {

	private final H2ConsoleProperties properties;

	public H2ConsoleAutoConfiguration(H2ConsoleProperties properties) {
		this.properties = properties;
	}

	@Bean
	public ServletRegistrationBean<WebServlet> h2Console() {
		String path = this.properties.getPath();
		String urlMapping = (path.endsWith("/") ? path + "*" : path + "/*");
		ServletRegistrationBean<WebServlet> registration = new ServletRegistrationBean<>(
				new WebServlet(), urlMapping);
		H2ConsoleProperties.Settings settings = this.properties.getSettings();
		if (settings.isTrace()) {
			registration.addInitParameter("trace", "");
		}
		if (settings.isWebAllowOthers()) {
			registration.addInitParameter("webAllowOthers", "");
		}
		return registration;
	}

}
```
조건 어노테이션에 따라 H2 자동 설정 적용 여부를 결정합니다.        
위 클래스에서는 다음 3가지 조건이 부합할 때 ```H2ConsoleAutoConfiguration```가 적용됩니다.        
      
1. ```@ConditionalOnWebApplication(type = Type.SERVLET)``` : 웹 어플리케이션일 때      
2. ```@ConditionalOnClass(WebServlet.class)``` : ```WebServlet.class```가 경로에 있을 때   
3. ```@ConditionalOnProperty(prefix = "spring.h2.console", name = "enabled", havingValue = "true", matchIfMissing = false)``` :   
```spring.h2.console.enabled``` 값이 ```true``` 일 때        
  
자동 설정 프로퍼티가 적용될 때 ```H2ConsoleProperties``` 클래스 타입으로 H2 관련 프로퍼티값을 매핑하여 사용하게 됩니다.         
이러한 작업을 스프링 프레임워크에서는 일일이 작업해야 했지만 부트에서는 미리 설정한 방식대로 애플리케이션에 적용하게 되어있습니다.   
   
## 5.4. H2 Console 자동 설정 적용하기   
기존 스프링 방 사용한다면 아래와 같은 방식을 취했어야 했습니다.     

```gradle
compile('com.h2database:h2')    
```
```java
@Configuration
public class DataSourceConfig{
	
	@Bean
	ServletRegistrationBean h2ServletRegistration(){
		ServletRegistration registrationBean = new ServletRegistrationBean(new WebServlet);   
		registrationBean.addUrlMappings("/console/*");   
		return registrationBean;
	}
}
```
하지만 위 같은 방법은 스프링 부트 자동 설정을 모를때 사용하는 방법입니다.         
우리는 스프링 부트 자동 설정 방식을 알고 있으니 단순히 **설정 프로퍼티값만 바꾸면 됩니다.**     
   
우선 H2프로퍼티값으로 기본 설정값이 무엇인지 살펴보겠습니다.   
     
![h2 프로퍼티](https://user-images.githubusercontent.com/50267433/85651823-03c0ff00-b6e4-11ea-9870-680f8caf2214.PNG)           
    
**spring-configuration-metadata.json**   
```
    {
      "sourceType": "org.springframework.boot.autoconfigure.h2.H2ConsoleProperties",
      "defaultValue": false,
      "name": "spring.h2.console.enabled",
      "description": "Whether to enable the console.",
      "type": "java.lang.Boolean"
    },
    {
      "sourceType": "org.springframework.boot.autoconfigure.h2.H2ConsoleProperties",
      "defaultValue": "\/h2-console",
      "name": "spring.h2.console.path",
      "description": "Path at which the console is available.",
      "type": "java.lang.String"
    },
    {
      "sourceType": "org.springframework.boot.autoconfigure.h2.H2ConsoleProperties$Settings",
      "defaultValue": false,
      "name": "spring.h2.console.settings.trace",
      "description": "Whether to enable trace output.",
      "type": "java.lang.Boolean"
    },
    {
      "sourceType": "org.springframework.boot.autoconfigure.h2.H2ConsoleProperties$Settings",
      "defaultValue": false,
      "name": "spring.h2.console.settings.web-allow-others",
      "description": "Whether to enable remote access.",
      "type": "java.lang.Boolean"
    },
```
**동작 예시**   
```properties
spring.h2.console.enabled=false
spring.h2.console.path=/h2-console
spring.h2.console.settings.trace=false
spring.h2.console.settings.web-allow-others=false
```
스프링부트 API 문서를 참조했을 때 위와 같은 결과가 나옵니다.           
다른 방법으로 알 수 있는 점은 **spring-configuration-metadata.json** 을 참고하시면 됩니다.       
      
분명히 ```H2ConsoleAutoConfiguration```에서      
```@ConditionalOnProperty(prefix = "spring.h2.console", name = "enabled", havingValue = "true", matchIfMissing = false)``` 형태     
즉, ```spring.h2.console.enabled``` property 의 값이 ```true```일 경우에 동작한다는 조건이 있었는데      
```spring.h2.console.enabled=false``` 디폴트 값으로 ```false``` 가 되어있습니다.        
그렇기에 h2 가 실행이 되지 않았던 것이고 다르게 말하면 ```spring.h2.console.enabled=true```로 바꿔주면 h2를 사용할 수 있습니다.         
     
이제 ```application.properties``` 나 ```application.yml``` 파일을 조작하여 콘솔을 활성화 하겠습니다.   
   
```yml  
# h2 메모리 db를 사용하기 위한 설정 
datasource:
  url: jdbc:h2:mem:testdb
  
spring:
  h2:
    console:
      enabled: true;
```
   
**application.yml**   
```yml
server:
  port: 80
---
spring:
  profiles: local
server:
  port: 8080
---
spring:
  profiles: dev
server:
  port: 8081
---
spring:
  profiles: real
server:
  port: 8082
---
property:
  test:
    name: property depth test
propertyTest: test
propertyTestList: a,b,c

fruit:
  list:
    - name: banana
      color: yellow
    - name: apple
      color: red
    - name: water melon
      color: green

# h2 메모리 db를 사용하기 위한 설정
datasource:
  url: jdbc:h2:mem:testdb

spring:
  h2:
    console:
      enabled: true
      path: /h2-test
```


컴파일에 포함되도록 H2의존성을 설정했습니다.        
H2 메모리 데이터베이스로 보통 테스트용으로만 쓰입니다.      
주 저장소가 아니기 때문에 불필요하게 컴파일 의존성에 포함될 필요가 없습니다.     
        
이제 콘솔을 위한 설정을 했다면 이제는 런타임 시점에만 의존하도록 다음과 같이 바꿔줘도 됩니다.    
   
**build.gradle**
```gradle

buildscript {
	ext{
		springBootVersion = '2.0.3.RELEASE'
	}
	repositories {
		mavenCentral()
		jcenter()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'community'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
	jcenter()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.projectlombok:lombok')
	compile('com.h2database:h2')
	testCompile('org.springframework.boot:spring-boot-starter-test'){
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}

```
사실 다른 관점에서 보면 이 모든 원리를 파악하는게 불필요하게 느껴질 수도 있습니다.   
하지만 자동 설정은 스프링 부트에서 매우 중요합니다.   
따라서 한 번쯤 어떻게 동작하는지 살펴볼 필요가 있습니다.   
스프링 부트의 환경을 제어하고 효과적으로 개발 환경을 최적화하는데 도움이 될 것입니다.    

***
# 6. 마치며   
인텔리제이와 그레이들을 설치하고 스프링 부트의 꽃이라고 할 수 있는 자동 환경 설정을 살펴보았습니다.   
환경 설정만큼은 내부를 이해하는 것이 좋습니다.    
그래야 앞으로 다양한 자동화 설정을 적용할 때 무엇이 적용되었는지 확인하고 원하는 최적의 설정을 반영할 수 있을테니까요   
앞에서 제시한 절차를 따라 본인 스스로가 다른 설정도 찾고 적용하는 연습을 한다면 누구보다 효과적으로 스프링 부트를 사용할 수 있을겁니다.   
  


  
```
스프링에서 프로퍼티에 대한 설정은 매우 중요하다.  
프로퍼티**   
* 다양한 방법으로 정의할 수 있는 설정값
* Environment의 역할은 프로퍼티 소스 설정 및 프로퍼티 값 가져오기

프로퍼티에는 우선 순위가 있다.
**StandardServletEnvironment의 우선순위**   

1. ServletConfig 매개변수
2. ServletContext 매개변수
3. JNDI (java:comp/env/)
4. JVM 시스템 프로퍼티 (-Dkey=”value”)
5. JVM 시스템 환경 변수 (운영 체제 환경 변수)

@PropertySource

   
# @PropertySource    
   
@PropertySource을 이용하면 **properties 파일을 불러오고 사용할 수 있다.**     


● Environment를 통해 프로퍼티 추가하는 방법
스프링 부트의 외부 설정 참고
● 기본 프로퍼티 소스 지원 (application.properties)
● 프로파일까지 고려한 계층형 프로퍼티 우선 순위 제공
```
## 프로파일에 따른 환경 구성 분리   
실제 서비스는 **`테스트`, `로컬`, `개발`, `운영`의 설정을 나누는 경우가 많다.**           
이러한 개발 환경을 나누기 위해 **`profile 설정` 및 `profile 별로 프로퍼티 파일`을 작성한다.**          

   
**YAML** 파일에서 프로퍼티 설정을 구분하는 방법은 간단합니다.   
다음과 같이 ```---``` 을 기준으로 설정값을 나눕니다.   
    
**application.yml**
```yml   
server:
  port: 80
---
spring: 
  profiles: local
server:
  port: 8080
---
spring:
  profiles: dev
server:
  port: 8081
---
spring:
  profiles: real
server:
  port: 8082
---
```
최상단에 ```server.port``` 프로퍼티값을 80으로 설정한 부분은 프로파일과는 상관없이 디폴트로 정의되는 영역입니다.       
**또다른 방법으로 ```application-[profile].yml```을 이용하는 겁니다.**                    
     
```[profile]```에 원하는 프로파일 값으로 YAML 파일을 추가하면             
애플리케이션 실행 시 ```application-[profile].yml```에서 지정한 프로파일값을 바탕으로 실행됩니다.          
즉, **호출한 ```yml```이 우선순위가 되고 이외에 존재하는 프로퍼티들도 ```applcaiton.yml```순서에 따라 설정됩니다.**            
   
### 이번에는 프로파일값을 적용하여 애플리케이션을 실행하는 방법을 알아보겠습니다.   
```local```, ```dev```, ```real```과 같이 각각의 프로파일값을 따로 지정하여 애플리케이션을 실행한다 가정합니다.   
스프링 부트 프로젝트는 JAR 파일로 빌드하기에 서버에서 직접 간단한 명령으로 실행할 수 있습니다.    
   
아래 와 같이 실행하여 프로파일 값을 활성화할 수 있습니다.

```
$ java -jar -D spring.profiles.active=dev
```
```
java -Dspring.profiles.active=dev -jar [jar파일명].jar
java -jar [jar파일명].jar --spring.profiles.active=dev
```
출처: https://freestrokes.tistory.com/106 [FREESTROKES DEVLOG]     
      
      
인텔리제이는 스프링 부트 실행 플러그인을 따로 사용하기 때문에       
```Edit Configurations``` 버튼을 눌러 ```Run/Debug configurations``` 창을 실행하고     
스프링 부트 플러그인의 프로파일 값을 할당하면 됩니다.      

# 참고 
출처 : https://velog.io/@lsb156/Spring-Boot-Properties-Usage#configurationproperties    
출처: https://luvstudy.tistory.com/99 [파란하늘의 지식창고]
