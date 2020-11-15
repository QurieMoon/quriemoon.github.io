---
title: "Python 3 Tutorial 공부 (5. 자료 구조 - 딕셔너리)"
date: 2019-03-27
category: Python3
tags:
- Python3
- data_structure
- dictionary
---

`이 이 포스트의 설명 및 코드는 Python official documentation을 참조했습니다.`

`url: https://docs.python.org/ko/3/tutorial/datastructures.html`



### 5.5  딕셔너리

딕셔너리를 자세히 살펴보기 위해서는 아래 링크를 보자.

https://docs.python.org/ko/3/tutorial/datastructures.html#dictionaries

이 본문에서는 일단 파이썬 공식문서에 등장한 내용부터 다루려고 한다.

-----------------

하나의 딕셔너리는...

- 키(key)로 인덱싱이 되고

- 이때, 키(key)들은 서로 중복되지 않는다라는 제약 조건을 지키며

- 키(key):값(value) 로 된 쌍(pair)의 집합

  이다.

또한, 집합(set)에서 언급되었지만 `{}` 은 빈 딕셔너리를 만들고, `{}` 안에 키:값 쌍들의 목록을 넣는 방식으로 딕셔너리를 초기화할 수 있다.

```python
>>> empty_dict = {}
>>> empty_dict
{}
>>> type(empty_dict)
<class 'dict'>
>>> len(empty_dict)
0
>>> new_dict = {'key1':1,'key2':2,'This_is_also_a_key':5}
>>> new_dict
{'key1': 1, 'key2': 2, 'This_is_also_a_key': 5}
>>> type(new_dict)
<class 'dict'>
>>> len(new_dict)
3
```

위에 첨부한 링크를 따라가면, 딕셔너리가 지원하는 연산들에 대해서 자세히 알 수 있다. 이때, 딕셔너리의 주 연산을 한줄로 요약한다면, **값에 대한 연산이 키를 이용하여 이루어진다** 는 것이다.

```python
>>> new_dict
{'key1': 1, 'key2': 2, 'This_is_also_a_key': 5}
>>> new_dict['key1'] #key를 이용하여 딕셔너리 내 value에 접근
1
>>> new_dict
{'key1': 1, 'key2': 2, 'This_is_also_a_key': 5}
>>> del new_dict['key1'] #del도 key를 이용
>>> new_dict
{'key2': 2, 'This_is_also_a_key': 5}
```

새로운 원소를 추가하는 것 역시 key를 중심으로 할 수 있다.

```python
>>> new_dict
{'key1': 1, 'key2': 2, 'This_is_also_a_key': 5}
>>> new_dict['new_key'] = 329020
>>> new_dict
{'key1': 1, 'key2': 2, 'This_is_also_a_key': 5, 'new_key': 329020}
```

추가적으로, 키에 대한 내용을 좀 더 살펴보면,

1) `list(d)` 는 딕셔너리에서 사용되고 있는 **모든 키의 리스트**를 **삽입 순서대로** 돌려준다.

​    정렬된 결과를 돌려받고 싶을 때는 `sorted(d)` 를 이용하면 된다.

```python
>>> tel = {'jack': 4098, 'sape': 4139}
>>> tel['guido'] = 4127
>>> tel
{'jack': 4098, 'sape': 4139, 'guido': 4127}
>>> del tel['sape']
>>> tel['irv'] = 4127
>>> tel
{'jack': 4098, 'guido': 4127, 'irv': 4127}
>>> list(tel)
['jack', 'guido', 'irv']
>>> sorted(tel)
['guido', 'irv', 'jack']
```

2) 하나의 키가 딕셔너리에 있는지 검사하려면, `in` 키워드를 사용한다.

```python
>>> 'guido' in tel
True
>>> 'jact' not in tel
True
>>> 'jack' not in tel
False
```

 다음으로, 딕셔너리를 생성하는 추가적인 방법을 알아보자.

- `dict()` 이용

  - 딕셔너리 생성 시, `dict()` 생성자를 사용하여 키-값 쌍들의 <u>시퀀스 (예: 리스트)</u>로부터 직접 딕셔너리를 만들 수도 있다.

  ```python
  >>> dict([('sape',4139),('guido',4127),('jack',4098)]) #(key, value)
  {'sape': 4139, 'guido': 4127, 'jack': 4098}
  ```

- 딕셔너리 컴프리헨션

  ```python
  >>> dict_1 = {x: x**2 for x in (2,4,6)}
  >>> dict_1
  {2: 4, 4: 16, 6: 36}
  >>> dict_1[6] #6 is the key for the value 36
  36
  ```

- 키가 간단한 문자열일 때, 키워드 인자를 이용해서 쌍을 만들 수 있다.

  ```python
  >>> sape
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  NameError: name 'sape' is not defined
  >>> dict(sape=4139, guido=4127, jack=4098) #sape, guido, jack 모두 '' 표시 없는데도 문자열로 인식됨
  {'sape': 4139, 'guido': 4127, 'jack': 4098}
  >>> sape
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  NameError: name 'sape' is not defined
  ```



### 5.6  루프 테크닉

시퀀스를 <u>*루핑*</u>할 때, 즉, <u>*반복문을 돌 때*,</u> 다음 세가지 함수를 이용하자: `items()` , `enumerate()` , `zip()`

- `items()`

  - `items()` 메서드를 사용하면 키와 해당 키에 대응하는 값을 동시에 얻을 수 있다.

    ```python
    >>> match_the_fruit = {'red':'apple','yellow':'banana','green':'kiwi'}
    >>> match_the_fruit
    {'red': 'apple', 'yellow': 'banana', 'green': 'kiwi'}
    >>> type(match_the_fruit)
    <class 'dict'>
    >>> for color, fruit_name in match_the_fruit.items():
    ...     print(color, fruit_name)
    ...
    red apple
    yellow banana
    green kiwi
    ```

- `enumerate()`

  - `enumerate()` 함수를 사용하면 위치 인덱스와 해당 인덱스에 대응하는 값을 동시에 얻을 수 있다.

    ```python
    >>> for i, v in enumerate(['tic','tac','toe']):
    ...     print(i,v)
    ...
    0 tic
    1 tac
    2 toe
    ```

- `zip()`

  - `zip()` 함수는 둘 혹은 그 이상의 시퀀스를 동시에 루핑할 때, 사용된다.

  - `zip()` 함수로 엔트리들의 쌍을 만들 수 있다.

    ```python
    >>> questions = ['name', 'favorite food', 'favorite color']
    >>> answers = ['lancelot', 'fried potatoes', 'blue']
    >>> for q, a in zip(questions, answers):
    ...     print('What is your {0}? It is {1}.'.format(q,a))
    ...
    What is your name? It is lancelot.
    What is your favorite food? It is fried potatoes.
    What is your favorite color? It is blue.
    ```

  추가적으로, 시퀀스를 거꾸로 루핑할 때는 `reversed()` 함수를, 정렬된 순서로 시퀀스를 루핑할 때는 `sorted()`   함수를 이용한다.

  그리고 루프를 돌고 있는 리스트를 변경하고 싶을 때는 이 자체를 변경하기보다 새 리스트를 만드는 게 종종 더 간단하고 **안전하다.** 아래 예제를 보면, 루프를 돌고 있는 리스트를 변경해서 **무한 루프**가 발생한다.

  ```python
  >>> #아래 코드 시, 무한 루프가 발생하니 되도록 실행을 권하지 않으며 혹시 실행 시 주의!!
  >>> import math
  >>> raw_data = [56.2, float('NaN'), 51.7, 55.3, 52.5, float('NaN'), 47.8]
  >>>
  >>> for value in raw_data:
  ...     if not math.isnan(value):
  ...             raw_data.append(value)
  ```

  위 코드를 보면, `for` 문 안에서 `value`가  `NaN` 이 아닐 때마다 해당 값을 `raw_data` 에 추가하는데, 이때, `NaN` 이 아닌 값이 계속해서 추가되기 때문에 무한 루프가 발생한다. 위 코드를 아래와 같이 수정해보자.

  ```python
  >>> import math
  >>> raw_data = [56.2, float('NaN'), 51.7, 55.3, 52.5, float('NaN'), 47.8]
  >>> filtered_data = []
  >>> for value in raw_data:
  ...     if not math.isnan(value):
  ...             print(value)
  ...             filtered_data.append(value)
  ...
  56.2
  51.7
  55.3
  52.5
  47.8
  >>> raw_data
  [56.2, nan, 51.7, 55.3, 52.5, nan, 47.8]
  >>> filtered_data
  [56.2, 51.7, 55.3, 52.5, 47.8]
  ```

  위 코드에서는 `filtered_data` 라는 새로운 리스트를 사용함으로써 사용자가 의도했던 `NaN` 이 아닌 값만 추출하려는 목적도 충족하고, 무한 루프도 발생하지 않는다.

### 5.7  조건 더 보기

​	이 부분에서는 아래 두 내용만 짚고 넘어가자.

- 논리 연산자 `and` 와 `or` 는 *단락-회로(short-circuit)* 연산자이다. 이 두 논리 연산자를 사용할 경우, 인자들은 왼쪽에서 오른쪽으로 값이 구해지고, **결과가 결정되자마자 값 구하기는 중단된다.** 즉, `A and B and C` 라는 식에서 `A` 와 `C` 가 `True` , `B` 가 `False` 이면 첫번째 `and` 뒤에 있는 `B` 에 의해서 이 식의 값이 `False` 로 정해지기 때문에 마지막 `C` 는 계산되지 않는다.

- 비교의 결과나 다른 논리 표현식의 결과를 변수에 대입할 수 있다.

  ```python
  >>> string1, string2, string3 = '', 'Trondheim', 'Hammer Dance'
  >>> non_null = string1 or string2 or string3
  >>> non_null #'Hammer Dance'는 출력되지 않는 것에 주의!
  'Trondheim'
  >>> result = (3+4==7) and (2+3==5)
  >>> result
  True
  ```

- 파이썬에서는 표현식 안에서는 **대입 연산자**가 등장할 수 없다.

### 5.8  시퀀스와 다른 형들을 비교하기





