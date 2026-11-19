---
title: "예제로 정리하는 EM 알고리즘 (1)"
date: 2023-07-04
tags:
- "Statistics"
- "em"
summary: "Allele Frequency Estimation 예제로 살펴보는 EM 알고리즘 (1)"
---
EM 알고리즘의 유명한 예제인 'Allele Frequency Estimation' 예제를 통해서, EM 알고리즘을 정리해보자.
> 이번 글은 연세대학교 박태영 교수님의 <데이터사이언스를위한통계전산 1 (STA6171.01-00)> 강의노트와 [다음 자료](http://galton.uchicago.edu/~eichler/stat24600/Handouts/s04.pdf)를 기반으로 작성되었습니다.

# EM 알고리즘이란?
## EM 알고리즘의 목표
- EM 알고리즘을 자세히 살펴보기 앞서, EM 알고리즘이 해결하고자 하는 상황이 무엇인지 먼저 살펴보자. **EM 알고리즘은 결측치가 있는 데이터가 주어졌을 때, 이 데이터로부터 likelihood를 최대화하는 값, 즉 MLE를 찾는 문제를 해결하고자 한다.**
- 그렇다면 MLE를 왜 찾으려고 하는걸까? 이를 자세히 설명하기 위해서는 MLE의 특성을 살펴봐야겠지만, 그건 이번 글에서 다루는 내용을 벗어나기 때문에 생략하고 간단하게 이야기해서 **MLE는 주어진 데이터의 분포를 가장 잘 설명하는 파라미터이기 때문이다.**
  - 아래 MLE에 대한 설명은 약간 곁가지 내용이기 때문에 스킵해도 무방하다!
  - MLE에 대해서 조금만 더 자세하게 살펴보면, MLE는 likelihood를 최대화하는 값이다. 이때, likelihood는 \(L(\theta|y)\)로, 데이터(\(y\))가 주어졌을 때, \(\theta\), 즉 주어진 분포를 설명하는 파라미터에 대한 함수이다. 만약에 우리가 가정하는 분포가 정규분포라면 \(\theta\)는 \(\mu, \sigma\)가 될 것이다.
  이를 좀 더 직관적으로 이해해본다면 likelihood는 **\(\theta\)가 참일 때, \(y\)가 관측될 확률**이라고 이해할 수 있다. Notation만 본다면 주어진 데이터 \(y\)를 관측했을 때, \(\theta\)의 확률을 의미하는 것같지만 이는 잘못된 해석이다. (참고자료: [Toronto University, Chapter 6. Likelihood Inference, p.2](https://www.utstat.toronto.edu/mikevans/jeffrosenthal/chap6.pdf))
  따라서 likelihood가 높다는 건 해당 \(\theta\)에서 주어진 데이터가 관측될 가능성이 높다고 이해할 수 있다. 예시로, 고등학교 학급의 일반적인 수학 성적에 대한 분포가 주어진 상황에서 평균(\(\mu\))를 추정하는 상황을 생각해보자. 이때, 주어진 데이터는 정규분포를 따른다고 가정하자. 이때, 평균이 80점이라는 가정에서 주어진 데이터가 관측될 확률이 높을까? 아님 평균이 1.5점이라는 가정에서 주어진 데이터가 관측된 확률이 높을까? 아마 전자에서 주어진 데이터가 관측될 확률이 높을 것이다. 다시 말해서, 평균이 80점일 때의 likelihood가 평균이 1.5점일 때의 likelihood보다 크다고 생각할 수 있다.
  따라서 결국 likelihood를 최대화하는 값인 MLE는 해당 값에서 주어진 데이터가 관측될 확률이 제일 높은 값이라고 이해할 수 있다.
  
- 다시 본론으로 돌아와서, 우리가 보통 수업시간에 배운 likelihood 공식을 써보면 아래와 같다.
$$
L(\theta|y) = \prod_{i} p(y_{i};\theta)
$$
  이때, EM 알고리즘에서 해결하고자 하는 상황은 주어진 데이터 \(y\)에 결측치가 포함된 상황이다.
  구체적으로, **주어진 데이터에서 관측된 값을 \(y_{obs}\), 결측치를 \(y_{mis}\)라고 했을 때, \(\log L(\theta|y_{obs}) = l(\theta|y_{obs})\)를 최대화하는 게 EM 알고리즘의 목표이다.**

## EM 알고리즘의 핵심 아이디어
![](https://velog.velcdn.com/images/quriemoon/post/b32baad3-7c64-4b69-bc21-9e2044f59970/image.jpg)
<a href="https://www.freepik.com/free-photo/close-up-hatched-small-chicken_10685631.htm#page=3&query=chicken%20or%20egg&position=9&from_view=search&track=ais">(Close up hatched small chicken) - Image by gpointstudio</a> on Freepik

- EM 알고리즘의 핵심 아이디어는 chicken and egg problem("닭이 먼저냐 달걀이 먼저냐")로 설명할 수 있다. Chicken and egg problem은 닭이 존재하기 위해서는 달걀이 있어야하고, 달걀이 생기려면 닭이 있어야하는 꼬리에 꼬리를 무는 상황을 가리킨다. 아래 서술된 것처럼, EM 알고리즘이 주목한 내용도 이와 닮았다.

  1. 만약 우리가 주어진 분포를 설명하는 파라미터를 알고 있다면 결측치를 채워넣는 건 쉽다.
  2. 만약에 결측치 없이 모든 데이터가 주어졌다면 파라미터 추정은 쉽다.
  
  **EM 알고리즘에서는 위 두 과정을 반복하면서 답을 찾아간다.** 구체적으로, 위 설명의 1번 상황은 기댓값을 활용해서 결측치를 채우는 **E-step(the expectation step)**, 2번 상황은 **M-step(the maximization step)**으로 이해할 수 있고, EM 알고리즘은 이 E-step과 M-step을 특정 조건이 만족할 때까지 반복해가며 문제를 해결한다.

## EM 알고리즘의 E-step과 M-step
- EM 알고리즘의 E-step은 아래 Q-function(\(Q(\theta|\theta^{(t)})\))을 계산하는 과정이다.
  $$
  \begin{aligned}
  Q(\theta|\theta^{(t)}) &= E[l(\theta|y_{com})|y_{obs},\theta^{(t)}]\\
  &= E[l(\theta|y_{obs},y_{mis})|y_{obs},\theta^{(t)}]  
  \end{aligned}
  $$
  
  위 수식에서 \(l(\theta|y_{com})\) 는 complete data log-likelihood이고, \(\theta^{(t)}\)는 이전 iteration에서 샘플링한 \(\theta\)값이다. 
  이때, 위 수식을 잘 보면 \(y_{obs},\theta^{(t)}\)가 주어져있다! 따라서 complete data log-likelihood에서 \(y_{obs},\theta^{(t)}\)가 들어간 부분은 상수항처럼 취급되고, \(y_{mis}\)만 random variable처럼 취급됨을 알 수 있다!
  
  따라서 결국 E-step은 간단하게 생각하면 \(y_{mis}\)의 conditional expectation값을 계산하는 것처럼도 생각할 수 있다. 물론 이 말은 틀렸다! 이해를 돕기 위해서 이렇게 표현했지만, 몇몇 예제를 통해서 E-step에서 \(y_{mis}\)의 function의 conditonal expectation값을 계산하는 등 추가적인 작업도 발생할 수 있음을 확인할 수 있기 때문이다.
  
  일단 지금은 이 설명이 와닿지 않을 수 있기 때문에.. 예제를 통해서 더 자세히 이해해보자.
  
  > 이때, \(y_{com}\)의 충분 통계량을 사용하면 \(Q(\theta|\theta^{(t)})\)의 \(l(\theta|y_{com})\)이 간단해진다는 점이 큰 장점이지만! 이번 포스트에서 이 내용은 생략한다.
  
- 다음으로, M-step은 앞서 구한 Q-function을 바탕으로 새로운 \(\theta\)값을 샘플링하는 단계이다. 구체적으로, Q-function을 최대화하는 값이 새로운 \(\theta\)값이 된다.
  $$
  \begin{aligned}
  \theta^{(t+1)} &= \arg \max_{\theta} Q(\theta|\theta^{(t)})
  \end{aligned}
  $$
  
  이 역시 예제를 통해서 이해해보자..!
  
# Allele Frequency Estimation 예제 소개
이번 포스트에서 살펴볼 예제는 Allele Frequency Estimation 예제로, 혈액형과 관련된 예제이다.
옛날에 과학 시간에 배웠던 내용을 떠올려보면 혈액형을 구성하는 blood type(?)은 다음과 같이 총 세가지이다: A,B,O. 그리고 이들을 조합해서 만들 수 있는 혈액형의 표현형은 총 네가지(A형, B형, AB형, O형)이며 이는 다시 아래와 같이 총 6개의 유전자형으로 구성된다.
- A형: A/A, A/O
- B형: B/B, B/O
- AB형: A/B
- O형: O/O

이 상황에서 각 혈액형(A형/B형/AB형/O형)의 인원이 데이터로 주어져있다고 해보자. **이 상황에서 A, B, O 각각 type의 frequency를 의미하는 \(p_{A},p_{B},p_{O}\)를 추정하려면 어떻게 해야할까?** 이때, EM 알고리즘을 활용해서 이 문제를 풀 수 있다!

포스트가 길어져서, 예제에 대한 단계별 풀이는 다음 포스트에서 살펴보자 :)

----------------

피드백은 언제나 환영입니다! 혹시 잘못된 부분을 발견하셨거나 궁금하신 점이 있다면 댓글창에 남겨주세요! 그럼 다음 포스트에서 또 만나요!
