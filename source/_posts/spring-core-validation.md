---
title: Spring 핵심 기술 (validation 추상화)
date: 2019-01-20 20:34:52
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- validation
---

애플리케이션에서 사용하는 객체 검증용 인터페이스로 org.springframework.validation.Validator 로 추상화하였다. 웹이, 서비스, 데이터 어떤 계층과도 관계없이 사용할 수 있다. 구현체 중 하나로, JSR-303(Bean Validation 1.0)과 JSR-349(Bean Validation 1.1)을 지원(LocalValidatorFactoryBean)하며,  DataBinder에 들어가 바인딩 할 때 같이 사용 가능하다.

<!--more-->  

## 인터페이스 

-  boolean supports(Class clazz): 어떤 타입의 객체를 검증할 때 사용할 것인지 결정
-  void validate(Object obj, Errors e): 실제 검증 로직을 이 안에서 구현 
  - 구현할 때 ValidationUtils 사용하며 편리 함. 

```java
public class EventValidator implements Validator{
    @Override
    public boolean supports(Class<?> clazz) {
        return Event.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "notempty", "Empty title is not allowed.");
        Event event = (Event) target;
        if(event.getTitle() ==null){
            errors.reject("title", "Empty title is not allowed.");
        }
    }
}
```

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Event event = new Event();
        EventValidator eventValidator = new EventValidator();
        Errors errors = new BeanPropertyBindingResult(event, "event");

        eventValidator.validate(event, errors);
        System.out.println(errors.hasErrors());
        errors.getAllErrors().forEach(e ->{
            System.out.println(" ===== error code ====");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
```



## 스프링 부트 2.0.5 이상 버전을 사용할 때 

- LocalValidatorFactoryBean 빈으로 자동 등록 

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Qualifier("defaultValidator")
    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(validator.getClass());

        Event event = new Event();
        event.setLimit(-1);
        event.setEmail("hsoh");

        Errors errors = new BeanPropertyBindingResult(event, "event");

        validator.validate(event, errors);
        System.out.println(errors.hasErrors());
        errors.getAllErrors().forEach(e ->{
            System.out.println(" ===== error code ====");
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```



- JSR-380(Bean Validation 2.0.1) 구현체로 hibernate-validator 사용. 
- https://beanvalidation.org/ 