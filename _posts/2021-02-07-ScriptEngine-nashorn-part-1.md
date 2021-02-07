---
title: "ScriptEngine(Nashorn) 내 javascript var 변수 사용 시, 유의점"
date: 2021-02-07
category: work
tags:
- ScriptEngine
- Nashorn
- java
- javascript
---

 스프링부트 어플리케이션에서 간단한 javascript 코드 실행을 위해서 `ScriptEngine(Nashorn)` 을 사용하고 있다. 그러던 중 아래 두가지 문제를 발견했다.

- javascript `var` 변수 scope 문제
- OutOfMemory(OOM) - MetaSpace 문제

 이번 글에서는 `ScriptEngine(Nashorn)`에 대한 간단한 소개와 함께, 위 두가지 문제 중 **javascript `var` 변수 scope 문제**

를 다뤄보려고 한다.

-------

#### `ScriptEngine(Nashorn)`이란

`ScriptEngine` 은 `javax.script` 패키지 내 있는 인터페이스로, scripting language 사용을 위한 기본적인 인터페이스이다.

 `ScriptEngine` 을 이해하기에 앞서, `javax.script` 패키지부터 알아보자. 이 패키지는 Java SE 6에 추가된 패키지로, 다양한 scripting language를 java application에서 쉽게 사용할 수 있게 도와준다. 또한, 스크립트에서 java object에 접근할 수 있게 해준다.

 그렇다면 `ScriptEngine` 의 역할은 무엇일까?  `ScriptEngine` 은 scripting code와 해당 코드를 실제로 실행하는 interpreter/compiler 사이에 있는 중간다리 역할을 한다. 

![ScriptEngine-overall-process]({{site.url}}/assets/images/post_images/work/script_engine_overall_process.png)

> [Introducing the Java scripting API - Use the javax.script API to change a running application, IBM](https://www.ibm.com/developerworks/library/j-javascripting1/index.html#artrelatedtopics) 의 Figure 1: Scripting API component relationships 참고

`ScriptEngine` 이라는 인터페이스를 사용함으로써, 각 script language의 interpreter가 해당 코드 실행을 위해서 어떤 클래스를 사용하는지에 대한 세부정보는 추상화되어 최종 사용자에게 보이지 않는다.

`Nashorn`은 ECMAScript 5.1 사양에 따라 구현된 jvm 공식 JavaScript 엔진이다. (java 8부터 지원)

 --> java code에서 javascript 사용이 필요할 경우, `nashorn` 이라는 이름을 가진 `ScriptEngine`을 사용하면 된다.

#### ScriptEngine(Nashorn) 간단한 활용법

- 기본적으로 아래와 같이 java application에서 javascript 코드를 실행할 수 있다.

  ```java
  ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
  engine.eval("print('Hello World!');");
  ```

  - 관련 튜토리얼: [Java 8 Nashorn Tutorial](https://winterbe.com/posts/2014/04/05/java8-nashorn-tutorial/)

#### javascript 변수 scope 문제 재현

 스프링부트 어플리케이션에서 ScriptEngine을 통해서 javascript를 실행하던 중 javascript 변수가 엉뚱하게 공유되고 있는 문제를 발견했다.

- 상황 설명 및 재현

  - 해당 스프링부트 어플리케이션에서 ScriptEngine 사용 시, ScriptEngine을 bean으로 등록하여 사용하였다. 

  ```java
  /*
  ScriptEngine --> bean으로 등록
  */
  @Configuration
  public class SEConfig {
      @Bean
      public ScriptEngine scriptEngine(){
          ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
  
          return engine;
      }
  }
  ```

  ```java
  /*
  위에서 등록한 ScriptEngine을 사용하는 서비스
  */
  @Service
  public class ScriptEngineService {
  
      private ScriptEngine scriptEngine;
  
      public ScriptEngineService(ScriptEngine scriptEngine){
          this.scriptEngine = scriptEngine;
      }
  
      public void runJS(String jsCode){
  
          ScriptContext context = scriptEngine.getContext();
  
          Map<String, Integer> variableMap = new HashMap<>();
          variableMap.put("Agent_A", 0);
          variableMap.put("Agent_B", 0);
  
          context.setAttribute("variables", variableMap, ScriptContext.ENGINE_SCOPE);
  
          try{
              scriptEngine.eval(jsCode);
          }catch(ScriptException e){
              System.out.println(String.format("ScriptException while running js code: {}", jsCode));
          }
      }
  }
  ```

  

  javascript 변수 scope 관련 문제가 발생했던 상황 재현을 위한 이벤트 리스너를 만들어보았다.

  ```java
  @Component
  public class SimpleListener implements ApplicationListener<ApplicationStartedEvent> {
  
      private ScriptEngineService scriptEngineService;
  
      public SimpleListener(ScriptEngineService scriptEngineService){
          this.scriptEngineService = scriptEngineService;
      }
  
      @Override
      public void onApplicationEvent(ApplicationStartedEvent event) {
          
          // 첫번째 js code
          Thread agentA = new Thread(() ->{
              scriptEngineService.runJS("var agent_a_score = variables.get('Agent_A'); for (var i = 0; i < 300; i++) { agent_a_score++; } print('Agent_A ' + String(agent_a_score));");
          });
  
          // 두번째 js code
          Thread agentB = new Thread(() -> {
              scriptEngineService.runJS("var agent_b_score = variables.get('Agent_B'); for (var i = 0; i < 10; i++) { agent_b_score++; } print('Agent_B ' + String(agent_b_score));");
          });
  
          agentA.start();
          agentB.start();
      }
  }
  ```

  - 위 코드를 짰을 때, 의도했던 결과는 아래와 같다.

    ```
    Agent_A 300
    Agent_B 10
    ```

    하지만, 위 코드를 실행 결과는 아래와 같다.

    ```
    Agent_B 9
    Agent_A 293
    ```

    javascript 내 for-loop에서 변수 `i`를 초기화했지만, 의도했던 결과가 나오지 않았다. 이는 javascript에서 `var` keyword의 scope 때문에 발생하는 문제이다.

    javascript에서 var 키워드의 경우, 함수 외부에서 선언한 변수가 모두 전역 변수로 취급되기 때문에 위 `i` 변수도 공유된다.

    > 참고 링크: [자바스크립트 변수(var, let, const)와 스코프(function vs block) - FE study5](https://velog.io/@kimtaeeeny/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%B3%80%EC%88%98var-let-const%EC%99%80-%EC%8A%A4%EC%BD%94%ED%94%84function-vs-block-FE-study6)

#### javascript 변수 scope 문제 해결방법

위에서 서술한 문제를 해결하기 위해서 아래 두가지 방법을 찾아보았다.

- 방법1: `var` 대신에 `let` 사용하기

  자바스크립트에서 `let` 키워드의 경우, 해당 블록 내에서만 유효하기 때문에 위 문제가 해결될 것으로 보인다. 하지만, 변수 `i` 를 `let` 으로 선언했더니, ScriptException 이 발생하였다.

  --> `Nashorn`이 ES5.1을 기준으로 구현이 되어서, `let` 키워드를 사용할 수 없는 것으로 보인다.

  > [Is it OK to use "let" and 'const" in pre-ES6 code?'](https://stackoverflow.com/questions/50607674/is-it-ok-to-use-let-and-const-in-pre-es6-code)

- 방법2: 매번 ScriptEngine context 초기화

  아래와 같이 매번 ScriptContext를 초기화하는 방법도 있다.

  ```java
  
  @Service
  public class ScriptEngineService {
  
      private ScriptEngine scriptEngine;
  
      public ScriptEngineService(ScriptEngine scriptEngine){
          this.scriptEngine = scriptEngine;
      }
  
      public void runJS(String jsCode){
  
          ScriptContext context = new SimpleScriptContext();
  
          Map<String, Integer> variableMap = new HashMap<>();
          variableMap.put("Agent_A", 0);
          variableMap.put("Agent_B", 0);
  
          context.setAttribute("variables", variableMap, ScriptContext.ENGINE_SCOPE);
  
          // ScriptContext 초기화
          scriptEngine.setContext(context);
          try{
              scriptEngine.eval(jsCode);
          }catch(ScriptException e){
              System.out.println(String.format("ScriptException while running js code: {}", jsCode));
          }
      }
  }
  ```

  이 상태에서 스프링부트 어플리케이션 실행 시, 처음에 의도했던 결과를 확인할 수 있다.

  ```
  Agent_B 10
  Agent_A 300
  ```

  

----

- 참고
  - [Introducing the Java scripting API - Use the javax.script API to change a running application, IBM](https://www.ibm.com/developerworks/library/j-javascripting1/index.html#artrelatedtopics)