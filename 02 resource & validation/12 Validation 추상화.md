Validation 추상화
===================  
스프링 프레임워크에서 제공하는 Validation 추상화에 대해서 알아보고자 한다.           
`org.springframework.validation.Validator`는 객체 검증용 인터페이스로          
주로 `MVC 기반의 웹 계층`에서 사용되지만 `서비스`, `데이터`에서 사용해도 좋다.         
(물론, javax 기반의 표준 validator가 존재하지만 스프링에서 사용하기에는 스프링 validator가 낫다.)    
         
```java
package org.springframework.validation;

public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```
`org.springframework.lang`와 연계하여 아래와 같은 어노테이션을 적용 및 지원해주고    
   
* NonNull
* NonNullApi
* NonNullFields
* Nullable 


가장 큰 특징으로 `org.springframework.validation.Validator`를 구현한 `LocalValidatorFactoryBean` 클래스를 통해       
자바 표준 Validation인 `Bean Validation`을 지원하여 더욱 확장성 있는 기능을 제공한다.           

```java
org.springframework.validation.Validator.Validator    javax.validation.Validator.Validator
  ▲                                                                    ▲
  ㄴ  org.springframework.validation.SmartValidator                    |
        ▲                                                              |     
        ㄴ org.springframework.validation.beanvalidation.SpringValidatorAdapter 
             ▲                                                                   
             ㄴ  LocalValidatorFactoryBean
```

* JSR-303(Bean Validation 1.0)   
* JSR-349(Bean Validation 1.1)  
* [JSR-380(Bean Validation 2.0)](https://beanvalidation.org/)    

여기서 언급된 자바 표준 Validation 은 아래와 같은 어노테이션을 지원한다.      
`javax.validation.constraints`에 속하고 있다.      

```java
import javax.validation.constraints.AssertTrue;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import javax.validation.constraints.Email;

public class User {

    @NotNull(message = "Name cannot be null")
    private String name;

    @AssertTrue
    private boolean working;

    @Size(min = 10, max = 200, message = "About Me must be between 10 and 200 characters")
    private String aboutMe;

    @Min(value = 18, message = "Age should not be less than 18")
    @Max(value = 150, message = "Age should not be greater than 150")
    private int age;

    @Email(message = "Email should be valid")
    private String email;

    // standard setters and getters 
}
```
|어노테이션|주요 속성|설명|지원 타입|
|---------|--------|----|---------|
|@AssertTrue<br>@AssertFalse||값이 true인지 또는 false인지 검사한다.<br>null은 유효하다고 판단한다.|boolean<br>Boolean|
|@DecimalMax<br>@DecimalMin|String value<br>최댓값 또는 최솟값<br><br>boolean inclusive<br>지정값 포함 여부<br>기본 값 true|지정한 값보다 작거나 같은지<br>또는 크거나 같은지 검사한다.<br>inclusive가 false면<br>value로 지정한 값은 포함하지 않는다.<br>null은 유효하다고 판단한다.|BigDecimal<br>BigInteger<br>CharSequence<br>byte, short, int, long 및<br>각 래퍼 타입|
|@Max<br>@Min|long value|지정한 값보다 작거나 같은지<br>또는 크거나 같은지 검사한다.<br>null은 유효하다고 판단한다.<br>|BigDecimal<br>BigInteger<br>byte, short, int, long 및<br> 관련 래퍼 타입|
|@Digits|int integer<br>허용 가능한<br>정수 자릿수<br><br>int fraction<br>허용 가능한<br>소수점 이하 자릿수|자릿수가 지정한 크기를 넘지 않는지 검사한다.<br>null은 유효하다고 판단한다.|BigDecimal<br>BigInteger<br>CharSequence<br>byte, short, int, long 및<br> 관련 래퍼 타입|
|@Size|int min<br>최소 크기<br>기본 값 0<br><br>int max<br>최대 크기<br>기본 값|길이나 크기가 지정한 값 범위에 있는지 검사한다.<br>null은 유효하다고 판단한다|CharSequence<br>Collection<br>Map<br>배열|
|@Null<br>@NotNull||값이 null인지 또는 null이 아닌지 검사한다.||
|@Pattern|String regexp<br>정규표현식|값이 정규표현식에 일치하는지 검사한다.<br>null은 유효하다고 판단한다.|CharSequence|
|@NotEmpty (2)||문자열나 배열의 경우<br>null이 아니고 길이가 0이 아닌지 검사한다.<br><br>콜렉션의 경우<br>null이 아니고 크기가 0이 아닌지 검사한다.|CharSequence<br>Collection<br>Map<br>배열|
|@NotBlank (2)||null이 아니고 최소한 한 개 이상의<br>공백이 아닌 문자를 포함하는지 검사한다.|CharSequence|
|@Positive (2)<br>@PositiveOrZero (2)||양수인지 검사한다.<br>OrZero가 붙은 것은<br>0 또는 양수인지 검사한다.<br>null은 유효하다고 판단한다.|BigDecimal<br>BigInteger<br>byte, short, int, long 및<br>관련 래퍼 타입|
|@Negative (2)<br>@NegativeOrZero (2)||음수인지 검사한다.<br>OrZero가 붙은 것은<br>0 또는 음수인지 검사한다.<br>null은 유효하다고 판단한다.|BigDecimal<br>BigIntegerbyte,<br> short, int, long 및<br>관련 래퍼 타입|
|@Email (2)||이메일 주소가 유효한지 검사한다.<br>null은 유효하다고 판단한다.|CharSequence| 
|@Future (2)<br>@FutureOrPresent (2)||해당 시간이 미래 시간인지 검사한다.<br>OrPresent가 붙은 것은 현재 또는 미래 시간인지 검사한다.<br>null은 유효하다고 판단한다.|시간 관련 타입|
|@Past (2)<br>@PastOrPresent (2)||해당 시간이 과거 시간인지 검사한다.<br>OrPresent가 붙은 것은 현재 또는 과거 시간인지 검사한다.<br>null은 유효하다고 판단한다.|시간 관련 타입|
  
   
**참고**
```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```
스프링 부트에서는 위와 같이 의존성을 추가해주면 java 표준 Bean Validation 을 사용할 수 있다.   
     
# 커스텀 Validator     
`org.springframework.validation.Validator` 인터페이스는 아래와 같은 형태로 구성되어있다.   
   
```java
package org.springframework.validation;

public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```
이를 활용하여 `커스텀 Validator` 를 만들 수 있다.         

* **boolean supports(Class clazz) :** 어떤 타입의 객체를 검증할 때 사용할 것인지 결정한다.   
* **void validate(Object obj, Errors e) :** 실제 검증 로직을 이 안에서 구현한다.     
    * 검증 로직을 구현할 때 ValidationUtils 사용하면 매우 편리 하다.     
    * 물론, ValidationUtils 말고도 구현할 수 있는 방법은 여러가지이다.     

```java
 public class UserLoginValidator implements Validator {

    private static final int MINIMUM_PASSWORD_LENGTH = 6;

    public boolean supports(Class clazz) {
       return UserLogin.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
       ValidationUtils.rejectIfEmptyOrWhitespace(errors, "userName", "field.required");
       ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "field.required");
       UserLogin login = (UserLogin) target;
       if (login.getPassword() != null
             && login.getPassword().trim().length() < MINIMUM_PASSWORD_LENGTH) {
          errors.rejectValue("password", "field.min.length",
                new Object[]{Integer.valueOf(MINIMUM_PASSWORD_LENGTH)},
                "The password must be at least [" + MINIMUM_PASSWORD_LENGTH + "] characters in length.");
       }
    }
 }
```
* `userName`필드가 null이거나 비어있다면 errors에  `field.required` 메시지를 넣는다.   
* `password`필드가 null이거나 비어있다면 errors에  `field.required` 메시지를 넣는다.    
  

스프링 부트 2.0.5 이상 버전을 사용할 때
● LocalValidatorFactoryBean 빈으로 자동 등록
● JSR-380(Bean Validation 2.0.1) 구현체로 hibernate-validator 사용.
● https://beanvalidation.org/
   
# 참고 
[자바 캔](https://javacan.tistory.com/entry/Bean-Validation-2-Spring-5-valiidatiion)
