---
title: Spring 핵심 기술 (IoC, Bean - 01)
date: 2019-01-05 11:30:00
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- IoC
- Bean
---

IoC 는 Inversion of Control의 약자로 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법으로 DI(Dependency Injection)이라고도 한다. 다시 말하면 객체 생명주기를 관리하며 DI 패턴을 제공하여 비즈니스 로직에 집중할 수 있도록 한다.

<!--more-->  

## 스프링 IoC 컨테이너

- BeanFactory - 가장 최상위 핵심 인터페이스
- 애플리케이션 컴포넌트의 중앙 저장소
- 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성/제공

스프링 빈(Bean)

- 스프링 IoC 컨테이너가 관리하는 객체
- 싱글톤, 프로토타입 스코프
- 라이프사이클 인터페이스 
- @Bean, @Component, @Service, @Repository 등 어노테이션으로 빈 설정
- application.xml 의 <bean> 테크로 빈 설정



## 빈 설정 방법

### 스프링 설정파일을 통한 빈 등록

- 고전적인 방법으로 application.xml 설정을 통해 빈을 등록
-  <bean> </bean>을 통해 빈으로 등록할 class를 설정
-  <property> </property>를 통해 빈 객체를 주입

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bookService" class="com.ex.forblog.book.BookService">
        <property name="bookRepository" ref="bookRepository"/>
    </bean>

    <bean id="bookRepository" class="com.ex.forblog.book.BookRepository"/>
</beans>
```



### component-scan을 이용한 빈 등록

- 위 방법은 빈을 하나하나 등록해야하는 불편
-  <component-scan> 을 통해 어노테이션을 통해 빈으로 설정한 빈 class들을 찾아 등록

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ex.forblog"/>
</beans>
```



### JAVA config을 이용한 빈 등록

- 최근(꾀 오래전?) 에는 xml파일을 사용하지 않고 JAVA를 사용하여 등록
- @Bean 통해 빈으로 등록
- @Autowire를 통해 빈 객체 주입

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public BookRepository bookRepository(){
        return new BookRepository();
    }

    @Bean
    public BookService bookService(){
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository());
        return bookService;
    }
}
```



### ComponentScan을 이용한 빈 등록

- xml과 마찬가지로 JAVA config 또한 @ComponentScan으로 basePackageClasses로 지정된 클래스 하위에서 어노테이션을 통해 빈으로 설정한 빈 class들을 찾아 등록

```java
@Configuration
@ComponentScan(basePackageClasses = Application.class)
public class ApplicationConfig {

}
```



### SpringBootApplication을 이용한 빈 등록

- 위의 과정들을 spring boot에서는 @SpringBootApplication를 통해 지원
- @SpringBootApplication을 확인해 보면 @ComponentScan를 확인 가능  
- Application 클래스 하위 패키지에서 빈 class들을 찾아 등록한다.

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
 ...
}
```



### @Autowired

IoC에 등록된 빈을 사용하는 방법은 @Autowired 어노테이션을 이용하여 객체의 타입에 해당하는 빈을 찾아 주입받아 사용한다. 

- 기본값은 true로 해당 객체 다입을 못찾으면 어플리케이션 구동 실패

- 같은 타입의 빈이 여러개 일 때 @Primary 통해 주입

```java
public interface BookRepository {
	...
}

@Repository @Primary
public class HsBookRepository implements BookRepository {
    ...
}

@Repository
public class MyBookRepository implements BookRepository{
    ...
}
```

```java
@Service
public class BookService {
    @Autowired
    public BookRepository bookRepository;
    
    ...
}
```



- 같은 타입의 빈이 여러개 일 때 @Qualifier를 통해 주입

- @Qualifier 사용할 경우 설정이 따로 없으면 빈에 등록된 이름은 다음과 같이 camel 표기법을 사용

```java
public interface BookRepository {
	...
}

@Repository
public class HsBookRepository implements BookRepository {
    ...
}

@Repository
public class MyBookRepository implements BookRepository{
    ...
}
```

```java
@Service
public class BookService {
    @Autowired @Qualifier("hsBookRepository")
    public BookRepository bookRepository;

    
    ...
}
```



- ### 여러 빈 주입 가능

```java
@Service
public class BookService {
    @Autowired
    List<BookRepository> bookRepositories;    
    ...
}
```



### @Commponent와 @ComponentScan

- @Commponent는 빈으로 등록할 class
- @Commponent는 @Controller, @Service, @Repository로 구체화 되어 사용
- @ComponentScan는 특정 패키지 안의 클래스들을 스캔하여 빈 등록
- 실제 스캐닝은 ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에
  의해 처리



## 빈 스코프

- 빈은 싱글톤, 프로토타입 스코프를 가짐
- 스코프 기본은 싱글톤, @Scope를 통해 설정 가능

```java
@Component
public class Single {
    ...
}

@Component @Scope("prototype")
public class Proto {
    ...
}

```



- 싱글톤 빈이 프로토타입 빈을 참조하려면 @Scope에 proxyMode 설정

```java
@Component @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class Proto {
    ...
}

@Component
public class Single {
    @Autowired
    Proto proto;

    public Proto getProto() {
        return proto;
    }
}
```

## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)