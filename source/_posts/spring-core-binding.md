---
title: Spring 핵심 기술(데이터 바인딩)
date: 2019-01-25 13:54:59
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- data binding
- DataBinder
---

데이터바이딩이란 어떤 프로퍼티의 값을 타겟객체에 설정하는 것을 뜻한다. Spring 사용자 관점에서 보면 사용자가 입력한 값을 애플리케이션 도메인 객체에 동적으로 할당하는 기능다.  Spring에서는 사용자가 입력한 값은 문자열이고도메인 객체에 맞는 자료형으로 변경 필요하기 때문에 추상화 되었다.

<!--more-->  

## 데이터 바인딩 추상화(PropertyEditor)

### 구현체

- org.springframework.validation.DataBinder
- java.beans.PropertyEditor

### PropertyEditor 사용

```java
public class EventEditor extends PropertyEditorSupport {

    @Override
    public String getAsText() {
        Event event = (Event)getValue();
        return event.id.toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
    }
}
```

- 스프링 3.0 이전까지 DataBinder가 변환 작업 사용하던 인터페이스
- 쓰레드-세이프 하지 않음 -> controller 단에 등록하여 사용

``` java
@RestController
public class EventController {

    @InitBinder
    public void init(WebDataBinder webDataBinder){
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event/{event}")
    public String getEvent(@PathVariable Event event){
        System.out.println(event);
        return event.getId().toString();
    }
}
```

- Object와 String 간의 변환만 할 수 있어, 사용 범위가 제한적 임.



## 데이터 바인딩 추상화(Converter, Formatter)

### Converter

-  S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기.
-  상태 정보 없음 (Stateless == 쓰레드세이프)

```java
public class EventConverter {

    public static class StringToEventConverter implements Converter<String, Event> {
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConverter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
    }
}
```

-  ConverterRegistry 에 등록해서 사용

```java
@Configuration
public class WebConfig implements WebMvcConfigurer{
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new EventConverter.StringToEventConverter());
    }
}
```

### Formatter

- PropertyEditor 대체제
- Object와 String 간의 변환을 담당
- 문자열을 Locale에 따라 다국화하는 기능도 제공

```java
public class EventFormatter implements Formatter<Event> {
    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(text));
    }

    @Override
    public String print(Event object, Locale locale) {
        return object.getId().toString();
    }
}
```

- FormatterRegistry 에 등록해서 사용

```java
@Configuration
public class WebConfig implements WebMvcConfigurer{
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new EventFormatter());
    }
}
```



### Converter, Formatter Bean 등록방법

- @Component 를 이용하여 등록

```java
public class EventConverter {
    @Component
    public static class StringToEventConverter implements Converter<String, Event>{
        @Override
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }
}
```

```java
@Component
public class EventFormatter implements Formatter<Event> {
    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(text));

    }
}

```



### ConversionService

- 실제 변환 작업은 이 인터페이스를 통해서 쓰레드-세이프하게 사용 가능
- 스프링 MVC , 빈 (value) 설정, SpEL에서 사용
- DefaultFormattingConversionService
  - FormatterRegistry
  - ConversionService
  - 여러 기본 컨버터, 포매터 자동 등록
- 스프링 부트
  - 웹 애플리케이션인 경우에 DefaultFormattingConversionSerivce를 상속하여 만든
    WebConversionService 를 빈으로 등록
  - Formatter와 Converter 빈을 찾아 자동 등록
- 등록된 ConversionService들 조회

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    ConversionService conversionService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(conversionService);
    }
}

```

```bash
ConversionService converters =
	@org.springframework.format.annotation.DateTimeFormat java.lang.Long -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@3b95a6db,@org.springframework.format.annotation.NumberFormat java.lang.Long -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@309cedb6
	@org.springframework.format.annotation.DateTimeFormat java.time.LocalDate -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@36b9cb99,java.time.LocalDate -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@4130955c
	@org.springframework.format.annotation.DateTimeFormat java.time.LocalDateTime -> 
.....
```



## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)