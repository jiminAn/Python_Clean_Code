
## Chapter 02 파이썬스러운 코드

##### "관용구를 따른 코드를 관용적이라고 부르고 , 파이썬에서는 파이썬스럽다(Pythonic)고 한다."

##### 파이썬스러운 코드로 얻을 수 있는 장점은
##### 첫번째로 일반적으로 더 나은 성능을 내고 
##### 두번째로는 동일한 패턴과 구조에 익숙해지면 실수를 줄이고 문제의 본질에 보다 집중할 수 있기 때문이다.

### 01.인덱스와 슬라이스
---
##### 음수 인덱스를 사용하여 끝에서 접근이 가능
```python
>>> my_number = (4,5,3,9)
>>> my_number[-1] # my_number[3]
9
```
##### Slice를 사용하여 특정 구간의 요소를 구할 수 있다.
```python
>>> my_numbers = (1,1,2,3,5,8,13,21)
>>> my_numbers[1:7:2]
(1,3,8) # index 1 to index 6 with step 2 
```
##### 시퀀스에 간격을 전달할 때 실제로는 슬라이스를 전달하는 것과 같다
```python
>>> interval = slice(1,7,2)
>>> my_number[interval]
>>> (1,3,8)
```
 
#### 1.1 자체 시퀀스 생성

##### 시퀀스나 이터러블 객체를 만들지 않고 키로 객체의 특정 요소를 가져오는 방법에 대해 다룬다.

##### 사용자정의 클래스에 __getitem__을 구현하려는 경우

```python
class Items:
	def __init__(self, *values):
		self._values = list(values)
	def __len__(self):
		return len(self.values)
	def __getitem__(self, item):
		return self._values.__getitem__(item)
# 리스트 래핑
```
###### *values → 파라미터를 몇개 받을 지 모르는 경우 사용, 튜플 형태로 전달
###### __ getitem __ : 슬라이싱을 구현할 수 있도록 도우며 리스트에서 슬라이싱을 하게 되면 내부적으로 __ getitem __ 메소드를 실행 
##### 래퍼도 아니고 내장 객체를 사용하지도 않은 경우는 자신만의 시퀀스를 구현할 수 있음

##### 유의사항

###### - 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 함

###### - slice에 의해 제공된 범위는 파이썬이 하는 것처럼 마지막 요소는 제외

###  02. 컨텍스트 관리자(context manager)
---

##### 주요 동작의 전후에 작업을 실행하려고 할 때 유용

###### ex) 파일을 열고 닫기, 데이터베이스의 연결을 열고 닫기
``` python
fd = open(filename)
try:
	process_file(fd)
finally:
	fd.close()
	
# pythonic sloution

with open(filename) as fd:
	process_file(fd)
```
##### -- 컨텍스트 관리자는  __ enter __와 __ exit __ 로 구성
##### -- with 문은 __ enter __ 메서드를 호출, 메서드가 무엇을 반환하든 as 이후 지정된 변수에 할당 함
##### -- 블록에 대한 마지막 문장이 끝나면 컨텍스트가 종료 되고 __ exit __ 를 호출
- ###### __ exit __ 는 블록에서 발생한 예외를 파라미터로 받는데,  
- ######  True를 반환하면 예외를 삼킬 수 있으므로 반환하지 않도록 주의해야 함

#### 2.1 컨텍스트 관리자 구현
```python
import contextlib

def open_file() :
    print ("open file")

def close_file() :
    print ("close file")

@contextlib.contextmanager
def my_context_manager():
    open_file()  #with 문에 진입할 때 __enter__ 메서드의 로직
    yield print("yield line")
    close_file() #with 문을 탈출할 때 __exit__ 메서드의 로직

with my_context_manager() as openen_file:
    print("with line")
    
output : 
open file
yield line
with line
close file
```
##### -- Contextlib 모듈을 사용하면 보다 쉽게 구현할 수 있음
##### -- contextmanager 데코레이터는 적용한 함수의 코드를 컨텍스트 관리자로 변환
#####  -- Contextlib.contextDecorator 클래스는 with문이 존재하지 않음

### 03. 프로퍼티, 속성과 객체 메서드의 다른 타입들
---
##### 파이썬 객체의 모든 프로퍼티와 함수는 public 이다.
##### 즉 호출자가 객체의 속성을 호출하지 못하도록 할 방법이 없음
##### 그러나, 밑줄로 시작하는 속성으로 private를 기대할 수 있음
#### 3.1 파이썬에서의 밑줄
```python
>>>class Connector:
		def __init__ (self, source) :
			self.source = source
			self._timeout = 60 # private(강제 X)
>>>conn = Connector("postgresql://localhost")
>>>conn.__dict__
{'source':'postgresql://localhost','_timeout':60}
```


##### 밑줄( _ ) 로 private 표현 하는데,  밑줄로 시작하는 속성에는 접근할 수 있지만 접근하지 말라고 명시적으로 표현한 것
```python
>>>class Connector:
		def __init__ (self, source) :
			self.source = source
			self.__timeout = 60 # 이중 밑줄
		def connect(self):
			print("connecting with {0}s".format(self.__timeout))
			
>>>conn = Connector("postgresql://localhost")
>>>conn.connect()
connecting with 60s
>>>conn.__timeout
Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
	AttributeError: 'Connector' object has no attribute '__timeout'
```
##### 에러 메세지로 "접근할 수 없다"가 아니라 존재하지 않는다고 말함
##### 이중 밑줄(__) 은 네임 맹글링(Name Mangling)이고 이것은 private로 표현한 것이 아님

##### 단지 속성이 다른 이름으로 존재하는 것에 주목해야 함

##### 따라서, 속성을 private로 정의하려고 할 때 하나의 밑줄을 사용하자.

#### 3.2 프로퍼티

##### 프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려는 경우 사용
##### JAVA와 같은 프로그래밍 언어에서는 접근 메서드를 만들지만 파이썬에서는 프로퍼티를 사용
``` python
class Person:
    def __init__(self, first_name, last_name, age):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age
    
    @property #getter 값을 가져온다
    def age(self):
        return self._age

    @property
    def full_name(self):
        return self.first_name + " " + self.last_name

    @age.setter #setter 값을 저장한다;
    def age(self, age):
        if age < 0:
            raise ValueError("Invalid age")
        self._age = age

person = Person("John","Doe",20)
print(person.age)
person.age = person.age + 1
print(person.age)
print(person.full_name)
```

##### 프로퍼티는 명령-쿼리 분리 원칙을 따르기 위한 좋은 방법

######  명령-쿼리 분리 원칙: 객체의 메서드가 무언가의 상태를 변경하는 커맨드이거나 무언가의 값을 반환하는 쿼리이거나 둘 중에 하나만 수행해야지 둘 다 동시에 수행하면 안된다는 것

##### 따라서, 무언가를 할당하고 유효성 검사를 하고 싶으면 두 개 이상의 문자로 나누어야 한다.

### 04. 이터러블 객체
---
##### 파이썬에는 기본적으로 반복 가능한 객체가 있다. 리스트, 튜플, 세트 등 이러한 객체들은 for 루프를 통해 값을 반복적으로 가져올 수 있다.

##### 이터러블 :__ iter __ 를 구현하여 반복 구문을 사용할 수 있게 정의한 객체
- ##### __ iter __는 이터레이터를 반환해야 한다.

##### 이터레이터 : __ next__ 를 구현하여 한 번에 하나의 값을 생산하는 로직을 정의한 객체

##### for e in myobject: 형태로 객체를 반복할 수 있는지 확인하기 위해 
###### -- 객체가 __ next __ 나 __ iter __ 이터레이터 메서드 중 하나를 포함하는지 여부
###### -- 객체가 시퀀스이고 __ len __과 __ getitem __를 모두 가졌는지 여부
##### 를 차례로 검사
#### 4.1 이터러블 객체 만들기

##### 객체를 반복하려고 하면 해당 객체의 iter() 함수를 호출함.
##### iter()함수는 해당 객체에 __iter__ 메서드가 있는지 확인하는 것.
#####  있다면, __ iter __메서드를 실행함.
```python
# 일정 기간의 날짜를 하루 간격으로 반복하는 객체의 코드
from datetime import timedelta

class DateRangeIterable:
 """자체 이터레이터 메서드를 가지고 있는 이터러블"""
	 def __init__(self, start_date, end_date):
		 self.start_date = start_date
		 self.end_date = end_date
		 self._present_day = start_date
	
	def __iter__(self):
		return self
	
	def __next__(self):
		if self._present_day >= self.end_date:
			raise StopIteration
		today = self._present_day
		self._present_day += timedelta(days=1)
		return today
	for day in DateRangeIterable(date(2019, 1, 1), date(2019, 1, 5)):
		print(day)
2019-01-01
2019-01-02
2019-01-03
2019-01-04
# StopIteration 예외가 발생할 때 까지 next()를 호출한다.
```
``` python
from datetime import timedelta 
from datetime import date

class DateRangeIterable:

    def __init__ (self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._present_day = start_date
    
    def __iter__(self):
        current_day = self.start_date
        while current_day < self.end_date:
            yield current_day
            current_day += timedelta(days=1)


r1 = DateRangeIterable(date(2019,1,1), date(2019,1,5))
print("_ ".join(map(str, r1)))
#'구분자'.join(리스트) 매개변수로 들어온 리스트에 있는 요소 하나하나를 합쳐서 하나의 문자열로 바꾸어 반환하는 함수
# map(함수,리스트) 리스트의 요소를 지정된 함수로 처리해준다.
print(max(r1))
```
##### 컨테이너 이터러블(container iterable)
#### 4.2 시퀀스 만들기

##### 시퀀스는 __len__과 __getitem__을 구현하고 첫 번째 인덱스 0부터 시작하여 포함된 요소를 한 번에 하나씩 차례로 가져올 수 있어야 함.

##### 이터러블은 메모리를 적게 사용하지만 시간복잡도는 O(n)

##### 시퀀스는 메모리를 많이 사용하지만 시간복잡도는 O(1) 
```python
from datetime import timedelta 
from datetime import date

class DateRangeIterable:
    
    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._range = self._create_range()

    def _create_range(self):
        days = []
        current_day = self.start_date
        while current_day < self.end_date:
            days.append(current_day)
            current_day += timedelta(days=1)
        return days

    def __getitem__(self, day_no):
        return self._range[day_no]

    def __len__(self):
        return len(self._range)
```
- ##### DateRangeSequence 객체가 모든 작업을 래핑된 객체인 리스트에 위임하기 때문에 호환성과 일관성을 유지할 수 있음.

### 05. 컨테이너 객체
---
##### 컨테이너는 __contains__ 메서드를 구현한 객체로 __contains__ 메서드는 일반적으로 Boolean 값을 반환한다. 이 메서드는 파이썬에서 in 키워드가 발견될 때 호출

##### 이 메서드를 잘 사용하면 코드의 가독성이 정말 높아진다.

##### Element in container 🡪 container.__contain__(element)

``` python
#__contain__ 메서드는 일반적으로 Boolean 값을 반환
#__contain__ 메서드는 파이썬에서 in 키워드가 발견될 때 호출된다.

from collections import Container

class Boundaries:
    def __init__ (self, width, height):
        self.width = width
        self.height = height

    def __contains__(self, coord):
        x, y = coord
        return 0 <= x < self.width and 0 <= y < self.height

class Grid:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.limit = Boundaries(width, height)

    def __contains__(self, coord):
        return coord in self.limit

def mark_coordinate(grid, coord):
    if coord in grid:
        print("coord in limit")
    else:
        print("coord not in limit")

grid = Grid(100,100)
coord1 = (3,4)
coord2 = (101,101)
mark_coordinate(grid, coord1)
mark_coordinate(grid, coord2)
		
```
### 06. 객체의 동적인 속성
---
```python
class DynamicAttributes:
    def __init__ (self, attribute):
        self.attribute = attribute

    def __getattr__ (self, attr):
        if attr.startswith("fallback_"): # str.startswith() 괄호 안에 적은 문자열로 시작하는지를 확인합니다. True or False
            name = attr.replace("fallback_", "") # replace("찾을값","바꿀값",[바꿀횟수])
            return f"[fallback resolved] {name}"
        raise AttributeError(f"{self.__class__.__name__}에는 {attr} 속성이 없음. ")

dyn = DynamicAttributes("value")
print(dyn.attribute)
print(dyn.fallback_test)
dyn.__dict__["fallback_new"] = "new value"

```
##### __ getattr__ 매직 메서드를 사용해 객체에서 속성을 얻는 방법을 제어할 수 있음.

##### < myobject >.< myattribute >를 호출하면 파이썬은 객체의 사전에서 < myattribute >를 찾고

##### __getattr__를 호출. 객체에 찾고자 하는 속성이 없을 경우 속성의 이름을 파라미터로 전달하여

##### __getattr__이라는 추가 메서드가 호출된다. 이 값을 사용해서 반환 값을 제어하거나, 새로운 속성을 만들 수 있음.

### 07. 호출형(callable) 객체
---
##### 매직 메서드 __call__ 을 사용하면 객체를 일반 함수처럼 호출가능

##### 여기에 전달된 모든 파라미터는 __call__ 메서드에 그대로 전달

##### 객체를 파라미터가 있는 함수처럼 사용하거나 정보를 기억하는 함수처럼 사용할 경우 유용
```python
# 함수처럼 동작하는 객체 
# object(*args, **kwargs) --> object.__call__(*args, **kwargs)

from collections import defaultdict # 외부함수이기 때문에 import 해야한다.

class CallCount:

    def __init__(self):
        self._counts = defaultdict(int)
#   defaultdict(<class 'int'>, {}) 디폴트값이 int인 딕셔너리
    def __call__(self, argument):
        self._counts[argument] += 1
        return self._counts[argument]

cc = CallCount()
print(cc(1)) #객체를 함수처럼 사용할 수 있다.
print(cc(2))
print(cc(1))
print(cc(1))
print(cc("something"))
```
### 08. 파이썬에서 유의할 점
---

#### 8.1 변경 가능한(mutable)파라미터의 기본 값

##### 변경 가능한 객체를 함수의 기본 인자로 사용하면 안됨.

#### 8.2 내장(built-in)타입 확장

##### 리스트, 문자열, 사전과 같은 내장 타입을 확장하는 올바른 방법은 collections 모듈을 사용하는 것.


