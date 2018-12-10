---
title: Spring boot를 이용한 RESTful API 개발(02)
date: 2018-08-30 22:48:48
categories:
- Spring Boot 정리
tags:
- java
- spring boot
- PostgreSQL
- JPA
- JUnit
- lombok
---

[Spring Boot](http://spring.io/projects/spring-boot)를 이용한 REST API 서버를 만들기위해 의존성을 추가하고 그게 맞는 환경설정을 방법을 알아보겠습니다. [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/1.5.4.RELEASE/reference/htmlsingle/#getting-started)를 참고하여 Spring Boot에서 제공하는 starter들과 Gradle을 사용하는 법을 정리하겠습니다.
<!--more-->  



### Spring Boot  Gradle 사용법

Spring Boot에서 Gradle 사용법은 [13.3](https://docs.spring.io/spring-boot/docs/1.5.4.RELEASE/reference/htmlsingle/#using-boot-gradle)에 정의 되어 있습니다. 

문서에서 설명하듯 Maven에서는 부모 의존성를 받아 spring boot에서 사용하는 추가적인 의존성의 버전을 신경쓰지 않고 개발할 수 있습니다. 

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
</parent>
```

Gradle에서는 `dependencies`섹션 에서 'starter'을 직접 가져올 수 있습니다 . Maven과는 달리 Gradle에서는 일부 구성을 공유하기 위해 가져올 수퍼 부모는 없습니다.

```groovy
repositories {
    jcenter()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.5.4.RELEASE")
}
```

[`spring-boot-gradle-plugin`](https://docs.spring.io/spring-boot/docs/1.5.4.RELEASE/reference/htmlsingle/#build-tool-plugins-gradle-plugin)도 사용할 수 있으며 실행 가능한 jar를 만들고 소스에서 프로젝트를 실행하는 작업을 제공합니다. 또한 다른 기능들 중에서도 스프링 부트에 의해 관리되는 모든 종속성의 버전 번호를 생략 할 수있는 종속성 관리를 제공합니다.

```groovy
plugins {
    id 'org.springframework.boot' version '1.5.4.RELEASE'
    id 'java'
}


repositories {
    jcenter()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```



### Spring Boot application starters

Spring Boot에는 다양한 [starter](https://docs.spring.io/spring-boot/docs/1.5.4.RELEASE/reference/htmlsingle/#using-boot-starter)들이 존재 합니다. 예를 들어 Spring Boot의 코어 기능을 가지는 `spring-boot-starter`가 있습니다. 또한 JPA를 사용하기 위한 `spring-boot-starter-data-jpa`또는 Spring Security 의존성을 가지는 `spring-boot-starter-security` 등등 다양한 starter들을 확인할 수 있습니다.

| Name                                  | Description                                                  | Pom                                                          |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `spring-boot-starter`                 | Core starter, including auto-configuration support, logging and YAML | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-data-jpa`        | Starter for using Spring Data JPA with Hibernate             | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-security`        | Starter for using Spring Security                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-social-facebook` | Starter for using Spring Social Facebook                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-social-facebook/pom.xml) |
| `spring-boot-starter-test`            | Starter for testing Spring Boot applications with libraries including JUnit, Hamcrest and Mockito | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`       | Starter for building MVC web applications using Thymeleaf views | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`      | Starter for using Java Bean Validation with Hibernate Validator | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`             | Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default embedded container | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-websocket`       | Starter for building WebSocket applications using Spring Framework’s WebSocket support | [Pom](https://github.com/spring-projects/spring-boot/tree/v1.5.4.RELEASE/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |
| ...                                   | ...                                                          | ...                                                          |

모든 공식 starter는 spring-boot-starter- *, *라는 비슷한 명명 패턴을 따릅니다. 이 명명 규칙은 starter를 찾아야 할 때 도움을 주기위한 것입니다. 많은 IDE의 Maven 통합을 통해 의존성을 이름으로 검색 할 수 있습니다. 개인적인 프로젝트에서는 spring-boot-starter- *, *로 시작해서는 안됩니다. 

### Gradle Practice

[Spring boot를 이용한 REST API 개발(01)](https://hsoh1990.github.io/2018/08/30/spring-boot-start-01/)에서 프로젝트를 생성했습니다. Intellij에서는 프로젝트를 생성하면 다음과 같이 기본적인 구조가 생성되며 API테스트를 위해 `compile('org.springframework.boot:spring-boot-starter-web')`를 추가 했습니다.

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.15.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

group = 'com.ex'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter')
	compile('org.springframework.boot:spring-boot-starter-web')

	testCompile('org.springframework.boot:spring-boot-starter-test')
}

```



이번에는 기본적으로 사용할 starter및 의존성을 추가해 보겠습니다. 추가할 의존성은 `spring-boot-starter-security` `org.projectlombok:lombok`입니다.

```groovy
dependencies {
   compile('org.springframework.boot:spring-boot-starter-security')
   compile('org.springframework.boot:spring-boot-starter-web')

   compileOnly('org.projectlombok:lombok')

   testCompile('org.springframework.boot:spring-boot-starter-test')
   testCompile('org.springframework.security:spring-security-test')
}
```

다음 Gradle이 빌드가 끝나면 의존성이 정상적으로 추가된것을 확인할 수 있습니다.

![springboot01](https://user-images.githubusercontent.com/33083822/47100747-2a723580-d273-11e8-8d18-42bf26a3206c.png)

다음 Intellij에서 lombok을 사용하기 위해서는 `Build, Execution, Deployment` ->  `Annotation Processors` -> `Enable annotation processing`을 체크해 줍니다.

그럼 lombok을 테스트 하기위해 간단한 클래스를 만들고 ForBlogController에서 사용해 보겠습니다.

```java
package com.ex.forblog;

import lombok.Data;

@Data
public class TestLombok {
    private String str;
}
```

```java
@RestController
public class ForBlogController {

    @GetMapping(value = "/")
    String hello(){
        TestLombok testLombok = new TestLombok();
        testLombok.setStr("Hello World!");
        return testLombok.getStr();
    }
}
```

여기까지 진행한 후 Run하면 정상적으로 실행이 되지만 `spring-boot-starter-security`에 대한 설정이 없어 다음과 같은 에러 페이지가 나옵니다.

```web-idl
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Thu Oct 18 01:16:16 KST 2018
There was an unexpected error (type=Unauthorized, status=401).
Bad credentials
```

현재는 보안은 그냥 넘어가도록 설정할 예정이며 추후에 OAuth2.0을 이용해 보겠습니다. `spring-boot-starter-security`에 대한 설정은 JAVA Config를 통해 진행 합니다.

```java
package com.ex.forblog.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(final HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("*").permitAll();
    }
}
```

마지막으로 Spring Boot를 실행하면 기본 포트는 8080으로 설정되어 있습니다. 기본 포트를 변경하는 방법은 application.properties 혹은 application.yml에서 설정할 수 있고 지금 예제에서는 application.yml를 사용하여 9090 포트를 이용하겠습니다. 경로는 `src/main/resources`에 생성합니다.

```yml
server:
  port: 9090
```

이상으로 Gradle 사용 및 환경설정에 대해 알아봤습니다.

감사합니다.