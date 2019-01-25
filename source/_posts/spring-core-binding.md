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



## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)