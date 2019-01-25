---
title: Spring Boot 정리(04)
date: 2018-12-08 02:41:18
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

[Spring Boot](http://spring.io/projects/spring-boot)를 이용한 RESTful API 개발 4번째 입니다. 간략하게 JPA 사용법과 도메인 정의에 대해서 알아보며,  [Spring boot를 이용한 REST API 개발(03)](https://hsoh1990.github.io/2018/08/30/spring-boot-start-03/)에서 사용한 구현한 Repository를 변경하겠습니다.
<!--more-->  

## Gradle 설정

먼저  dependenccy를 설정하기 위해 build.gradle을 수정하겠습니다. [Spring boot를 이용한 REST API 개발(03)](https://hsoh1990.github.io/2018/08/30/spring-boot-start-03/)에서 설정까지 따라 하셨다면 build.gradle파일에서 `dependencies` 에 db에 관련된 dependency만 추가 하시면 됩니다.

```groovy
dependencies {
    ...
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    runtime('org.postgresql:postgresql')
    ...
}

```



## application.yml 설정

PostgreSQL을 사용하며, 다운로드 및 설치는 [여기](https://www.postgresql.org/download/)에서 각 OS에 맞는 버전으로 다운받으신 후 설치하시면 됩니다. 

다음 `resources/application.yml`을 수정 합니다.

```yaml
server:
  port: 9090
spring:
  datasource:
    platform: postgres
    url: jdbc:postgresql://localhost:5432/spring_start
    username: hsoh
    password: ****
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create
    generate-ddl: true
```

table을 자동으로 생성하기 위해 hibernate를 사용하였습니다. jpa:show-sql: true로 설정하여 서버 구동시 쿼리를 확인합니다. 또한  jpa:hibernate:ddl-auto: create로 하게되면 서버를 실행할 떄 혹은 테스트를 진행할때 기존에 있던 테이블을 지우고 다시 만들게 됩니다.

먼저 코드를 변경했다고 치고 서버를 구동시켜 보면 다음과 같은 로그를 확인할 수 있습니다.

```bash
Hibernate: drop table if exists account cascade
Hibernate: create table account (id  serial not null, email varchar(50) not null, name varchar(15) not null, password varchar(255) not null, primary key (id))
Hibernate: alter table account add constraint UK_q0uja26qgu1atulenwup9rxyr unique (email)
Hibernate: alter table account add constraint UK_bb9lrmwswqvhcy1y430ki00ir unique (name)
```

그러니 실제로 운영중인 DB를 사용할 경우에는 조심하여야 합니다. (한번 실수가...) 



## Account 도메인 수정

이제 설정을 완료했으니 도메인을 정의해보죠.

```java
package com.ex.forblog.account;

@Data
@NoArgsConstructor
@Entity(name = "account")
class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @NotNull
    @Size(min = 4, max = 15)
    @Column(unique = true, nullable = false)
    private String name;

    @NotNull
    private String password;

    @NotNull
    @Size(max = 50)
    @Column(unique = true, nullable = false)
    private String email;
}
```

처음 보이는 @Entity 어노테이션의 경우 엔티티 클래스임을 지정하여 JPA가 엔티티로써 관리하며, 테이블과 매핑합니다. 보통 @Table 어노테이션을 이용하여 엔티티와 매핑할 DB 테이블을 지정하지만 생략시 엔티티 클래스 이름 혹은 (name = '....')에 명시된 이름의 테이블로 매핑됩니다.

다음 @Id는 기본키를 매핑시켜주면  @GeneratedValue(strategy = GenerationType.IDENTITY)기본키 생성을 DB에 위임하여 사용하도록 설정됩니다.

 @Column은 컬럼의 이름을 이용하여 지정된 필드나 속성을 테이블의 칼럼에 매칭한다. 역시 생략하면 속겅과 같은 이름의 칼럼으로 매핑됩니다.  자주 사용하는 속성만 정의해 보겠습니다. 

`name`은 매핑할 table 컬럼 이름, 기본은 객체의 필드 이름을 사용합니다. `nullable` 은 false로 설정하면 DDL생성시에 “NOT NULL” 제약조건을 추가해 줍니다. `unique`는 true로 설정하면 uniqueConstraints와 같은 동작을 합니다.

추가적인 속성들은 학습하시는걸 추천드립니다.



## AccountRepository 수정

다음은 JpaRepository를 extends하여  Repository를 수정합니다.

```java
package com.ex.forblog.account;

@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
}

```

음.. 먼가 다 사라졌습니다. 지금까지 구현해야했던 sava(), findById(), delete() 메소드들은 JpaRepository에 이미 정의되어 있습니다.  코드를 확인하고 싶으시면 `SimpleJpaRepository` 를 확인하시기 바랍니다.



## TEST

수정을 했으니 테스트를 확인해보죠. 처음 구현할 때 자신있게 될꺼라고 생각했지만 실제로 테스트를 해보면 먼가 문제가 있는듯 보입니다.

```bash
Detail: Key (email)=(hsoh@gmail.com) already exists.
```

이미 있다고 하는데 지금까지는 `unique = true`를 하지않았기 떄문입니다. 그럼 실제 테스트에서 처리하기 위해 `@Transactional`를 붙혀줍니다.

```java
package com.ex.forblog.account;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class AccountControllerTest {
	...
}
```

다시 테스트를 해보면 성공적으로 마쳤을꺼라고 예상됩니다.

## 마치며

원래는 자동 문서화도구인 Swagger도 연동해보려고 했지만 springdoc를 사용하는게 기존 controller코드를 로직에만 집중할 수 있을꺼 같아서 학습을 하려고 합니다. 정리가 되면 spring doc 사용법과 OAuth2.0 사용법을 정리하겠습니다. 

## Contributors

- 오형석[(ohs4123@gmail.com)](ohs4123@gmail.com)