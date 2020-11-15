---
title: "Fluent Python - Chap 3. 딕셔너리와 집합 (1)"
date: 2019-06-08
category: Python3
tags: 
- Python3 
- dict
- mapping
- hash_table
- dictcomp
- hashable
- duck_typing
- setdefault

---

`이 포스트는 <<Fluent Python (전문가를 위한 파이썬) >> (written by Luciano Ramalho, 강권학 옮김) 스터디를 진행하면서 공부한 내용을 정리하는 목적으로 작성되었습니다. 따라서 이 포스트 내 설명 및 코드의 출처는 <<Fluent Python (전문가를 위한 파이썬) >> 임을 미리 밝힙니다.`



## Chapter 3. 딕셔너리와 집합

 파이썬에서 딕셔너리( `dict` 형)은 애플리케이션에서 뿐만 아니라 파이썬 구현 부분에서도 중요한 역할을 한다. 따라서 파이썬 `dict` 클래스는 상당히 최적화되어 있는데, 이러한 최적화 뒤에는 **해시 테이블** 이 있다. 이 해시 테이블을 이용해서 **딕셔너리**뿐만 아니라 **집합** 도 구현된다. 해시 테이블의 작동 방식 및 의미에 대해서는 뒤에서 더 알아보자.

### 3.1 일반적인 매핑형

 딕셔너리 챕터에서 매핑형이 등장하는 이유는 파이썬의 컨테이너 추상 클래스인 `collections.abc` 모듈에서 **`dict` 및 이와 비슷한 자료형의 인터페이스를 정의하기 위해 `Mapping` 및 `MutableMapping` 추상 베이스 클래스를 제공하기 때문**이다. 이러한 추상 베이스 클래스는 **매핑이 제공해야하는 최소한의 인터페이스를 정의**한다.

 책을 보면, 표준 라이브러리에서 제공하는 매핑형은 모두 `dict` 를 이용해서 구현하기 때문에 값은 해시 가능할 필요 없지만, **키가 해시 가능해야한다는 조건**이 있다. 그렇다면 **해시 가능하다**는 말은 무슨 말일까? 책의 설명에 따르면, 아래와 같이 정의된다.

> 주어진 **수명 주기 (lifetime)** 동안 **결코 변하지 않는 해시값을 갖고 있고** ( `__hash__()` 메서드가 필요하다) 다른 객체와 **비교할 수 있으면** (`__eq__()` 메서드가 필요하다), **객체를 해시 가능하다**고 한다. 또한, **동일**하다고 판단되는 **객체**는 반드시 **해시값이 같아야한다.**

이에 따르면 다음 type을 가진 객체는 해시 가능하다.

- 원자적 불변형 (atomic immutable types): `str` , `byte` , `int` 를 비롯한 수치형

- `frozenset`

- `tuple` —> 튜플은 **원소들이 모두 해시 가능할 때만 해시 가능하다.**

  ```python
  >>> t1 = (1,2,3)
  >>> hash(t1) # hashable
  2528502973977326415
  >>> t2 = (1, 2, (30, 40))
  >>> hash(t2) # hashable
  8027212646858338501
  >>> t3 = (1, 2, [30, 40]) # list --> mutable object
  >>> hash(t3) # unhashable
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: unhashable type: 'list'
  >>> t4 = (1, 2, frozenset([30,40]))
  >>> hash(t4) # hashable
  985328935373711578
  ```

* 사용자 정의 자료형
  * 사용자 정의 자료형의 경우, 해쉬값이 자신의 `id()` 값이기 때문에 서로 다르다. 따라서 사용자 정의 자료형은 기본적으로 해시 가능하다. 만약에 사용자가 직접 `__eq__` 를 구현해서 사용하는 경우에는, 해시값 계산에 사용되는 속성이 불변(immutable)형 일때만 해당 객체가 해쉬 가능하다.

 다시 본문으로 돌아와서, 결국 **키가 해시 가능하다**는 말은, 키로 사용되는 객체가 선언되어 수명 주기 동안에 변하지 않는 해시값을 가지고 있고, `__eq__()` 메서드로 다른 객체와 비교할 수 있으면 된다.

 위를 참고하여, **해시 가능한 키값**이라는 조건만 유의하여 아래와 같이 다양하게 딕셔너리를 구현할 수 있다.

```python
>>> a = dict(one=1, two=2, three=3) # one에 '' 없는 것 주의!
>>> b = {'one':1, 'two':2, 'three':3}
>>> c = dict(zip(['one', 'two', 'three'], [1,2,3]))
>>> d = dict([('two', 2), ('one', 1), ('three', 3)])
>>> e = dict({'three':3, 'one':1, 'two':2})
>>>
>>> a == b == c == d == e
True
>>> a
{'one': 1, 'two': 2, 'three': 3}
```

 다음 절에서 **지능형 딕셔너리** (dict comprehensions)를 이용해서 딕셔너리 객체를 만드는 법을 알아보자.

### 3.2 지능형 딕셔너리

 시퀀스 형의 하나인 리스트 부분에서 등장했던 지능형 리스트(list_comp)와 비슷하게, 딕셔너리에 대해서도 **지능형 딕셔너리(dictcomp)** 가 있다. 지능형 딕셔너리는 **모든 반복형 객체**에서 키-값 쌍(pair)을 생성함으로써 딕셔너리 객체를 만든다.

```python
>>> DIAL_CODES = [
... (11, 'Seoul'),
... (21, 'Busan'),
... (22, 'Daegu'),
... (23, 'Incheon'),
... (24, 'Gwangju'),
... ]
>>> city_code = {city: code for code, city in DIAL_CODES}
>>> city_code
{'Seoul': 11, 'Busan': 21, 'Daegu': 22, 'Incheon': 23, 'Gwangju': 24}
>>> type(city_code)
<class 'dict'>
>>> {code: city.upper() for city, code in city_code.items() if code>21}
{22: 'DAEGU', 23: 'INCHEON', 24: 'GWANGJU'}
```



### 3.3 공통적인 매핑 메서드

 책에 나와있는 `dict`, `collections.defaultdict`, `collections.OrderedDict` 매핑형의 메서드 정리 부분을 보다보면, `update()` 메서드 설명 중에서 **덕 타이핑(duck typing)** 이라는 표현을 볼 수 있다.

 덕 타이핑(duck typing)이 무엇일까? 덕 타이핑을 아래 문장으로 요약할 수 있다.

> 만약 오리처럼 걷고, 오리처럼 꽥꽥 거린다면, 그것은 오리라고 할 것이다.

 즉, 사람이 오리처럼 행동한다면 그 사람을 **오리로 봐도 무방하다**는 게 덕 타이핑이다. 좀 더 메서드 관점에서 설명해보면, 타입을 미리 정해두는 것이 아니라 **실행이 되었을 때, 해당 메서드를 확인 후, 타입을 정한다.** 아래 예제 코드를 보자.

```python
class Duck:
        def quack(self): print("꽥꽥!")
        def feathers(self): print("오리에게 흰색, 회색 깃털이 있습니다.")

class Person:
        def quack(self): print("이 사람이 오리를 흉내내네요.")
        def feathers(self): print("사람은 바닥에서 깃털을 주어서 보여 줍니다.")

def in_the_forest(duck):
        duck.quack()
        duck.feathers()

def game():
        donald = Duck()
        john = Person()
        in_the_forest(donald)
        in_the_forest(john)

if __name__ == "__main__":
    game()
# 실행 결과
# 꽥꽥!
# 오리에게 흰색, 회색 깃털이 있습니다.
# 이 사람이 오리를 흉내내네요.
# 사람은 바닥에서 깃털을 주어서 보여 줍니다.
```

> 코드 출처: [덕 타이핑(위키피디아)]([https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91](https://ko.wikipedia.org/wiki/덕_타이핑))

 위 예제 코드에서 덕 타이핑이 적용되지 않았다면 `duck` (오리) 라는 타입의 객체를 인자로 받아 해당 객체의 `quack` (꽥꽥거리기) 함수와 `feathers` 함수를 호출했을 것이다. 하지만, 위 예제에는 덕 타이핑 기법이 적용되었기 때문에 인자로 받은 객체가 `quack` 메소드와 `feathers` 메소드를 갖고 있다면, **해당 객체를 `duck` (오리) 타입으로 간주한다.**

 이렇듯 덕 타이핑은 런타임 데이터를 기반으로, 타입을 판단하기 때문에 타입에 대해 매우 자유롭다. 하지만, 미리 예상하기 어려운 런타임 자료형 오류가 발생할 수도 있다는 단점이 있다.

> 덕 타이핑 장단점 설명 출처: [덕 타이핑(Duck Typing)이란?](https://nesoy.github.io/articles/2018-02/Duck-Typing)

 이러한 덕 타이핑 기법 덕분에 `d.update(m, [**kargs])` 에서 `m` 을 유연하게 처리할 수 있다. 구체적으로, `m` 이 **`keys()` 메서드를 가지고 있으면 매핑으로 간주하고,** **`keys()` 메서드를 가지고 있지 않더라도 `m` 의 항목들이 (키, 값) 쌍으로 되어 있다고 간주**하고 `m` 을 반복하여 처리한다. 대부분의 파이썬 매핑이 이처럼 1) 다른 매핑으로 초기화하거나 2) (키, 값) 쌍을 생성할 수 있는 반복행 객체로 초기화 (매핑형으로 **간주**)하여 초기화할 수 있다.

 다음으로 `setdefault()` 메서드를 알아보자.

### 3.3.1 존재하지 않는 키를 `setdefault()` 로 처리하기

 아래 코드를 실행시켜 보자.

```python
>>> dict_1={'Minsu':1, 'Okdal':2, 'Cheeze':3}
>>> dict_1['Okdal']
2
>>> dict_1['new_key']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'new_key'
```

 위처럼 파이썬에서 `dict` 에서 존재하지 않는 ` key`를 호출 하면, `KeyError`  가 발생한다. 이때, `KeyError`를 발생시키지 않고, 기본값(`default-value`)을 주어서 처리해보자.

```python
>>> dict_1.get('Okdal')
2
>>> dict_1.get('Okdal','default-value')
2
>>> dict_1.get('new_key','default-value')
'default-value'
```

 이처럼 `d[k]` 대신에 `d.get(k, default)` 함수를 쓰면, 기본값을 손쉽게 줄 수 있다.

 하지만, 발견한 값을 갱신(update)할 때, `__getitem__` 혹은 `get` 을 쓰는 건 비효율적이고 추천하지 않는 방법이라고 한다. 이때, 해결책으로 사용할 수 있는 함수가 `setdefault()` 인데, 이 함수가 왜 좋은지 아래 코드를 통해서 알아보자.

>  <참고> 아래 코드 실행을 위해서 아래 `practice_file.txt` 파일을 아래 파이썬 코드 파일과 같은 위치 (같은 디렉토리 내에) 저장해야한다.

```
Hello, Hi! Hi ~
Nice to meet you! ^^
```



 1)  `setdefault()` 함수를 사용하지 않았을 때

```python
import sys
import re
WORD_RE = re.compile(r'\w+') # 원하는 문자만 골라내는 정규식! # [a-zA-Z0-9_])와 동일
index={}
with open(sys.argv[1]) as fp: # cmd 창에서 입력한 경로에 있는 파일을 연다
        for line_no, line in enumerate(fp, 1): # enumerate() 를 통해서 index 부여서 두번째 인자값을 1로 설정함으로써 index가 0이 아닌 1부터 시작한다.
             for match in WORD_RE.finditer(line):
                     word = match.group()
                     column_no = match.start()+1
                     location = (line_no, column_no)
                     occurences = index.get(word, [])
                     occurences.append(location)
                     index[word] = occurences

for word in sorted(index, key=str.upper):
        print(word, index[word])
# 실행 결과
# Hello [(1, 1)]
# Hi [(1, 8), (1, 12)]
# meet [(2, 9)]
# Nice [(2, 1)]
# to [(2, 6)]
# you [(2, 14)]
```

> 참고
>
> 위 코드를 실행시킬 때는
>
> ```shell
> python practice_python.py ./practice_file.txt
> ```
>
> 라고 입력하면 된다. (`practice_python.py` 는 위 코드를 담고 있는 파이썬 파일명)



 2)  `setdefault()` 함수를 사용했을 때

```python
import sys
import re
WORD_RE = re.compile(r'\w+')
index={}
with open(sys.argv[1]) as fp:
        for line_no, line in enumerate(fp, 1):
             for match in WORD_RE.finditer(line):
                     word = match.group()
                     column_no = match.start()+1
                     location = (line_no, column_no)
                     index.setdefault(word, []).append(location) # 위 코드에서 바뀐 부분
                     
for word in sorted(index, key=str.upper):
        print(word, index[word])
```

 위 코드에서 바뀐 부분만 떼어 놓고 보면,

```python
index.setdefault(word, []).append(location)
```

 `index` 라는 딕셔너리 안에 `word` 라는 키가 존재하면, 해당 키값을 반환하고, `word` 라는 키가 존재하지 않으면 두번째 인자인 `[]` 을 반환한다. 이처럼 `setdefault()` 를 사용하여 간결하게 처리할 수 있다.

------------

 이번 포스트에서 매핑 관련된 내용을 끝내려고 했는데, 계획보다 시간이 오래 걸렸다ㅠㅠ 3.4절부터의 내용은 다음 포스트에서 보자.