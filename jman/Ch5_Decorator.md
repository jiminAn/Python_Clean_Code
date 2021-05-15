> 해당 게시글은 <파이썬 클린 코드 : 유지보수가 쉬운 파이썬 코드를 만드는 비결, 마리아노 아나야 지음> 책의 5장을 참고하여 작성되었습니다

# Chapter 05. 데코레이터를 사용한 코드 개선

해당 장에서는 데코레이터가 SOLID 원칙을 준수하는데 어떻게 도움이 되는지 살펴보도록 하자

### 5장의 목표

- 파이썬에서 데코레이터의 동작 방식 이해
- 함수와 클래스에 적용되는 데코레이터를 구현하는 방법 배우기
- 일반적인 실수를 피하여 데코레이터를 효과적으로 구현하는 방법 배우기
- 데코레이터를 활용한 코드 중복 회피(DRY 원칙 준수)
- 데코레이터를 활용한 관심사의 분리
- 좋은 데코레이터 사례
- 데코레이터가 좋은 선택이 될 수 있는 일반적인 상황, 관용구, 패턴



## 파이썬의 데코레이터

------

### Decorator란?

classmethod나 staticmethod 같은 함수를 사용하여 원래 메서드의 정의를 변형할 경우 추가 코드가 필요하고 함수의 원래 정의를 수정해야하므로 **함수와 메서드의 기능을 쉽게 수정하기 위한 수단**으로 등장한 개념

- 아래의 예제는 original 함수에 그 기능을 약간 수정한 modifer함수가 있는 경우이다

```python
def original(...):
  ...
  original = modifier(original)
```

- 이 경우 아래와 같은 이유로 혼란스럽고 오류가 발생하기 쉽고 번거로운 코드임
  - 함수를 재할당하는 것을 잊어버리는 경우
  - 함수 정의가 멀리 떨어져 있는 경우

- 이러한 이유로 새로운 구문이 추가되었으며 앞의 예제는 다음과 같이 작성하면 된다

```python
@modifier
def original(...):
  ...
```

**정리**

- 즉, 데코레이터는 데코레이터 이후에 나오는 것을 첫 번째 파라미터로 하고 결과값을 반환하게 하는 Syntax sugar임
- 위의 예제에서 말하는 modifier는 파이썬 용어로 데코레이터라고 함
- original을 데코레이팅된 함수 또는 래핑된 객체라고 한다
  - decorated function or wrapped obeject

- 원래는 함수와 메서드를 위해 고안되었으나 실제로는 모든 종류의 객체에 적용 가능하다

  - 따라서 해당 장에서는 함수와 메서드, 제너레이터, 클래스에 데코레이터를 적용하는 방법을 살펴본다

- 주의 : 데코레이터 디자인 패턴과 혼동하지 말 것

  

### 함수 데코레이터

함수에 데코레이터를 사용할 경우 어떤 종류의 로직이라도 적용 가능

- 파라미터의 유효성 혹은 사전조건 검사
- 기능 전체를 새롭게 정의
- 서명 변경 
- 원래 함수의 결과 캐시 

아래의 예제는 도메인의 특정 예외에 대해서는 특정 횟수만큼 재시도하는 데코레이터이다

```python
class ControlledException(Exception):
  """도메인에서 발생하는 일반적인 예외"""
  
  def retry(operation):
    @wraps(operation) # wrap의 자세한 설명은 '데코레이터 활용-흔한 실수 피하기'에서 다룸
    def wrapped(*arg, **kargs):
      last_raised = None
      RETRIES_LIMIT = 3
      for _ in range(RETRIES_LIMIT): # _ 는 무시해도 되는 값(관습표현)
        try:
          return operation(*args, **kwargs)
        except ControlledException as e:
          logger.info("retrying %s", operation.__qualname__)
          last_raised = e
          
     return wrapped
```

retry 데코레이터는 파라미터가 필요 없으므로 어떤 함수에도 쉽게 적용 가능. 다음은 적용 예제이다

```python
@retry
def run_operation(task):
  """실행 중 예외가 발생할 것으로 예상되는 특정 작업을 실행"""
  return task.run()
```

- `@retry` : 파이썬에서 `run_operation = retry(run_operation)`을 실행하게 해주는 syntax sugar임
- 해당 예제에서는 timeout 같은 예외가 발생할 경우 여러번 호출을 반복하는 retry로직을 데코레이터로 만드는 방법을 살펴봄



### 클래스 데코레이터

함수 데코레이터와 차이점은 데코레이터 함수의 파라미터로 클래스를 받는다는 점이다

**클래스 데코레이터의 장점**

- 코드 재사용과 DRY 원칙의 모든 이점을 공유
  - 여러 클래스가 특정 인터페이스나 기준을 따르도록 강제할 수 있다
  - 여러 클래스에 적용할 검사를 데코레이터에서 한 번만 하면 됨
- 당장은 작고 간단한 클래스를 생성하고 나중에 데코레이터로 기능 보강 가능
- 유지보수 시 데코레이터를 사용해 기존 로직을 훨씬 쉽게 변경 가능
  - 메타클래스와 같은 방법은 복잡하므로 권장되지 않음

데코레이터가 유용하게 사용될 수 있는 예제를 살펴보자 

예제1: 각 이벤트마다 직렬화 방법을 정의한 클래스

```python
class LoginEventSerializer:
  """로그인 이벤트에 직접 매핑할 클래스"""
  def __init__(self, event):
    self.event = event
    
  def serialize(self) -> dict:
    return{
      ... # username, password, ip
      "timestamp":self.event.timestamp.strftime("...")
    }
  
class LoginEvent:
  SERIALIZER = LoginEventSerializer
  
  def __init__(self, username, password, ip, timestamp):
    ... # username,password,ip초기화
    self.timestamp = timestamp
  
  def serialize(self) -> dict:
    return self.SERAILIZER(self).serialize()
```

해당 코드의 문제점

- 클래스가 너무 많아짐 

  : 이벤트 클래스와 직렬화 클래스가 1:1로 매핑되어 직렬화 클래스가 점점 많아지게 됨

- 유연하지 않음(코드 재사용성)

  :만약 패스워드를 가진 다른 클래스에서도 이 필드를 숨기려 한다면 함수로 분리한 다음 여러 클래스에서 호출해야 함

- 표준화 문제

  : `serialize()` 메서드는 모든 이벤트 클래스에 있어야 함 즉, 상속을 제대로 사용하지 않음



예제2: 이벤트 인스턴스와 변형 함수를 필터로 받아서 동적으로 객체를 만드는 방법

```python
def hide_field(field) -> str:
  return "**민감한 정보 삭제**"

def format_time(field_timestamp: datetime) -> str:
  return filed_timestamp.strftime("...")

def show_original(event_field):
  return event_field
```

```python
class EventSerializer:
    def __init__(self, serialization_fields: dict) -> None:
        self.serialization_fields = serialization_fields
    
    def serialize(self, event) -> dict:
        return{
            field: transformation(getattr(event,field))
            for field, transformation in self.serialization_fields.items()
      }
 
class Serialization:
    def __init__(self, **transformation):
        self.serializer = EventSerializer(transformation)
        
    def __call__(self, event_class):
        def serialize_method(event_instance):
            return self.serializer.serialze(event_instance)
        event_class.serialize = serialze_method
        return event_class
      
```

```python
@Serialization(username = show_original,password = hide_field,ip = show_original,timestamp = format_time) # 확인
class LoginEvent:
  def __init__(self, username, password, ip, timestamp):
    ... # username,password,ip초기화
    self.timestamp = timestamp
```

해당 코드의 장점

- 데코레이터를 사용해 다른 클래스의 코드를 확인하지 않고도 각 필드가 어떻게 처리되는지 알 수 있음
  - username, ip는 수정되지 않음
  - passowrd 필드는 숨겨짐
  - Timestamp는 포매팅 됨
- 개별 클래스에 `seriallize()`메서드를 정의하거나 믹스인 확장 필요 없음
  - 데코레이터에만 추가하면 됨



예제3: 파이썬 3.7이상 버전에서는 아래와 같이 간단하게 작성 가능

```python
from dataclasses import dataclass

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
    ip : str
    timestamp: datetime
```



### 다른 유형의 데코레이터 

앞서 함수와 메서드, 클래스에서 데코레이터가 적용되는 예제를 보았는데 다른 유형의 데코레이터 역시 존재 함

- 제너레이터와 코루틴(7장에서 자세히 다룰 예정)
  - 새로 생성된 제너레이터에 데이터를 보내기 전에 `next()` 를 호출하여 다음 yield문으로 넘어가야 함(에러가 발생하기 쉬움)
  - 제너레이터를 파라미터로 받아 `next()`를 호출한 다음 다시 제너레이터를 반환하는 데코레이터를 만들면 쉽게 해결 됨
- 데코레이터된 객체에서의 데코레이트(이 경우 데코레이터는 스택형태로 쌓임)



### 데코레이터에 인자 전달

파라미터를 갖는 데코레이터 구현 방법

- 간접 참조(indirection)을 통해 새로운 레벨의 중첩 함수 생성
- 데코레이터를 위한 클래스 생성
  - 첫 번째 방법보다 가독성이 좋음

방법 1: **중첩 함수의 데코레이터**

- 데코레이터는 함수를 파라미터로 받아서 함수를 반환하는 함수(higher-order function)

  - 실제로는 데코레이터의 본문에 정의된 함수가 호출

  ```python
  @retry(arg1, arg2, ...)
  ```

  - @구문은 데코레이팅 객체에 대한 연산 결과를 반환하는 것이기 때문에 위의 코드는 의미상 아래와 같음

  ```python
  <original_function> = retry(arg1, arg2, ....)(<original_function>)
  ```

  - 원하는 재시도 횟수외에도 제어하려는 예외 유형을 아래와 같이 반영할 수 있다

  ```python
  RETRIES_LIMIT = 3
  # 데코레이터의 파라미터를 받는 함수
  def with_retry(retries_limit=RETRIES_LIMIT, allowed_exceptions=None): 
    allowed_exceptions = allowed_exceptions or (ControlledException)
    
    # 전달된 파라미터를 로직에서 사용하는 클로저
    def retry(operation):
      @wraps(operation)
      def wraaped(*args, **kargs):
        last_raised = None
        for _ in range(retries_limit):
          ...#try...excecp e문
        raise last_raised
      return wrapped
      
    return retry
  ```

  - 위에서 정의한 데코레이터를 아래와 같이 함수에 적용할 수 있음

  ```python
  @with_retry()
  def ...:
    ....
  ```

  



방법2 : **데코레이터 객체**

- 클래스를 사용하여 보다 깔끔하게 데코레이터를 정의할 수 있다

- `__init__`메서드에 파라미터를 전달 후 `__call__`이라는 매직 메서드에서 데코레이터의 로직을 구현하면 됨

  - 데코레이터 객체는 아래와 같이 구현할 수 있음

  ```python
  class WithRetry:
    def __init__(self, retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
      self.retries_limit = retries_limit
      self.allowed_exceptions = allowed_exceptions or (ControlledExecption,)
      
   def __call__(self, operation):
    	@wraps(operation)
      def wraaped(*args, **kargs):
        last_raised = None
        for _ in range(retries_limit):
          ...#try...excecp e문
        raise last_raised
    return wrapped  
  ```

  - 사용방법은 이전과 유사
    - 전달된 파라미터를 사용해 데코레이터 객체 생성
      - `__init__`메서드에서 정해진 로직에 따라 초기화 진행
    - @연산 호출
      - 데코레이터 객체는 `run_with_custom_retries_limit` 함수를 래핑하여 `__call__` 매직 메서드 호출
      - 매직메서드는 원본 함수를 래핑하여 원하는 로직이 적용된 새로운 함수를 반환

  ```python
  @WithRetry(retries_limit = 5) 
  def run_with_custom_retries_limit(task):
    return task.run()
  ```

  ​	

  ### 데코레이터 활용 우수 사례

  데코레이터가 좋은 선택이 될 수 있는 경우들

  - 파라미터 변환
    - 일반적으로 파라미터를 다룰 때 데코레이터를 많이 사용함
      - 데코레이터를 사용하여 파라미터의 유효성 검사 가능
      - DbC원칙에 따라 사전조건/사후조건 강제 가능
    - 특히 유사한 객체를 반복 생성하거나 추상화를 위해 유사한 변형을 반복하는 경우 데코레이터를 통해 쉽게 처리 가능
  - 코드 추적
    - code tracing이란 모니터링 하고자 하는 함수의 실행과 관련
  - 파라미터 유효성 검사
  - 재시도 로직 구현
  - 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화





## 데코레이터 활용 - 흔한 실수 피하기

------

해당 섹션에서는 효과적인 데코레이터를 만들기 위해 피해야 할 몇가지 공통된 상황을 살펴보도록 하자

### 래핑된 원본 객체의 데이터 보존

원본 함수의 일부 프로퍼티나 속성을 유지하지 않아 원치않는 부작용을 유발함 

```python
"""함수가 실행 될 때 로그를 남기는 데코레이터 사용"""
def trace_decorator(function):
  def wrapped(*args, **kwargs):
    logger.info("%s 실행", function.__qualname__)
    return function(*ars, **kwargs)
  return wrapped

@trace_decorator
def process_account(account_id):
  """id별 계정 처리"""
  logger.info("%s 계정 처리", account_id)
```

- 위의 코드의 경우 데코레이터가 실제로 원본 함수를 wrapped라 불리는 새로운 함수로 변경했기 때문에 원본함수의 이름이 아닌 새로운 함수의 이름을 출력하게 됨
  - 즉, 이 데코레이터를 이름이 다른 여러 함수에 적용하더라도 결국은 wrapped라는 이름만 출력하게 됨
  - 실제 실행된 함수를 알 수 없으므로 디버깅이 더 어려워지는 문제 발생
- docstring을 함께 작성한 경우 데코레이터에 의해 덮어씌어짐



function 파라미터 함수를 래핑하는 것이라고 알려주도록 코드 수정

- docstring에 포함된 단위 테스트 기능이 복구 됨
- `wraps` 데코레이터를 사용해 `__wrapped__` 속성을 통해 수정되지 않은 원본에도 접근할 수 있게 됨

```python
def trace_decorator(function):
  @wraps(function) # function 파라미터 함수를 래핑하는 것을 알려주는 코드
  def wrapped(*args, **kwargs):
    logger.info("%s 실행", function.__qualname__)
    return function(*ars, **kwargs)
  return wrapped

@trace_decorator
def process_account(account_id):
  """id별 계정 처리"""
  logger.info("%s 계정 처리", account_id)
```



일반적인 데코레이터의 경우 아래와 같은 구조에 따라 `functools.wraps` 를 추가하면 됨

```python
def decorator(original_fuction):
  @wraps(original_fuction)
  def decorated_fuction(*args, **kwargs):
    # 데코레이터에 의한 수정 작업 ...
    return original_fuction(*args, **kwargs)
  return decorated_function
```



### 데코레이터 부작용 처리

데코레이터의 부작용을 허용하는 경우도 있으나 의심이 된다면 원칙에 따라 그렇게 하지 않기로 결정 해야 함. 즉 데코레이터의 부작용을 최소화하는 방법에 대해 알아보자

1. 데코레이터 부작용의 잘못된 처리

   - 래핑된 함수 바깥에 추가 로직을 구현하는 것이 바람직하지 않은 예

   ```python
   # function execution & logging the execution time
   def traced_function_wrong(function):
     logger.info("%s function execution")
     start_time = time.time()
     
     @functools.wraps(function)
     def wrapped(*args, **kwargs):
       result = function(*args, *kwargs)
       logger.info(
       	"function %s \'s execuation time", function, time.time() - start_time
       )
       return result
    return wrapped
   ```

   ```python
   @traced_function_wrong
   def process_with_delay(callback, delay=0):
     time.sleep(delay)
     return callback()
   ```

   일반 함수에서 위 데코레이터를 적용하면 문제없이 작동하나 아래와 같이 중요한 버그가 존재

   - `process_with_delay`함수를 import만 해도 로그가 실행됨
   - `@traced_function_wrong` 은 `process_with_delay = traced_function_wrong(process_with_delay)`로 즉, 해당 문장은 모듈을 import 할 때마다 실행되게 됨

   

   이는 아래와 같이 수정하면 위의 버그들을 해결할 수 있다

   ```python
   def traced_function(function):
     @functools.wraps(function)
     
     def wrapped(*args, **kwargs):
       logger.info("%s 함수 실행", function.__qualnale__)
       start_time = time.time()
       result = function(*args, *kwargs)
       logger.info(
         "function %s took %.2fs",
         function.__qualname__,
         time.time() - start_time
       )
       return result
    return wrapped
   ```

   

2. 데코레이터 부작용의 활용 

   이러한 부작용을 의도적으로 사용하여 실제 실행이 가능한 시점까지 기다리지 않는 경우도 있음

   - 많은 웹 프레임워크나 널리 알려진 라이브러리들은 이 원리로 객체를 노출하거나 활용하고 있음
   - 데코레이터는 래핑된 객체를 변경하지도, 동작 방식을 수정하지도 않고 원래 함수 그대로를 반환
   - **래핑된 객체를 일부 수정하거나 수정하는 내부 함수를 정의했다고 해도 결과 객체를 외부에 노풀하는 코드가 있어야 함**
     - 결과 객체가 같은 클로저에 있지 않고 외부 스코프에 있음
     - 데코레이팅한 것만으로 스스로 결과 객체에 저장됨

   

   예:) 모듈의 공용 레지스트리에 객체를 등록하는 경우

   - 이전 이벤트 시스템에서 일부 이벤트만 사용하려는 경우를 살펴보자

     - 이벤트 계층 구조의 중간에 가상의 클래스를 만들고 일부 파생 클래스에 대해서만 이벤트를 처리하도록 할 수 있음
     - 각 클래스마다 처리 여부에 플래그 표시를 하는 대신에 데코레이터를 사용해 명시적으로 표시 할 수 있음

     ```python
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
     ```



### 어느 곳에서나 동작하는 데코레이터 만들기

데코레이터를 만들 때 재사용을 고려하여 함수 뿐 아니라 메서드에서도 동작하길 바람

- `*args`, `*kwargs` 서명을 사용하여 데코레이터를 정의하면 모든 경우에 사용할 수 있음
- 단, 원래 함수의 서명과 비슷하게 데코레이터를 정의하는 것이 좋은 경우가 있음
  - 원래 함수와 모양이 비슷하기 때문에 가독성이 좋음
  - 파라미터를 받아서 뭔가를 하려면 `*args`, `*kwargs` 를 사용하는 것이 불편함



예:) 문자열을 받아서 빈번히 드라이버 객체를 초기화하는 경우 -> 파라미터를 변환해주는 데코레이터를 만들어 중복 제거 가능

```python
import logging
from functools import wraps

logger = logging.getLogger(__name__)

class DBDriver:
  def __init__(self, dbstring):
    self.dbstring = dbstring
    
  def execute(self, query):
    return f"{self.dbstring} 에서 쿼리{query} 실행"

def inject_db_driver(function):
  """DB에서 dns 문자열을 받아서 DBDriver 인스턴스를 생성하는 데코레이터"""
  @wraps(function)
  def wrapped(dbstring):
    return function(DBDriver(dbstring))
  return wrapped

@inject_db_driver
def run_query(driver):
  return driver.execute("test_function")
  
```

- 이 경우 같은 기능을 하는 데코레이터를 클래스 메서드에 재사용할 경우 동작하지 않는다

  ```python
  class DataHandler:
    @inject_db_driver
    def run_query(self, driver):
      return driver.execute(self.__class__.__name__)
  ```

  - 클래스 메서드의 `self`라는 추가 변수를 항상 첫 번째 파라미터로 받게 되어 있음

    - 따라서 하나의 파라미터만 받도록 설계된 이 데코레이터는 연결 문자열 자리에  `self`를 전달
    - 두 번째 파라미터에는 아무것도 전달하지 않아 에러가 발생하게 됨

  - 해결법: 메서드와 함수에 대해서 동일하게 동작하는 데코레이터를 만들기

    - 데코레이터를 클래스 객체로 구현하고 `__get__` 메서드를 구현한 디스크립터 객체 만들기
      - 디스크립터에 대한 자세한 내용은 6장에서 다룸
      - 호출할 수 있는 객체를 메서드에 다시 바인딩한다는 정도만 알고 지나가기 

    ```python
    from functools import wraps
    from types import  MethodType
    
    class inject_db_driver:
      """문자열을 DBDriver 인스턴스로 변환하여 래핑된 함수로 전달"""
      def __init__(self, funciton):
        self.function = function
        wraps(self.function)(self)
        
      def __call__(self,dbstring):
        return self.function(DBDriver(dbstring))
    
      def __get__(self,instance, owner):
        if instance is None:
          return self
        return self.__class__(MethodType(self.function, instance))
      
    ```







## 데코레이터와 DRY 원칙

------

### DRY : Do not Repeat Yourself

- 소스 코드에서 동일한 코드를 반복하지 마라

1. 처음부터 데코레이터를 만들지 않는다

   : 패턴이 생기고 데코레이터에 대한 추상화가 명확해지면 그 때 리펙토링을 한다

2. 데코레이터가 적어도 3회 이상 필요한 경우에만 구현한다

3. 데코레이터 코드를 최소한으로 유지한다



## 데코레이터와 관심사의 분리

------

코드 재사용의 핵심은 응집력 있는 컴포넌트를 만드는 것

- 최소한의 책임을 가져서 오직 한 가지 일만 해야하며 그 일을 잘해야 함

- 컴포넌트가 작을 수록 재사용성이 높아진다

- 결합과 종속성을 유발하고 SW의 유연성을 떨어트리는 추가 동작이 필요없이 여러 상황에서 쓰일 수 있음



## 좋은 데코레이터의 분석

------

### 훌륭한 데코레이터가 갖추어야 할 특성

- 캡슐화와 관심사의 분리
  - 좋은 데코레이터는 실제로 하는일과 데코레이팅하는 일의 책임을 명확히 구분해야 함
  - 데코레이터의 클라이언트 블랙박스 모드로 동작해야 함
- 독립성
  - 데코레이터가 하는 일은 독립적이어야 함
  - 데코레이팅되는 객체와 최대한 분리되어야 함
- 재사용성
  - 여러 유형에 적용 가능한 형태가 바람직함
  - 충분히 범용적이어야 함















