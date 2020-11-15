---
title: "Python 3 Tutorial 공부 (5. 자료 구조 - 튜플과 시퀀스)"
date: 2019-03-22
category: Python3
tags:
- Python3
- data_structure
- tuple

---

`이 포스트의 설명 및 코드는 Python official documentation을 참조했습니다.`

`url: https://docs.python.org/ko/3/tutorial/datastructures.html`



### 5.3  튜플과 시퀀스

Python에는 세 가지 기본 시퀀스 형이 있다: `list, tuple, range`

> 파이썬 공식 문서를 보면, 추후에 다른 시퀀스 자료형이 추가될 수도 있다고 한다.

기본적으로, 튜플은 쉼표로 구분되는 여러 값으로 구성된다.

```python
>>> t = 12345, 54321, 'hello!' #하나의 튜플 안에 서로 다른 자료형을 가진 원소들이 포함될 수 있다.
>>> t
(12345, 54321, 'hello!')
```

튜플은 중첩된(nested) 형태로도 사용될 수 있다.

```python
>>> u = t, (1,2,3,4,5)
>>> u
((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))
>>> u[0]
(12345, 54321, 'hello!')
>>> u[0][0]
12345
```

이때, 튜플은 수정될 수 없다(immutable)는 특징이 있다.

```python
>>> t
(12345, 54321, 'hello!')
>>> t[0] = 1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

추가적으로, 빈 튜플이랑 하나의 항목을 갖는 튜플을 만들어보자.

```python
>>> empty = ()
>>> singleton = 'hello', #쉼표에 주의하자
>>> len(empty)
0
>>> len(singleton)
1
>>> empty
()
>>> singleton
('hello',)
```



--------

추가적으로 튜플에서는 '패킹'과 '언패킹' 개념이 등장한다. 이에 대해서는 별개의 포스트에서 집중적으로(?) 살펴보자.