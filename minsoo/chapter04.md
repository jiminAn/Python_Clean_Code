## Chapter 04. SOLID 원칙
#### S : 단일 책임 원칙
#### O : 개방/폐쇄의 원칙
#### L : 리스코프 치환 원칙
#### I : 인터페이스 분리 원칙
#### D : 의존성 역전 원칙
##### 다섯 가지 원칙을 지키다 보면 서로 상충되는 경우가 발생할 수 있음
---
### 01. 단일 책임 원칙 (Single Responsibility Principle)
 #### 소프트웨어 컴포넌트가 단 하나의 책임을 져야한다는 원칙
---
##### 즉,  하나의 구체적인 일을 담당한다는 것을 의미하며 변화해야 할 이유는 단 하나라는 것
 ##### 1) 이 디자인 원칙은 보다 응집력 있는 추상화를 하는데 도움이 됨
###### 응집력 : 객체가 작고 잘 정의된 목적을 가져야하며 가능하면 작아야함

 ##### 2) 어떤 경우에도 여러 책임을 가진 객체를 만들어서는 안 됨
###### 다른 행동을 그룹화한 것이므로 유지보수가 어려워지기 때문
 ##### 3) 클래스에 있는 프로퍼티와 속성이 항상 메서드를 통해서 사용되도록 하는 것
###### 관련된 개념이기 때문에 동일한 추상화로 묶는 것이 가능
##### 4) 더 작은 클래스로 분해할 수 있어야 함.
###### 이 원칙은 클래스의 메서드들이 서로 관련이 없고 다른 책임을 가지고 있기 때문

#### 1.1 책임분산 

##### 모든 메서드를 다른 클래스로 분리하여 각 클래스마다 단일 책임을 갖게 해야 함 
###### 따라서 , 솔루션을 관리하기 쉬워짐
##### 각 클래스가 딱 하나의 메서드를 가져야 한다는 것은 아님 
##### 처리 해야할 로직이 같은 경우 하나의 클래스에 여러 메서드를 추가할 수 있음
----
### 02. 개방/폐쇄 원칙 (Open/Close Principle)
#### 모듈이 개방되어 있으면서도 폐쇄되어야 한다는 원칙
___ 
##### 클래스를 디자인할 때는 유지보수가 쉽도록 로직을 캡슐화하여 확장에는 개방, 수정에는 폐쇄되도록 해야 함
##### 확장 가능하고, 새로운 요구사항이나 도메인 변화에 잘 적응하는 코드를 작성해야 함
###### 즉, 새로운 문제가 발생할 경우 코드를 유지한 채 새로운 것을 추가만 해야 함
#### 2.1 OCP를 따르지 않을 경우 유지보수의 어려움
##### 
```python
class Event:
	def __init__(self, raw_data):
		self.raw_data = raw_data

class UnknownEvent(Event):
"""데이터만으로 식별할 수 없는 이벤트"""

class LoginEvent(Event):
"""로그인 사용자에 의한 이벤트"""

class LogoutEvent(Event):
"""로그아웃 사용자에 의한 이벤트"""

class SystemMonitor:
"""시스템에서 발생한 이벤트 분류"""

	def __init__(self, event_data):
		self.event_data = event_data
	
	def identify_event(self):
		if (
			self.event_data["before"]["session"] == 0
			and self.event_data["after"]["session"] == 1
		):
				return LoginEvent(self.event_data)
		elif (
			self.event_data["before"]["session"] == 1
			and self.event_data["after"]["session"] == 0
		):
			return LogoutEvent(self.event_data)
		
		else
			return UnknownEvent(self.event_data) # 기본 로직을 가진 null 객체
		#다형성:하나의 타입에 여러 객체를 대입할 수 있는 성질
```
##### 위의 코드는 이벤트 유형의 계층 구조와 이를 구성하는 일부 비지니스 로직을 명확하게 할 수 있음
##### 그러나, 문제점이 존재
##### 이벤트 유형을 결정하는 논리가 일체형으로 중앙 집중화 된다는 점
##### 새로운 유형의 이벤트를 시스템에 추가할 때마다 메서드를 수정해야 함

#### 2.2 확장성을 가진 이벤트 시스템으로 리팩토링

##### 메서드를 변경하지 않고도 새로운 유형의 이벤트를 추가 (폐쇄 원칙)
#####  새로운 이벤트가 추가될 때 이미 존재하는 코드를 변경하지 않고 코드를 확장하여 새로운 유형의 이벤트를 지원 (개방 원칙)
----
##### SystemMonitor 클래스를 추상적인 이벤트와 협력하도록 변경하고
##### 이벤트에 대응하는 개별 로직은 각 이벤트 클래스에 위임

```python
class Event:
	def __init__(self, raw_data):
		self.raw_data = raw_data
		
	@staticmethod
	def meets_condition(event_data: dict):
		return False

	#@staticmethod : 인스턴트를 통하지 않고 클래스에서 바로 호출할 수 있는 정적메서드	

class UnknownEvent(Event):
"""데이터만으로 식별할 수 없는 이벤트"""

class LoginEvent(Event):
	@staticmethod
	def meets_condition(event_data: dict):
		return (
		event_data["before"]["session"] == 0
		and event_data["after"]["session"] == 1
		)
		
class LogoutEvent(Event):
	@staticmethod
	def meets_condition(event_data: dict):
		return (
		event_data["before"]["session"] == 1
		and event_data["after"]["session"] == 0
		)
		
class SystemMonitor:
"""시스템에서 발생한 이벤트 분류"""

	def __init__(self, event_data):
		self.event_data = event_data
	
	def identify_event(self):
		for event_cls in Event.__subclasses__():
			try:
				if event_cls.meets_condition(self.event_data):
					return event_cls(self.event_data)
				except KeyError:
					continue
		return UnknownEvent(self.event_data)
```
##### 상호작용이 추상화를 통해 이뤄지고 있음
##### 분류 메서드는 일반적인 인터페이스를 따르는 제네릭 이벤트와 동작
##### 이 인터페이스를 따르는 제네릭들은 모두 meets_condition 메서드를 구현하여 다형성을 보장
###### 제네릭 : 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법
#### 2.3 이벤트 시스템 확장
```python
class Event:
	def __init__(self, raw_data):
		self.raw_data = raw_data
		
	@staticmethod
	def meets_condition(event_data: dict):
		return False
	
	#@staticmethod : 인스턴트를 통하지 않고 클래스에서 바로 호출할 수 있는 정적메서드	
	
class UnknownEvent(Event):
"""데이터만으로 식별할 수 없는 이벤트"""

class LoginEvent(Event):
	@staticmethod
	def meets_condition(event_data: dict):
		return (
		event_data["before"]["session"] == 0
		and event_data["after"]["session"] == 1
		)
		
class LogoutEvent(Event):
	@staticmethod
	def meets_condition(event_data: dict):
		return (
		event_data["before"]["session"] == 1
		and event_data["after"]["session"] == 0
		)
#Event 추가	
class TransactionEvent(Event):
	"""시스템에서 발생한 트랜잭션 이벤트"""

	@staticmethod
	def meets_condition(event_data: dict):
		return event_data["after"].get("transaction") is not None
		
class SystemMonitor:
"""시스템에서 발생한 이벤트 분류"""

	def __init__(self, event_data):
		self.event_data = event_data
	
	def identify_event(self):
		for event_cls in Event.__subclasses__():
			try:
				if event_cls.meets_condition(self.event_data):
					return event_cls(self.event_data)
				except KeyError:
					continue
		return UnknownEvent(self.event_data)
```
##### 이전 이벤트에 대해서는 동일하게 동작하고 새로운 이벤트에 대해서는 정확하게 분류하는 것을 볼 수 있음
#### 2.4 OCP 최종 정리
##### 1) OCP는 다형성의 효과적인 사용과 밀접한 관련이 있음
###### 즉, 모델을 쉽게 확장할 수 있는 일반적인 구조로 디자인하는 것 
##### 2) 유지보수성에 대한 문제를 해결
###### OCP를 따르지 않을 경우, 파급 효과가 생겨 코드 전체에 영향을 미침 
##### 3) 모든 프로그램에서 이 원칙을 적용하는 것이 가능한 것은 아님
###### 일부 추상화의 경우 충돌이 발생할 수 있기 때문에 확장 가능한 요구 사항에 적합한 폐쇄를 선택해야함
### 03. 리스코프 치환 원칙 (Liskov Substitution Principle)
#### 설계 시 안정성을 유지하기 위해 객체 타입이 유지해야 하는 일련의 특성
---
##### 1) 어떤 클래스에서든 클라이언트는 특별한 주의를 기울이지 않고도 하위 타입을 사용할 수 있어야 함
###### 즉, 클라이언트는 완전히 분리되어 있으며 클래스 변경 사항과 독립
##### 2) 자식 클래스는 부모클래스를 대체할 수 있어야 함
##### 3) LSP는 계약을 통한 설계와도 관련이 있음
#### 3.1 도구를 사용해 LSP 문제 검사하기
#### 1) 메서드 서명의 잘못된 데이터타입 검사
###### 메서드 서명 : 메서드의 이름과 매개변수 리스트의 조합
```python
class Event:
	...
	def meets_condition(self, event_data: dict) -> bool:
		return False

class LoginEvent(Event):
	def meets_condition(self, event_data: list) -> bool:
		return bool(event_data)
# Mypy를 실행하면 다음과 같은 오류 메세지 표시
 error: Argument 1 of "meets_condition" incompatible with supertype "Event"
```
#####  LSP 위반 사례
##### - 파생 클래스가 부모 클래스에서 정의한 파라미터와 다른 타입을 사용
##### - 반환 값을 다른 값으로 변경 
#### 2) Pylint로 호환되지 않는 서명 검사
##### 메서드의 서명 자체가 완전히 다른 경우 (탐지가 쉽지 않음)
```python
class LogoutEvent(Event):
	def meets_condition(self, event_data: dict, override: bool) -> bool:
		if override:
			return True
		...
	#Pylint 오류 감지 
parameters differ from overridden 'meets_condition' method (argumentsdiffer)
``` 
#### 3.2 애매한 LSP 위반 사례
##### LSP를 위반한 것이 명확하지 않아서 자동화된 도구로 검사하기 애매한 경우  → '코드 리뷰'
##### LSP 에서 하위 클래스는 상위 클래스와 호환 가능하단 점에서 계약은 계층 구조 어디에서든 항상 유지 되어야 함
##### 부모 클래스는 클라이언트와의 '계약' 을 정의 (하위 클래스는 '계약' 을 따라야 함)
###### - 하위 클래스는 부모 클래스에 정의된 것 보다 사전조건을 엄격하게 만들면 안됨
###### - 하위 클래스는 부모 클래스에 정의된 것 보다 약한 사후조건을 만들면 안됨

##### 사전조건에서 파라미터가 사전 타입인지 & "before"와 "after"키를 가지고 있는지 확인하는 예제
``` python
class Event:
	def __init__(self, raw_data):
		self.raw_data = raw_data
		
	@staticmethod
	def meets_condition(event_data: dict):
		return False
		
	@staticmethod
	def meets_condition_pre(event_data: dict):
		""" 인터페이스 계약의 사전조건
		''event_data'' 파라미터가 적절한 형태인지 유효성 검사
		"""
		assert isinstance(event_data, dict), f"{event_data!r} is not a dict"
		for moment in ("before", "after"):
			assert moment in event_data, f"{moment} not in {even_data}"
			assert isinstance(event_data[moment], dict)

# 올바른 이벤트 유형을 탐지하기 위해 사전조건을 먼저 검사

class SystemMonitor:
"""시스템에서 발생한 이벤트 분류"""

	def __init__(self, event_data):
		self.event_data = event_data
	
	def identify_event(self):
		Event.meets_condition_pre(self.event_data)
		event_cls = next(
			(
				event_cls
				for event_cls in Event.__subclasses__()
				if event_cls.meets_condition(self.event_data)
			),
			UnknownEvent,
		)
		return event_cls(self.event_data)
```
##### 올바르게 설계된 이벤트 클래스
``` python
class TransactionEvent(Event):
	""" 시스템에서 발생한 이벤트 분류 """
	
	@staticmethod
	def meets_condition(event_data: dict):
		return event_data["after"].get("transaction") is not None
```
##### LoginEvent와 LogoutEvent 클래스는 before 와 after의 "session" 이라는 키를 사용하기 때문에 그대로 사용할 수 없음
##### TransactionEvent 처럼 대괄호 대신 .get() 메서드로 수정하여 해결해야 함
#### 3.3 LSP 최종 정리
##### LSP는 객체지향 소프트웨어 설계의 핵심이 되는 다형성을 강조하기 떄문에 좋은 디자인의 기초가 됨
##### 인터페이스의 메서드가 올바른 계층구조를 갖도록 하여 상속된 클래스가 부모 클래스와 다형성을 유지하도록 하는 것
##### LSP가 OCP에 기여함 

### 04. 인터페이스 분리 원칙 (Interface Segregation Principle)
#### 반복적으로 재검토했던 "작은 인터페이스"에 대한 가이드라인을 제공
----
##### 인터페이스 
###### 1) 객체가 노출하는 메서드의 집합
###### 2) 객체가 수신하거나 해석할 수 있는 모든 메세지로 구성됨
###### 3) 다른 클라이언트에서 호출할 수 있는 요청
###### 4) 클래스에 노출된 동작의 정의와 구현을 분리
##### 인터페이스는 클래스 메서드의 형태를 보고 암시적으로 정의됨 → "덕 타이핑(duck typing) 원리"를 따르기 때문
###### 덕 타이핑  : 모든 객체가 자신이 가지고 있는 메서드와 자신이 할 수 있는 일에 의해서 표현된다는 점에서 출발
###### 즉, 객체의 본질을 정의하는 것은 궁극적으로 메서드의 형태임
##### 추상 기본 클래스 : 파생 클래스가 구현해야 할 일부분을 기본 동작 또는 인터페이스로 정의하는 것
##### 가상 하위 클래스(virtual subclass)  : 타입을 계층구조에 등록하는 기법 →  덕 타이핑의 개념을 조금 더 확장하는 것
```python
# ISP 는 '다중 메서드를 가진 인터페이스가 있다면, 
# 정확하고 구체적인 구분에 따라  
# 적은 수의 메서드를 가진 여러 개의 메서드로 분할하는 것이 좋다' 를 의미
→ 응집력이 높아짐
```
#### 4.1 너무 많은 일을 하는 인터페이스
##### XML과 JSON 포맷의 데이터를 파싱하는 경우
```python
# XML과 JSON 포맷의 데이터를 파싱하는 경우
# 추상 기본 클래스를 만들고, from_xml()과 from_json()이라는 메서드를 정의
class EventParser:
	def from_xml():
		...
	def from_json():
		...
```
##### 만약, 어떤 클래스는 XML 메서드를 필요하지 않고 JSON으로만 구성할 수 있다면 ?
##### 인터페이스에서는 여전히 필요하지 않은 메서드 ( from_xml ) 를 제공할 것임
##### 즉 , 이 인터페이스는 결합력은 높히게 하고 유연성을 떨어트리게 됨.
#### 4.2 인터페이스는 작을수록 좋다
```python
# 인터페이스 분리
class XMLEventParser:
	def from_xml():
		...

class JSONEventParser:
	def from_json():
		...
```
##### 독립성을 유지하게 되었고, 새로운 작은 객체를 사용해 모든 기능을 유연하게 조합할 수 있게 됨
#### 4.3 인터페이스는 얼마나 작아야 할까?
```python
# 컨텍스트 관리자
class File(object):

	def __init__ (self, file_name, method):
		self.file_obj = open(file_name, method)
	def __enter__(self):
		return self.file_obj
	def __exit__ (self, exc_type, exc_val, exc_tb):
		self.file_obj.close()
```
##### 컨텍스트 관리자를 보면 __enter__와 __exit__ 두 가지 메서드를 필요로 함
##### 따라서, 서로 관련이 있는 메서드들은 분리할 필요가 없음
##### 관련이 없는 메서드 → 분리

### 05. 의존성 역전 (Dependency Inversion Principle)
#### 코드가 깨지거나 손상되는 취약점으로부터 보호해주는 흥미로운 디자인 원칙
---
##### 의존성 역전 : 코드가 세부사항이나 구체적인 구현에 적응하도록 하지 않고, API 같은 것에 적응하도록 하는 것
##### 추상화를 통해 세부 사항에 의존하지 않도록 해야 하지만, 반대로 세부 사항은 추상화에 의존해야 한다.
##### A와 B 두 객체가 상호 교류를 한다고 생각해보자 
##### A는 B의 인스턴트를 사용하지만 B의 모듈을 직접 관리하지 않음
##### 코드가 B에 크게 의존해서 B 코드가 변경되면 원래의 코드는 쉽게 깨지게 된다
##### 이 때, 의존성 역전이 필요
##### 즉, 인터페이스를 개발하고 코드가 B의 구체적인 구현에 의존하지 않도록 해야 함
#### 5.1 엄격한 의존의 예
##### 저수준의 내용에 따라 고수준의 클래스가 변경되는 것은 좋은 디자인이 아님
##### 새로운 요구사항에 대해서 지속적으로 수정해야 하기 때문이다.
#### 5.2 의존성을 거꾸로
##### 따라서 세부 구현사항을 가진 저수준 클래스가 인터페이스의 구현을 담당하게 해야한다
