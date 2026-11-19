---
title: "예제로 정리하는 EM 알고리즘 (2)"
date: 2023-07-12
tags:
- "Statistics"
- "em"
summary: "Allele Frequency Estimation 예제로 살펴보는 EM 알고리즘 (2)"
---
이번 글에서는 [지난 글](/posts/2023-07-04-em-algorithm-example-1/)의 끝부분에 소개된 예제를 활용해서, EM 알고리즘을 마저 이해해보자.
> 이번 글은 연세대학교 박태영 교수님의 <데이터사이언스를위한통계전산 1 (STA6171.01-00)> 강의노트와 [다음 자료](http://galton.uchicago.edu/~eichler/stat24600/Handouts/s04.pdf)를 기반으로 작성되었습니다.

-----
# Allele Frequency Estimation 예제 소개
<center>
    <img src="https://velog.velcdn.com/images/quriemoon/post/930ff3b3-3f67-4fbb-b570-62b85d7fec70/image.png" width="30%" height="30%">
</center>

지난 포스트에서 정리한 것처럼 혈액형을 구성하는 blood type(?)은 다음과 같이 총 세가지(A,B.O)이며 이들을 조합해서 만들 수 있는 혈액형의 표현형은 총 4가지(A형, B형, AB형, O형), 유전자형은 총 6가지로 아래와 같다.
- A형: A/A, A/O
- B형: B/B, B/O
- AB형: A/B
- O형: O/O

이 상황에서 각 혈액형(A형/B형/AB형/O형)의 인원이 데이터로 주어져있다고 해보자. **이 상황에서 A, B, O 각각 type의 frequency를 의미하는 \(p_{A},p_{B},p_{O}\)를 추정하려면 어떻게 해야할까?** 이때, EM 알고리즘을 활용해서 이 문제를 풀 수 있다!

# EM 알고리즘을 활용한 Allele Frequency Estimation 단계별 풀이
## 0. Notation 정리
본론에 들어가기 앞서, 간단히 notation을 정리하자.
 - \(n_{X}\): 혈액형이 X인 인원 수이다. 이때, X에 들어가는 값이 혈액형의 표현형/유전자형 둘 다 가능하다. 
    - 즉, \(n_{A/A}\)라면 혈액형이 \(A/A\), 즉 혈액형의 유전자형이 \(A/A\)인 인원수이다. 그렇지만 \(n_{A}\)라면 혈액형이 A형, 즉 혈액형의 표현형이 A형인 인원 수이기 때문에 다음과 같이 쓸 수 있다: \(n_{A} = n_{A/A}+n_{A/O}\)
 - \(\theta\): 우리가 추정하려는 대상, 즉 파라미터로 이번 문제에서는 \((p_{A}, p_{B},p_{O})\) 이다.

## 1. 결측치 두기
지난 포스트에서 살펴본 것처럼 EM 알고리즘은 결측치가 있는 상황에서 \(l(\theta|y_{obs})\)를 최대화시키는 걸 목표로 한다. 따라서 EM 알고리즘을 모델링하기 위해서는 어떤 값을 결측치로 볼 것인가를 먼저 정해야한다. 이는 실제 관측치 내 결측된 값보다 latent variable처럼 모델링을 위해서 추가적으로 가정한 어떤 값이라고 이해할 수 있다.
### 1-1. 이번 예제에서의 결측치를 개념적으로 이해해보기
 - 이번 예제에서 생각해본다면 현재 관측된 값은 \(n_{A}\), \(n_{B}\), \(n_{AB}\), \(n_{O}\)이다. 이때, \(n_{A}\), \(n_{B}\), \(n_{AB}\), \(n_{O}\) 안에 실제로 관측이 누락된 값이 있는지 살펴보는 것이 아니라, latent variable처럼 새로운 변수를 두어 해당 값을 결측치처럼 생각해보자. \(A\)형과 \(B\)형의 경우, 혈액형의 표현형을 알아도 세부적인 유전형의 정확한 인원을 모른다는 점에서 힌트를 얻어서 \(n_{A/A}\), \(n_{A/O}\), \(n_{B/B}\), \(n_{B/O}\)를 결측치로 둘 수 있다.
### 1-2. 이번 예제에서의 결측치를 log-likelihood를 활용하여 이해해보기
\(n_{A/A}\), \(n_{A/O}\), \(n_{B/B}\), \(n_{B/O}\)를 결측치로 두는 게 타당하다는 것은 log-likelihood를 통해서도 확인할 수 있다.
 - EM 알고리즘의 원래 목적이 \(l(\theta|y_{obs})\)를 최대화하는 것이라는 점에서, \(l(\theta|y_{obs})\)를 먼저 확인하자.
   - 이번 데이터의 특성을 고려했을 때, 주어진 데이터는 multinomial distribution을 따른다고 생각할 수 있다.
    $$
    \begin{aligned}
        n_{A}, n_{B}, n_{AB}, n_{O} &\sim \text{Multinomial}(p_{A}^{2}+2p_{A}p_{O},p_{B}^{2}+2p_{B}p_{O}, 2p_{A}p_{B}, p_{O}^{2})
    \end{aligned}
    $$
   - \(y_{obs}\)가 \(n_{A}, n_{B}, n_{AB}, n_{O}\)인 상황이기 때문에 아래와 같이 observed data log-likelihood를 구할 수 있다. 이때, 파라미터인 \(\theta =(p_{A}, p_{B}, p_{O})\)가 포함되지 않은 항은 MLE를 구할 때, 필요없는 항이기 때문에 likelihood를 쓸 때, 파라미터가 포함된 항만 남겼다. 마찬가지의 맥락에서 log-likelihood 역시 어떤 constant \(C\)를 사용해서 표현하였다.
    $$
    \begin{aligned}
        L(\theta|y_{obs}) &\propto (p_{A}^{2}+2p_{A}p_{O})^{n_{A}} \cdot (p_{B}^{2}+2p_{B}p_{O})^{n_{B}} \cdot (2p_{A}p_{B})^{n_{AB}} \cdot (p_{O}^{2})^{n_{O}}\\
        l(\theta|y_{obs}) &= n_{A}\cdot \log(p_{A}^{2}+2p_{A}p_{O}) + n_{B} \cdot \log(p_{B}^{2}+2p_{B}p_{O})\\&+ n_{AB}\cdot \log(2p_{A}p_{B}) + n_{O} \log(p_{O}^{2})+C
    \end{aligned}
    $$
    이때, \(l(\theta|y_{obs})\) 식을 보면 \(\log(p_{A}^{2}+2p_{A}p_{O})\) 와 같이 \(log(\sum \theta_{i})\) 형태의 항이 포함되어 있다. 따라서 MLE를 구하기 위해 미분을 할 때, 이 부분이 깔끔하지 못하다.
    - 다음으로, \(n_{A/A}\), \(n_{A/O}\), \(n_{B/B}\), \(n_{B/O}\)를 결측치로 둔 상황에서 complete data log-likelihood를 구해보자. 마찬가지로, 아래 \(C\) 역시 어떤 constant값이다.
    $$
    \begin{aligned}
        L(\theta|y_{com}) &\propto (p_{A}^{2})^{n_{A/A}} \cdot (2p_{A}p_{O})^{n_{A/O}}\cdot (p_{B}^{2})^{n_{B/B}} \cdot (2p_{B}p_{O})^{n_{B/O}} \cdot (2p_{A}p_{B})^{n_{AB}} \cdot (p_{O}^{2})^{n_{O}}\\
        l(\theta|y_{com}) &= n_{A/A}\cdot \log(p_{A}^{2}) + n_{A/O}\cdot \log(2p_{A}p_{O})+n_{B/B}\cdot\log(p_{B}^{2})+n_{B/O}\cdot \log(2p_{B}p_{O})\\
        &+n_{AB}\cdot\log(2p_{A}p_{B})+n_{O}\cdot\log(p_{O}^{2})+C
    \end{aligned}
    $$
    
    위 \(l(\theta|y_{com})\)을 보면 결측치를 둠으로써, \(log(\sum \theta_{i})\) 형태의 항이 쪼개졌음을 확인할 수 있다. 이를 통해서, MLE를 구하기 위한 미분이 한결 수월해졌음을 확인할 수 있다.

## 2. E-step
다음으로, E-step을 수행하자. E-step은 간단히 이야기해서 \(Q(\theta|\theta^{(t)})\)를 계산하는 단계이다. 이전 포스트의 공식과 위에서 도출한 \(l(\theta|y_{com})\)을 활용해서 아래와 같이 \(Q(\theta|\theta^{(t)})\)가 구해진다.
$$
\begin{aligned}
Q(\theta|\theta^{(t)}) &= E[l(\theta|y_{com})|y_{obs},\theta^{(t)}]\\
&= E[n_{A/A}\cdot \log(p_{A}^{2}) + n_{A/O}\cdot \log(2p_{A}p_{O})+n_{B/B}\cdot\log(p_{B}^{2})+n_{B/O}\cdot \log(2p_{B}p_{O})\\
   &+n_{AB}\cdot\log(2p_{A}p_{B})+n_{O}\cdot\log(p_{O}^{2})+C|y_{obs},\theta^{(t)}]
\end{aligned}
$$

이어지는 M-step에서 위 \(Q(\theta|\theta^{(t)})\)를 최대화할 것인데, \(Q(\theta|\theta^{(t)})\)의 식을 자세히 보면 \(y_{obs}\)와 \(\theta^{(t)}\)가 주어져있다. 다시 말해, expectation 에서 random variable로 취급되는 부분은 \(n_{A/A}\), \(n_{A/O}\), \(n_{B/B}\), \(n_{B/O}\) 밖에 없으며 나머지는 상수항처럼 취급되어 expectation 바깥으로 빠져나오게 된다. 다시 말해, \(E[aX+bY] = aE[X]+bE[Y]\) 형태로 \(Q(\theta|\theta^{(t)})\)를 정리할 수 있다. 마저 정리해보자!
$$
\begin{aligned}
Q(\theta|\theta^{(t)}) &= E[l(\theta|y_{com})|y_{obs},\theta^{(t)}]\\
&= \log(p_{A}^{2})\cdot E[n_{A/A}|y_{obs},\theta^{(t)}]+\log(2p_{A}p_{O})\cdot E[n_{A/O}|y_{obs},\theta^{(t)}]\\
&+\log(p_{B}^{2})\cdot E[n_{B/B}|y_{obs},\theta^{(t)}]+\log(2p_{B}p_{O})\cdot E[n_{B/O}|y_{obs},\theta^{(t)}]\\
&+n_{AB}\cdot\log(2p_{A}p_{B})+n_{O}\cdot\log(p_{O}^{2})+C
\end{aligned}
$$
이때, 표기의 편의를 위해서 \(E[n_{X}|y_{obs},\theta^{(t)}] = \hat{n}_{X}\) 라고 표기하자.
$$
\begin{aligned}
Q(\theta|\theta^{(t)}) &= E[l(\theta|y_{com})|y_{obs},\theta^{(t)}]\\
&= \log(p_{A}^{2})\cdot \hat{n}_{A/A} + \log(2p_{A}p_{O})\cdot \hat{n}_{A/O}+\log(p_{B}^{2})\cdot \hat{n}_{B/B}+\log(2p_{B}p_{O})\cdot \hat{n}_{B/O}\\
&+n_{AB}\cdot\log(2p_{A}p_{B})+n_{O}\cdot\log(p_{O}^{2})+C\\
&= 2\cdot\log(p_{A})\cdot \hat{n}_{A/A} + \log(2p_{A}p_{O})\cdot \hat{n}_{A/O}+2\cdot \log(p_{B})\cdot \hat{n}_{B/B}+\log(2p_{B}p_{O})\cdot \hat{n}_{B/O}\\
&+n_{AB}\cdot\log(2p_{A}p_{B})+2\cdot n_{O}\cdot\log(p_{O})+C\\
\end{aligned}
$$

이때, E-step의 의미를 되새겨보면 expectation을 활용해서 결측치 부분을 채워넣는 거라고 이해할 수 있다. 즉, 결측치 대신에 \(\hat{n}_{A/A}, \hat{n}_{A/O}, \hat{n}_{B/B}, \hat{n}_{B/O}\) 를 사용함으로써 결측치를 채워넣는 것이라고 이해할 수 있다. 따라서 여기서의 E-step은 결론적으로 \(\hat{n}_{A/A}, \hat{n}_{A/O}, \hat{n}_{B/B}, \hat{n}_{B/O}\)를 구하는 문제로 귀결된다.
> 💡 **주의!** 그렇지만 [이전 포스트](/posts/2023-07-04-em-algorithm-example-1/)에서 언급한 것처럼 E-step은 결측치의 conditional expectation을 계산하는 것과 동일하다는 말에는 오류가 있다. 왜냐하면 다른 예제에서는 E-step이 결측치의 function 형태에 대한 conditional expectation을 계산해야하는 경우도 있기 때문이다.

E-step의 최종 관문인 \(\hat{n}_{A/A}, \hat{n}_{A/O}, \hat{n}_{B/B}, \hat{n}_{B/O}\) 를 구하자. 이를 위해서는 Multinomial distribution의 아래 성질을 활용해야한다.
$$
\begin{aligned}
(y_{1}, ..., y_{k}) &\sim Multinomial(n;p_{1},...,p_{k})\\
\therefore y_{i}|y_{i}+y_{j} &\sim Binomial(y_{i}+y_{j}, \frac{p_{i}}{p_{i}+p_{j}})
\end{aligned}
$$
\(\hat{n}_{A/A}\)의 경우, 관측치와 이전 iteration에서의 parameter값(\(\theta^{(t)}\))이 주어졌을 때의 \(n_{A/A}\)의 기댓값이다. 이때, \(n_{A}\)가 관측치이며 \(n_{A}=n_{A/A}+n_{A/O}\)인 점을 활용해서 아래와 같은 결과를 도출할 수 있다.
$$
\begin{aligned}
n_{A/A}|n_{A/A}+n_{A/O},\theta^{(t)} &\sim Binomial(n_{A/A}+n_{A/O}, \frac{(p_{A}^{(t)})^{2}}{(p_{A}^{(t)})^{2}+2p_{A}^{(t)}p_{O}^{(t)}})
\end{aligned}
$$
\(Binomial(n, p)\)를 따른 변수의 기댓값이 \(np\)라는 점을 떠올리면...
$$
\begin{aligned}
\hat{n}_{A/A} &= E[n_{A/A}|y_{obs}, \theta^{(t)}]\\
&= (n_{A/A}+n_{A/O})\cdot \frac{(p_{A}^{(t)})^{2}}{(p_{A}^{(t)})^{2}+2p_{A}^{(t)}p_{O}^{(t)}}\\
&= n_{A} \cdot \frac{(p_{A}^{(t)})^{2}}{(p_{A}^{(t)})^{2}+2p_{A}^{(t)}p_{O}^{(t)}}
\end{aligned}
$$
다음으로, \(n_{A/O} = n_{A}-n_{A/A}\)인 점을 활용해서, \(\hat{n}_{A/O}\)를 구하자.
$$
\begin{aligned}
\hat{n}_{A/O} &= E[n_{A}-n_{A/A}|y_{obs}, \theta^{(t)}]\\
&= n_{A} - \hat{n}_{A/A}
\end{aligned}
$$
같은 방식으로, \(\hat{n}_{B/B}, \hat{n}_{B/O}\)를 아래와 같이 구할 수 있다.
$$
\begin{aligned}
\hat{n}_{B/B} &= n_{B} \cdot \frac{(p_{B}^{(t)})^{2}}{(p_{B}^{(t)})^{2}+2p_{B}^{(t)}p_{O}^{(t)}}
\end{aligned}
$$
$$
\begin{aligned}
\hat{n}_{B/O} &= n_{B} - \hat{n}_{B/B}
\end{aligned}
$$

## 3. M-step
M-step에서는 앞서 구한 \(Q(\theta|\theta^{(t)})\)를 최대화하는 \(\theta\)값을 찾으면 된다. 이는 모든 데이터가 주어진 상황에서 MLE를 구하는 상황처럼 이해할 수 있다. 왜냐하면 E-step에서 expectation값으로 결측치 값을 채운 상황에서 이를 최대화하는 값을 찾으려 하기 때문이다.

Lagrange multiplier method로 이번 예제의 M-step을 풀 수 있다. Lagrange multiplier method는 간단하게 이야기해서, 제약 조건이 있는 상황에서 최적값을 찾으려는 것이다. 이번 예제에서의 parameter \(\theta\)에 대해서 생각해보며 모두 확률값이며 혈액형을 구성하는 blood type은 \(A\), \(B\), \(O\) 이렇게 세개밖에 없다. 따라서 다음과 같은 제약 조건이 존재하게 된다: \(p_{A}+p_{B}+p_{O} = 1\) 
정리해보면, M-step에서는 아래 \(\Lambda\)를 최대화하는 parameter 값을 찾아야한다.
$$
\begin{aligned}
\Lambda(\theta, \lambda) &= Q(\theta|\theta^{(t)}) + \lambda(p_{A}+p_{B}+p_{O}-1)
\end{aligned}
$$
최종적으로, M-step에서는 아래 식들을 풀어야한다.
$$
\begin{aligned}
\begin{cases}
     \frac{d\Lambda}{dp_{A}} = 0 \\
     \frac{d\Lambda}{dp_{B}} = 0 \\
     \frac{d\Lambda}{dp_{O}} = 0 \\
     \frac{d\Lambda}{d\lambda} = 0 \\
  \end{cases}
\end{aligned}
$$

이때, 위 식의 solution, 즉 \(Q(\theta|\theta^{(t)})\)를 최대화하는 값이 바로 이번 iteration에서 구한 새로운 \(\theta\)값인 \(\theta^{(t+1)}\) 이다. 직접 미분을 해서 결과를 정리하면 아래와 같은 결과가 나온다.

$$
\begin{aligned}
p_{A}^{(t+1)} &= \frac{\hat{n}_{A/A}+\frac{\hat{n}_{A/O}}{2}+\frac{\hat{n}_{A/B}}{2}}{n}
\end{aligned}
$$
$$
\begin{aligned}
p_{B}^{(t+1)} &= \frac{\hat{n}_{B/B}+\frac{\hat{n}_{B/O}}{2}+\frac{\hat{n}_{A/B}}{2}}{n}
\end{aligned}
$$
$$
\begin{aligned}
p_{O}^{(t+1)} &= \frac{n_{O}+\frac{\hat{n}_{A/O}}{2}+\frac{\hat{n}_{B/O}}{2}}{n}
\end{aligned}
$$

## 4. E-step, M-step 반복
M-step의 결과를 다시 E-step에서 사용하고, 이 E-step의 결과를 다시 M-step에서 사용하고.. 이런 식으로 EM 알고리즘을 수렴할 때까지 반복하면 된다. EM 알고리즘의 수렴 여부는 아래 4가지 조건으로 확인할 수 있다.
- \(l(\theta^{(t+1)}|y_{obs})-l(\theta^{(t)}|y_{obs}) < \epsilon\)
- \(||\theta^{(t+1)}-\theta^{(t)}|| < \epsilon\)
- \(\max_{i}|\theta_{i}^{(t+1)}-\theta_{i}^{(t)}| < \epsilon\)
- \(\max_{i}|(\theta_{i}^{(t+1)}-\theta_{i}^{(t)})/\theta_{i}^{(t)}| < \epsilon\)

## 결론
위 과정을 통해서 EM 알고리즘에서 목표했던 \(l(\theta|y_{obs})\) 를 최대화하는 \(\theta\)값을 찾을 수 있다.

다음 포스트에서는 Python으로 이번 예제에 대한 시뮬레이션을 해보자 :)

--------------------
피드백은 언제나 환영입니다! 혹시 잘못된 부분을 발견하셨거나 궁금하신 점이 있다면 댓글창에 남겨주세요! 그럼 다음 포스트에서 또 만나요!
