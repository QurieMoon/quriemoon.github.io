---
title: "Fluent Python - Chap 2. 시퀀스 (2)"
date: 2019-05-06
category: Python3
tags: 
- Python3 
- data_structure 
- augmented_assignment
- sort

---

`이 포스트는 <<Fluent Python (전문가를 위한 파이썬) >> (written by Luciano Ramalho, 강권학 옮김) 스터디를 진행하면서 공부한 내용을 정리하는 목적으로 작성되었습니다. 따라서 이 포스트 내 설명 및 코드의 출처는 <<Fluent Python (전문가를 위한 파이썬) >> 임을 미리 밝힙니다.`



##  Chapter 2. 시퀀스

### 2.6 시퀀스의 복합 할당(augmented assignment)

 `+=`, `*=` 와 같은 연산자가 **복합 할당 연산자**에 해당한다. 책에서 설명했듯이, `+=` 에 적용되는 개념은 `*=` 에도 적용되기 때문에 `+=` 에 대해 집중적으로 알아보자.

 우선, `+=` 연산자를 실제로 구현하는 메소드는 `__iadd__()` 이다.  `__iadd__()` 메소드가 구현되어 있지 않은 경우, 파이썬은 `__add__()` 메소드를 대신 호출한다.

> 참고:
>
> [`__iadd__()` 메소드](https://docs.python.org/ko/3/reference/datamodel.html#object.__iadd__)

> `*=` 의 경우, `__imul__()` 이라는 메소드로 구현되어 있다.
>
> [`__imul__()` 메소드](https://docs.python.org/ko/3/reference/datamodel.html#object.__imul__)



 이러한 복합 할당 연산자의 작동은 첫 번째 피연산자에 따라서 다르게 작동한다. 아래 식을 보자.

```python
>>> a += b
```

 위 덧셈 복합 할당 연산자 `+=` 의 동작은 `__iadd()__` 라는 특수한 메소드의 구현 여부에 따라서 달라진다.

* `__iadd()__` 메소드가 구현된 경우

  위 덧셈 복합 할당 연산자 `+=` 의 동작은 **첫 번째 피연산자인 `a` 에 의해서 결정**된다. 구체적으로, `a` 의 자료구조(자료형)에서  `__iadd()__` 메소드가 구현되어 있으면, 구현된 해당 메서드가 호출된다.

  * 대표적인 활용 예: 가변 시퀀스 (`list` , `bytearray` , `array.array`)

* `__iadd()__` 메소드가 구현되지 않은 경우

  아래 과정을 거쳐서 위 `+=` 연산자가 동작한다.

  1. `a+=b` 라는 표현식이 `a=a+b` 가 되고
  2. `a+b` 식이 먼저 평가(evalutate) 되고
  3. 위에서 평가된 `a+b`  식을 토대로 **새로** 객체가 생성되고
  4. 이렇게 **새로** 생성된 객체를 `a` 에 할당한다.

 위에서 언급된 두 방식이 어떻게 차이가 나는지 아래 예제 코드를 통해서 알아보자.

* `__iadd()__` 메소드가 구현된 경우

  ```python
  >>> l=[1,2,3]
  >>> id(l) # initial ID of the list l
  4408277768
  >>> l *= 2
  >>> l
  [1, 2, 3, 1, 2, 3]
  >>> id(l)
  4408277768
  ```

* `__iadd()__` 메소드가 구현되지 않은 경우

  ```python
  >>> t=(1,2,3)
  >>> id(t) # initial ID of the tuple t
  4408294136
  >>> t *= 2
  >>> id(t)
  4408035464
  ```

  위 `id()` 함수에서 보이듯이, `__iadd()__` 메소드가 구현된 경우,`*=` 를 통해서 **기존 객체의 내용을 변경**한다. 하지만, `__iadd()__` 메소드가 구현되지 않은 경우에는 곱셈 연산을 수행한 후, **새로운 튜플 객체**를 만들어  `t` 에 대입한다. 이를 통해, 불변 시퀀스에서 반복적으로 연결 연산을 수행하고자 복합 할당 연산자를 쓰는 것은 **비효율적**임을 알 수 있다.

  > `str` 객체 역시 불변 시퀀스이지만, `str` 객체는 반복적으로 수행되는 연결 연산에 대해서 효율적이다. 이는 실제로 문자열에 대해서는 `+=` 연산자를 이용한 작업이 빈번히 수행된다는 점을 CPython에서 고려했기 때문이다. 따라서 `str` 객체의 경우, 메모리 안에 여분의 공간을 갖고 할당되어 이후에 `str` 객체 연결 작업 수행 시, **매번 전체 문자열을 다시 생성하지 않는다.**

### 2.7 list.sort()와 sorted() 내장 함수

 list.sort() 와 sorted() 함수의 차이는 아래와 같다.

|                 list.sort()                 |                 sorted()                 |
| :-----------------------------------------: | :--------------------------------------: |
| 타깃 객체(리스트 내부)를 직접 변경해서 정렬 |   타깃 객체(리스트 내부)를 직접 변경 X   |
|   **None**을 반환 (새로운 리스트 생성 X)    | 정렬된 **새로운 리스트**를 생성해서 반환 |

 아래 코드를 보자.

```python
>>> backpack = ['paper', 'glue', 'notebook', 'pencils', 'eraser']
>>> sorted(backpack)
['eraser', 'glue', 'notebook', 'paper', 'pencils']
>>> backpack # 원본 리스트인 backpack은 바뀌지 않았다.
['paper', 'glue', 'notebook', 'pencils', 'eraser']
>>> sorted(backpack, reverse=True) # 역순으로 정렬
['pencils', 'paper', 'notebook', 'glue', 'eraser']
>>> sorted(backpack, key=len) # 문자열의 길이에 따라 정렬
['glue', 'paper', 'eraser', 'pencils', 'notebook']
>>> sorted(backpack, key=len, reverse=True)
['notebook', 'pencils', 'eraser', 'paper', 'glue']
>>> backpack
['paper', 'glue', 'notebook', 'pencils', 'eraser']
>>> backpack.sort()
>>> backpack # 원본 리스트인 backpack이 바뀌었다.
['eraser', 'glue', 'notebook', 'paper', 'pencils']
```

 책을 보다보면, "정렬 알고리즘이 **안정적**" 이라는 표현이 나온다. 각주를 보면, **안정적**인 정렬 알고리즘이란 *비교해서 동일한 항목들이 상대적인 순서를 유지*하는 알고리즘이다. 아래 코드를 보자.

```python
>>> fruit = ['banana', 'apple', 'grape']
>>> sorted(fruit)
['apple', 'banana', 'grape']
>>> sorted(fruit, reverse=True)
['grape', 'banana', 'apple']
```

```python
>>> fruit_2 = ['banana', 'grape', 'apple']
>>> sorted(fruit_2)
['apple', 'banana', 'grape']
>>> sorted(fruit_2, reverse=True)
['grape', 'banana', 'apple']
```

 위 `fruit` 리스트와 `fruit_2` 리스트의 원소는 `'banana', 'apple', 'grape'` 로 동일하다. 두 리스트의 유일한 차이점은 `'apple'` 원소와 `'grape'` 원소의 순서다. 따라서 주어진 리스트를 알파벳순으로 정렬하는 경우에는 정렬 결과에 **차이가 없다.**

 하지만, 아래 코드를 보자.

```python
>>> fruit = ['banana', 'apple', 'grape']
>>> sorted(fruit, key = len)
['apple', 'grape', 'banana']
>>> sorted(fruit, key=len, reverse=True)
['banana', 'apple', 'grape']
```

```python
>>> fruit_2 = ['banana', 'grape', 'apple']
>>> sorted(fruit_2, key=len)
['grape', 'apple', 'banana']
>>> sorted(fruit_2, key=len, reverse=True)
['banana', 'grape', 'apple']
```

 분명 동일한 원소를 포함하고 있는데, `key=len` 이라는 옵션을 설정하자 **두 리스트의 정렬 결과가 달라졌다.** 왜 이런 결과가 나왔을까?

 `'apple'` 원소와 `'grape'` 원소의 경우, 길이가 `5`로 동일하다. 따라서 길이를 기준으로 리스트를 정렬할 경우, `'apple'` 원소와 `'grape'` 원소 중에서 어떤 원소가 먼저 와야하는가라는 문제가 발생한다. 이때, 파이썬은 정렬 알고리즘의 **안정성** 을 유지하는 방향으로 정렬한다. 즉, 기존 리스트의 원소 정렬 순서가 유지되는 방향으로 정렬한다. 위 예시를 보면, 기존에 `fruit` 리스트에서는 `'apple'` 원소가, `fruit_2` 리스트에서는 `'grape'` 원소가 더 앞에 위치하기 때문에 위와 같은 결과가 나온다.

> list.sort()는 사본을 만들지 않고, **리스트 내부를 변경해서 정렬**한다는 말에서 예상할 수 있듯이, `tuple` 타입의 객체에 대한 `sort()` 내장 함수는 구현되어 있지 않다.
>
> ```python
> >>> t=(4,2,1,5)
> >>> type(t)
> <class 'tuple'>
> >>> t.sort()
> Traceback (most recent call last):
>   File "<stdin>", line 1, in <module>
> AttributeError: 'tuple' object has no attribute 'sort'
> ```

### 2.8  정렬된 시퀀스를 bisect로 관리하기

 2.8절은 표준 이진 검색 알고리즘이 구현된 파이썬 표준 라이브러리인 `bisect` 모듈에 대한 내용이다. 구체적으로, 이미 정렬된 시퀀스의 정렬 상태를 유지한 채로 새로운 항목을 추가하는 `bisect.insort()` 함수 등에 대해 살펴보지만 이 포스트에서 자세한 내용은 생략한다.

-----------------------------------

 지금까지 살펴보았듯이, 리스트형은 매우 편리한 자료형(자료구조)이다. 하지만, 때때로 배열이나 덱(deque) 등 다른 자료형을 사용하는 게 더 효율적일 때가 있다. 다음 포스트에서 리스트형 이외에 어떤 시퀀스형이 있는지, 해당 시퀀스형이 어떤 상황에서 유용한지 알아보자.

