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

Environment는 프로파일과 프로퍼티를 다루는 인터페이스로 테스트 환경, 프로덕션 환경등 각각에 환경에 따라 다른 빈들을 써야하는 경우 혹은 특정한 빈을 써야하는 경우 사용

<!--more-->  

## 프로파일

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

## 프로퍼티

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

## MessageSource

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



## ApplicationEventPublisher

이벤트 프로그래밍에 필요한 인터페이스를 제공(옵저버 패턴의 구현체)

- ApplicationEventPublisher의 메소드 publishEvent(ApplicationEvent event)로 이벤트 발생
- 4.2 이전에는  ApplicationEvent를 상속받아서 이벤트 구현
- 이벤트를 발생시키고 EventListener를 등록하여 이벤트 처리

```java
public class AppEvent {

    private int data;

    private Object source;

    public AppEvent(Object source, int data) {
        this.data = data;
        this.source = source;
    }

    public Object getSource() {
        return source;
    }

    public int getData() {
        return data;
    }
}

```

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    ApplicationEventPublisher publisher;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        publisher.publishEvent(new AppEvent(this, 100));
    }
}
```

```java
@Component
public class AppEventHandler {

    @EventListener
    @Async
    public void handle(AppEvent event){
        System.out.println(Thread.currentThread().toString());
        System.out.println("AppEventHandler = " +event.getData());
    }
}

```

- 순서지정은 @Order로 지정
- 비동기적 실행은 @Async 사용

## ResourceLoader

리소스를 읽어오는 기능을 제공하는 인터페이스

- ResourceLoader의 getResource(java.lang.String location)로 리소스 조회
- 다양한 방법으로 조회가능
  - 파일 시스템에서 읽기
  - 클래스 패스에서 읽기
  - URL로 읽기
  - 상대/절대 경로로 읽기

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:text.txt");
        System.out.println(resource.exists());
        System.out.println(resource.getDescription());
        Files.lines(Paths.get(resource.getURI())).forEach(System.out::println);
    }
}
```

## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)