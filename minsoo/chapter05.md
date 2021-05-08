## Chapter 05. 데코레이터
#### 목표 
##### 1) 파이썬에서 데코레이터가 동작하는 방식 이해
##### 2) 함수와 클래스에 적용되는 데코레이터를 구현 방법 학습
##### 3) 일반적인 실수를 피하여 데코레이터를 효과적으로 구현하는 방법 학습
##### 4) 데코레이터를 활용한 코드 중복을 회피 (DRY 원칙 준수)
##### 5) 데코레이터를 활용한 관심사의 분리
---
### 01. 파이썬의 데코레이터
##### 함수와 메서드의 기능을 쉽게 수정하기 위한 수단
##### classmethod & staticmethod : https://wikidocs.net/21054
```python
# 함수에 변형을 할 때
def original(...):
	...
original = modifier(original)

# 데코레이터를 활용
@modifier // 데코레이터
def original(...): // 데코레이팅된 함수 or 래핑된 객체
	...
```
##### 데코레이터 이후에 나오는 것을 파라미터로하고 그 결과 값을 반환하게 함
---
#### 1.1  함수 데코레이터
##### 파라미터 유효성 체크, 사전조건 검사, 기능 전체를 새롭게 정의, 서명을 변경 하는 등의 작업 가능

```python
# 특정 예외에 대해서 특정 횟수만큼 재시도하는 데코레이터
class ControlledException(Exception):
	""" 도메인에서 발생하는 일반적인 예외"""
def retry(operation):
	@wraps(operation)
	def wrapped(*args, **kwargs):
		last_raised = None
		RETRIES_LIMIT = 3
		for _ in range(RETRIES_LIMIT): # '_' for 문 안에서 해당 변수에 관심이 없음을 표현
 			try:
				return operation(*args, **kwargs) 
			except ControlledException as e:
				logger.info("retrying %s", operation.__qualname__)
				last_raised = e
			raise last_raised
	return wrapped
```
##### @wraps  관련 내용 :  https://cjh5414.github.io/wraps-with-decorator/
##### retry  데코레이터는 파라미터가 필요 없으므로 어떤 함수에도 쉽게 적용할 수 있음
```python
# timeout 같은 예외가 발생할 경우 여러 번 호출을 반복하는 retry 로직
@retry
def run_operation(task):
	"""실행 중 예외가 발생할 것으로 예상되는 특정 작업을 실행"""
	return task.run()
```
##### @retry → run_operation = retry(run_operation)

#### 1.2 클래스 데코레이터
##### 파라미터로 클래스를 받음
##### 클래스 데코레이터의 장점
###### 1) 클래스 데코레이터는 코드 재사용과 DRY 원칙의 모든 이점을 공유/ 여러 클래스가 특정 인터페이스나 기준을 따르도록 강제할 수 있음
###### 2) 당장은 작고 간단한 클래스를 생성하고 나중에 데코레이터로 기능을 보강할 수 있음
###### 3) 어떤 클래스에 대해서는 유지보수 시 데코레이터를 사용해 기존 로직을 훨씬 쉽게 변경할 수 있음
```python
# 모니터링을 플랫폼을 위한 이벤트 시스템
# 각 이벤트의 데이터를 변환하여 외부 시스템으로 보내야 함
# 따라서, 이벤트마다 직렬화 방법을 정의한 클래스를 만드는 것

class LoginEventSerializer:
	def __init__(self, event):
		self.event = event
	
	def serialize(self) -> dict:
		return {
			"username" : self.event.username,
			"password" : "**민감한 정보 삭제**",
			"ip" : self.event.ip,
			"timestamp" : self.event.timestamp.strftime("%Y-%m-%d %H:%M"),
			}
	
class LoginEvent:
	SERIALIZER = LoginEventSerializer
	
	def __init__(self, username, password, ip, timestamp):
		self.username = username
		self.password = password
		self.ip = ip
		self.timestamp = timestamp
	
	def serialize(self) -> dict:
		return self.SERIALIZER(self).serialize()

```
##### 시스템을 확장할수록 다음과 같은 문제 발생
###### 1) 클래스가 너무 많아짐 : 1대1 매핑
###### 2) 충분히 유연하지 않음  : 
###### 3) 표준화

```python
# 이벤트 인스턴스와 변형 함수를 필터로 받아서 동적으로 객체를 만드는 것
from datetime import datetime

def hide_field(field) -> str:
	return "**민감한 정보 삭제**"

def format_time(field_timestamp: datetime) -> str:
	return field_timestamp.strftime("%Y-%m-%d %H:%M")

def show_original(event_field):
	return event_field

class EventSerializer:
	def __init__(self, serialization_fields: dict) -> None:
		self.serialization_fields = serialization_fields

	def serialize(self, event) -> dict:
		return {
			field: transformation(getattr(event, field))
			for field, transformation in
			self.serialization_fields.items()
			}

class Serialization:
	def __init__(self, **transformations):
		self.serializer = EventSerializer(transformations)
	
	def __call__(self, event_class):
		def serialize_method(event_instance):
			return self.serializer.serialize(event_instance)
		event_class.serialize = serialize_method
		return event_class

@Serialization(
	username = show_original,
	password = hide_field,
	ip = show_original,
	timestamp = format_time,
	)
class LoginEvent:
	def __init__(self, username, password, ip, timestamp):
		self.username = username
		self.password = password
		self.ip = ip
		self.timestamp = timestamp
```
##### 클래스 데코레이터를 사용하여 다른 클래스의 코드를 확인하지 않고도 각 필드가 어떻게 처리되는지 쉽게 알수 있음.
```python
from dataclasses import dataclass
from datetime import datetime

@Serialization(
	username = show_original,
	password = hide_field,
	ip = show_original,
	timestamp = format_time,
)
@dataclass
class LoginEvent:
	username: str
	password: str
	ip: str
	timestamp: datetime
```
#### 1.3 다른 유형의 데코레이터
##### 데코레이터는 함수, 클래스 뿐만 아니라 제너레이터나 코루틴, 이미 데코레이터된 객체에도 적용이 가능
#### 1.4 데코레이터에 인자 전달
##### 파라미터를 갖는 데코레이터를 구현하는 방법
###### 1) 간접 참조(indirection)를 통해 새로운 레벨의 중첩 함수 만들기
###### 2) 데코레이터를 위한 클래스 만들기 → 가독성이 좋음

#### 1.4.1 중첩 함수의 데코레이터
##### 함수를 파라미터로 받아서 함수를 반환하는 함수 → '고차 함수 (higher-order function)'
##### 데코레이터를 파라미터에 전달하는 다른 수준의 간접 참조가 필요
###### 1) 첫 번째 함수 : 파라미터를 받아서 내부 함수에 전달
###### 2) 두 번째 함수 : 데로케이터가 될 함수
###### 3) 세 번째 함수 : 데코레이팅의 결과를 반환하는 함수
##### 즉, 세 단계의 중첩 함수가 필요
```python
# 재시도 기능
# 인스턴트마다 재시도 회수를 지정 & 파라미터에 기본 값 추가
@retry(arg1, arg2, ...)
=> <original_function> = retry(arg1, arg2, ....)(<original_function>)

RETRIES_LIMIT = 3

def with_retry(retries_limit = RETRIES_LIMIT, allowed_execptions=None):
	allowed_exceptions = allowed_exceptions or (ControlledException,)

def retry(operation):
	
	@wraps(operation)
	def wrapped(*args, **kwargs):
		last_raised = None
		for _ in range(RETRIES_LIMIT): # '_' for 문 안에서 해당 변수에 관심이 없음을 표현
 			try:
				return operation(*args, **kwargs)
			except ControlledException as e:
				logger.info("retrying %s", operation.__qualname__)
				last_raised = e
			raise last_raised
	return wrapped
return retry

# 위의 데코레이터를 함수에 적용한 예

@with_retry()
def run_operation(task):
	return task.run()
@with_retry(retries_limit=5)
def run_with_custom_retries_limit(task):
	return task.run()
@with_retry(allowed_exeptions = (AttributeError,))
def run_with_custom_exceptions(task):
	return task.run()
@with_retry(
	retries_limit = 4, allowed_exceptions = (ZeroDivisionError, AttributeError)
)
def run_with_custom_parameters(task):
	return task.run()
	
```
#### 1.4.2 데코레이터 객체
##### 클래스를 사용하여 데코레이터를 정의
```python
# __init__ 메서드에 파라미터를 전달한 다음 
# __call__이라는 매직 메서드에서 데코레이터의 로직을 구현

class WithRetry:

	def __init__(self, retries_limit = RETRIES_LIMIT, allowed_exceptions=None):
		self.retries_limit = retries_limit
		self.allowed_exceptions = allowed_exceptions or (ControlledException,)

	def __call__(self, operation):
	
		@wraps(operation)
		def wrapped(*args, **kwargs):
			last_raised = None
	
			for _ in range(self.retries_limit):
			try:
				return operation(*args, **kwargs)
			except self.allowed_exceptions as e:
				logger.info("retrying %s due to %s", operation, e)
				last_raised = e
			raise last_raised
	
	return wrapped
	
# 사용방법
@WithRetry(retries_limit = 5)
def run_with_custom_retries_limit(task):
	return task.run()

```
##### 동작하는 순서
###### 1) @ 연산 전에 전달된 파라미터를 사용해 데코레이터 객체를 생성
###### 2) 데코레이터 객체는 init 메서드에서 정해진 로직에 따라 초기화를 진행
###### 3) @ 연산이 호출
###### 4) 데코레이터 객체는 run_with_custom_retries_limit 함수를 래핑하여 call 메서드를 호출

#### 1.5 데코레이터 활용 우수 사례
##### 1) 파라미터 변환
##### 2) 코드 추적
##### 3) 파라미터 유효성 검사
##### 4) 재시도 로직 구현
##### 5) 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화
---
#### 1.5.1 파라미터 변환
##### 파라미터의 유효성 검사 가능
##### DbC의 원칙에 따라 사전조건 or 사후조건을 강제할 수 있음
##### 유사한 객체를 반복적으로 생성 or 추상화를 위해 유사한 변형을 반복 하는 경우 유용
#### 1.5.2 코드 추적
##### 추척 (tracing)
###### 실제 함수의 실행 경로 추척
###### 함수 지표 모니터링
###### 함수의 실행 시간 측정
###### 언제 함수가 실행되고 전달된 파라미터는 무엇인지 로깅

### 02. 데코레이터의 활용 - 흔한 실수 피하기
##### 효과적인 데코레이터를 만들기 위해 피해야 할 몇 가지 공통된 사항
#### 2.1 래핑된 원본 객체의 데이터 보존
##### 원본 함수의 일부 프로퍼티나 또는 속성을 유지하지 않으면 부작용을 유발함
##### qualname : https://www.python2.net/questions-204000.htm
```python
# 함수가 실행될 때 로그를 남기는 데코레이터
def trace_decorator(function):
	def wrapped(*args, **kwargs):
		logger.info("%s 실행", function.__qualname__)
		return function(*args, **kwargs)

	return wrapped
# 원본 함수의 정의와 비교해 함수가 수정되지 않은 것 처럼 보인다.
@trace_decorator
def process_account(acoount_id):
	"""id별 계정 처리"""
	logger.info("%s 계정 처리", account_id)
	...
# 코드 결함이나 docstring을 변경하는 경우 확인

>>> help(process_account)
Help on function wrapped in module decorator_wraps_1:

wrapped(*args, **kwargs)
>>> print(process_account.__qualname__)
'trace_decorator.<locals>.wrapped'
	
```
##### 데코레이터가 원본 함수를 wrapped로 불리는 새로운 함수로 변경
##### 이 경우, 개별 함수를 확인하고 싶은 경우에 실제 실행된 함수를 알 수 없게 됨
##### 다른 문제점은, 함수에 테스트와 함께 docstring을 작성한 경우 데코레이터에 의해 덮어짐
##### 이를 수정하기 위해 wrapped 함수에 @wraps 데코레이터를 적용하면 됨
``` python
def trace_decorator(function):
	@wraps(function)
	def wrapped(*args, **kwargs):
		logger.info("running %s", function.__qualname__)
		return function(*args, **kwargs)
	
	return wrapped
# help 함수 확인
>>> Help on function process_account in module decorator_wraps_2:
process_account(account_id)
	id별 계정 처리

# __qualname__ 확인
>>> process_account.__qualname__
'process_account'
```
#### 2.2 데코레이터 부작용 처리
##### 데코레이터 함수가 되기 위한 조건 : 가장 안쪽에 정의된 함수 → 임포트에 문제가 발생할 수 있음
#### 2.2.1 데코레이터 부작용의 잘못된 처리
```python
# 함수의 실행과 실행 시간을 로깅하는 데코레이터

def traced_function_wrong(function):
	logger.info("%s 함수 실행", function)
	start_time = time.time()
	
	@functools.wraps(function)
	def wrapped(*args, **kwargs):
		result = function(*args, **kwargs)
		logger.info(
			"함수 %s의 실행시간: %.2fs", function, time.time() - start_time
			)
			return result
	return wrapped

@traced_function_wrong
def process_with_delay(callback, delay=0):
	time.sleep(delay)
	return callback()
# 함수를 여러 번 임포트할 경우
>>> from decorator_side_effects_1 import process_with_delay
INFO:<function process_with_delay at 0x...> 함수 실행

# 함수가 호출되지 않았으므로 로그가 남지 않아야 함
>>> main()
...
INFO:<function process_with_delay at 0x>의 실행시간: 8.67s
>>> main()
...
INFO:<function process_with_delay at 0x>의 실행시간: 13.39s
>>> main()
...
INFO:<function process_with_delay at 0x>의 실행시간: 17.01s
# 실행할 때 마다 오래 걸림
```
##### @traced_function_wrong 은 모듈을 임포트할 때 실행 됨 → 잘못된 시점에 기록
##### 래핑된 함수 내부로 코드를 이동하면 해결
```python
def traced_function(function):
	@functools.wraps(function)
	def wrapped(*args, **kwargs):
		logger.info("%s 함수 실행", fuction.__qualname__)
		start_time = time.time()
		result = function(*args, **kwargs)
		logger.info(
			"function %s took %.2fs",
			function.__qualname__,
			time.time() - start_time
			)
			return result
		return wrapped
```
#### 2.2.2 데코레이터 부작용의 활용
##### 모듈의 공용 레지스트리에 객체를 등록하는 경우
```python
# 이전 이벤트 시스템에서 일부 이벤트만 사용하려는 경우
# 각 클래스마다 처리여부에 데코레이터를 사용해 명시적으로 표시 가능
# UserLoginEvent & UserLogoutEvent 만 처리한다고 가정

EVENTS_REGISTRY = {}

def register_event(event_cls):
	"""모듈에서 접근 가능하도록 이벤트 클래스를 레지스트리에 등록"""
	EVENTS_REGISTRY[event_cls.__name__] = event_cls
	return event_cls

class Event:
	"""기본 이벤트 객체"""
class UserEvent:
	TYPE = "user"

@register_event
class UserLoginEvent(UserEvent):
	"""사용자가 시스템에 접근했을 때 발생하는 이벤트"""
	
@register_event
class UserLogoutEvent(UserEvent):
	"""사용자가 시스템에서 나갈 때 발생하는 이벤트"""
	
>>> from decorator_side_effects_2 import EVENTS_REGISTRY
>>> EVENTS_REGISTRY
{'UserLoginEvent': decorator_side_effects_2.UserLoginEvent,
'UserLogoutEvent': decorator_side_effects_2.UserLogoutEvent}
```

#### 2.3 어느 곳에서나 동작하는 데코레이터 만들기
##### *args 와 **kwargs 서명을 사용하여 데코레이터를 정의하면 모든 경우에 사용 가능
```python
# 연결 문자열을 받아서 데이터베이스에 연결하고 DB 연산을 수행하는 객체

import logging
from fuctools import wraps

logger = logging.getLogger(__name__)

class DBDriver:
	def __init__(self, dbstring):
		self.dbstring = dbstring

	def execute(self, query):
		return f"{self.dbstring} 에서 쿼리 {query} 실행"

def inject_db_driver(function):
	"""데이터베이스 dns 문자열을 받아서 DBDriver 인스턴스를 
	생성하는 데코레이터
	"""
	@wraps(function)
	def wrapped(dbstring):
		return function(DBDriver(dbstring))
	return wrapped

@inject_db_driver
def run_query(driver):
	return driver.execute("test_function")

>>> run_query("test_OK")
'test_OK 에서 쿼리 test_function 실행'
```
##### 같은 기능을 하는 데코레이터를 클래스 메소드에 재사용 할 때
```python
class DataHandler:
	@inject_db_driver
	def run_query(self, driver):
		return driver.execute(self.__class__.__name__)

>>> DataHandler().run_query("test_fails")
Traceback (most recent call last):
...
TypeError: wrapped() takes 1 positional argument but 2 were given

# 하나의 파라미터만 받도록 설계된 이 데코레이터에
# 연결 문자열 자리에 self를 전달하고 
# 두 번째 파라미터에는 아무것도 전달하지 않아서 에러가 발생
```
##### 해결책 : 데코레이터를 클래스 객체로 구현하고 get 메서드를 구현한 디스크립터 객체를 만드는 것 
```python
from functools import wraps
from types import MethodType

class inject_db_driver:
	"""문자열을 DBDriver 인스턴스로 변환하여
	래핑된 함수에 전달"""

	def __init__(self, function):
		self.function = function
		wraps(self.function)(self)

	def __call__(self, dbstring):
		return self.function(DBDriver(dbstring))

	def __get__(self, instance, owner):
		if instance is None:
			return self
		return self.__class__(MethodType(self.function, instance))
```
##### 호출할 수 있는 객체를 메서드에 다시 바인딩한다
