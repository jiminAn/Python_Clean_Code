## Chapter 02 파이썬스러운 코드

"관용구를 따른 코드를 관용적이라고 부르고 , 파이썬에서는 파이썬스럽다(Pythonic)고 한다."

파이썬스러운 코드의 장점
1. 일반적으로 더 나은 성능을 냄 
2.  동일한 패턴과 구조에 익숙해지면 실수를 줄이고 문제의 본질에 보다 집중할 수 있음.

### 01. 인덱스와 슬라이스

음수 인덱스를 사용하여 끝에서 접근이 가능
```python
>>> my_number = (4,5,3,9)
>>> my_number[-1] 
9
```
`slice`를 사용하여 특정 구간의 요소를 구할 수 있다.

시작 인덱스는 포함, 끝 인덱스는 제외하고 선택한 구간의 값을 가져옴
```python
>>> my_numbers = (1,1,2,3,5,8,13,21)
>>> my_numbers[1:7:2]
(1,3,8) # index 1 to index 6 with step 2 
```
시퀀스에 간격을 전달할 때 실제로는 슬라이스를 전달하는 것과 같다
```python
>>> interval = slice(1,7,2)
>>> my_number[interval]
>>> (1,3,8)
```
 `slice`의 (시작, 중지, 간격) 중 하나를 지정하지 않는 경우 `None`으로 간주
 
 튜플, 문자열, 리스트의 특정요소를 가져오려고할 때 유용함
 
#### 1.1 자체 시퀀스 생성

`__getitem__ `은 `myobject[key]`와 같은 형태를 사용할 때 호출되는 메서드로 key에 해당하는 대괄호 안의 값을 파라미터로 전달함

시퀀스는 `__getitem__`과 `__len__`을 모두 구현하는 객체이므로 반복이 가능. 리스트, 튜플과 문자열 등이 있다.

 시퀀스나 이터러블 객체를 만들지 않고 키로 객체의 특정 요소를 가져오는 방법
  
```python
# 객체가 어떻게 리스트를 래핑하는지 보여주는 클래스. 
# 필요한 메서드가 있을 때는 list 객체에 있는 동일한 메서드에 위임하며 됨.
class Items:
	def __init__(self, *values):
		self._values = list(values)
	def __len__(self):
		return len(self.values)
	def __getitem__(self, item):
		return self._values.__getitem__(item)
# 리스트 래핑
``` 
`*values` : 파라미터를 몇 개 받을 지 모르는 경우 사용하고 , 튜플 형태로 전달할 때 사용 

`__ getitem __` : 슬라이싱을 구현할 수 있도록 도우며 리스트에서 슬라이싱을 하게 되면 내부적으로 `__ getitem __` 메소드를 실행 

래퍼도 아니고 내장 객체를 사용하지도 않은 경우는 자신만의 시퀀스를 구현할 수 있음

유의사항
- 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 함

 - slice에 의해 제공된 범위는 파이썬이 하는 것처럼 마지막 요소는 제외

###  02. 컨텍스트 관리자(context manager)

주요 동작의 전후에 작업을 실행하려고 할 때 유용 ( ex 파일, 데이터 베이스 )

예외가 발생하거나 오류를 처리해야 하는 경우 리소스를 해제하는 방법은 `finally` 블록에 정리 코드를 넣는 것
```python
fd = open(filename)
try:
	process_file(fd)
finally:
	fd.close()
	
# pythonic sloution

with open(filename) as fd:
	process_file(fd)
``` 
`with`문은 컨텍스트 관리자로 진입하게 함. `open` 함수는 컨텍스트 관리자 프로토콜을 구현 따라서 **예외가 발생한 경우**에도 블록이 완료되면 파일이 자동으로 닫힘.

컨텍스트 관리자는  `__ enter __`와 `__ exit __ `로 구성. 

 `with` 문은` __ enter __` 메서드를 호출, 메서드가 무엇을 반환하든 as 이후 지정된 변수에 할당 

라인이 실행되면 새로운 컨텍스트로 진입하고 블록에 대한 마지막 문장이 끝나면 컨텍스트가 종료 되고 `__ exit __` 를 호출

'컨텍스트 관리자'는  관심사를 분리하고 독립적으로 유지되어야하는 코드를 분리하는 좋은 방법
```python
# 컨텍스트 관리자를 사용하지 않은 코드
def stop_database():
	run("systemctl stop postgresql.service")

def start_database():
	run ("systemctl start postgresql.service")

class DBHandler:
	def __enter__(self):
		stop_database()
		return self

	def __exit__(self, exc_type, ex_value, ex_traceback):
	start_database()

def db_backup():
	run("pg_dump database")

def main()
	with DBHandler():
		db_backup()
```
` __ exit __` 는 블록에서 발생한 예외를 파라미터로 받는데,  예외가 없으면 모두 `None`이고
`True`를 반환하면 잠재적으로 발생한 예외를 호출자에게 전파하지 않고 멈춘다는 것을 의미함.

따라서 `True`를 반환하지 않도록 주의해야하고, 예외를 삼키는 것은 좋지 않은 습관임.

#### 2.1 컨텍스트 관리자 구현

`__enter__`와 `__exit__` 매직 메서드만 구현하면 해당 객체는 컨텍스트 관리자 프로토콜을 지원할 수 있음

 `Contextlib` 모듈을 사용하면 컨텍스트 관리자를 구현하거나 더 간결한 코드를 작성하는 데 도움이 됨
`contextlib.contextmanager` 데코레이터를 적용면 함수의 코드를 컨텍스트 관리자로 변환. 함수는 제너레이터여야 함
```python
import contextlib

@contextlib.contextmanager
def db_handler():
	stop_database()
	yield
	start_database()

with db_handler():
	db_backup()

# 제너레이터를 정의하고 데코레이터를 적용
```
데코레이터를 적용함으로써, `yield`문 앞의 모든 것은 `__enter__`의 로직이고 `yield`문 뒤의 모든 것은 `__exit__`의 로직임

이런식의 컨텍스트 매니저를 작성하면 기존 함수를 **리팩토링**하기 쉬워짐.

컨텍스트 관리자 안에서 실행될 함수에 데코레이터를 적용하기 위한 로직을 제공하는 믹스인 클래스인 `contextlib.ContextDecorator`라는 클래스가 존재함.

```python
class dbhandler_decorator(contextlib.ContextDecorator):
	def __enter__(self):
		stop_database()

	def __exit(self, ext_type, ex_value, ex_traceback):
		start_database()

@dbhandler_decorator()
def offline_backup():
	run("pg_dump database")
```
`with`문이 없이 함수를 호출하면 `offline_backup` 함수가 컨텍스트 관리자 안에서 자동으로 실행됨.

단점은 완전히 '독립적'이라는 것.  `__enter__` 메서드가 반환한 객체를 사용해야 하는 경우는 `with offline_backup() as bp:` 와 같은 접근 방식을 선택해야 함


### 03. 프로퍼티, 속성과 객체 메서드의 다른 타입들

파이썬 객체의 모든 프로퍼티와 함수는 **public** 이다. 따라서, 호출자가 객체의 속성을 호출하지 못하도록 할 방법이 없음

그러나, 밑줄로 시작하는 속성으로 **private**를 기대할 수 있음
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

`connector`객체는 `source`로 생성되며
`source`와 `_timeout`을 속성으로 가짐.

여기서 `source`는 `public`이고 `_timeout`은 `private`임. 그러나, 두 개의 속성에 모두 접근이 가능함. 


밑줄( _ ) 로 `private` 표현 하는데,  밑줄로 시작하는 속성에는 접근할 수 있지만 접근하지 말라고 명시적으로 표현한 것

위의 규칙을 준수하게 되면 유지보수가 쉽고 견고한 코드를 작성할 수 있음.

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
에러 메세지로 "접근할 수 없다"가 아니라 "존재하지 않는다" 고 말함

이중 밑줄(__) 은 **네임 맹글링(Name Mangling)** 이고 이것은 `private`로 표현한 것이 아니고 단지 속성이 다른 이름으로 존재하는 것에 주목해야 함

따라서, 속성을 private로 정의하려고 할 때 하나의 밑줄을 사용해야 함.

#### 3.2 프로퍼티

프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려는 경우 사용

JAVA와 같은 프로그래밍 언어에서는 접근 메서드를 만들지만 파이썬에서는 프로퍼티를 사용

```python
# 사용자가 등록한 정보에 잘못된 정보가 입력되지 않게 보호하려는 경우

import re # 정규 표현식

EMAIL_FORMAT = re.compile(r"[^@]+@[^@]+[^@]+")

def is_valid_email(potentially_valid_email: str):
	return re.match(EMAIL_FORMAT, potentially_valid_email) is not None

class User:
	def __init__ (self, username):
	self.username = username
	self._email = None
	
@property # 응답 쿼리
def email(self):
	return self._email

@email.setter # 명령 쿼리
def email(self, new_email):
	if not is_valid_email(new_email):
		raise ValueError(f"유효한 이메일이 아니므로 {new_email} 값을 사용할 수 없음")
	self._email = new_email
```
프로퍼티를 사용하여 얻은 이점 
1. `@property`메서드는 `private`속성인 `email`값을 반환
2. `@email.setter`를 추가하여 `<user>.email = <new_email>`이 실행될 때 호출되어 <new_email>이 파라미터가 되게 함
3
```python
>>> u1 = user("jsmith")
>>> u1.email = "jsmith@"
유효한 이메일이 아니므로 jsmith@ 값을 사용할 수 없음
>>> u1. email = "jsmith@g.co"
>>> u1. email
'jsmith@g.co'
```
프로퍼티는 **명령-쿼리 분리 원칙**을 따르기 위한 좋은 방법

명령-쿼리 분리 원칙: 객체의 메서드가 무언가의 상태를 변경하는 커맨드이거나 무언가의 값을 반환하는 쿼리이거나 둘 중에 하나만 수행해야지 둘 다 동시에 수행하면 안된다는 것

프로퍼티를 사용하여 실제 코드가 무엇을 하는지 명확하게 할 수 있음

`@property` 데코레이터는 무언가에 응답하기 위한 쿼리이고 `@<property_name>.setter`는 무언가를 하기 위한 커맨드임.


### 04. 이터러블 객체

파이썬에는 기본적으로 반복 가능한 객체가 있다. 리스트, 튜플, 세트 등 이러한 객체들은 for 루프를 통해 값을 반복적으로 가져올 수 있다.

`for e in myobject:` 형태로 객체를 반복할 수 있는 지  
1. 객체가 `__ next __` 나 `__ iter __` 이터레이터 메서드 중 하나를 포함하는지 여부
2. 객체가 시퀀스이고 `__ len __`과 `__ getitem __`를 모두 가졌는지 여부

를 차례로 검사

#### 4.1 이터러블 객체 만들기

객체를 반복하려고 하면 해당 객체의 `iter()` 함수를 호출함.
`iter()`함수는 해당 객체에 __iter__ 메서드가 있는지 확인하고 있으면, 실행함.

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
```
`StopIteration` 예외가 발생할 때 까지 `next()`를 호출한다.

실행하고 끝의 날짜에 도달한 상태이므로 이후에 호출하면 계속 `StopIteration` 예외가 발생. 

두 개 이상의 `for` 루프에서 이 값을 사용하면 두번째 루프부터는 작동하지 않게 됨.

이 문제를 해결하기 위해 

1. 매번 새로운 `DateRangeIterable`인스턴스를 만드는 것

2. `__iter__` 에서 제너레이터를 사용하는 것

각각의 `for`루프는 `__iter__`를 호출하고, `__iter__`는 다시 제너레이터를 생성

```python
# 컨테이너 이터러블 (container iterable)
class DateRangeContainerIterable:
	 def __init__(self, start_date, end_date):
		 self.start_date = start_date
		 self.end_date = end_date
	 def __iter__(self):
		 current_day = self.start_date
		 while current_day < self.end_date:
			 yield current_day
			 current_day += timedelta(days=1)
```
#### 4.2 시퀀스 만들기

시퀀스는 `__len__`과 `__getitem__`을 구현하고 첫 번째 인덱스 0부터 시작하여 포함된 요소를 한 번에 하나 씩 차례로 가져올 수 있어야 함.

이터러블은 메모리를 적게 사용하지만 시간복잡도는 O(n). 이는 전형적인 트레이드오프임.

시퀀스는 메모리를 많이 사용하지만 시간복잡도는 O(1) 

```python
# 시퀀스로 구현

class DateRangeSequence:
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
```python
>>> s1 = DateRangeSequence(date(2019, 1, 1), date(2019, 1, 5))
>>> for day in s1:
		print(day)
2019-01-01
2019-01-02
2019-01-03
2019-01-04
>>> s1[0]
datetime.date(2019,1,1)
>>> s1[3]
datetime.date(2019,1,4)
>>> s1[-1]
datetime.date(2019,1,4)
```
DateRangeSequence 객체가 모든 작업을 래핑된 객체인 리스트에 위임하기 때문에 호환성과 일관성을 유지할 수 있음.

### 05. 컨테이너 객체

컨테이너는 `__contains__` 메서드를 구현한 객체로 `__contains__` 메서드는 일반적으로 Boolean 값을 반환한다. 이 메서드는 파이썬에서 `in` 키워드가 발견될 때 호출됨.

```python
element in container
# Python이 해석하는 방식
container.__contains__(element)
```

`__contains__` 메서드를 잘 사용하면 코드의 가독성이 정말 높아짐.

``` python
# 2차원 게임 지도에서 특정 위치에 표시하는 예제

class Boundaries:
	def __init__(self, width, height):
		self.width = width
		self.height = height
	
	def __contains__(self, coord):
		x, y = coord
		return 0 <= x < self.width and 0 <= y < self. height

class Grid:
	def __init__ (self, width, height):
		self.width = width
		self.height = height
		self.limits = Boundaries(width, height)

	def __contains__(self, coord):
		return coord in self.limits

def mark_coordinate(gird, coord):
	if coord in grid:
		grid[coord] = MARKED
		
```
지도 자체적으로 `grid`의 영역을 판단하고 일을 더 작은 객체에 위임


### 06. 객체의 동적인 속성

` __ getattr__` 매직 메서드를 사용해 객체에서 속성을 얻는 방법을 제어할 수 있음.

`< myobject >.< myattribute >`를 호출하면 파이썬은 객체의 사전에서 `< myattribute >`를 찾아서 `__getattribute__`를 호출. 

객체에 찾고자 하는 속성이 없을 경우 속성(myattribute)의 이름을 파라미터로 전달하여 `__getattr__`이라는 추가 메서드가 호출됨 

이 값을 사용해서 반환 값을 제어하거나, 새로운 속성을 만들 수 있음.
```python
class DynamicAttributes:
	def __init__(self, attribute):
		self.attribute = attribute

	def __getattr__(self, attr):
		if attr.startswith("fallback_"):
			name = attr.replace("fallback_", "")
			return f"[fallback resolved] {name}"
		raise AttributeError(f"{self.__class__.__name__}에는 {attr} 속성이 없음.")

>>> dyn = DynamicAttributes("value")
>>> dyn.attribute
'value'
# 객체에 있는 속성을 요청하고 그 결과 값을 반환
>>> dyn.fallback_test
'[fallback resolved] test'
# 객체에 없는 fallback_test 라는 메서드를 호출하기 때문에 
# __getattr__이 호출되어 값을 반환
>>> dyn.__dict__["fallback_new"] = "new value"
>>> dyn.fallback_new
'new value'
# dyn.fallback_new = "new value"를 실행한 것과 동일
>>> getattr(dyn, "something", "default")
'default'
# 값을 검색할 수 없는 경우 AttributeError 예외가 발생. 
# 이 예외가 발생하면 기본 값을 반환
```


### 07. 호출형(callable) 객체

매직 메서드 `__call__` 을 사용하면 객체를 일반 함수처럼 호출가능

여기에 전달된 모든 파라미터는 `__call__` 메서드에 그대로 전달되어 객체를 파라미터가 있는 함수처럼 사용하거나 정보를 기억하는 함수처럼 사용할 경우 유용

파이썬은 `object(*args, **kwargs)`와 같은 구문을 `object.__call__(*args, **kwargs)`로 변환함.

```python
from collections import defaultdict

class Callcount:

	def __init__(self):
		self._counts = defaultdict(int)

#defaultdict key값이 없을 경우 미리 지정해 놓은 초기값을 반환하는 dictionary이다.
	
	def __call__(self, argument):
		self._counts[argument] += 1
		return self._counts[argument]

>>> cc = Callcount()
>>> cc(1)
1
>>> cc(2)
1
>>> cc(1)
2
>>> cc(1)
3   
>>> cc("something")
1 
# 동일한 값으로 몇번 호출 되었는지 반환하는 객체
```
### 08. 파이썬에서 유의할 점

잠재적인 문제를 피할 수 있는 관용적인 코드

#### 8.1 변경 가능한(mutable)파라미터의 기본 값

변경 가능한 객체를 함수의 기본 인자로 사용하면 안됨.
```python
def wrong_user_display(user_metadata: dict = {"name":"John","age":30}):
	name = user_metadata.pop("name")
	age = user_metadata.pop("age")
	return f"{name} ({age})"

# pop()은 리스트의 맨 마지막 요소를 돌려주고 그 요소는 삭제한다.
```
두 가지 문제가 존재
1. 변경 가능한 인자 사용
2. 함수의 본문에서 가변 객체를 수정하여 부작용 발생

위의 함수는 인자를 사용하지 않고 처음 호출할 때만 동작, 다음  호출 시 명시적으로 `user_metadata`를 전달하지 않으면 `KeyError`가 발생

```python
>>> wrong_user_display()
'John (30)'
>>> wrong_user_display({"name": "Jane", "age": 25})
'Jane (25)'
>>> wrong_user_display()
...
KeyError: 'name'
```
기본 초기 값으로 `None`을 사용하고 함수 본문에서 기본 값을 할당해서 수정할 수 있음.
#### 8.2 내장(built-in)타입 확장

리스트, 문자열, 사전과 같은 내장 타입을 확장하는 올바른 방법은 `collections` 모듈을 사용하는 것.

입력 받은 숫자를 접두어가 있는 문자열로 변환하는 리스트
```python
class BadList(list):
	def __getitem__(self, index):
		value = super().__getitem__(index)
		if index % 2 == 0:
			prefix = "짝수"
		else:
			prefix = "홀수"
		return f"[{prefix}] {value}"
```
결국 리스트이므로 반복해보면 원하는 결과가 나오지 않음.
```python
>>> b1 = BadList((0,1,2,3,4,5))
>>> b1[0]
'[짝수] 0'
>>> b2[1]
'[홀수] 1'
>>> "".join(b1)
Traceback (most recent call last):
TypeError: sequence item 0: expected str instance, int found
# join은 문자열 리스트를 반복하는 함수
```
이식 가능하고 호환 가능한 코드를 위해 UserList에서 확장하여 수정
```python
from collections import UserList

class GoodList(UserList):
	def __getitem__(self, index):
		value =  super().__getitem__(index)
		if index % 2 == 0:
			prefix = "짝수"
		else:
			prefix = "홀수"
		return f"[{prefix}] {value}"
```
```python
>>> g1 = GoodList((0,1,2))
>>> g1[0]
'[짝수] 0'
>>> g1[1]
'[홀수] 1'
>>> "; ".join(g1)
'[짝수] 0; [홀수] 1; [짝수] 2'
```
