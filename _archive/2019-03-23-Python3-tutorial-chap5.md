---
title: "Python 3 Tutorial 공부 (5. 자료 구조 - 집합)"
date: 2019-03-23
category: Python3
tags:
- Python3 
- data_structure 
- set

---

`이 포스트의 설명 및 코드는 Python official documentation을 참조했습니다.`

`url: https://docs.python.org/ko/3/tutorial/datastructures.html`



### 5.4  집합

일단, 집합은 **중복되는 요소가 없는** 그리고 **순서가 없는** 컬렉션이다. 따라서 집합의 기본적인 용도는 멤버십 검사와 중복 엔트리 제거이다.

집합을 만들 때는 중괄호나 `set()` 함수를 사용할 수 있다.

```python
>>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> print(basket) #basket is a set
{'apple', 'orange', 'banana', 'pear'}
>>> a= set('abracadabra')
>>> a
{'r', 'd', 'b', 'c', 'a'}
>>> b=set('alacazam')
>>> b
{'l', 'm', 'z', 'c', 'a'}
```

이때, 주의할 점이 있다. 바로 빈 집합을 만들려면 `{}` 이 아닌 `set()` 을 사용한다. `{}` 은 빈 집합이 아닌 **빈 딕셔너리** 를 만든다.

```python
>>> empty_dict = {}
>>> type(empty_dict)
<class 'dict'>
>>> empty_set = set()
>>> type(empty_set)
<class 'set'>
```

특정 요소가 해당 집합에 포함되어 있는지 아닌지를 확인하기 위해서 `in` 을 사용할 수 있다.

```python
>>> 'orange' in basket
True
>>> 'crabgrass' in basket
False
```

기존 집합 연산 역시 지원된다.

```python
>>> a= set('abracadabra')
>>> a
{'r', 'd', 'b', 'c', 'a'}
>>> b=set('alacazam')
>>> b
{'l', 'm', 'z', 'c', 'a'}
>>> a-b #letters in a but not in b
{'r', 'd', 'b'}
>>> a|b #letters in a or b or both #합집합
{'l', 'r', 'd', 'b', 'm', 'z', 'c', 'a'}
>>> a&b #letters in both a and b #교집합
{'a', 'c'}
>>> a^b #letters in a or b but not both #Exclusive OR
{'l', 'b', 'm', 'z', 'r', 'd'}
```

또한, 리스트 컴프리헨션처럼 **집합 컴프리헨션**도 지원다.

```python
>>> a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'r', 'd'}
```

----------------------------

다음 포스트에서 딕셔너리에 대해서도 알아보자.