---
title: "SpringBoot RestTemplate Timeout, interceptor logging 적용기"
date: 2021-08-12
categories: ["work"]
tags:
- spring-boot
- restTemplate
- 글또
---

 SpringBoot 에서 외부 API 호출을 위해서 사용한 RestTemplate에 timeout 설정과 interceptor logging 을 적용했던 과정 및 문제점과 해결책을 공유하려고 합니다.

---------------------------------------------------

## RestTemplate에 timeout 설정과 interceptor logging 적용하는 방법

### 1. RestTemplate에 read timeout 설정

- `HttpComponentsClientHttpRequestFactory` 속성 중 `readTimeout` 값을 설정하고, 위 `HttpComponentsClientHttpRequestFactory` 을 기반으로 `restTemplate` 을 생성한다.

  ``````java
  /*
  외부 API 호출 시, 사용될 restTemplate
   - Bean 으로 등록해서 사용
  */
  @Bean
  public RestTemplate restTemplate(){
    HttpComponentsClientHttpRequestFactory httpComponentsClientHttpRequestFactory = new HttpComponentsClientHttpRequestFactory();
    httpComponentsClientHttpRequestFactory.setReadTimeout(3000); // read Timeout 설정되는 부분
    HttpClient httpClient = HttpClientBuilder.create()
      .setMaxConnTotal(500)
      .setMaxConnPerRoute(100)
      .build(); // <참고> 부분 확인
    httpComponentsClientHttpRequestFactory.setHttpClient(httpClient);
  
    RestTemplate restTemplate = new RestTemplate(httpComponentsClientHttpRequestFactory);
    return restTemplate;
  }
  ``````
  
  <div class="notice">
    <p><strong>[참고] httpClient 설정 이유</strong></p>
    <p>
      RestTemplate 은 기본적으로 connection pool을 활용하지 않는다. 즉, 매 REST 호출마다 새로운 포트를 열어서 새로운 연결을 맺는다. 이때, close 직후에 해당 socket이 일정 시간동안 'TIME_WAIT' 상태에 들어가기 때문에 요청량이 많다면 이러한 상태의 소켓들을 재사용하지 못하고, 응답 지연 현상이 초래된다.
    </p>
    <p>
      이때, httpClient를 활용하여, RestTemplate 에서의 connection pool 설정을 할 수 있다.
      <ul>
        <li>'maxConnPerRoute': IP, port 번호 1쌍에 가능한 연결 수 제한</li>
        <li>'maxTotal': 최대 오픈되는 연결 수 제한</li>
  	  </ul>
    </p>
  </div>

### 2. RestTemplate에 interceptor logging 부분 추가

- `RestTemplate` 에 interceptor logging 부분을 추가하지 않았을 때는 실행 로그에 API 호출 request/response 내역이 남지 않는다.

  따라서 아래처럼 API 호출 request/response에 대한 기록을 하고 싶을 경우에는 interceptor 를 활용하여 API request/response 를 로깅할 수 있다.

  - 예시) request param.으로 param1, param2 가 있고, response param.으로 msg, code 가 있는 더미 API를 호출한 상황

    ``````
    Request body: {"param1":"value1","param2":"value2"}
    Response body: {"msg":"success","code":"0000"}
    ``````

- 방법

  1. `ClientHttpRequestInterceptor` 를 상속받아서 customizing 된 restTemplate interceptor 를 구현한다.

     - 1번 interceptor 에서 API request/response 내역 로깅을 한다.

     ``````java
     /*
      코드 출처: https://www.baeldung.com/spring-resttemplate-logging
     */
     public class RestTemplateInterceptor implements ClientHttpRequestInterceptor {
     
         static Logger logger = LoggerFactory.getLogger(RestTemplateInterceptor.class);
     
         @Override
         public ClientHttpResponse intercept(HttpRequest request, byte[] reqBody, ClientHttpRequestExecution execution) throws IOException {
             logger.info("Request body: {}", new String(reqBody, StandardCharsets.UTF_8));
     
             ClientHttpResponse response = execution.execute(request, reqBody);
     
             InputStreamReader isr = new InputStreamReader(
                     response.getBody(), StandardCharsets.UTF_8
             );
             String body = new BufferedReader(isr).lines()
                     .collect(Collectors.joining("\n"));
             logger.info("Response body: {}", body);
             return response;
     
         }
     
     }
     ``````

  2. RestTemplate 에 1번 단계에서 구현한 interceptor 를 추가한다.

     ``````java
     /*
      코드 출처: https://www.baeldung.com/spring-resttemplate-logging
      */
     RestTemplate restTemplate = new RestTemplate(httpComponentsClientHttpRequestFactory);
     List<ClientHttpRequestInterceptor> interceptorList = restTemplate.getInterceptors();
     if(CollectionUtils.isEmpty(interceptorList)){
       interceptorList = new ArrayList<>();
     }
     
     interceptorList.add(new RestTemplateInterceptor());
     restTemplate.setInterceptors(interceptorList);
     ``````

     - 실행 결과: API 호출 시, request/response 모두 로깅이 잘 된다.

     ``````
     2021-08-14 17:09:51.840  INFO 4308 --- [           main] c.s.s.utils.RestTemplateInterceptor      : Request body: {"param1":"value1","param2":"value2"}
     2021-08-14 17:09:53.048  INFO 4308 --- [           main] c.s.s.utils.RestTemplateInterceptor      : Response body: {"msg":"success","code":"0000"}
     ``````

## RestTemplate 에 interceptor logging 적용 과정에서 겪었던 문제점

RestTemplate 에 interceptor 를 추가한 후, 코드를 실행했더니 아래와 같은 에러가 발생하였다.

``````
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "http://localhost:8088/sample": Attempted read from closed stream.; nested exception is java.io.IOException: Attempted read from closed stream.
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:785) ~[spring-web-5.3.9.jar:5.3.9]
	...
Caused by: java.io.IOException: Attempted read from closed stream.
 ... 
``````

관련 내용을 찾아보면, 문제의 원인은 아래와 같다.

> **As the interceptor consumes the response stream, our client application will see an empty response body**. 
>
> 내용 출처 : [Baeldung - Spring RestTemplate Request/Response Logging](https://www.baeldung.com/spring-resttemplate-logging)

즉, interceptor 에서 response stream 을 먼저 사용해버렸기 때문에 application에는 빈 response body 가 전달되어서 위 문제가 발생하였다.

## 해결책

위 문제는 `BufferingClientHttpRequestFactory` 를 사용하여 해결할 수 있다.  `BufferingClientHttpRequestFactory` 는 메모리에 스트림에 있는 내용을 올려놓기 때문에 interceptor와 application 모두 response body 내용을 정상적으로 사용할 수 있다. 

- 해결책 적용 후 코드

  ``````java
  RestTemplate restTemplate = new RestTemplate(new BufferingClientHttpRequestFactory(httpComponentsClientHttpRequestFactory)); // 해결책 적용된 부분
  
  List<ClientHttpRequestInterceptor> interceptorList = restTemplate.getInterceptors();
  if(CollectionUtils.isEmpty(interceptorList)){
    interceptorList = new ArrayList<>();
  }
  
  interceptorList.add(new RestTemplateInterceptor());
  restTemplate.setInterceptors(interceptorList);
  ``````
  
- 한계점

  `BufferingClientHttpRequestFactory` 를 쓰면 application에 빈 response body가 넘어가는 문제는 해결되지만, 퍼포먼스 측면에서 단점이 있다. 구체적으로, response body data 전체를 메모리로 올리기 때문에 퍼포먼스 측면에서 이슈가 있고, 최악의 경우에는 OutOfMemoryError 도 발생할 수 있다.

   따라서 로깅 레벨이 DEBUG 레벨일 때만 `BufferingClientHttpRequestFactory` 를 써서, API request/response 내역을 로깅하는 방안도 제시되고 있다.

------

### 참고/인용 출처

- [Baeldung - Spring RestTemplate Request/Response Logging](https://www.baeldung.com/spring-resttemplate-logging)
  - 이번 블로그의 sample code 출처
  - 해결책 및 한계점 내용 원문

- [권남 - Spring RestTemplate](https://kwonnam.pe.kr/wiki/springframework/resttemplate)

- [마로의 Spring Framework 공부 - RestTemplate Logging(로그)](https://hoonmaro.tistory.com/49)

- [빨간색코딩 - RestTemplate (정의, 특징, URLConnection, HttpClient, 동작원리, 사용법, connection pool 적용)](https://sjh836.tistory.com/141)
  - [참고] httpClient 설정 이유 부분 원문

- [Stackoverflow - Using Spring REST template, either creating too many connections or slow](https://stackoverflow.com/questions/31869193/using-spring-rest-template-either-creating-too-many-connections-or-slow/)
