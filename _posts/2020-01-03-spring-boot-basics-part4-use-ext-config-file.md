---
title: "[강의 내용 정리] <<스프링 부트 개념과 활용>> - 스프링부트에서 application.properties 설정파일 사용법"
date: 2021-01-03
category: spring-boot-basics
tags:
- spring-boot
- application.properties
- properties
---

 현재 수강중인 백기선 님의 [&laquo;스프링 부트 개념과 활용&raquo;](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard) 강의 내용을 정리하고 있습니다.

####application.properties 설정 파일에 있는 값 활용하기

application.properties 설정 파일에 있는 값을 불러올 때, 종종 아래와 같이 `@Value` 어노테이션을 활용하여, 불러옵니다.

```
# application.properties 내 설정값
sample.value = Sample
```

```java
@Value("sample.value")
private String sampleValue;
```

 이번 강좌에서는 `@Value` 어노테이션을 활용하는 방법이 아닌, 관련 설정을 묶어서 bean으로 등록한 후에 활용하는 방안을 배웠습니다.

설정 파일에서 아래 3개의 설정값(`user.name` , `user.age`, `user.fullName`) 을 활용하려는 상황을 가정해봅시다.

```
# application.properties 내 설정값
user.name = Gildong
user.age = ${random.int(1,100)}
user.fullName = ${user.name} Hong
```

 1단계: 위 설정 파일과 매핑될 클래스를 만들어줍니다.

```java
@Component
@ConfigurationProperties("user")
public class UserProperties {

    private String name;

    private int age;

    private String fullName;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getFullName() {
        return fullName;
    }

    public void setFullName(String fullName) {
        this.fullName = fullName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

 `@ConfigurationProperties`에 사용하려는 설정들의 공통 prefix인 "user"를 명시합니다.

2단계: 위 설정을 사용하려는 곳에서 위 Bean으로 등록된 위 설정을 활용합니다.

```java
@Component
public class PracticeApplicationRunner implements ApplicationRunner {

    @Autowired
    private UserProperties userProperties;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("==================");
        System.out.println(userProperties.getName());
        System.out.println("==================");
    }
}

```



#### 관련 설정을 묶어서 bean으로 등록한 후, 활용 vs. @Value 어노테이션 활용

`@Value` 어노테이션을 사용하는 대신에, 위와 같이 관련 설정을 bean으로 등록하여, 사용 시, 아래와 같은 장점이 있습니다.

- 융통성 있는 바인딩

  - 프로퍼티 값을 camel-case(예: fullName), kebab-case(예: full-name) 등등 다양한 형식으로 작성하더라도 융통성 있게 바인딩됩니다.

- 프로퍼티 타입 컨버전

- 프로퍼티 값 검증

  - `@Validated` 어노테이션을 사용하여, 아래와 같이 프로퍼티가 특정 조건에 부합하는지 검사할 수 있습니다.

    ```java
    @Component
    @ConfigurationProperties("user")
    @Validated
    public class UserProperties {
    
        @NotEmpty
        private String name; // name 필드에 빈 값이 들어올 수 없음
      
      ...
        
    }
    ```



`@Value` 어노테이션 사용 시, SpEL을 적용할 수 있다는 장점이 있으나, 위에서 언급한 장점들을 활용할 수 없습니다.

따라서 이번 강의에서는 `@Value` 어노테이션보다는 위와 같이 관련 있는 설정들을 bean으로 등록하여, 사용하는 걸 권장하고 있습니다.

