---
title: "RabbitMQ 알아보기 #2"
date: 2021-09-26
categories: ["study"]
tags:
- rabbitmq
- publish/subscribe
- exchange
- binding
- 글또
---

 이번 글에서는 RabbitMQ 튜토리얼 #3, #4 에서 다루고 있는 pub/sub 개념을 정리하려고 합니다.

> 아래 글의 경우, RabbitMQ 공식 튜토리얼을 기반으로 하고 있습니다.
>
> [RabbitMQ 공식 튜토리얼](https://www.rabbitmq.com/getstarted.html)

---------------------------------------------------

## Publish/Subscribe messaging (Pub/Sub messaging)

### 등장 배경

- 앞선 Rabbitmq Tutorial 에서 다룬 messaging model 을 보면 아래와 같다.
  ![rabbitmq_basic_flow](/images/post_images/rabbitmq/rabbitmq_basic_flow.png)

  이때, 위 work queue에서는 "하나의 task(message)를 1개의 worker(consumer)에게만 전달된다"는 가정이 숨어있다.

  하지만, 1개의 message를 서로 다른 여러 개의 consumer로 전달 필요한 경우도 있다.
  
  - 예시 상황: logging level 이 error 인 log는 console 에 출력 + disk에 저장하고, info-level log는 console에 출력만 하기
  
  위 상황, 즉, 1개의 message를 특정 조건에 따라서 여러 개의 consumer에게 전달하고 싶으면 어떻게 해야할까?
  
  ![rabbitmq_pub_sub_flow](/images/post_images/rabbitmq/rabbitmq-pub_sub.png)
  
  ➡️ 위 질문에 대한 답이 바로 **"Publish/Subscribe"** 패턴이다.

### 정리

- **"Publish/Subscribe"** 패턴은 1개의 메세지를 여러 개의 consumer 에게 보내는 패턴이다.

## Exchanges

 Pub/Sub 패턴을 이해하기 위해서는 Rabbit의 full messaging model 과 이 모델 내 위치한 "exchange" 에 대한 이해가 필요하다.

RabbitMQ의 full messaging model 구조는 아래와 같다.

![rabbitmq_full_messaging_model_structure](/images/post_images/rabbitmq/rabbitmq-full_messaging_model.png)

 위 구조를 보면 exchange(파란색 X표)가 추가되었다. 즉, producer 가 보낸 메세지는 work queue로 바로 전달되는 것이 아니라, **반드시** exchange를 거쳐서 work queue로 전달된다.

 정리해보면, exchange란 producer로부터 메세지를 받아서 해당 메세지를 queue에 push 하는 역할을 수행한다.

<div class="notice"><p>Q: 앞선 튜토리얼(#1, #2)에서는 exchange가 사용되지 않은걸까?</p>
  <p><strong>그렇지 않다.</strong> 앞선 예제에서는 default exchange(nameless exchange)가 사용되었다.</p>
  </div>


 그렇다면 왜 메세지를 바로 work queue에 넣지 않고, 반드시 exchange를 거쳐서 queue에 넣는 것일까?

답은 exchange를 통해서 여러 개의 queue에 메세지를 전달하는 과정을 정교하게 할 수 있기 때문이다. exchange는 사전에 정의된 규칙을 참고하여, 새로 들어온 메세지를 특정 queue에만 전달하거나 어떤 queue에도 전달하지 않는(버리는) 작업을 수행한다. 이러한 규칙을 **"exchange type"** 이라고 부른다.

### Exchange type

- 의미: exchange가 메세지를 처리하는 유형

- 종류: `direct`, `topic`, `headers`, `fanout`

  --> Tutorial #3, #4에서는 `fanout` 타입과 `direct` 타입에 대해서만 살펴본다.

  - `fanout` 타입의 exchange: 해당 exchange와 바인딩된 모든 queue로 메세지 전달

  - `direct` 타입의 exchange: 들어온 메세지의 routing key 와 일치하는 routing key를 가진 queue에만 메세지 전달

### Exchange를 생성/사용하는 방법

- 명시적으로 exchange를 생성하여 사용하지 않으면 default exchange(nameless exchange)가 사용된다. 명시적으로 exchange를 선언하여 사용하고 싶을 때는 아래와 같이 한다.

  ```java
  channel.exchangeDeclare(exchangeName, "fanout"); // 첫번째 인자는 해당 exchange의 이름, 두번째 인자는 해당 exchange type
  ```

- Publisher가 exchange로 메세지를 보내는 방법은 아래와 같다.

  ```java
  channel.basicPublish(exchangeName, "", null, message.getBytes()); // 첫번째 인자가 exchange의 이름 (default exchange의 경우, 첫번째 인자에 빈 String("") 입력)
  ```

<div class="notice"><p><strong>Temporary queues</strong></p>
<p>Producer와 consumer가 queue를 공유해야하는 경우, queue의 이름을 지정해야하지만, 그외 경우 temporary queue와 exchange를 활용하자.
  <ol>
    <li>temporary queue 생성</li>
    <li>Publisher와 Consumer에서 사용되는 exchange name 통일</li>
  </ol>
</p>
</div>

## Bindings

### 개념

- exchange를 생성한 후에는 해당 exchange가 어떤 queue에 메세지를 전달해줄지 정해야한다. 이때, 이렇게 exchange와 queue의 관계를 정의한 걸 **binding**이라고 부른다.

  ![rabbitmq_bindings](/images/post_images/rabbitmq/rabbitmq-Bindings.png)

### Binding 생성하는 방법

```java
channel.queueBind(queueName, exchangeName, ""); // 세번째 인자는 routingKey(binding key)
```

<div class="notice"><p><strong>💡 binding은 producer 가 아니라, consumer 쪽에서 정의된다.</strong></p></div>

-------

Tutorial #5 부터의 내용은 다음 포스트에서 이어서 정리하겠습니다.

--------

### 참고/인용 출처

- [RabbitMQ Tutorial #3 - Publish/Subscribe](https://www.rabbitmq.com/tutorials/tutorial-three-java.html)
- [RabbitMQ Tutorial #4 - Routing](https://www.rabbitmq.com/tutorials/tutorial-four-java.html)
  --> 이번 글은 위 2개 튜토리얼을 기반으로 하고 있습니다.

