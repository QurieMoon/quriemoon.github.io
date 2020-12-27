---
title: "[강의 내용 정리] <<스프링 부트 개념과 활용>> - 독립적으로 실행 가능한 jar"
date: 2020-12-27
category: spring-boot-basics
tags:
- spring-boot
- spring-boot-maven-plugin
- inflearn
---

 현재 수강중인 백기선 님의 [&laquo;스프링 부트 개념과 활용&raquo;](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard) 강의 내용을 정리하고 있습니다.

---------------------------------------------------

#### 독립적으로 실행 가능한 jar

 스프링부트를 처음해보면 jar 파일 하나만으로 실행 가능하다는 점이 참 신기합니다.

실행하려는 애플리케이션 클래스 정보부터 필요한 외부 라이브러리까지 모조리 땅땅 jar 파일 하나에 어떻게 담겨 있을까요?

이는 스프링부트 기본 프로젝트에 포함되어 있는 `spring-boot-maven-plugin` 덕분에 가능해집니다.

위 `spring-boot-maven-plugin` 플러그인은 크게 아래 두가지 일을 수행합니다.

1. jar 파일에서 필요한 기본 정보 추가

   - 패키징된 스프링부트 jar 파일 압축 풀어보면, `META-INF/MANIFIEST.MF` 내 jar 파일에서 필요한 아래 정보가 추가된 걸 확인할 수 있습니다.

     ```
     Manifest-Version: 1.0
     Created-By: Maven Jar Plugin 3.2.0
     Build-Jdk-Spec: 15
     Implementation-Title: spring-boot-getting-started
     Implementation-Version: 1.0-SNAPSHOT
     Main-Class: org.springframework.boot.loader.JarLauncher
     Start-Class: me.quriemoon.Application
     Spring-Boot-Version: 2.4.0
     Spring-Boot-Classes: BOOT-INF/classes/
     Spring-Boot-Lib: BOOT-INF/lib/
     Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
     Spring-Boot-Layers-Index: BOOT-INF/layers.idx
     
     ```

   - 위 정보들 중 특히 중요한 정보는 `Main-Class`, `Start-Class`, `Spring-Boot-Classes`, `Spring-Boot-Lib`입니다.

     - `Main-Class` : 스프링부트 프로젝트의 경우, Main-Class의 값으로 스프링부트의 `org.springframework.boot.loader.JarLauncher`를 지정합니다.

     - `Start-Class` : 현재 개발중인 프로젝트의 메인클래스를 Start-Class값으로 지정합니다.

       > 스프링부트 프로젝트가 아니라면, `Main-Class` 에 개발중인 프로젝트의 메인클래스가 위치해야합니다.

     - `Spring-Boot-Classes` : 현재 개발중인 프로젝트의 클래스 파일들이 위치한 경로입니다.

     - `Spring-Boot-Lib` : `pom.xml` 에서 추가한 외부 라이브러리들이 위치한 경로입니다.  

       

2. 패키징 시, 아래와 같은 계층 구조로 jar 파일 생성

   - 패키징된 스프링부트 jar 파일 압축을 풀어보면, 아래와 같은 구조를 확인할 수 있습니다.

     ![img](http://wiki.sys4u.co.kr/download/attachments/10585052/image2019-9-20_10-56-46.png?version=1&modificationDate=1568944606000&api=v2)

   - 위 구조에서 `BOOT-INF/classes` 에는 애플리케이션 클래스가, `BOOT-INF/lib` 에는 프로젝트에서 사용하는 라이브러리들이 위치한 걸 확인할 수 있습니다.



이렇게 하나의 jar 파일이 생성되면...

- `org.springframework.boot.loader.jar.JarFile` 을 사용해서 내장 jar를 읽고,
- `org.springframework.boot.loader.jar.Launcher` 를 사용해서 스프링부트 애플리케이션을 실행합니다.



------

#####참고/인용 자료

#####- [spring-boot-maven-plugin의 역할](http://wiki.sys4u.co.kr/pages/viewpage.action?pageId=10585052)

- 본문에 등장한 jar 파일 구조 사진 자료도 위 링크에서 가져왔습니다.