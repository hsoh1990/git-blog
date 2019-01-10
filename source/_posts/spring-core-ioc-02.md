---
title: Spring 핵심 기술 (IoC, Bean - 02)
date: 2019-01-6 11:30:00
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- IoC
- Bean

---

### Environment

프로파일과 프로퍼티를 다루는 인터페이스로 테스트 환경, 프로덕션 환경등 각각에 환경에 따라 다른 빈들을 써야하는 경우 혹은 특정한 빈을 써야하는 경우 사용

<!--more-->  

##### 프로파일

- 프로파일은 빈들의 그룹
- ApplicatioContext의 getEnvironment()를 통해 호출
- 활성화할 프로파일 확인 및 설정

```java
@Component
public class AppRunner implements ApplicationRunner{
    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
    }
}
```

- 클래스 정의 @Configuration @Profile("test") 를 통해 설정

- 메소드 정의 @Bean  @Profile("test")를 통해 설정

  ```java
  @Configuration
  @Profile("test")
  public class TestConfiguration {
      ...
  }
  ```

- -Dspring.profiles.active="test,A,B..."으로 설정가능

##### 프로퍼티

- 다양한 방법으로 정의할 수 있는 설정값
- -Dapp.name=spring5
- properties파일 사용

```properties
app.about=spring
```

- java 사용방법

```java
@Component
public class AppRunner implements ApplicationRunner{
    @Autowired
    ApplicationContext ctx;

    @Value("${app.about}")
    String appAbout;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment environment = ctx.getEnvironment();
        System.out.println(environment.getProperty("app.name"));
        System.out.println(this.appAbout);

    }
}
```

- 우선순위
  - ServletConfig 매개변수
  - ServletContext 매개변수
  - JNDI (java:comp/env/)
  - JVM 시스템 프로퍼티 (-Dkey="value")
  - JVM 시스템 환경 변수(운영 체제 환경 변수)

### MessageSource

i18n 기능을 제고하는 인터페이스로 스프링 부트를 사용하면 별다른 설정 없이 messages.properties 사용가능

- ApplicatioContext의 getMessageSource()를 통해 호출
- messages.properties, messages_ko_kr.properties...

```properties
# messages.properties
greeting=Hello, {0}
#messages.properties
greeting=안녕, {0}
```

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getMessage("greeting", new String[]{"hsoh"}, Locale.KOREA));
        System.out.println(messageSource.getMessage("greeting", new String[]{"hsoh"}, Locale.getDefault()));
    }
}
```



### ApplicationEventPublisher



### ResourceLoader

