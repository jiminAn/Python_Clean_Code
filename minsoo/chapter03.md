## Chapter 03. 좋은 코드의 일반적인 특징

### 01. 계약에 의한 디자인

컴포넌트는 기능을 숨겨 캡슐화하고 함수를 사용할 고객에게는 API를 노출

컴포넌트의 함수, 클래스, 메서드는 특별한 유의사항에 의해 동작. 그렇지 않을 경우 코드가 깨지게 됨

클라이언트는 특정 응답을 기대. 다를 경우 함수 호출에 실패하고 결함이 발생

API를 디자인할 때 예상되는 입력, 출력 및 부작용을 문서화를 하게 되지만,  문서화가 런타임 시의 소프트웨어의 동작을 강제하는 것은 X

따라서 제대로 작동하길 기대하는 것과 호출자가 반환 받기를 기대하는 것은 **디자인의 하나** 가 되어야 함

**계약에 의한 디자인 ( Design by Contract )** 이란,  관계자가 기대하는 바를 암묵적으로 코드에 삽입하는 대신 양측이 동의하는 계약을 먼저 한 다음, 계약을 어겼을 경우는 명시적으로 예외를 발생하는 것

**계약**은 주로 사전조건과 사후조건을 명시하지만 때로는 불변식과 부작용을 기술

여기서, 사전조건과 사후조건은 저수준 ( 코드 ) 레벨에서 강제함.

계약에 의해 디자인을 하는 이유
1. 사전조건 또는 사후조건 검증에 실패할 경우 오류를 쉽게 찾을 수 있음
2. 잘못된 가정 하에 코드의 핵심 부분이 실행되는 것을 방지 할 수 있음

#### 1.1 사전 조건 (precondition)

함수나 메서드가 제대로 동작하기 위해 보장해야 하는 모든 것. 즉, 적절한 데이터를 전달하는 것.
 
유효성 검사
1. 관용적인 접근법
클라이언트가 함수를 호출하기 전에 모든 유효성 검사를 하게 하는 것
2. 까다로운 접근법
함수가 자체적으로 로직을 실행하기 전에 검사하도록 하는 것

일반적으로 가장 안전하고 견고한 방법인DbC에 대한 까다로운 접근법을 사용

어떤 방식으로 유효성 검사를 하든 '중복 제거 원칙'을 적용해야 함. 즉, 검증 로직을 클라이언트에 두거나 함수 자체에 두어야 함

#### 1.2 사후조건 (postcondition)

메서드 또는 함수가 반환된 후의 상태를 강제하는 계약의 일부. 특정 속성이 보존되도록 보장해야 함.

호출자가 컴포넌트에서 기대한 것을 제대로 받았는지 확인하기 위해서 수행

#### 1.3 파이썬스러운 계약

PEP – 316  (Programming by Contract for Python) 를 적용하는 가장 좋은 방법

1. 메서드, 함수 및 클래스에 `RuntimeError` 예외 또는 `ValueError` 예외를 발생시키는 제어 메커니즘을 추가

2. 문제를 정확하게 특정하기 어려울 땐 '사용자 정의 예외'를 만드는 것

3. 코드를 가능한 격리된 상태로 유지하는 것

#### 1.4 계약에 의한 디자인 ( Dbc )– 결론

1. 문제가 있는 부분을 효과적으로 식별할 수 있음.

즉, 오류가 발생했을 때 코드의 어떤 부분이 손상되었는지 계약이 파손되었는지가 명확해짐

2. 프로그램 구조를 명확히 하는 목적

계약은 **명시적으로** 함수나 메서드가 정상 동작하기 위해 기대하는 것이 무엇인지, 무엇을 기대할 수 있는지 정의

위의 방법이 효과적이기 위해서는 함수에 전달되는 객체의 속성과 반환 값을 검사하고 이들이 유지해야하는 조건을 확인하는 등의 작업을 해야함

### 02. 방어적(defensive) 프로그래밍

 DbC와는 다른 접근 방식을 따름

계약에서 예외를 발생시키고 실패하게 되는 모든 조건을 기술하는 대신 객체, 함수 또는 메서드와 같은 코드의 모든 부분을 유효하지 않은 것으로부터 **스스로 보호할 수** 있게 하는 것

DbC와 다른 철학을 가졌다는 의미가 아닌, **다른 디자인 원칙과 서로 보완 관계**에 있을 수 있다는 것을 의미

예상할 수 있는 시나리오의 오류를 처리하는 방법 **( 에러 핸들링 프로시저 )** 과 불가피한 조건에 의해서  발생하지 않아야 하는 오류 **( 어썰션 )** 를 처리하는 방법에 대해 다룸
#### 2.1 에러 핸들링 

오류가 발생하기 쉬운 상황에서 사용

  ex) 데이터 입력 확인

주요 목적은 에러에 대해서 실행을 계속할 수 있을지 아니면 극복할 수 없는 오류여서 프로그램을 중단할 지를 결정하는 것

에러 처리 방법
1. 값 대체
2. 에러 로깅
3. 예외 처리

#### 2.1.1 값 대체 

오류가 발생해서 소프트웨어가 잘못된 값을 생성하거나 전체가 종료될 위험이 있을 경우 결과 값을 안전한 다른 값으로 대체하는 것

**안전한 값**이란 기본 값 또는 잘 알려진 상수, 초기 값 즉, 정합성을 깨지 않는 다른 값 

항상 값 대체가 가능한 것은 아니기 때문에 대체 값이 안전한 옵션인 경우에 한해서 신중하게 선택해야 하는데, 

이 때 견고성과 정확성 간의 트레이드오프로써 결정을 내릴 수 있음

다른 방향의 안전한 방법은 제공되지 않은 데이터에 기본 값을 사용하는 것 ( 설정되지 않은 환경 변수의 기본 값, 설정 파일의 누락된 항목, 함수의 파라미터 )
```python
# 사전에서 get 메서드의 두 번째 파라미터를 사용하면 기본값을 나타내는 예제

>>> configuration = {"dbport": 5432 }
>>> configuration.get ("dbport", "localhost" )
'localhost'
>>> configuration.get("dbport")
5432
```
일반적으로 누락된 파라미터를 기본 값으로 바꾸어도 큰 문제가 없지만 "오류가 있는 데이터를 유사한 값으로 대체하는 것은 더 위험하며 일부 오류를 숨겨버릴 수 있다"
#### 2.1.2 예외 처리 

에러가 발생하기 쉽다는 가정으로 계속 실행하는 것 보다 실행을 멈추고 호출자에게 실패했음을 알리는 것

함수 호출 실패는 함수 자체의 문제이거나 외부 컴포넌트 중 하나의 문제일 수 있음.

따라서, 심각한 오류에 대해 명확하고 분명하게 알려주어야 함.

또한 예외적인 상황을 명확하게 알려주고 원래의 **비즈니스 로직에 따라 흐름을 유지**하는 것이 중요.

예외는 **캡슐화를 약화**시키기고 **문맥에서 자유롭지 않다는 것을 의미**하기 때문에 신중하게 사용해야함.

예외가 너무 많이 발생하면 함수가 응집력이 약하고 많은 책임을 가지고 있다는 것이므로 여러개의 작은 함수로 나눠야한다는 신호임.

#### 2.1.2.1 올바른 수준의 추상화 단계에서 예외 처리

예외는 오직 한 가지 일을 하는 함수의 일부분이어야 함.

```python
# 서로 다른 수준의 추상화를 혼합하는 예
# 애플리케이션에서 디코딩한 데이터를 외부 컴포넌트에 전달하는 객체

class DataTransport:
	""" 다른 레벨에서 예외를 처리하는 객체의 예 """
	retry_threshold: int = 5
	retry_n_times: int = 3

	def __init__(self, connector):
		self._connector = connector
		self.connection = None

	def deliver_event(self, event):
		try:
			self.connect()
			data = event.decode()
			self.send(data)
		except ConnectionError as e:
			logger.info("연결 실패: %s", e)
			raise
		except ValueError as e:
			logger.error("%r 잘못된 데이터 포함: %s", event, e)
			raise

	def connect(self):
		for _ in range(self.retry_n_times):
			try:
				self.connection = self._connector.connect()
			except ConnectionError as e:
				logger.info(
					"%s: 새로운 연결 시도 %is",e,
					self.retry_threshold,
					)
				time.sleep(self.retry_threshold)
			else:
				return self.connection
			raise ConnectionError(
				f"{self.retry_n_times} 번째 재시도 연결 실패"
			)
	
	def send(self, data):
		return self.connection.send(data)
```
`ConnectionError`는 `connect` 메서드 내에서 처리 하고 `ValueError`는 `decode`메서드에 내에서 처리. 따라서, `deliver_event`에서는 예외를 `catch`할 필요가 없음.

```python
class DataTransport:
	""" 추상화 수준에 따른 예외 분리를 한 객체의 예제 """
	retry_threshold: int = 5
	retry_n_times: int = 3

	def __init__(self, connector):
		self._connector = connector
		self.connection = None

	def deliver_event(self, event):
		self.connection = connect_with_retry(
				self._connector, self.retry_n_times, self.retry_threshold
			)
		self.send(event)
		
	def send(self, data):
		try:
			return self.connection.send(event.decode())
		except ValueError as e:
			logger.error("%r 잘못된 데이터 포함: %s", event, e)
			raise

	def connect_with_retry(connector, retry_n_times, retry_threshold = 5):
	""" connector의 연결을 맺는다. <retry_n_times>회 재시도.
	
	연결에 성공하면 connection 객체 반환
	재시도까지 모두 실패하면 ConnectionError 발생

	:param connector: '.connect()' 메서드를 가진 객체
	:param retry_n_times int: ''connector.connect()''를 호출 시도하는 횟수
	:param retry_threshold int: 재시도 사이의 간격
	"""
	for _ in range(retry_n_times):
			try:
				return connector.connect()
			except ConnectionError as e:
				logger.info(
					"%s: 새로운 연결 시도 %is", e, retry_threshold,
				)
				time.sleep(self.retry_threshold)
			exc = ConnectionError(f"{self.retry_n_times} 번째 재시도 연결 실패")
			logger.exception(exc)
			raise exc
```
#### 2.1.2.2 Traceback 노출 금지
보안을 위한 고려 사항

특정 문제를 나타내는 예외가 있는 경우 문제를 효율적으로 해결할 수 있도록 traceback 정보, 메세지 및 기타 수집 가능한 정보를 로그로 남기는것은 중요. 

그러나, 위의 세부사항은 사용자에게 보여서는 안됨.  traceback은 매우 유용한 정보여서 중요 정보나 지적 재산의 유출이 발생할 수 있기 때문

따라서, 예외가 전파되도록 하는 경우 중요한 **정보를 공개하지 않도록 주의** 해야 함

#### 2.1.2.3 비어있는 except 블록 지양

일부 오류에 대비하여 프로그램을 지나치게 방어적이게 하는 것은 더 심
각한 문제로 이어짐

```python
try:
	process_data()
except:
	pass
```

다음과 같은 코드는 오류를 발생시키지 않음. 

문제점은 실패해야만 할 때조차도 실패하지 않는다는 것임. 이는 전혀 파이썬스러운 코드가 아니고 오히려 문제를 숨기고 유지보수를 어렵게 만들게 됨.

두 가지 대안
1. 보다 구체적인 예외 사용
2. except 블록에서 실제 오류 처리

두 항목을 동시에 적용하는 것이 가장 좋은 방법

#### 2.1.2.4 원본 예외 포함

오류 처리 과정에서 다른 오류를 발생시키고 메시지를 변경할 수 있음. 이 때 원래 예외를 포함하는 것이 좋음.

```python
raise <e> from <orginal_exception>
```
구문을 사용하여 예외의 타입을 변경할 수 있음.

원본의 `traceback`이 새로운 `exception`에 포함되고 원본 예외는 `__cause__` 속성으로 설정됨.

```python 
# 기본 예외를 사용자 정의 예외로 래핑하고 싶을 때

class InternalDataError(Exception):
	"""업무 도메인 데이터의 예외"""
def process(data_dictionary, record_id):
	try:
		return data_dictionary[record_id]
	except KeyError as e:
		raise InternalDataError("Record not present") from e
```


#### 2.2 파이썬에서 어설션 사용하기

어설션은 절대로 일어나지 않아야 하는 상황에 사용 됨. 즉,  `assert` 문에 사용된 표현식은 불가능한 조건을 의미. 

오류가 발생했다는 것은 결함이 있다는 것이므로 잘못된 시나리오에 도달할 경우 프로그램이 더 큰 피해를 입지 않도록 하는 것
 
```python
# 어설션을 비즈니스 로직과 섞거나 
# 소프트웨어의 제어 흐름 메커니즘으로 사용한 예제
# 좋지 않은 생각
try:
	assert condition.holds(), "조건에 맞지 않음."
except AssertionError:
	alternative_procedure()
```

어설션에 실패하면 반드시 프로그램을 종료시켜야 함

```python
result = condition.holds()
assert result > 0, "에러 {0}".format(result)
```
### 03. 관심사의 분리 

책임이 다르면 컴포넌트 ,계층 또는 모듈로 분리되어야 함.

프로그램의 각 부분은 **기능의 일부분 (관심사)** 에 대해서만 책임을 지며 나머지 부분에 대해서는 알 필요가 없음.

관심사를 분리하는 목표는 파급 효과를 최소화하여 유지보수성을 향상 시키는 것.


####  3.1 응집력(cohension)과 결합력(coupling) 

응집력이란 객체가 작고 잘 정의된 목적을 가져야 하며 가능하면 작아야 한다는 것을 의미

응집력이 높을수록 더 유용하고 재사용성이 높아짐

결합력이란 두 개 이상의 객체가 서로 어떻게 의존하는지 나타내는 것을 의미

객체 또는 메서드의 두 부분이 서로 의존적일 경우 
1. 낮은 재사용성
2. 파급 효과
3. 낮은 수준의 추상화

와 같은 바람직하지 않은 결과를 가져온다.

**"잘 정의된 소프트웨어는 높은 응집력과 낮은 결합력을 갖는다"**
( high cohesion and low coupling )

### 04. 개발 지침 약어

#### 4.1 DRY (Do not  Repeat Yourself) &  OAOO (Once And Only Once)

코드에 있는 지식은 단 한번, 단 한 곳에 정의되어야 함 즉, 코드 중복을 피해야 함

코드 중복의 부정적인 영향

1. 오류가 발생하기 쉽다

2. 비용이 비싸다

3. 신뢰성이 떨어진다

코드 중복은 기존 코드의 지식을 무시함으로써 발생하므로 코드의 특정 부분에 의미를 부여함으로써 해당 지식을 식별하고 표시할 수 있음.

```python
# 연구 센터에서 학생들을 다음과 같은 기준으로 평가
# 평가 기준 : 시험 통과 11점, 시험 통과 실패 -5점, 1년이 지날 때 마다 -2점
# 나쁜 코드 예제
def process_students_list(students):
	# 중간 생략...

	students_ranking = sorted(
		students, key=lambda s: s.passed * 11 - s.failed * 5 - s.years * 2 )
	# 학생별 순위 출력
	for student in student_ranking:
		print(
			"이름: {0}, 점수: {1}".format(
			student.name,
			(student.passed * 11 - student.failed * 5 - student.years * 2),
		)
	)
```
`sorted`함수의 `key`로 사용되는`lambda`가 특별한 도메인 지식을 나타내지만 아무런 정의가 없음. 

코드에서 의미를 부여하지 않았기 때문에 순위를 출력하는 동안에 중복이 발생 

```python
def score_for_student(student):
	return student.passed * 11 - student.failed * 5 - student.year * 2

def process_students_list(students):
	# 중간 생략...

	students_ranking = sorted(students, key=score_for_student)
	# 학생별 순위 출력
	for student in students_ranking:
		print(
			"이름: {0}, 점수: {1}".format(
			student.name, score_for_student(student)
		)
	)
```
코드 중복을 제거하는 방법
1. 함수 생성 기법

2. 새로운 객체 만들기

3. 컨텍스트 관리자 

4. 이터레이터, 제너레이터

5. 데코레이터


#### 4.2 YAGIN (You Ain’t Gonna Need it)

과잉 엔지니어링을 하지 않기 위해 솔루션 작성 시 계속 염두에 두어야 하는 원칙

현재의 요구사항을 잘 해결하기 위한 소프트웨어를 작성하고 가능한 나중에 수정하기 쉽도록 작성하는 것

즉, 디자인을 할 때 내린 결정으로 특별한 제약 없이 개발을 계속 할 수 있다면, 굳이 필요 없는 추가 개발을 하지 말라는 것.

#### 4.3 KIS (Keep It Simple)

문제를 올바르게 해결하는 최소한의 기능을 구현하고 필요한 것 이상으로 솔루션을 복잡하게 만들지 않도록 해야 함.

```python
# 제공된 키워드 파라미터 세트에서 네임 스페이스를 작성하지만
# 다소 복잡한 코드 인터페이스를 가지고 있는 예제
class ComplicatedNamespace:
	"""프로퍼티를 가진 객체를 초기화하는 복잡한 예제"""
	
	ACCEPTED_VALUES = ("id_", "user", "location")
	
	@classmethod
	def init_with_data(cls, **data):
		instance = cls()
		for key, value in data.items():
			if key in cls.ACCEPTED_VALUES:
				setattr(instance, key, value)
			return instance
```
객체를 초기화 하기 위해 추가 메서드 `init_with_data`를 만드는것, 반복을 통해 `setattr`을 호출하는 것은 상황을 더 이상하게 만들게 됨
```python
class Namespace:
	"""Create on object from keyword arguments."""
	
	ACCEPTED_VALUES = ("id_", "user", "location")

	def __init__(self, **data):
		accepted_data = {
			k: v for k, v in data.items() if k in self.ACCEPTED_VALUES
		}
		self.__dict__.update(accepted_data)
```

"단순한 것이 복잡한 것보다 낫다."

#### 4.4 EAFP(Easier to Ask Forgiveness than Permission) / LBYL(Look Before You Leap)

[ EAFP ]  
일단 코드를 실행하고 실제 동작하지 않을 경우 대응한다는 뜻

일반적으로 코드를 실행하고 발생한 예외를 `catch`하고 `except`블록에서 바로 잡는 코드를 실행하게 됨.
```python
	try:
		with open(filename) as f:
		...
	except FileNotFoundError as e:
		logger.error(e)
```
[ LBYL ] 
도약하기 전에 살피라는 뜻
즉, 무엇을 실행하기 전에 실행할 수 있는지 확인하라는 것
```python
	if os.path.exists(filename):
		with open(filename) as f:
			...
```
파이썬은 EAFP 방식으로 만들어졌으며, EAFP 방식을 사용해야함.

**"암묵적인 것보다 명시적인 것이 더 좋다"**

### 05. 컴포지션과 상속

코드를 재사용하는 올바른 방법은 여러 상황에서 동작 가능하고 쉽게 조합할 수 있는 응집력 높은 객체를 사용하는 것

#### 5.1 상속이 좋은 선택인 경우

파생 클래스를 만드는 것은 양날의 검이 될 수 있음.

부모 클래스의 메서드를 얻을 수 있는 장점이 있지만 정의에 너무 많은 기능을 추가하게 되는 단점이 있기 때문임.

하위 클래스를 만들 때 상속된 모든 메서드를 실제로 사용할 것인지 생각해 보아야 함. 

그렇지 않다면 다음과 같은 이유로 설계상의 실수라고 할 수있음
1. 상위 클래스는 잘 정의된 인터페이스 대신 막연한 정의와 너무 많은 책임을 가짐
2. 하위 클래스는 확장하려고 하는 상위 클래스의 적절한 세분화가 아님

[ 상속을 잘 사용한 예 ]

1. 클래스의 기능을 그대로 물려받으면서 추가 기능을 더하거나 특정 기능을 수정하려는 경우
	
	상위 `BaseHTTPRequestHandler` 하위 `SimpleHTTPRequestHandler`
2. 인터페이스의 정의
	 
	 어떤 객체에 인터페이스 방식을 강제하고자 할 때 기본 추상 클래스를 만들어 사용

3. 예외 

	모든 예외가 `Exception`에서 상속받은 클래스임

#### 5.2 상속 안티패턴

상속을 올바르게 사용하면 객체를 **전문화**하고 기본 객체에서 출발하여 **세부적인 추상화**를 할 수 있음.

부모 클래스는 새 파생 클래스의 공통 정의의 일부가 됨. 따라서 클래스의 `public` 메서드는 부모 클래스가 정의하는 것과 일치해야 함.

코드 재사용만을 목적으로 상속을 사용하면 불필요한 메서드 까지 상속받을 수 있음

```python
class TransactionalPolicy(collections.UserDict):
	""" 잘못된 상속의 예 """

	def change_in_policy(self, customer_id, **new_policy_data):
		self[customer_id].update(**new_policy_data)
```

```python
>>> dir(policy)
[ # 모든 매직 메서드는 간략화를 위해 생략...
'change_in_policy', 'clear', 'copy', 'data', 'fromkeys', 'get', 'items',
'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']
```
위 디자인의 두 가지 주요 문제점
1. 계층 구조가 잘못됨

	기본 클래스에서 새 클래스를 만드는 것은 개념적으로 확장되고 세부적이라는 것을 의미.  사용자가 객체의 `public` 인터페이스를 통해 노출된 `public` 메서드를 확인하게 되면 특이하게 전문화된 이상한 계층구조라고 느끼게 될 것임.

2. 결합력에 대한 문제

	`TransactionPolicy`에 `pop()` 또는 `items()`와 같은 메서드는 필요하지 않지만 포함되어있음. 위 메서드는 `public`메서드 이므로 사용자가 호출할 수 있고 부작용이 있을 수 있음.

올바른 해결책은 **컴포지션**을 사용하는 것. `TransactionalPolicy` 자체가 사전이 되는 것이 아닌 사전을 활용하는 것.

```python
# 사전을 private 속성에 저장하고 
# __getitem()__으로 사전의 프록시를 만든다
# 나머지 필요한 public 메서드를 추가적으로 구현

class TransactionalPolicy:
	""" 컴포지션을 사용한 리팩토링 예제 """
	def __init__(self, policy_data, **extra_data):
		self._data = {**policy_data, **extra_data):

	def change_in_policy(self, customer_id, **new_policy_data):
		self._data[customer_id].update(**new_policy_data)

	def __getitem__(self, customer_id):
		return self._data[customer_id]

	def __len__(self):
		return len(self.data)
```
현재 사전인 데이터 구조를 향후 변경하려고 해도 인터페이스만 유지하면 사용자는 영향을 받지 않음.

결합력을 줄이고 파급 효과를 최소화하며 보다 나은 리팩토링을 허용하고 코드를 유지 관리하기 쉽게 만듦

#### 5.3 파이썬의 다중상속

다중 상속은 양날의 검이므로 올바르게 구현되지 않으면 문제가 커짐.

#### 5.3.1 메서드 결정 순서 ( MRO )

```python
class BaseModule:
	module_name = "top"

	def __init__ (self, module_name):
		self.name = module_name

	def __str__(self):
		return f"{self.module_name}:{self.name}"

class BaseModule1(BaseModule):
	module_name = "module-1" 

class BaseModule2(BaseModule):
	module_name = "module-2" 

class BaseModule3(BaseModule):
	module_name = "module-3" 

class ConcreteModuleA12(BaseModule1, BaseModule2):
	"""1과 2 확장""""

class ConcreteModuleA23(BaseModule2, BaseModule3):
	"""2과 3 확장""""

>>> str(ConcreteModuleA12(" test "))
	'module-1:test'
```

파이썬은 **C3 linearization** 또는 **MRO** 알고리즘을 사용하여 문제를 해결

```python
# 클래스의 결정 순서
>>> [cls.__name__ for cls in ConcreteModuleA12.mro()]
['ConcreteModuleA12', 'BaseModule1', 'BaseModule2', 'BaseModule', 'object']
```

#### 5.3.2 믹스인 ( mixin )

코드를 재사용하기 위해 일반적인 행동을 캡슐화해놓은 기본 클래스.

믹스인 클래스 자체로는 유용하지 않기 때문에, 다른 클래스와 함께 믹스인 클래스를 다중 상속하여 믹스인에 있는 메서드나 속성을 사용

```python
# 문자열을 받아서 하이픈(-)으로 구분된 값을 반환하는 파서

class BaseTokenizer:
	def __init__(self, str_token):\
		self.str_token = str_token

	def __iter__(self):
		yield from self.str_token.split("-")

>>> tk = BaseTokenizer("28a2320b-fd3f-4627-9792-a2b98e3c46b0")
>>> list(tk)
['28a2320b', 'fd3f', '4627', '9792', 'a2b38e3c46b0']
```

```python
# 기본 클래스를 변경하지 않고 값을 대문자로 변환하는 예제
class UpperIterableMixin:
	def __iter__(self):
		return map(str.upper, super().__iter__())
class Tokenizer(UpperIterableMixin, BaseTokenizer):
	pass
```
믹스인 을 사용하기 때문에 새로운 코드가 필요하지 X

일종의 데코레이터 역할을 하는데, `Tokenizer`는 믹스인에서 `__iter__`을 호출하고 다시 `super()`호출을 통해 다음 클래스 `BaseTokenizer`에 위임

### 06. 함수와 메서드의 인자

파이썬은 여러 가지 방법으로 인자를 받도록 함수를 정의할 수 있으며 사용자도 여러 가지 방법으로 인자를 제공할 수 있음

#### 6.1 파이썬의 함수 인자 동작방식

#### 6.1.1 인자는 함수에 어떻게 복사되는가

파이썬에서 모든 인자는 값에 의해 전달 ( passed by a value ) 됨.

인자를 변경하는 함수는 인자의 타입에 따라 다른 결과를 낼 수 있음.

```python
def function(arg):
	arg += " in function"
	print (arg)

>>> immutable = "hello"
>>> function(immutable)
hello in function
>>> mutable = list("hello")
>>> immutable 
'hello' # 변하지 않음
>>> function(mutable)
['h','e','l','l','o', ' ', 'i', 'n', ' ', 'f', 'u','n','c','t','i','o','n']
>>> mutable	
['h','e','l','l','o', ' ', 'i', 'n', ' ', 'f', 'u','n','c','t','i','o','n'] # 변함
```

불변형 타입객체 : `int` , `float`, `str`, `tuples`

`"arg + = <expression>"` 문장은 새로운 객체를 만들어 `arg`에 다시 할당함. `arg`는 단지 함수 스코프 내에 있는 로컬 변수이므로 호출자의 원래 변수와는 상관이 없이 작동함.

변형 타입객체 : `list`, `set`, `dict`

**참조**를 보유하고 있는 변수를 통해 값을 수정하므로 함수 외부에서도 실제 값을 수정할 수 있음.

이런 유형의 파라미터를 사용하면 예상치 못한 부작용을 유발할 수 있으므로 주의해야 함.

#### 6.1.2 가변인자


가변 인자를 사용하려면 해당 인자를 패킹할 변수의 이름 앞에 **별표( * )** 를 사용

```python
# 패킹 기법을 사용하여 하나의 명령어로 전달할 수 있음
# 파이썬스러운 코드

def f (first, second, third):
	print(first)
	print(second)
	print(third)

>>> l = [1, 2, 3]
>>> f(*l)
1
2
3
```
패킹 기법의 장점은 다른 방향으로도 동작한다는 것
```python
# 리스트의 값을 각 위치별로 변수에 언패킹하는 예제
>>> a, b, c = [1, 2, 3]
>>> a
1
>>> b
2
>>> c
3
```
부분적인 언패킹도 가능
```python
def show (e, rest):
	print("요소: {0} - 나머지: {1}".format(e, rest))

>>> first, *rest = [1, 2, 3, 4, 5]
>>> show(first, rest)
요소: 1 - 나머지: [2, 3, 4, 5]
>>> *rest, last = range(6)
>>> show(last, rest)
요소: 5 - 나머지: [0, 1, 2, 3, 4]
>>> first, *middle, last = range(6)
>>> first
0
>>> middle
[1, 2, 3, 4]
>>> last
5
>>> first, last, *empty = (1,2)
>>> first
1
>>> last
2
>>> empty
[]
```
변수 언패킹에 가장 좋은 사용은 반복. 일련의 요소를 반복해야 하고 각 요소가 차례로 있다면 각 요소를 반복할 때 언패킹하는 것이 좋음

```python
# 데이터베이스의 결과를 리스트로 받는 함수
# 첫번째 구현 : 레코드의 각 칼럼에 해당하는 값을 받아서 사용자를 생성
# 두번째 구현 : 언패킹을 사용해 반복을 수행

USERS = [(i, f "first_name_{i}", "last_name_{i}") for i in range(1_000)]

class User:
	def __init__(self, user_id, first_name, last_name):
		self.user_id = user_id
		self.first_name = first_name
		self.last_name = last_name
# 첫번째 구현
	def bad_users_from_rows(dbrows) -> list:
		""" DB 레코드에서 사용자를 생성하는 파이썬스럽지 않은 잘못된 예 """
		return [User(row[0], row[1], row[2]) for row in dbrows]
# 두번째 구현
	def users_from_rows(dbrows) -> list:
		return [
			User(user_id, first_name, last_name)
			for (user_id, first_name, last_name) in dbrows
		]
```
`row[0], row[1], row[2]` 형태는 무엇을 뜻하는지 전혀 알 수 없음. 반면, `user_id, first_name, last_name`과 같은 변수는 명확함.

이중 별표( ** )를 키워드 인자에 사용할 수 있음.

키워드 인자란, 함수를 호출할 때 인자의 값 뿐만 아니라 그 이름까지 명시적으로 지정해서 전달하는 것을 의미함

```python
function(**{"key": "value"})
```
위의 코드는 다음과 동일함
```python
function(key="value")
```

#### 6.2 함수 인자의 개수

너무 많은 인자를 사용하는 함수나 메서드는 나쁜 디자인의 징후임.

해결할 대안은 다음과 같음
1. 일반적인 소프트웨어 디자인의 원칙을 사용하는 것. **( 구체화 )**
	즉, 전달하는 모든 인자를 포함하는 새로운 객체를 만드는 것

2. 가변 인자나 키워드 인자 같은 파이썬의 특정 기능을 사용하는 것.
	그러나, 매우 동적이어서 유지보수가 어렵기 때문에 남용하지 않도록 주의해야 함 

#### 6.2.1 함수 인자와 결합력

함수 서명의 인수가 많을수록 호출자 함수와 밀접하게 결합될 가능성이 커짐.

함수가 보다 일반적인 인터페이스를 제공하고 더 높은 수준의 추상화로 작업할 수 있다면 코드 재사용성이 높아짐.  이는, 클래스의 `__init__` 메서드를 포함하여 모든 종류의 함수와 객체 메서드에 적용됨. 

이와 같은 메서드가 있다는 것은 일반적으로 새로운 상위 레벨의 추상화 객체를 필요로 하거나 누락된 객체가 있음을 의미.

#### 6.2.2 많은 인자를 취하는 작은 함수의 서명

```python
track_request = (request.headers, request.ip_addr, request.request_id)
```
파라미터가 `request`와 관련이 있으므로 `track_request(request)`의 형태로 파라미터로 전달하는 것이 좋음

명시적으로 의도한 것이 아니면 함수는 전달받은 객체를 변경해서는 안됨. 

`*args`와 `**kwargs`로 정의된 함수가 융통성 있고 적응력이 좋지만, 서명을 잃어버린다는 것과 가독성을 거의 상실하므로 인터페이스에 대한 문서화를 하고 정확하게 사용했는지 확인해야 함

### 07. 소프트웨어 디자인 우수 사례 결론

#### 7.1 소프트웨어의 독립성 ( orthogonality )

이들 중 하나가 변화해도 다른 하나에는 전혀 영향을 미치지 않는 **직교성**이 소프트웨어에서 염두에 두어야하는 방법임.

```python
# 세금과 할인율을 고려하여 가격을 계산하는 함수를 가지고 있고,
# 최종 계산된 값을 포매팅하는 예제

	def calculate_price(base_price: float, tax: float, discount: float) -> float
		return (base_price * (1 + tax)) * (1 - discount)

	def show_price(price: float) -> str:
		return "$ {0:,.2f}".format(price)

	def str_final_price(
		base_price: float, tax: float, discount: float, fmt_function=str) -> str:
		return fmt_function(calculate_price(base_price, tax, discount))

>>> str_final_price(10, 0.2, 0.5)
'6.0'
>>> str_final_price(1000, 0.2, 0)
'1200.0'
>>> str_final_price(1000, 0.2, 0.1, fmt_function = show_price)
'& 1,080.00'
```
`calculate_price`와 `show_price` 함수는 서로 독립성을 갖게 됨. 

코드의 두 부분이 독립적이라는 것은 다른 부분에 영향을 주지 않고 변경할 수 있다는 것을뜻하고 이는 단위 테스트에도 적용됨.

#### 7.2 코드 구조

코드를 구조화하는 방법은 팀의 작업 효율성과 유지보수성에 영향을 미치게 됨.

여러 정의가 들어있는 큰 파일을 만드는 것은 좋지 않고, 유사한 컴포넌트끼리 정리하여 구조화해야 함.

해결방안은 `__init__.py`파일을 가진 새 디렉토리를 만들어 파이썬 패키지를 만드는 것
