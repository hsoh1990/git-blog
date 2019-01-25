---
title: Spring Boot 정리(01)
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

[Spring Boot](http://spring.io/projects/spring-boot)를 이용한 REST API 서버를 만들기를 정리하려고 합니다. 처음 Spring을 접하고 SpringBoot를 처음 사용했을 때 간단히 따라할 강의나 자료가 부족했고 러닝커브도 심해서 당장 먼가 팍팍 진행된다는 느낌을 못받아서 힘들었던거 같습니다.

<!--more-->  

 또한 구글링을 하면 node, python등과 같은 언어로 간단히 API서버를  만드는 방법을 많이 볼 수 있는데 그런 이유때문에 처음 웹개발을 Java가 아닌 다른언어로 시작하는 경우가 많이 증가한거 같습니다. 물론 저도 express가 처음 접한 웹프레임워크입니다. 하지만 아직 국내 많은 기업들은 Spring을 사용하고 있습니다.    

그래서 공부 혹은 개발을 진행하면서 적용한 부분들 중심으로 정리를 하려하며, 내용은 다음과 같습니다.

1. Spring Boot 프로젝트 생성
2. 의존성 추가 및 환경설정
3. 도메인(account) CRUD 및 테스트
4. 데이터 베이스 연동
5. Swagger 연동 및 사용법

 

## Spring Boot 프로젝트 생성

### 개발 환경

intellij

spring boot

gradle

JUnit

### Intellij를 이용한 Spring Boot 프로젝트 생성

Intellij를 이용하여 spring프로젝트를 생성합니다.

Intellij에서는 `New Project`  -> `pring Initializr` 를 선택하면 Spring Boot 프로젝트를 생성할 수 있습니다. 



![springboot01](https://user-images.githubusercontent.com/33083822/45309349-7dfbb000-b55e-11e8-9947-9308ff3b1d61.png)

다음으로 넘어가면 `Project Metadata`를 설정할 수 있습니다. 적당한 이름을 기입한 후 type을 `Gradle Project`로 변경합니다.

![springboot02](https://user-images.githubusercontent.com/33083822/45309350-7e944680-b55e-11e8-9e4a-5f9aec5273de.png)

다음으로 넘어가면 Dependencies와 Spring Boot 버전을 선택할 수 있습니다. Spring Boot 버전은 1.5.15.버전을 사용할 예정이며,  테스트를 위해 `spring-boot-starter-web`만 추가하고 나머지 Dependecies는 진행 하면서 추가합니다. 

설정이 완료되면 프로젝트가 생성되고 gradle이 자동으로 빌드됩니다.

![springboot03](https://user-images.githubusercontent.com/33083822/45309351-7e944680-b55e-11e8-8be5-48600377478b.png)



## Hello World! 

간단히 동작을 확인하기 위해 `ForBlogController.java` 파일을 생성합니다. 생성은 com.ex.forblog에서 오른쪽 버튼 클릭 후 `new` -> `Java Class`  또는 `command + n` 을 통해 생성합니다. 생성된 파일에  다음과 같이 같단한 API를 만들어 보죠.

```java
package com.ex.forblog;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ForBlogController {

    @GetMapping(value = "/")
    String hello(){
        return "Hello World!";
    }
}
```

build.gradle파일은 다음과 같습니다.

```yaml
dependencies {
	compile('org.springframework.boot:spring-boot-starter')
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

다음 서버를 구동시킨 후 localhost:8080으로 접속해 보죠.

![springboot04](https://user-images.githubusercontent.com/33083822/45309352-7f2cdd00-b55e-11e8-9317-7b0c69c1c444.png)



## git, github연동 

git은 현재 버전관리 시스템으로 많이 활용되고 있으며, 많은 기업들이 github, gitlab등를 통해 협업하고 있다. git에 대한 자세한 내용은 인터넷이 더 자세히 나와있기 때문에 생략합니다.

프로젝트 생성시 만들어진 `.gitignore`파일을 새로만든 github repository에 push한 다음 나머지 코드들도 push하겠습니다. 먼저 github에 새로운 repository를 만듭니다. 

![springboot05](https://user-images.githubusercontent.com/33083822/45309353-7f2cdd00-b55e-11e8-85f4-4f52103d0bc3.png)

![springboot06](https://user-images.githubusercontent.com/33083822/45309356-7f2cdd00-b55e-11e8-9fcf-902c2e793179.png)

다시 Intellij로 돌아와 터미널을 열어 git프로젝트를 만든 후 github repository를 추가합니다. 

```bash
$ git init
$ git remote add origin git@github.com:hsoh1990/spring-for-blog.git
```

다음으로 Intellij를 통해서 `add` -> `commit` -> `push` 를 진행하면 다음과 같습니다.

![springboot07](https://user-images.githubusercontent.com/33083822/45309357-7fc57380-b55e-11e8-990f-e534154659ab.png)

![springboot08](https://user-images.githubusercontent.com/33083822/45309358-7fc57380-b55e-11e8-8940-5fe97737f336.png)

![springboot09](https://user-images.githubusercontent.com/33083822/45309359-7fc57380-b55e-11e8-88da-93caa76ee52e.png)

![springboot10](https://user-images.githubusercontent.com/33083822/45309361-805e0a00-b55e-11e8-86d2-f8c36834b1d3.png)

![springboot11](https://user-images.githubusercontent.com/33083822/45309362-805e0a00-b55e-11e8-966f-8a49b77b2d10.png)



이것으로 spring boot 프로젝트 생성 및 기본 설정에 대해서 알아봤고 다음에는 사용할 모듈을 추가하고 그에 맞는 설정들을 진행하겠습다. 

감사합니다.