---
title: "Fluent Python - Chap 2. 시퀀스 (3)"
date: 2019-05-12
category: Python3
tags: 
- Python3 
- data_structure 
- array
- memoryview
- deque

---

`이 포스트는 <<Fluent Python (전문가를 위한 파이썬) >> (written by Luciano Ramalho, 강권학 옮김) 스터디를 진행하면서 공부한 내용을 정리하는 목적으로 작성되었습니다. 따라서 이 포스트 내 설명 및 코드의 출처는 <<Fluent Python (전문가를 위한 파이썬) >> 임을 미리 밝힙니다.`



##  Chapter 2. 시퀀스

### 2.9 리스트가 답이 아닐 때

 이 장에서는 특정 상황에서 리스트보다 더 효율적인, **리스트를 대체할 수 있는 여러 시퀀스형**을 살펴보자.

#### 2.9.1 배열

```python
>>> from array import array
>>> from random import random
>>> floats = array('d', (random() for i in range(10**7)))
```

 배열을 생성할 떄는 배열에 저장되는 각 항목의 형(type)을 결정하는 문자인 **타입코드(typecode)** 를 지정한다. 위 코드에서 타입코드는 `'d'` 이다. 는 '실수'를 의미하기 때문에 `floats` 배열은 실수를 담고 있는 배열임을 알 수 있다.

> 위 코드에서 배열 초기화 시, *제너레이터 표현식*을 사용했다.

 그렇다면, 리스트 대신에 **배열을 왜 써야할까?** 배열은 **숫자만 들어 있는**, 특히 **숫자가 아주 많이 들어 있는 시퀀스**를 처리할 때, 효율적이다. 아래 코드를 실행해보면서 **실행 시간**을 비교해보자.

- 리스트의 각 원소를 한 행에 하나씩 텍스트 파일에 저장하는 예제

```python
import timeit
from array import array
from random import random

floats_list = [random() for i in range(10**7)]
start_time = timeit.default_timer()
with open('floats_default.txt', 'w') as fp:
	for i in range(len(floats_list)):
		fp.write(str(floats_list[i])+'\n')

stop_time = timeit.default_timer()
elapsed_time = stop_time - start_time

print("Program Executed in {} seconds".format(elapsed_time))
# Program Executed in 19.114658007980324 seconds
```

- `array.tofile()` 메서드를 이용해서 배열을 파일에 저장하는 예제

```python
import timeit
from array import array
from random import random

floats = array('d', (random() for i in range(10**7)))
start_time = timeit.default_timer()
with open('floats.bin', 'wb') as fp:
	floats.tofile(fp)

stop_time = timeit.default_timer()
elapsed_time = stop_time - start_time

print("Program Executed in {} seconds".format(elapsed_time))
# Program Executed in 0.163494489970617 seconds
```

 실행 시간을 비교해봤을 때, 배열(array)을 사용했을 때, 파일 쓰기를 위해 소요되는 시간이 약 1/100 배로 줄어들었다. 또한, 결과 파일 크기 역시 `floats.bin`  의 경우, 80MB인 반면에 `floats_default.txt` 의 경우, 192.7MB 로 차이가 상당하다.

 역으로 파일을 읽어오는 예제 역시 리스트보다 배열을 사용했을 때, 실행 속도가 훨씬 빨랐다.

- `floats_default.txt` 에서 값을 읽어와 리스트에 저장하는 예제

```python
import timeit
from array import array
from random import random

floats_list = []
start_time = timeit.default_timer()
with open('floats_default.txt', 'r') as fp:
	lines = fp.readlines()

for i in range(len(lines)):
	floats_list.append(float(lines[i].split('\n')[0]))

stop_time = timeit.default_timer()
elapsed_time = stop_time - start_time

print("Program Executed in {} seconds".format(elapsed_time))
# Program Executed in 10.558079929091036 seconds
```

- `floats.bin` 에서 값을 읽어와 배열에 저장하는 예제

```python
import timeit
from array import array
from random import random

floats = array('d')
start_time = timeit.default_timer()
with open('floats.bin', 'rb') as fp:
	floats.fromfile(fp, 10**7)

stop_time = timeit.default_timer()
elapsed_time = stop_time - start_time

print("Program Executed in {} seconds".format(elapsed_time))
# Program Executed in 0.09963303396943957 seconds
```

#### 2.9.2 메모리 뷰

 **메모리 뷰(memoryview)** 라는 내장 클래스는 공유 메모리 시퀀스형이다. 책에서는 16비트 정수 배열에서 바이트 하나를 변경하는 예제를 보여준다.

```python
>>> import array
>>> numbers = array.array('h', [-2,-1,0,1,2])
>>> memv = memoryview(numbers)
>>> len(memv)
5
>>> memv[0]
-2
>>> memv_oct = memv.cast('B')
>>> memv_oct.tolist()
[254, 255, 255, 255, 0, 0, 1, 0, 2, 0]
>>> memv_oct[5]=4
>>> memv_oct.tolist()
[254, 255, 255, 255, 0, 4, 1, 0, 2, 0]
>>> memv.tolist() # 2바이트 unsigned int의 최상위 비트에 4가 대입되었기에 memv[2] == 1024이다.
[-2, -1, 1024, 1, 2]
>>> numbers
array('h', [-2, -1, 1024, 1, 2])
```

 memoryview 는 왜 필요할까? 책에 따르면, 메모리뷰 덕분에 SQLlite 데이터베이스, NumPy 배열 들 데이터 구조체를 복사하지 않고 **메모리를 공유하여 사용할 수 있다** . 즉, 데이터셋이 커지는 상황에서 유용하다.

 그렇지만 여전히 **굳이** memoryview라는 새로운 클래스를 왜 사용해야할까?? 일단, **동일한** 메모리를 공유한다는 말을 다시 곱씹어보자. 위 예제를 보면, `memory_oct` 를 바꾸었을 때, `memv` 와`numbers` 의 내용 역시 바뀌었다. 이를 통해서 `memory_oct` 와 `memv` 와`numbers` 가 동일한 메모리를 공유한다고 볼 수 있다. 그렇다면, 이 셋은 동일한 걸까? 아래 코드를 보자.

```python
>>> numbers == memv # memv = memoryview(numbers)
True
>>> numbers == memv_oct # memv_oct = memv.cast('B')
False
>>> memv == memv_oct
False
>>> id(numbers)
4542538032
>>> id(memv)
4541520200
>>> id(memv_oct)
4541520584
```

 우선, `memv` 는 `memoryview()` 함수를 통해서 `numbers`를 참조하는 **새로** 만들어진 memoryview 객체이다. 또한, `memv_oct` 의 경우, `memoryview.cast()` 에 의해서 반환된 **또 다른** memoryview 객체이다. 따라서 `id(numbers)`, `id(memv)`, `id(memv_oct)` 가 서로 다 다르다.

**질문**: 왜 `numbers == memv_oct` 와 `meme == memv_oct` 는 `False` 일까? `memv_oct`는  `memv` 요소를 unsigned char(타입코드 'B') 로 **형변환**한 결과로 생성되었기 때문일까? 



> 추가 링크: [memoryview() in Python - GeeksforGeeks](https://www.geeksforgeeks.org/memoryview-in-python/)

> 추가로, 위 코드에서 왜 `numbers` 와 `memv` 의 `id()` 값이 다른데, `numbers == memv` 가 `True` 인지 궁금할 경우, Python에서 두 객체를 비교할 때, 이용되는 `==` 와 `is` 의 차이를 설명해주는 아래 글을 읽어보자.
>
> [Python에서 is 와 == 의 차이](https://tech.songyunseop.com/post/2017/09/python-comparing/)

#### 2.9.3 NumPy와 SciPy

 생략



#### 2.9.4 덱 및 기타 큐

 책에 나와있듯이, append()와 pop() 메서드를 이용해서 리스트를 스택이나 큐로 사용할 수 있다. 하지만, 이럴 경우 0번 인덱스에 위치한 요소처럼 끝에 위치한 값을 새로 삽입하거나 삭제하고 싶은 경우, 전체 요소를 이동시켜야하는 부담이 있다. 이때, 유용하게 이용할 수 있는 것이 바로 **덱(collections.deque) 클래스**이다.

 덱 클래스는 **큐의 양쪽 어디에서든 빠르게 삽입 및 삭제**할 수 있도록 만들어진 양방향 큐다. 따라서 '최근에 본 항목' 등의 목록을 만들 때도 유용하게 이용할 수 있다. 아래 예시를 보자.

```python
>>> from collections import deque
>>> dq = deque(range(10), maxlen = 10)
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
```

 위 `maxlen` 이라는 parameter를 통해서 deque 객체가 수용할 수 있는 최대 항목 수를 설정한다.

```python
>>> dq.rotate(3) # rotate(n): n이 양수일 때는 오른쪽 끝에 있는 항목 중 n개를 왼 쪽 끝으로
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4) # rotate(n): n이 음수일 때는 왼쪽 끝에 있는 항목 중 n개를 오른쪽 끝으로
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
```

 rotate() 함수를 통해서 deque을 조작할 수 있다.

```python
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.append(100)
>>> dq
deque([2, 3, 4, 5, 6, 7, 8, 9, 0, 100], maxlen=10)
>>> dq.appendleft(-1)
>>> dq
deque([-1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.extend([10,20,30])
>>> dq
deque([4, 5, 6, 7, 8, 9, 0, 10, 20, 30], maxlen=10)
>>> dq.extendleft([-40,-30,-20,-10]) # extendleft()의 경우, 항목이 역순으로 추가된다.
>>> dq
deque([-10, -20, -30, -40, 4, 5, 6, 7, 8, 9], maxlen=10)
```

 `dq`가 가득 찬 상태에서 새로운 항목을 추가하려고 할 때는 반대쪽 항목을 삭제한다.

 이처럼 덱은 리스트 메서드 대부분을 구현했을 뿐만 아니라 rotate() 처럼 고유한 메서드를 추가로 가지고 있다. 하지만, 덱의 중간 항목을 삭제하는 연산은 그리 빠르지 않다는 단점이 있다.



> 참고: 아래 상황에서는 시퀀스는 아니지만 **set 형**으로 구현하는 것도 고려해보자.
>
> - `item in my_collection` 처럼 특정 항목이 주어진 `my_collection`  에 포함되 어 있는지 아닌지 자주 검사할 때
> - 위 상황에서 특히 `my_collection` 의 항목 수가 아주 많을 때

