---
title: Spring 핵심 기술 (AOP)
date: 2019-02-25 14:30:17
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- AOP
---

AOP 는 Aspect-oriendted Programming의 약자로 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법을 뜻하며, OOP를 보완하는 수단으로 사용된다.

<!--more-->  

## AOP 개념

### AOP 주요 개념

- Aspect 
  - 관점 지향적으로 모듈화된 모듈
  - Advice와 Pointcut을 담고 있음
- Target
  - Advice가 적용되어지는 대상(class)
- Advice 
  -  모듈에서 해야할 일을 정의
- Pointcut
  - 어떤 부분에 적용해야 하는지에 대한 정보
- Join point
  - Target에 Advice가 실행되는 여러가지 합류 지점
  - 생성자 호출직전, 생성자 호출이후, 필드에 접근하기전, 필드에서 값을 가져갔을 때 등
- AOP 구현체
  - https://en.wikipedia.org/wiki/Aspect-oriented_programming
  - 자바
    - AspectJ
    - 스프링 AOP



### AOP 적용 방법

- 컴파일 
  - 자바 파일을 클래스 파일로 만들때 조작된 바이트 코드를 생성하여 적용
- 로드 타임
  - JVM이 클래스를 로딩하는 시점에 추가하여 로딩(로드 타임 위빙)
- 런타임
  - JVM이 클래스를 로딩한 후 Bean을 생성할 때 해당 클래스의 프록시빈을 생성하여 적용



## 스프링 AOP: 프록시 기반 AOP

### 스프링 AOP 특징

- 프록시 기반의 AOP 구현체
- 스프링 빈에만 AOP를 적용 가능
- 모든 AOP 기능을 제공하는 것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는 것이 목적.



### 프록시 패턴 AOP

기존 코드 변경 없이 접근 제어 또는 부가 기능 추가하기 위해 프록시 패턴을 사용하여 AOP 구현



```java
public interface EventService {
    void createEvent();
}
```

```java
@Service
public class SimpleEventService implements EventService{
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Create an event");
    }
}
```

```java
@Primary //@Primary를 통해 기본 Service로 사용
@Service
public class ProxySimpleEventService implements EventService {

    @Autowired
    SimpleEventService simpleEventService;

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }
}
```

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
    }
}

```

- 매번 프록시 클래스를 정의 해야하고 객체의 관계도 복잡한 문제 발생




## 스프링 AOP: @AOP

스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용하여 여러 복잡한 문제 해결

- 동적 프록시: 동적으로 프록시 객체 생성하는 방법
  - 자바가 제공하는 방법은 인터페이스 기반 프록시 생성.
  - CGlib은 클래스 기반 프록시도 지원.
- 스프링 IoC: 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록
  - 클라이언트 코드 변경 없음.
  -  AbstractAutoProxyCreator implements BeanPostProcessor

### 애노테이션 기반의 스프링 @AOP

- `spring-boot-starter-aop` 의존성 추가하여 사용 가능

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

- 애스팩트 정의
  - @Aspect 모듈 정의
  - 빈으로 등록을 위해 (컴포넌트 스캔을 사용한다면) @Component 추가
- 포인트컷 정의
  -  @Pointcut(표현식)
  - 주요 표현식
    - execution
    - @annotation
    - bean
  -  포인트컷 조합 가능 - &&, ||, !
- 어드바이스 정의
  - @Before 
  - @AfterReturning
  - @AfterThrowing
  - @Around

```java
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerfLogging {
}
```

```java
@Component
@Aspect
public class perfAspect {

    @Around("@annotation(PerfLogging)") //@PerfLogging 가 붙은 클래스에 적용
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }

    @Before("bean(simpleEventService)") //simpleEventService로 등록된 빈이 실행되기전 호출
    public void hello(){
        System.out.println("Hello");
    }
}
```

```java
@Service
public class SimpleEventService implements EventService{

    @PerfLogging
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Create an event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.deleteEvent();
    }
}
```

```bash
출력:
Hello
Create an event
1003

Hello
Delete an event
```



## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)