---
title: "Fluent Python - Chap 3. 딕셔너리와 집합 (2)"
date: 2019-06-08
category: Python3
tags: 
- Python3 
- dict
- mapping
- defaultdict
- collections
- OrderedDict
- ChainMap
- Counter

---

`이 포스트는 <<Fluent Python (전문가를 위한 파이썬) >> (written by Luciano Ramalho, 강권학 옮김) 스터디를 진행하면서 공부한 내용을 정리하는 목적으로 작성되었습니다. 따라서 이 포스트 내 설명 및 코드의 출처는 <<Fluent Python (전문가를 위한 파이썬) >> 임을 미리 밝힙니다.`



## Chapter 3. 딕셔너리와 집합

### 3.4 융통성 있게 키를 조회하는 매핑

#### 3.4.1 defaultdict: 존재하지 않는 키에 대한 또 다른 처리

 저번 포스트에서 봤던 `setdefault()` 예제 코드를 다시 보자.

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
                     index.setdefault(word, []).append(location)
                     
for word in sorted(index, key=str.upper):
        print(word, index[word])
```

 이때, `index`를 기본 `dict` 가 아닌 `collections.defaultdict` 객체로 생성하여 위 코드를 다시 쓸 수 있다.

```python
import sys
import re
import collections

WORD_RE = re.compile(r'\w+')

index = collections.defaultdict(list)
with open(sys.argv[1]) as fp:
        for line_no, line in enumerate(fp, 1):
             for match in WORD_RE.finditer(line):
                     word = match.group()
                     column_no = match.start()+1
                     location = (line_no, column_no)
                     index[word].append(location) # 수정된 부분
                     
for word in sorted(index, key=str.upper):
        print(word, index[word])
```

 위 코드에서 수정된 부분만 보면, `defaultdict` 객체인 `index` 내에 `word` 라는 키가 존재하지 않으면, 새로운 빈 리스트를 생성하기 위해 `list()` 를 호출한다. 그리고 새롭게 생성된 빈 리스트를 `word` 라는 키에 매핑시켜서 `index` 에 삽입한다. 그 다음에 이 리스트에 대한 참조를 반환한다.

#### 3.4.2 `__missing__()` 메서드

 추후 업데이트!

### 3.5 그 외 매핑형

 `defaultdict` 이외에 표준 라이브러리의 `collections` 모듈에서 제공하는 여러 매핑형들을 보자.

- `collections.OrderedDict`

  순서가 저장되지 않는 기본 `dict` 와 다르게 `OrderedDict` 에서는 **키를 삽입한 순서를 유지한다.**

  ```python
  >>> from collections import OrderedDict
  >>> d = {'banana': 3, 'apple':4, 'pear': 1, 'orange': 2} # dict
  >>> d.items()
  dict_items([('banana', 3), ('apple', 4), ('pear', 1), ('orange', 2)])
  >>> OrderedDict(sorted(d.items(), key = lambda t: t[0]))
  OrderedDict([('apple', 4), ('banana', 3), ('orange', 2), ('pear', 1)])
  >>> OrderedDict(sorted(d.items(), key = lambda t: t[1]))
  OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])
  >>> OrderedDict(sorted(d.items(), key = lambda t: len(t[0])))
  OrderedDict([('pear', 1), ('apple', 4), ('banana', 3), ('orange', 2)])
  ```

  > 코드 출처: [8.3. collections — Container datatypes - 8.3.6.1. OrderedDict Examples and Recipes](https://docs.python.org/3.4/library/collections.html?highlight=ordereddict)

- `collections.ChainMap`

  이는 여러 개의 `dict` 혹은 mapping형을 합칠 때, 유용하다.

  ```python
  >>> dict1 = {'apple':1, 'banana':2}
  >>> dict2 = {'coconut':3, 'date':4, 'apple':5}
  >>> combined_dict = collections.ChainMap(dict1, dict2)
  >>> reverse_ordered_dict = collections.ChainMap(dict2, dict1)
  >>> for key, value in combined_dict.items():
  ...     print(key, value)
  ...
  date 4
  coconut 3
  banana 2
  apple 1
  >>> for key, value in reverse_ordered_dict.items():
  ...     print(key, value)
  ...
  date 4
  coconut 3
  banana 2
  apple 5
  ```

  이 코드에서는 `apple` 라는 `key` 에 대한 `value` 에 주목하자. `ChainMap(dict1, dict2)` 에 대해서는 `dict1` 에서의 `apple` 의 `value` 가,  `ChainMap(dict2, dict1)` 에 대해서는 `dict2` 에서의 `apple` 의 `value` 가 사용되었다.

  > 코드 출처: [RIP Tutorial - Python Language collections.ChainMap](https://riptutorial.com/ko/python/example/30550/%EC%BB%AC%EB%A0%89%EC%85%98--chainmap)

- `collections.Counter`

  ```python
  >>> alphabet_counter = Counter('abracadabra')
  >>> alphabet_counter['a'] # 'a' 가 5번 등장했음
  5
  >>> alphabet_counter['z'] # missing key
  0
  >>> sorted(alphabet_counter.elements())
  ['a', 'a', 'a', 'a', 'a', 'b', 'b', 'c', 'd', 'r', 'r']
  >>> alphabet_counter.most_common(3) # return a list of the n most common elements and their counts
  [('a', 5), ('b', 2), ('r', 2)]
  ```

- `collections.UserDict`

  표준 `dict` 처럼 작동하는 매핑을 순수 파이썬으로 구현한 클래스

 위에서 `collections.OrderedDict`, `collections.ChainMap`, `collections.Counter` 는 바로 사용할 수 있지만, `UserDict`는 3.6절에 나오듯이 **상속해서 사용해야한다.**

### 3.6 UserDict 상속하기

 3.4.2 `__missing__()` 보충하면서 같이 보충하기!

### 3.7 불변 매핑

 표준 라이브러리에서 제공하는 매핑형은 모두 가변형이지만, 사용자가 임의로 매핑을 변경하지 못하게 하기 위해서 파이썬에서 제공하는 `types` 모듈의 `MappingProxyType` 이라는 래퍼 클래스를 이용할 수 있다.

-----

 다음 포스트에서 딕셔너리처럼 해시 테이블을 이용하는 또 다른 자료형인 집합에 대해 알아보자.