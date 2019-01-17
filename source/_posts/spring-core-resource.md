---
title: Spring 핵심 기술 (Resource 추상화)
date: 2019-01-16 21:36:32
categories:
- Spring 핵심 기술
tags:
- java
- spring boot
- IoC
- Bean
---

java.net.URL을 org.springframework.core.io.Resource로 감싸 추상화 한 것으로 클래스패스 기준으로 리소스 읽어오는 기능 부재, ServletContext를 기준으로 상대 경로로 읽어오는 기능 부재     , 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부족하여 추상화 하였다.

<!--more-->  

#### 인터페이스

- Resource extends InputStreamSource
- getInputStream() 
- exitst() : 리소스가 존재하는지 확인
- isReadable() : 리소스를 읽을 수 있는지 확인
- isFile() : 리소스가 파일인지 확인
- isOpen() : 리소스가 열려있는지 확인
- getDescription() : 전체 경로 포함한 파일 이름 또는 실제 URL 
- ...

#### 구현체 

- UrlResource: java.net.URL 참고, 기본으로 지원하는 프로토콜 http, https, ftp, file, jar. 
- ClassPathResource: 지원하는 접두어 classpath: 
- FileSystemResource 
- ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스 찾는다. 
-  ... 

#### 리소스 읽어오기 

- Resource의 타입은 locaion 문자열과 ApplicationContext의 타입에 따라 결정 된다.
  - ClassPathXmlApplicationContext -> ClassPathResource
  - FileSystemXmlApplicationContext -> FileSystemResource
  - WebApplicationContext -> ServletContextResource 
- ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+ classpath:)중 하나를 사용할 수 있다. 
  - classpath:me/whiteship/config.xml -> ClassPathResource 
  - file:///some/resource/path/config.xml -> FileSystemResource 

```java
@Component
public class AppRunner implements ApplicationRunner{

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(resourceLoader.getClass());

        Resource resource = resourceLoader.getResource("classpath:text.txt");
        System.out.println(resource.getClass());

        System.out.println(resource.exists());
        System.out.println(resource.getDescription());
        Files.lines(Paths.get(resource.getURI())).forEach(System.out::println);
    }
}
```

