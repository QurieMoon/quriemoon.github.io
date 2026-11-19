---
title: "RabbitMQ 알아보기 #1"
date: 2021-08-29
categories: ["study"]
tags:
- rabbitmq
- message-broker
- 글또
---

 RabbitMQ 튜토리얼을 따라가면서, **메세지 브로커** 개념과 그 중 대표주자인 `RabbitMQ` 사용법을 공부하고 있습니다.

이번 글에서는 그중 Tutorial #1, #2 내용을 정리하였습니다.

> 아래 글의 경우, RabbitMQ 공식 튜토리얼을 기반으로 하고 있습니다.
>
> [RabbitMQ 공식 튜토리얼](https://www.rabbitmq.com/getstarted.html)

---------------------------------------------------

## 메시징 시스템(MOM 기반 시스템)

### 개념

- 어플리케이션 간에 메시지를 교환하기 위해 사용되는 시스템
  ![oracle_mom_system_diagram](/images/post_images/rabbitmq/oracle_mom_system_diagram.gif)

  > 출처: Oracle - MOM 기반 시스템 다이어그램

- 기본 요소: 클라이언트, 메세지, MOM 공급자 (+ API, 관리 도구)
- 동작 방법
  1. 클라이언트에서 MOM 시스템 내 API를 호출하여, 공급자가 관리하는 대상에게 메시지를 보냄
  2. 공급자는 수신한 메세지가 참조(조회) 될 때까지 메세지 보관
  3. 또 다른 클라이언트에서 MOM 시스템 내 API를 호출하여, 보관되어 있는 메세지 받아감

### 유형

1. `Point-to-point messaging`
   - 메세지를 보내는 측과 받는 측 간의 관계가 1:1임을 가정
     --> queue 에 있는 메세지는 오직 1곳으로부터 조회되고, 1번만 사용됨
   - 메세지가 단한번 처리되어야할 때, 사용되는 모델
     --> 대표 활용 예시: 결제 처리
2. `Publish/subscribe messaging`
   - 메세지를 보낼 때, queue에 직접 보내는 대신에 `topic` 에 해당 메세지를 보내고,
     메세지를 받는 측도 `topic` 에서 받아오려는 메세지를 받아옴
     --> 1개의 메세지를 받는 곳이 여러 곳일 수 있음
     --> 메세지를 보내는 측과 받는 측 간의 관계가 `one-to-many relationship` (`broadcast-style`)

### 대표 용어 정리

- `Producer` : 메세지를 보내는 측
- `queue` : MOM 공급자 내부에 메세지를 보관하는 *임시* 저장소
  --> `broker` : `RabbitMQ` 와 같은 message queue SW가 수행하는 역할로, `producer` 로부터 받은 메세지를 보관했다가 `consumer` 가 해당 메세지를 다시 가져가도록 대기하는 역할 (중간자)
- `Consumer`: 메세지를 받는 측

## Work Queues(a.k.a "Task Queues")

 앞서 배운 개념을 토대로, `rabbitmq`가 동작하는 방식을 제일 단순하게 정리하면 아래와 같다.

![rabbitmq_basic_flow](/images/post_images/rabbitmq/rabbitmq_basic_flow.png)

- 동작 방식
  1. Producer 가 queue 로 메세지를 보내고
  2. Consumer 가 해당 queue에세 필요한 메세지를 받아온다.

`RabbitMQ`는 아래 그림처럼 해당 메세지를 조회하는 Consumer가 1개가 아니라, 여러 개인 경우도 지원한다. 

![rabbitmq-work queues](/images/post_images/rabbitmq/rabbitmq-work-queues.png)

이때, 사용되는 queue를 `Work queue` 라고 부르는데, 아래 내용을 유념해야한다.

- 1개의 task는 정확히 1개의 worker(consumer)에게만 전송된다.
- RabbitMQ는 기본적으로 Round-Robin(RR) 방식으로 message를 보낸다.

위 `Work queue` 를 사용한 상황에서 발생할 수 있는 문제상황과 해결책을 알아보자.

| 문제상황                                                     | 해결책                                          |
| ------------------------------------------------------------ | ----------------------------------------------- |
| Consumer 측 문제로 메세지 손실<br />- 예시 상황: Consumer 가 메세지를 받고 처리하던 중 비정상 종료되어 메세지 손실 발생 | Message acknowledgement                         |
| Broker 측 문제로 메세지 손실<br />- 예시 상황: RabbitMQ 서버가 비정상적으로 멈춘 경우 | Queue의 durable 속성 활성화(Message durability) |
| Round-robin 방식으로, 일감 분배가 불균형하게 이뤄지는 경우<br />- 예시 상황: consumer가 총 2개 있는데, 홀수번째 작업은 처리 시간이 길고, 짝수번째 작업은 처리 시간이 짧은 경우 | Fair dispatch                                   |

 각 해결책에 대해서는 아래서 좀 더 자세히 알아보자.

### Message Acknowledgement

- `Acknowledgement(ack)` 란: **Consumer가** RabbitMQ에게 해당 메세지가 정상 도착/처리되었으므로 삭제해도 됨을 알려주는 것

- 동작 방식

  1. Consumer 가 메세지 처리 중 비정상 종료되어서, RabbitMQ에게 ack를 보내지 못함
  2. RabbitMQ는 메세지가 비정상 종료되었음을 알고, queue에 해당 메세지를 다시 넣음
  3. RabbitMQ에서 해당 메세지를 보낼 수 있는 다른 consumer가 있으면 거기로 보냄

- 주의사항

  - RabbitMQ에서는 `Manual message acknowledgements` 가 기본적으로 활성화되어있다.

    해당 옵션을 비활성화려면 `autoAck` 속성값에 `true` 를 넣어준다.

    <div class="notice"><p><strong>[참고] automatic acknowledgement mode(autoAck)</strong></p><p><ul>
      <li>Consumer로 메세지가 전송된 직후에 성공적으로 전송되었다고 체크되는 것</li>
      <li>fire-and-forget mode라고도 칭해짐</li>
      <li>문제점: Consumer로 메세지가 정상적으로 전달되기 전에, consumer의 TCP connection이나 채널이 비정상적으로 닫히면 메세지 손실됨</li>
      </ul></p></div>

### Message durability

- durable queue: RabbitMQ node가 재시작되어도 사라지지 않고, 그대로 남아있는 queue
- RabbitMQ에서 해당 기능 사용하는 방법
  - queue 생성 시, durable 속성에 true 설정

### Fair dispatch

- 현재 작업을 진행중인(busy) worker에게는 메세지를 추가로 보내지 않는 방법

- RabbitMQ에서 해당 기능 사용하는 방법

  - `basicQos` 메소드에 `prefetchCount` 파라미터 값으로 1을 대입

    --> 처리 중인 메세지가 있는 worker에게는 추가로 메세지를 보내지 않도록 RabbitMQ에게 알려주는 설정

-------

Tutorial #3 부터의 내용은 다음 포스트에서 이어서 정리하겠습니다.

--------

### 참고/인용 출처

- [RabbitMQ Tutorial #1 - Introduction](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)
- [RabbitMQ Tutorial #2 - Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-java.html)
  --> 이번 글은 위 2개 튜토리얼을 기반으로 하고 있습니다.



- [Oracle - 1장. 메시징 시스템 소개](https://docs.oracle.com/cd/E19636-01/819-3567/intro.html#wp553053)
  - 포스트 내 첫번째 다이어그램 출처
  - 해결책 및 한계점 내용 원문
- [IBM Cloud Learn Hub - Message Brokers](https://www.ibm.com/cloud/learn/message-brokers)
  - 메시지 브로커 유형 정리 시, 참고
- [RabbitMQ - Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html)
  - autoAck  관련 내용
- [Iron blog - What is RabbitMQ?](https://blog.iron.io/what-is-rabbitmq/)

