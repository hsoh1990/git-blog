---
title: Spring Boot 정리(03)
date: 2018-12-07 23:07:31
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

[Spring Boot](http://spring.io/projects/spring-boot)를 이용한 RESTful API 개발 3번째 입니다. 이번에는 요구사항을 정의한 후 Account 도메인을 통해 CRUD 로직을 만들어 보겠습니다.
<!--more-->  

 

## Gradle 설정

먼저  dependenccy를 설정하기 위해 build.gradle을 확인하겠습니다. [Spring boot를 이용한 REST API 개발(02)](https://hsoh1990.github.io/2018/08/30/spring-boot-start-02/)에서 설정을 했다면 다음과 같습니다.

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
	compile('org.springframework.boot:spring-boot-starter-security')
	compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.modelmapper:modelmapper:0.7.5')

    compileOnly('org.projectlombok:lombok')

	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('org.springframework.security:spring-security-test')
}

```



## TEST 코드 작성

spring boot에서 테스트는 `src/test/java/packagename/..`에 `@SpringBootTest`를 사용해 작성합니다.` AccountControllerTest` Class를 사용하여 기본 틀을 작성하면 다음과 같습니다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class AccountControllerTest {
    @Autowired
    private WebApplicationContext webApplicationContext;

    @Autowired
    AccountService accountService;

    private MockMvc mockMvc;

    private AccountDto.AccountRegistDto registerAccountDto;

    private ObjectMapper mapper = new ObjectMapper();

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders
                .webAppContextSetup(webApplicationContext)
                .apply(springSecurity())
                .alwaysDo(print())
                .build();
    }
}
```

현재 만들지 않은 Class도 있지만 그대로 진행합니다. 필요한 Class는 진행하면서 구현하도록 하며, 코드에 대한 설명을 추후에 하나씩 살펴보도록 하고 일딴 개발을 진행해 보겠습니다.  

## Account 도메인 CRUD 명세 정의 

다음 개발에 앞서 Test코드를 작성하기 위한 명세를 정의하겠습니다.

1. 계정 등록
   - 계정은 name, password, email을 입력하여 등록한다
   - 등록이 정상적으로 수행되면 id, name, password, email를 가지는 데이터를 저장한다.
   - 등록이 정상적으로 수행되면 201 상태코드와 password를 제외한 계정정보를 반환한다.
   - 등록 시 name, password, email 중 하나라도 없으면 400에러를 반환한다.
2. 계정 리스트 조회
   - 조회가 정상적으로 수행되면 200 상태코드와  password를 제외한 계정 리스트를 반환한다. 
3. id를 이용하여 계정 조회
   - 특정 id로 조회가 정상적으로 수행되면 200 상태코드와 password를 제외한 계정정보를 반환한다.
   - 특정 id에 해당하는 계정이 없으면 400에러를 반환하며, 조회한 id를 반환한다.
4. id를 이용하여 계정 정보 수정
   - 특정 id로 수정이 정상적으로 수행되면 200 상태코드와 password를 제외한 계정정보를 반환한다.
   - 특정 id에 해당하는 계정이 없으면 400에러를 반환하며, 조회한 id를 반환한다.
5. id를 이용하여 계정 삭제
   - 특정 id로 수정이 정상적으로 수행되면 200 상태코드와 password를 제외한 계정정보를 반환한다.
   - 특정 id에 해당하는 계정이 없으면 400에러를 반환하며, 조회한 id를 반환한다.



## 계정 등록

### 등록 성공

먼저 명세에 정의된 내용을 Test 코드로 옮겨 보겠습니다. 테스트는 given, when, then 순서로 특정 상황(given)에서 특정 API로 요청했습때(when) 원하는 결과가 리턴(then)되는지 확인하는 방식으로 진행합니다.

```java
    /**
     * 계정을 등록한다
     * id:1, name:hsoh, password:password, email:hsoh@gmail.com
     * 성공적으로 등록되면 201 상태코드를 반환한다.
     * 성공적으로 등록되면 등록한 계정정보가 반환된다.
     */
    @Test
    public void registAccount() throws Exception {
        // Given

        // When
        final ResultActions resultActions = mockMvc.perform(
                post("/accounts").contentType(MediaType.APPLICATION_JSON)
                        .content(mapper.writeValueAsString(this.registerAccountDto))
                        .with(csrf()));

        // Then
        resultActions.andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("hsoh"))
                .andExpect(jsonPath("$.email").value("hsoh@gmail.com"));
    }
```

다음과 같이 Test 코드를 작성 했하고 실행해보면 구현되지 않은 내용이기 때문에 에러가 발생됩니다. 이제 다시 코드로 돌아가 구현해 보겠습니다.

먼저 account package를 생성하고 Account 도메인을 정의 합니다. 

```java
package com.ex.forblog.account;

@Data
class Account {
    Account(String name, String password, String email) {
        this.name = name;
        this.password = password;
        this.email = email;
    }

    private int id;

    private String name;

    private String password;

    private String email;
}
```

다음 `/accounts`요청을 처리할 controller, service, repository, dto를 생성합니다.

- AccountController

```java
package com.ex.forblog.account;

@RestController
@RequestMapping("/accounts")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @Autowired
    ModelMapper modelMapper;

    @PostMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity registAccount(@RequestBody final AccountRegistDto accountDto) {
        Account account = accountService.register(accountDto);
        AccountResponseDto accountResponseDto = modelMapper.map(account, AccountResponseDto.class);
        return new ResponseEntity<>(accountResponseDto, HttpStatus.CREATED);
    }
}
```

- AccountService

```java
package com.ex.forblog.account;

@Service
public class AccountService {
    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    ModelMapper modelMapper;

    public Account register(AccountRegistDto accountDto) {
        Account account = modelMapper.map(accountDto, Account.class);
        return accountRepository.save(account);
    }
}

```

- AccountRepository

```java
package com.ex.forblog.account;

@Repository
public class AccountRepository {

    private List<Account> accounts = new ArrayList<Account>();
    private int accountId;


    AccountRepository() {
        accountId = 1;
    }

    public Account save(Account account) {
        if (account.getId() == 0){
            account.setId(accounts.size()+1);
            accounts.add(account);
            accountId += 1;
        } else {
            accounts.set(account.getId()-1, account);
        }

        return account;
    }
}

```

- AccountDto.AccountRegistDto, AccountDto.AccountResponseDto

```java
package com.ex.forblog.account;

@SuppressWarnings("squid:S1118")
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class AccountDto {
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class AccountRegistDto {
        private String name;
        private String password;
        private String email;
    }
    
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class AccountResponseDto {
        private int id;
        private String name;
        private String email;
    }
}

```

다음 다시 테스트를 돌려보면 역시 에러... 이번에는 `ModelMapper`가 없어서 그렇습니다. `@Bean`를 등록해보죠.

- ForBlogApplication

```java

@SpringBootApplication
public class ForBlogApplication {
    ...
        
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

다시 테스트 코드 실해하면 녹색으로 처리된 테스트 코드를 만날 수 있습니다.



### 등록 실패

테스트는 등록에 실패하느 경우도 생각해야합니다. 등록에 실패하는 경우에는 어떤 처리를 해야하는지 어떤 동작을 해야하는지 명세에 정의되어 있고 그대로 동작하는지 확인하기 위해 다음과 같은 Test 코드를 작성 했습니다.

```java
    /**
     * name, password, email 필드중 하나라도 없으면 등록하면 실패한다.
     * 404 상태코드를 반환한다.
     */
    @Test
    public void registAccountBadRequest() throws Exception {
        //Given
        this.registerAccountDto.setName(" ");
        this.registerAccountDto.setPassword("1234");

        //When
        final ResultActions resultActions = mockMvc.perform(
                post("/accounts").contentType(MediaType.APPLICATION_JSON)
                        .content(mapper.writeValueAsString(this.registerAccountDto))
                        .with(csrf()));

        //Then
        resultActions.andExpect(status().isBadRequest());
    }
```

이번에 테스트를 실행시켜보면 역시 에러를 만날 수 있습니다. 잘못된 요청에 대한 처리를 하지 않았기때문이죠.

그럼 코드를 다시 수정해 보겠습니다. 저는 Spring에서 제공하는 ` @Valid`를 사용하여 요청 데이터의 유효성 검사를 진행하겠습니다.

- AccountController

```java
package com.ex.forblog.account;

@RestController
@RequestMapping("/accounts")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @Autowired
    ModelMapper modelMapper;

    @PostMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity registAccount(@RequestBody @Valid final AccountRegistDto accountDto,
                                        final BindingResult result) {
        if (result.hasErrors()) {
            return new ResponseEntity<>(result.getFieldErrors(), HttpStatus.BAD_REQUEST);
        }

        Account account = accountService.register(accountDto);
        AccountResponseDto accountResponseDto = modelMapper.map(account, AccountResponseDto.class);
        return new ResponseEntity<>(accountResponseDto, HttpStatus.CREATED);
    }
}
```

AccountController의 registAccount함수에서 AccountRegistDto앞에 ` @Valid`를 사용하면 AccountRegistDto에서 정의된 유효성 검사를 진행합니다. 그럼 AccountRegistDto로 이동하여 어떤 유효성을 검사하는지 정의해 보죠

- AccountDto.AccountRegistDto, AccountDto.AccountResponseDto

```java
package com.ex.forblog.account;

@SuppressWarnings("squid:S1118")
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class AccountDto {
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class AccountRegistDto {
        @NotBlank
        private String name;
        @NotBlank
        private String password;
        @NotBlank
        private String email;
    }
    
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class AccountResponseDto {
        private int id;
        private String name;
        private String email;
    }
}

```

`@NotBlank`가 생겼습니다. 해당 필드가 비어있으면 에러를 발생시킵니다. 유효성 검사는 @Min, @Max, @NotBlank 등 다양한 어노테이션들이 있으니 확인해 보세요.



이와 같은 방법으로 위에 정의한 요구사항들을 구현해보세요. 구현한 내용을 [여기](https://github.com/hsoh1990/spring-for-blog/tree/spring-start-03)에 있습니다.



## 에러처리 

추가적으로 spring으로 개발하다보면 위와 같이 에러처리를 해야합니다. 이걸 각 함수마다 따로 정의를 하다보면 중복되는 에러처리가 발생합니다. `@ExceptionHandler`를 통해 이런 중복된 코드 줄이겠습니다. 

- AccountController

```java
package com.ex.forblog.account;
@RestController
@RequestMapping("/accounts")
public class AccountController {
	...

    @ExceptionHandler(value = NotFoundException.class)
    public ResponseEntity accountNotfoundException(NotFoundException e) {
        ExceptionDto exceptionDto = new ExceptionDto();
        exceptionDto.setMessage("id가 " + e.getMessage() + "인 계정이 없습니다.");
        return new ResponseEntity<>(exceptionDto, HttpStatus.BAD_REQUEST);
    }

}

```

해당 코드는 NotFoundException이 발생하면 처리합니다.



## 마치며

이상으로  글재주가 없다는걸 다시한번 느끼면서 account CRUD에 대한 정리를 마치겠습니다. 다음에는 JPA를 사용하여 DB(Postgresql)와 연동해 보겠습니다.

감사합니다.