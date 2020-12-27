---
title: "[강의 내용 정리] <<스프링 부트 개념과 활용>> - 의존성 관리, 자동설정"
date: 2020-12-27
category: spring-boot-basics
tags:
- spring-boot
- dependency
- configuration
- inflearn
---

현재 수강중인 백기선 님의 [&laquo;스프링 부트 개념과 활용&raquo;](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard) 강의 내용을 정리하고 있습니다.

---------------------------------------------------

#### 의존성 관리

Springboot 프로젝트에서 의존성 관리를 하는 방법은 아래 두 가지 방법이 있습니다.

1. (권장) pom.xml에서 `spring-boot-starter-parent` 활용

   - `spring-boot-starter-parent` 의 parent 프로젝트인 `spring-boot-dependencies` 프로젝트로 버전 관리가 이루어집니다.

     **![spring-boot-basics-part3-dependency-01]({{site.url}}/assets/images/post_images/spring-boot-basics-baek/2020-12-13-spring-boot-basics-1.jpg)**

   

   - 위 `<parent>` 설정을 마친 이후에는 아래와 같이 의존성을 추가합니다.

     - 스프링부트가 버전 관리 해주는 의존성 의 경우, `<dependency>` 태그를 사용합니다.

       ```xml
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-undertow</artifactId>
               </dependency>
       ```

       > [스프링부트 관련 문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)를 보면 `spring-boot-starter-parent` 자체가 의존성을 제공하지 않기 때문에 필요한 의존성을 추가하기 위해서는 `spring-boot-starter-web` 의존성을 추가해야한다고 합니다.

     - 스프링부트에서 이미 관리하고 있는 의존성 버전을 변경하고 싶을 경우, `<properties>` 태그를 활용하여 변경합니다.

       ```xml
               <properties>
                   <spring.version>5.0.6.RELEASE</spring.version>
               </properties>
       ```

       - 스프링부트에서 버전 관리를 하지 않는 의존성의 경우, 꼭 `<version>`까지 명시하여 의존성을 추가합니다.

2. parent POM 없이`dependencyManangement` 활용

   - parent POM 에서는 dependency 뿐만 아니라, java version, encoding 등 다른 property에 대한 설정도 이루어지기 때문에 이 방법은 권하지 않습니다.

#### 자동설정

Springboot 에서는 bean을 통상 2번 등록합니다.

1. `@ComponentScan` 기준으로 bean 등록
   - `@ComponentScan`은 `@Component` 기준으로 bean을 스캔/등록합니다.
     - 이때, `@ComponentScan` 어노테이션이 붙은 클래스가 속한 패키지의 bean을 등록합니다.
2. `@EnableAutoConfiguration` 기준으로 bean 등록
   - 이때, 자동설정 내용을 기준으로 한 bean이 생성됩니다.

> `@SpringBootApplication` 어노테이션에는 `@ComponentScan`, `@EnableAutoConfiguration` 이 포함되어 있습니다.

따라서 같은 이름의 bean이 두 번 등록될 경우에는 `@EnableAutoConfiguration` 기준으로 생성된 bean이 `@ComponentScan` 기준으로 생성던 bean을 덮어씁니다. 따라서 이 경우에는  `@ConditionalOnMissingBean` 어노테이션을 통하여, `@ComponentScan` 때, 등록한 bean을 활용할 수 있습니다.

자동설정 관련 어노테이션 및 설정 파일을 추가로 정리해보면 아래와 같습니다.

- `@Configuration`
  
  - Bean을 등록하는 java 설정 파일
  
- `@ConfigurationProperties`

  - property 값을 직접 설정하여, bean 을 생성 시, 필요함

    > 위 어노테이션 사용 시, 아래 설정 필요함
    >
    > **[3.1. Configure the Annotation Processor](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html#configuration-metadata-annotation-processor)**

    - property를 활용 시, 해당 starter를 참조하는 메인 프로젝트의 설정 파일(`application.properties`)에서 설정한 property 값대로 bean 생성됨

- `@EnableConfigurationProperties`

  - property 설정 참고 시, 필요한 어노테이션
    - 예) `@EnableConfigurationProperties(SampleProperties.class)`

- `src/main/resources/META-INF/spring.factories`

  - 자동설정 시 불러오려는 설정 클래스명(예: `SampleConfiguration`)을 `spring.factories` 에 아래와 같이 명시합니다.

    ```
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
      com.sample.SampleConfiguration
    ```

    - 나중에 위 설정을 포함한 starter 프로젝트를 타 프로젝트에서 참조 시, 해당 jar 파일의 `META-INF/spring.factories` 에 위에서 설정한 `src/main/resources/META-INF/spring.factories` 내용을 확인할 수 있습니다.

---------

다음 글에서는 스프링부트 내장 웹 서버와 독립적으로 실행 가능한 jar 에 대해서 다뤄보겠습니다.