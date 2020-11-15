---
title: "Fluent Python - Chap 2. 시퀀스 (1)"
date: 2019-04-27
category: Python3
tags: 
- Python3 
- data_structure 
- listcomp genexpr 
- tuple
- tuple_unpacking 
- named_tuple 
- slicing

---

`이 포스트는 <<Fluent Python (전문가를 위한 파이썬) >> (written by Luciano Ramalho, 강권학 옮김) 스터디를 진행하면서 공부한 내용을 정리하는 목적으로 작성되었습니다. 따라서 이 포스트 내 설명 및 코드의 출처는 <<Fluent Python (전문가를 위한 파이썬) >> 임을 미리 밝힙니다.`



##  Chapter 2. 시퀀스

### 2.2 지능형 리스트 (list comp) 와 제너레이터 표현식 (genexpr)

아래 코드는 주어진 문자열의 문자에 해당하는 아스키 코드를 담고 있는 리스트를 새로 생성하고 있다.

```python
>>> new_str = "convert to ASCII"
>>> char_list = []
>>> for one_char in new_str:
...     char_list.append(ord(one_char))
...
>>> char_list
[99, 111, 110, 118, 101, 114, 116, 32, 116, 111, 32, 65, 83, 67, 73, 73]
```

위 코드를 지능형 리스트(listcomp)를 통해서 표현하면, 아래와 같이 쓸 수 있다.

```python
>>> codes = [ord(one_char) for one_char in new_str]
>>> codes
[99, 111, 110, 118, 101, 114, 116, 32, 116, 111, 32, 65, 83, 67, 73, 73]
```

위와 같은 지능형 리스트는 왜 사용해야할까? 그 이유는 바로 지능형 리스트의 가독성이 좋고 때로는 지능형 리스트를 통해서 실행 속도가 빠른 코드를 만들기 쉽기 때문이다. 또한, (Python3의) 지능형 리스트는 더 이상 메모리를 누수하지 않는다.

> 물론 지능형 리스트를 남용해서 이해가 정말 어려운 코드를 만들 수도 있다. 따라서 책에서는 다음과 같은 상황에서는 지능형 리스트를 사용하지 않도록 권한다: 생성된 리스트를 사용하지 않을 경우, 코드를 짧게 쓰기 어려운 경우, 지능형 리스트 구문이 두 줄 이상 넘어가는 경우

> 지능형 리스트는 **오로지 새로운 리스트를 만드는 일만 한다**는 것 주의하자.


위에서 살펴본 지능형 리스트를 이용하여 아래와 같이 튜플, 배열 등 리스트 이외의 시퀀스형을 초기화할 수 있다.

```python
>>> codes = [ord(one_char) for one_char in new_str]
>>> codes
[99, 111, 110, 118, 101, 114, 116, 32, 116, 111, 32, 65, 83, 67, 73, 73]
>>> type(codes)
<class 'list'>
>>> new_tuple = tuple(codes)
>>> new_tuple
(99, 111, 110, 118, 101, 114, 116, 32, 116, 111, 32, 65, 83, 67, 73, 73)
>>> type(new_tuple)
<class 'tuple'>
```

 하지만, 책에서는 방금 살펴본 코드처럼 다른 생성자에 전달할 리스트를 통째로 만드는 대신에 **반복자 프로토콜 (iterative protocol)**을 이용해서 항목을 하나씩 생성하는 제너레이터 표현식을 권한다. 데카르트 곱(Cartesian product)를 각각 지능형 리스트와 제너레이터 표현식을 이용해서 구해봄으로써 이에 대한 내용을 살펴보자.

 먼저, 지능형 리스트를 이용해서 Cartesian product를 구하면 아래와 같이 구할 수 있다.

```python
>>> weather=['sunny','cloudy','rainy','snowy']
>>> drinks=['coffee','coke']
>>> what_to_drink_today=[(weather, drink) for weather in weather
... for drink in drinks]
>>> what_to_drink_today
[('sunny', 'coffee'), ('sunny', 'coke'), ('cloudy', 'coffee'), ('cloudy', 'coke'), ('rainy', 'coffee'), ('rainy', 'coke'), ('snowy', 'coffee'), ('snowy', 'coke')]
```

 이때, 날씨와 음료수의 조합을 메모리에 유지할 필요 없는 데이터를 생성하고 싶다면, 아래와 같이 제너레이터 표현식을 사용하는 게 더 효율적이다.

```python
>>> for weather_drink in ('%s %s' % (weather, drink) for weather in weather for drink in drinks):
...     print(weather_drink)
...
sunny coffee
sunny coke
cloudy coffee
cloudy coke
rainy coffee
rainy coke
snowy coffee
snowy coke
```

 제너레이터 표현식은 이처럼 단순히 for 루프에 전달하기 위한 용도를 위해서만 리스트를 생성하는 일을 피할 수 있다.

### 2.3 튜플은 단순한 불변 리스트가 아니다

 튜플은 종종 리스트와 비교되어, '불변 리스트'로 소개된다. 하지만, 책에서는 튜플의 용도를 다음 2가지로 정리한다.

- 필드명 없는 레코드
- 불변 리스트

 각각에 대해서 살펴보자.

#### 2.3.1 튜플: 필드명 없는 레코드

 튜플을 필드명 없는 레코드를 쓴 예제는 다음과 같다.

```python
>>> scoreboard = [('Min-su','Math',20), ('Woo-ri','Korean',30),('Kyung-mi','English',25)]
>>> for score in sorted(scoreboard):
...     print('Name: {}, Subject: {}, Score: {}'.format(*score))
...
Name: Kyung-mi, Subject: English, Score: 25
Name: Min-su, Subject: Math, Score: 20
Name: Woo-ri, Subject: Korean, Score: 30
```

 위 코드에서 보이듯이 튜플의 각 항목은 레코드의 필드 하나를 의미한다. 위 예시에서 각 튜플의 첫번째 항목은 이름, 두번째 항목은 과목명, 세번째 항목은 점수를 의미한다. 이처럼 각 항목의 위치에 의해서 항목의 의미가 결정되기 때문에 각 항목의 위치/순서가 중요하다.

 뿐만 아니라 튜플은 **언패킹 메커니즘 (tuple unpacking/iterative unpacking)** 덕분에 레코드로써 기능을 더욱 잘 수행한다. 아래 예를 통해서 언패킹이 사용된 사례를 살펴보자.

```python
>>> # 예제 1
>>> basket = ('banana', 'apple')
>>> fruit_1, fruit_2 = basket
>>> fruit_1
'banana'
>>> fruit_2
'apple'
```

```python
>>> # 예제 2
>>> scoreboard = [('Min-su','Math',20), ('Woo-ri','Korean',30),('Kyung-mi','English',25)]
>>> for score in sorted(scoreboard):
...     print('Name: %s, Subject: %s, Score: %s' % score)
...
Name: Kyung-mi, Subject: English, Score: 25
Name: Min-su, Subject: Math, Score: 20
Name: Woo-ri, Subject: Korean, Score: 30
```

 이 기능이 특히 유용할 때는 바로 swap 기능을 사용할 때다!

```python
>>> a = 10
>>> b = 20
>>> a,b
(10, 20)
>>> a,b = b,a
>>> a,b
(20, 10)
```

 위에서 살펴보았듯이 튜플의 여러 항목들을 함수의 인자로 전달해줘야할 때, *를 활용하면 유용하다.

```python
>>> #*를 이용한 튜플 언패킹
>>> for score in sorted(scoreboard):
...     print('Name: {}, Subject: {}, Score: {}'.format(*score))
...
Name: Kyung-mi, Subject: English, Score: 25
Name: Min-su, Subject: Math, Score: 20
Name: Woo-ri, Subject: Korean, Score: 30
```

 *은 초과 항목을 잡기 위해서도 사용될 수 있다. 초과 항목을 잡는다는 표현은 함수 매개변수 전달 시, *를 이용해서 초과된 인수를 가져온다는 뜻인데, 파이썬 3에서는 이 개념을 확장하여 **병렬 할당** 상황에서도 이 기능을 이용한다.

> 병렬 할당: 반복형 데이터를 변수로 구성된 튜플에 할당하는 것

```python
>>> a,b,*rest = range(3)
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> type(rest)
<class 'list'>
>>> len(rest)
3
>>> rest
[2, 3, 4]
>>> a,b,*rest = range(3)
>>> a,b,rest
(0, 1, [2])
>>> a,b,*rest = range(2)
>>> a,b,rest
(0, 1, [])
```

 병렬 할당의 경우, *는 단 하나의 변수에만 적용할 수 있지만 어떠한 변수에도 적용 가능할 수 있다.

```python
>>> a, *b, *rest = range(5)
  File "<stdin>", line 1
SyntaxError: two starred expressions in assignment
  
>>> *a, b, rest = range(5)
>>> a,b,rest
([0, 1, 2], 3, 4)
>>> a,*b,rest = range(5)
>>> a,b,rest
(0, [1, 2, 3], 4)
>>> a,b,*rest = range(5)
>>> a,b,rest
(0, 1, [2, 3, 4])
```

 추가적으로, 튜플 언패킹은 내포된 구조, 즉 nested 형태에도 적용이 가능하다.

 이처럼 튜플은 아주 편리하나 때로는 필드에 이름을 붙일 필요가 있다. 그래서 고안된 함수가 collections.namedtuple() 함수로, 이를 통해 명명된 튜플을 이용할 수 있다.

```python
>>> from collections import namedtuple
>>> Student = namedtuple('Student', 'name student_id favorite_subject')
>>> Minsu = Student('Minsu', 23, 'Math')
>>> Minsu
Student(name='Minsu', student_id=23, favorite_subject='Math')
>>> Minsu.name
'Minsu'
```

 명명된 튜플은 튜플에서 상속받은 속성 외에 추가적인 몇 가지 속성들을 더 가지고 있다.

```python
>>> Student._fields
('name', 'student_id', 'favorite_subject')
>>> Student._fields #_fields: 클래스의 필드명을 담고 있는 튜플
('name', 'student_id', 'favorite_subject')
>>> mina_data = ('Mina', 25, 'English')
>>> Mina = Student._make(mina_data)
>>> Mina
Student(name='Mina', student_id=25, favorite_subject='English')
>>> Mina = Student(*mina_data) #_make() 함수를 이용한 것과 결과 동일
>>> Mina
Student(name='Mina', student_id=25, favorite_subject='English')
>>> Mina._asdict()
OrderedDict([('name', 'Mina'), ('student_id', 25), ('favorite_subject', 'English')])
>>> for key, value in Mina._asdict().items():
...     print(key+":",value)
...
name: Mina
student_id: 25
favorite_subject: English
```

 <추가 내용> 명명된 튜플의 장점

- 튜플과 마찬가지로, 객체당 오버헤드가 적다.
- 이름을 이용해서 필드에 간단히 접근할 수 있다.
- `_asdict()` 메서드를 사용하여 레코드를 `OrderedDict` 객체로 내보낼 수 있다(export).



#### 2.3.2 튜플: 불변 리스트

 튜플과 리스트에서 동시에 사용할 수 있는 기능과 튜플이 immutable하기 때문에 리스트에서는 가능하지만 튜플에서는 가능하지 않은 기능(예: append, insert, remove)을 구분해서 알자.

### 2.4 슬라이싱

 파이썬의 강력한 기능인 슬라이싱은 []  안에서 a: b: c 표기법을 통해서 이루어진다. 이렇게 이루어지는 **slice(a,b,c) 객체를 생성**한다. 슬라이스 객체를 알아두어야 하는 이유는 슬라이스 객체를 통해서 슬라이스에 이름을 붙일 수 있기 때문이다.

```python
>>> shopping_list = """
... 0....5...................25...29.....
... 001  banana                  2   $4.9
... 002  kiwi                    4    $12
... 003  apple                   8  $20.2
... """
>>> id_num = slice(0,5) #shopping_list.split('\n')[1] 을 통해서 슬라이싱 때, 나누어야할 지점의 인덱스를 알 수 있다.
>>> item_name = slice(5,25)
>>> quantity = slice(25,29)
>>> price = slice(29, None)
>>> line_items = shopping_list.split('\n')[2:]
>>> for item in line_items:
...     print(item[item_name], item[quantity], item[price])
...
banana                    2   $4.9
kiwi                      4    $12
apple                     8  $20.2
```

 또한, 슬라이스에 할당하는 것 역시 가능하다.

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20,30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> l[2:5] = [100,200,300]
>>> l
[0, 1, 100, 200, 300, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 100, 200, 300, 8, 9]
>>> l[3::2] = [11,22]
>>> l
[0, 1, 100, 11, 300, 22, 9]
```

 이때, 위처럼 슬라이스에 할당할 경우, 항목 하나만 할당하는 경우에도 할당문 오른쪽에 **반복 가능한 객체가 와야 한다.**

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 5, 6, 7, 8, 9]
```



### 2.5 시퀀스에 덧셈과 곱셈 연산자 사용하기

 시퀀스에 덧셈과 곱셈 연산자를 적용하여 새로운 시퀀스를 만들 수 있다.

```python
>>> l = [1,2,3]
>>> l * 5
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
>>> 5 * 'abcd'
'abcdabcdabcdabcdabcd'
```

 위 `l * 5` 와 같은 표현식을 이용해서 nested list를 만든 후, 이 nested list를 이용하여 리스트를 초기화할 수 있다. 이때, **동일 리스트를 여러 번 참조하지 않도록 주의**해야한다. 아래 코드를 보면 동일한 리스트를 (의도치 않게) 여러 번 참조했을 때의 문제점을 볼 수 있다.

```python
>>> weird_board = [['_'] * 3] * 3
>>> weird_board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> weird_board[1][2] = 'X'
>>> weird_board
[['_', '_', 'X'], ['_', '_', 'X'], ['_', '_', 'X']]
```

 사용자가 의도했던 결과는  `[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]` 인데, 위 결과는 그렇지 않다. 위 코드를 사용자가 의도한 결과가 나오도록 고치면 아래와 같다.

```python
>>> board = [['_'] * 3 for i in range(3)]
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = 'X'
>>> board
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]
```

 왜 두 코드의 실행 결과가 다른 걸까? 위 두 코드는 아래와 같이 풀어서 쓸 수 있다.

```python
>>> # weird_board 사례
>>> row = ['_'] * 3
>>> board = []
>>> for i in range(3):
...     board.append(row) # 동일한 행이 board 리스트에 세 번 추가된다
...
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = 'X'
>>> board
[['_', '_', 'X'], ['_', '_', 'X'], ['_', '_', 'X']]
```

```python
>>> board = []
>>> for i in range(3):
...     row = ['_'] * 3 # 반복할 때마다 row 객체를 새로 만들어서
...     board.append(row)  # board에 추가한다
...
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = 'X'
>>> board
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]
```



 다음 포스트에서 시퀀스의 복합 할당 ( `+=`, `*=` ) 부터 이어서 보자.