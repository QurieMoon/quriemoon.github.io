---
title: "Rcpp, Eigen 활용해서 R 패키지 만들기"
date: 2023-05-24
tags:
- "Eigen"
- "R"
- "R-pkg"
- "Rcpp"
- "usethis"
summary: "usethis를 활용한 R 패키지 개발기"
---
최근 Rcpp와 C++의 Eigen 을 활용해서 개발을 진행하였다. 지난 [포스트](/posts/2022-12-02-rcpp/)에서처럼 `sourceCpp()`를 활용해서 로컬에서 어느 정도 개발을 마치고, 서버에서 이를 실행하려고 했는데.. 

**에러가 발생하였다..**

 <p align="center">
  <img src="https://velog.velcdn.com/images/quriemoon/post/f1c0b141-1cde-47dd-b958-b2d710580989/image.png" alt="collect2: error: ld returned 1 exit status ; Error in sourceCpp(&quot;src/***.cpp&quot;) : Error 1 occured building shared library. Calls: source -> with Visible -> eval -> eval -> sourceCpp Execution halted" width="400" />
</p>

위 에러를 해결하기 위해 이것저것 찾아본 결과, 현재 R 프로젝트를 패키지로 빌드해야한다는 결론에 도달하였다. 하지만, 처음 해보는 R 패키지 빌드 역시 험난한 과정이었으니..!

 <p align="center">
  <img src="https://velog.velcdn.com/images/quriemoon/post/b2f431b0-4cc1-4b25-947a-946da284f62c/image.jpeg" alt="text" width="200" />
</p>


[](https://velog.velcdn.com/images/quriemoon/post/b2f431b0-4cc1-4b25-947a-946da284f62c/image.jpeg)

이번 글에서는 간단한 R 패키지를 개발해보면서 R 패키지 빌드 및 설치 과정을 단계별로 정리하려고 한다.

> 이번 글의 경우, `usethis` 패키지를 활용하며 아래 글에 기초해서 작성되었습니다.
> [R Packages (2e) by Hadley Wickham and Jenny Bryan - 2. The Whole Game](https://r-pkgs.org/whole-game.html)
>
> 따라서 `usethis` 패키지가 익숙하지 않으시다면 위 링크에 있는 글을 먼저 읽고 따라해주세요!

---

이번 글에서는 1) \(p\) 차원에 대한 identity matrix 를 출력하는 함수와 2) 행렬 덧셈을 수행하는 함수를 포함한 R 패키지를 개발할 것이다.

위 과정을 단계별로 정리해보면 다음과 같다.
<ol>
  <li>C++ `Eigen` 라이브러리를 활용해서, 목표하는 행렬 연산을 수행하는 C++ 함수를 작성하고</li>
  <li>앞서 개발된 C++ 함수를 호출하는 R 함수를 작성한 다음</li>
  <li>이 소스코드들을 포함하는 패키지인 `regexcite` 를 빌드하고</li>
  <li>R을 사용하는 end-user 입장에서 `regexcite` 패키지를 설치하고, 행렬 연산 함수를 호출해보자!</li>
</ol>

## Step 0. RcppEigen을 사용하기 위한 세팅하기
 - 본격적인 시작에 앞서, `devtools` 라이브러리를 import 했는지 확인해야한다. 만약에 이를 빠뜨렸다면 `library(devtools)` 로 먼저 `devtools` 라이브러리를 불러오자.

 - 다음으로, 이번 패키지에서는 RcppEigen 을 사용하기 때문에 아래 구문을 실행하자.
   ```r
   use_rcpp_eigen()
   ```
   + 위 구문 실행 시, RcppEigen 실행을 위해서 필요한 부분이 자동으로 추가된다. (예: `src` 디렉토리 추가)

## Step 1. C++ `Eigen`으로 행렬 연산을 수행하는 함수 작성하기
 - C++ `Eigen`을 활용해서 다음과 같이 행렬 연산을 수행하는 함수를 작성하자.
   + `print_identity_matrix`는 \(p\)차원에 대한 identity matrix 를 출력하는 함수이고, `matrix_addition`은 행렬 덧셈을 수행하는 함수이다.
   
```cpp
#include <Rcpp.h>
#include <iostream>
#include <cmath>

//[[Rcpp::depends(RcppEigen)]]
#include <RcppEigen.h>

using namespace Rcpp;

// [[Rcpp::export]]
void print_identity_matrix(size_t p){
  Eigen::MatrixXd identityMatrix = Eigen::MatrixXd::Identity(p, p);

  std::cout << "Identity matrix with " << pow(p, 2) << " elements" << std::endl;
  std::cout << identityMatrix << std::endl;

}

// [[Rcpp::export]]
Eigen::MatrixXd matrix_addition(Eigen::MatrixXd A, Eigen::MatrixXd B){

  Eigen::MatrixXd rslt_mat = A+B;

  return rslt_mat;

}

```

## Step 2. Step 1의 C++ 함수를 호출하는 R 함수 작성하기
 - R 패키지 내 R 함수를 추가할 때는 `usethis` 패키지의 `use_r()` 함수를 사용해보자. 예를 들어, `show_identity_matrix` 라는 이름의 R 함수를 추가하고 싶다면 아래와 같이 R 콘솔에 입력하자.
     `use_r("show_identity_matrix")`
 - 이번 프로젝트에서는 한개의 기능을 한개의 함수에, 그리고 한개의 함수를 한개의 파일에 담아보자.
```r
# show_identity_matrix.R

#' Test Rcpp Eigen using the identity matrix
#'
#' @param p
#'
#' @return
#' @export
#'
#' @examples
#' show_identity_matrix(5)
show_identity_matrix <- function(p){
  print_identity_matrix(p)
}

```

```r
# test_eigen_matrix_additon.R

#' Test Rcpp Eigen using the matrix addition
#'
#' @param A
#' @param B
#'
#' @return A matrix
#' @export
#'
#' @examples
#' A <- matrix(seq(1,4), nrow=2)
#' B <- matrix(seq(5,8), nrow=2)
#' test_eigen_matrix_addition(A, B)
test_eigen_matrix_addition <- function(A, B){
  rslt_mat = matrix_addition(A,B)

  return(rslt_mat)

}

```
   > 💡 **R 함수를 선언할 때, 위 주석 부분을 꼭 지키자! 위 주석 부분이 있어야 Roxygen을 통해서 해당 함수가 `NAMESPACE`에 정상적으로 등록될 수 있고, 해당 함수에 대한 문서(`.Rd`)도 정상적으로 생성된다.**

## Step 3. R 패키지 새로 빌드하기
  
 - 이제 패키지가 잘 실행되는지 아래 구문으로 테스트해보자.
```r
load_all()
```
   + 위 코드 실행 결과, `src/`에 c++ 파일에 대한 object file(`*.o`)이 생성되었음을 확인할 수 있다.

 - 이제 RStudio 콘솔에서 위 함수를 불러보면..! 두둥...! **실패다..**
   + 에러 내용을 대략 살펴보면 R 함수 내부에서 c++ 함수를 호출 시에, 이를 찾을 수 없다는 내용이다. 이 에러를 잡기까지 고생을 꽤 많이 했는데, 문제의 해결책은 간단하였다. 바로 NAMESPACE 파일에 아래 내용을 포함하면 되는 것이었다.
```
importFrom(Rcpp,evalCpp)
useDynLib(regexcite, .registration=TRUE)
```

> 위 첫번째 줄의 코드는 Rcpp이 정상적으로 import 되지 않은 상황에서 Rcpp 함수가 호출되는 에러 상황을 방지한다.다음으로, 두번째 줄의 코드는 컴파일된 c++ 파일의 object 파일이 R 함수에서 호출될 수 있도록 연결(link)하는 역할을 담당하다.

- 하지만, 우리 패키지에서 NAMESPACE는 R 패키지를 개발하는 우리가 직접 수정할 수 없다. 따라서 Roxygen 을 활용해서 NAMESPACE에 위 내용을 추가해야한다. 이 과정은 아래와 같다.
   1. `R` 폴더 아래 `{패키지명}-package.R` 이라는 파일을 만들자. 우리 예제에서는 `regexcite-package.R` 라는 파일을 만들자
   2. 해당 파일에 아래 내용을 복붙하자.
```r
## usethis namespace: start
#' @importFrom Rcpp evalCpp
#' @useDynLib regexcite, .registration = TRUE
## usethis namespace: end
NULL
```
   - 패키지명이 다를 경우, `@useDynLib` 뒤에 패키지명을 자신이 정한 패키지명을 넣으면 된다.
   > 💡 `Rcpp` 외 다른 패키지를 import 하여 해당 패키지의 함수를 사용하는 경우에도 `@importFrom`에 해당 내용을 추가해야한다.
   >    - 예시: `stats` 패키지의 `runif` 함수를 호출하고 싶다면 `#' @importFrom stats runif` 를 위 파일에 추가해야한다.
   >    - 추가적으로, 특정 패키지에서 사용하려는 함수가 여러개라면 공백을 사이에 두고, 함수명을 나열하면 된다.
   >        + `#' @importFrom stats lm sd quantile`

   3. 그리고 `load_all()`을 다시 불러보면 `NAMESPACE` 파일에 우리가 추가하려는 내용이 포함되었음을 확인할 수 있다!
    - 혹시 `load_all()` 로 `NAMESPACE` 파일 업데이트가 안 된다면 R 콘솔에 `check()`를 입력하여, `check` 함수를 호출해서 확인할 수도 있다.
  
 - 결과적으로, `show_identity_matrix` 함수와 `test_eigen_matrix_addition` 함수가 정상적으로 출력됨을 확인할 수 있다!!
   + 원래 실행 결과 이미지가 있었지만, 이미지 크기 조정이 불가하여 삭제했습니다..ㅠㅠ

 - 이제 최종 단계로 R 패키지를 `.tar.gz` 파일로 빌드하자.
   + 빌드에 앞서 최종 R 프로젝트 형상을 확인해보면 아래와 같다.
     ```
    ├── .gitignore
    ├── .Rbuildignore
    ├── .Rhistory
    ├── DESCRIPTION
    ├── LICENSE
    ├── LICENSE.md
    ├── man
    │   ├── strsplit1.Rd
    │   ├── show_identity_matrix.Rd
    │   ├── test_eigen_matrix_addition.Rd
    ├── NAMESPACE
    ├── R
    │   ├── strsplit1.R
    │   ├── show_identity_matrix.R
    │   ├── test_eigen_matrix_addition.R
    │   ├── regexcite-package.R
    │   ├── RcppExports.R
    ├── regexcite.Rproj
    ├── src
    │   ├── .gitignore
    │   ├── test_eigen.cpp
    │   ├── RcppExports.cpp
     ```

   + (Mac OS 기준) 터미널에 `R CMD BUILD regexcite`라고 치면 `regexcite_{버전}.tar.gz` 라는 파일이 생성된다.
     * 이때, `R CMD build regexcite` 는 `regexcite` 프로젝트 디렉토리의 부모 디렉토리에서 실행해야한다는 점을 주의하자.

## Step 4. 직접 개발한 R 패키지를 설치하고, 함수를 호출해보자!
이제 R에서 `regexcite` 패키지를 설치하고, 행렬 연산 함수를 호출해보자!
 - 현재 환경에 `regexcite` 패키지가 설치되지 않았음을 먼저 확인해보자.
 !['regexcite'이라고 불리는 패키지가 없습니다](https://velog.velcdn.com/images/quriemoon/post/478f5ffc-6eb6-48f4-ab33-07909723386c/image.png)
   + 만약 이미 설치되었다면 아래와 같이 기존 `regexcite` 패키지를 삭제할 수 있다.
```r
remove.packages("regexcite")
```
 
 - 이제 R 콘솔에서 아래 구문을 실행하여, `regexcite` 패키지를 설치하자.
```r
 install.packages(
 "{regexcite 패키지 tar.gz 파일 경로}",
 repos=NULL,
 type="source"
 )
 ```
 - 최종적으로, 설치된 `regexcite` 패키지를 import하여, 목표 함수를 호출하는 것까지 성공하였다!
   > velog 에서 이미지 크기 조절이 불가하여, 실행 결과 캡쳐 화면이 조금 너저분하게 보이는 점 양해 부탁드립니다...
   > -> 혹시 velog 본문에서 이미지 크기 조절을 하는 방법을 아신다면 댓글로 공유 부탁드립니다 😭😭
 
 ![screenshot - success result of show_identity_matrix function](https://velog.velcdn.com/images/quriemoon/post/62908e76-dd4c-4f57-9b4b-0dea5409ce5e/image.png)
 ![screenshot - success result of test_eigen_matrix_addition function](https://velog.velcdn.com/images/quriemoon/post/968b201f-5be8-4720-ac76-83e04318aea2/image.png)

----------
피드백은 언제나 환영입니다! 혹시 잘못된 부분을 발견하셨거나 궁금하신 점이 있다면 댓글창에 남겨주세요! 그럼 다음 포스트에서 또 만나요!
