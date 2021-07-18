Validation 추상화
===================  
스프링 프레임워크에서 제공하는 Validation 추상화에 대해서 알아보고자 한다.           
`org.springframework.validation.Validator`는 객체 검증용 인터페이스로          
주로 `MVC 기반의 웹 계층`에서 사용되지만 `서비스`, `데이터`에서 사용해도 좋다.       \  
(물론, javax 기반의 표준 validator가 존재하지만 스프링에서 사용하기에 스프링 validator가 낫다.)    
         
```java
package org.springframework.validation;

public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```
           
가장 큰 특징으로    
`org.springframework.validation.Validator`를 구현한 `LocalValidatorFactoryBean` 클래스를 통해       
자바 표준 Validation인 `Bean Validation`을 지원하여 더욱 확장성 있는 기능을 제공한다.           

```java
org.springframework.validation.Validator.Validator
    ㄴorg.springframework.validation.SmartValidator
       ㄴ
         org.springframework.validation.beanvalidation.SpringValidatorAdapter <- ★LocalValidatorFactoryBean★
       r
javax.validation.Validator.Validator    
```

* JSR-303(Bean Validation 1.0)   
* JSR-349(Bean Validation 1.1)  
* JSR-380(Bean Validation 2.0)  






```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```
스프링 부트에서는 위와 같이 의존성을 추가해주면 java 표준 Bean Validation 을 사용할 수 있다.   








___  


]
인터페이스
● boolean supports(Class clazz): 어떤 타입의 객체를 검증할 때 사용할 것인지 결정함
● void validate(Object obj, Errors e): 실제 검증 로직을 이 안에서 구현
○ 구현할 때 ValidationUtils 사용하며 편리 함.

스프링 부트 2.0.5 이상 버전을 사용할 때
● LocalValidatorFactoryBean 빈으로 자동 등록
● JSR-380(Bean Validation 2.0.1) 구현체로 hibernate-validator 사용.
● https://beanvalidation.org/
