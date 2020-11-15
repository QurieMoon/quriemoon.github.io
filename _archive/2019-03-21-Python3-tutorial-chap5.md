---
title: "Python 3 Tutorial 공부 (5. 자료 구조 - 리스트)"
date: 2019-03-21
category: Python3
tags:
- Python3
- data_structure list
---

`이 포스트의 설명 및 코드는 Python official documentation을 참조했습니다.`

`url: https://docs.python.org/ko/3/tutorial/datastructures.html`



### 5.1  리스트

우선, 리스트의 메서드 관련해서 살펴보자.

* sorted() vs. list.sort(*key=None*, *reverse=False*)

  ```python
  >>> fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  >>> sorted(fruits)
  ['apple', 'apple', 'banana', 'banana', 'kiwi', 'orange', 'pear']
  >>> fruits
  ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  ```

  위 코드에서 보이듯이 sorted() 함수를 사용 시, 정렬된 리스트를 확인할 수 있지만 **리스트 자체가 변하지 않는다.**

  반면에

  ```python
  >>> fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  >>> fruits.sort()
  >>> fruits
  ['apple', 'apple', 'banana', 'banana', 'kiwi', 'orange', 'pear']
  ```

  리스트의 method인 sort() 이용 시, 해당 리스트 자체가 정렬된다.

  > reversed(), list.reverse() 도 위와 같다. 즉, reversed() 를 이용하면, 순서가 뒤집어진 리스트를 확인할 수 있지만 원래 리스트 자체가 변화하지 않는다. 반면에 list.reverse()를 사용 시, 원래 리스트 자체에 변화가 있지만 None을 return 받는다.

  > insert(), remove(), sort()와 같은 메서드들은 리스트 자체를 수정하고 None을 return한다.

* list.index(*x*[, *start*[, *end*]])

  ```python
  >>> fruits = ['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange']
  >>> fruits.index('apple')
  1
  >>> fruits.index('beet')
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ValueError: 'beet' is not in list
  
  ```

  fruits.index('apple') 입력 시, 'apple'이라는 항목이 fruits라는 리스트 안에서 몇 번째 위치해있는지, 즉 해당 인덱스를 반환(return)해준다.

  > Python에서도 리스트의 인덱스는 0부터 시작한다.

  이때, 해당 항목이 리스트 안에 존재하지 않으면 **ValueError**가 발생한다.

  

  또한, Python 공식 문서에서 index() 메소드를 보면 다음과 같이 추가적인 인자(*start, end*)를 줄 수 있다.

  `list.index(x[, start[, end]])`

  > 위 메서드 설명에 등장하는 [] 표기법의 경우, 파이썬 라이브러리 레퍼런스에 자주 등장하는 표기법이다. 
  >
  > 메서드 설명 중 [, *i* ]와 같은 식이 등장했다면 이는 해당 매개변수 *i* 가 선택적임을 뜻한다. 즉, 그 위치에 []를 입력해야 한다는 뜻이 아니라 해당 매개변수 *i* 를 선택적으로 사용하라는 뜻이다.

  이때, *start*와 *end*는 슬라이스 표기법처럼 해석된다. 따라서 index() 메소드를 통해 검색을 수행할 때, 전체 리스트가 아닌 *start*와 *end*를 통해서 제한된 서브 시퀀스 내에서 검색을 수행한다.

  ```python
  >>> fruits = ['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange']
  >>> fruits.index('apple',2,3)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ValueError: 'apple' is not in list
  >>> fruits.index('apple',2)
  5
  ```

  위 코드를 보면, 'apple'이 fruits 리스트 안에 존재한다.

  하지만, `fruits.index('apple',2,3)` 의 경우, fruits라는 전체 리스트가 아닌 `fruits[2:3]` 를 통해 슬라이싱된 서브 리스트에서 'apple'을 검색하는 상황과 같다.

  이때, `fruits[2:3]` 은 `['kiwi']`와 같기 때문에 'apple'이 검색하려는 리스트 내에 존재하지 않는 상황과 같다. 따라서 **ValueError**가 발생한다.

  비슷하게 생각해보면, `fruits.index('apple',2)` 의 경우, `fruits[2:]` 로 슬라이싱된 서브 리스트 내에서 이루어지는 검색으로 이해할 수 있다.

* list.pop([*i*])

  리스트의 항목 중에서 인덱스가 *i* 인 항목을 삭제하고, 해당 항목을 반환한다. 이때, *i* 를 입력하지 않을 경우, default로 마지막 항목을 삭제하고 돌려준다.

  ```python
  >>> fruits
  ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  >>> fruits.pop(2)
  'pear'
  >>> fruits
  ['orange', 'apple', 'banana', 'kiwi', 'apple', 'banana']
  >>> fruits.pop()
  'banana'
  >>> fruits
  ['orange', 'apple', 'banana', 'kiwi', 'apple']
  ```

  

* list.copy()

  list.copy()는 리스트의 얕은 사본을 돌려준다. a[:]와 동등하다.

  ```python
  >>> fruits
  ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  >>> fruits.copy()
  ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  >>> fruits[:]
  ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
  ```



다음으로, 리스트의 활용 방안을 생각해보자.

​	1) 리스트를 스택으로 사용하기

​	2) 리스트를 큐로 사용하기

> 이때, 리스트로 큐를 구현하는 것은 효율적이지 않다.
>
> 왜냐하면 리스트의 머리(head)에 덧붙이거나 꺼내기 위해서는 다른 요소들을 모두 한 칸씩 이동시켜야하기 때문에 느리다. 따라서 이때는 **collections.deque**를 이용하자.



- 리스트 컴프리헨션

  리스트를 만드는 간결한 방법

  ```python
  >>> squares = [x**2 for x in range(10)]
  >>> squares
  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  ```

  위 내용을 아래와 같은 코드로도 표현할 수 있다.

  ```python
  >>> squares = list(map(lambda x:x**2, range(10)))
  >>> squares
  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  ```

  하지만, 책 "Fluent Python"을 보면 저자는 위와 같은 상황에서 map()과 lambda를 사용하는 것보다 기존 리스트 컴프리헨션 방식을 권한다.

  **추가 예제**

  ```python
  >>> [(x,y) for x in [1,2,3] for y in [3,1,4] if x!= y]
  [(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
  ```

  ```python
  #example using tuple
  #the tuple must be parenthesized
  >>> [(x, x**2) for x in range(6)]
  [(0, 0), (1, 1), (2, 4), (3, 9), (4, 16), (5, 25)]
  >>> [x, x**2 for x in range(6)]
    File "<stdin>", line 1
      [x, x**2 for x in range(6)]
                 ^
  SyntaxError: invalid syntax
  ```

  ```python
  # flatten a list using a listcomp with two 'for'
  >>> vec = [[1,2,3],[4,5,6],[7,8,9]]
  >>> vec
  [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
  >>> [num for elem in vec for num in elem]
  [1, 2, 3, 4, 5, 6, 7, 8, 9]
  #order is important
  >>> [num for num in elem for elem in vec]
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  NameError: name 'elem' is not defined
  ```

  ```python
  >>> from math import pi
  >>> [str(round(pi, i)) for i in range(1,6)]
  ['3.1', '3.14', '3.142', '3.1416', '3.14159']
  ```



### 5.2 del 문

del문은 리스트에서 값 대신 인덱스로 특정 요소를 삭제하게 해준다.

```python
>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> del a[0]
>>> a
[1, 66.25, 333, 333, 1234.5]
>>> del a[2:4]
>>> a
```

또한, 리스트에서 슬라이스를 삭제하거나 전체 리스트를 비우는 데도 사용할 수 있다.

```python
>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> a
[-1, 1, 66.25, 333, 333, 1234.5]
>>> del a[:]
>>> a
[]
```

이때, del 문은 변수 자체를 삭제하는데도 사용될 수 있다. 하지만, `del a` 직후에 a를 참조하면 에러가 발생한다.

```python
>>> a = [-1, 1, 66.25, 333, 333, 1234.5]
>>> a
[-1, 1, 66.25, 333, 333, 1234.5]
>>> del a
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
```



다음 포스트에서 또다른 기본 시퀀스 형인 튜플에 대해서 알아보자.

