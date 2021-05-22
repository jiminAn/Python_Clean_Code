## Chapter 06. 디스크립터
##### 하나의 객체(A)는 다른 객체(B)를 속성으로 가질 수 있음
##### 이 때 속성이 되는 그 객체(B)의 값을 
##### 읽거나, 쓰거나, 객체(B)를 속성에서 삭제하려고 할 때
##### 동작들이 미리 정의된 객체(B)를 디스크립터라고 함.
#### 목표 
##### 1) 디스크립터가 무엇인지 어떻게 동작하는지 어떻게 효율적으로 구현하는지 이해한다.
##### 2) 두 가지 유형의 디스크립터
##### 3) 디스크립터를 활용한 코드 재사용 방법
##### 4) 디스크립터의 좋은 사용 예를 살펴보고 자체 라이브러리의 API에 어떻게 활용할 수 있는지 살펴본다.
---
### 01. 디스크립터 개요
---
#### 1.1 디스크립터 메커니즘
##### 디스크립터 구현하려면 최소 두 개의 클래스가 필요함.
##### 1) 클라이언트 클래스 
##### → 디스크립터 구현의 기능을 활용할 도메인 모델, 솔루션을 위해 생성한 일반적인 추상화 객체 
##### 2) 디스크립터 클래스 
##### → 디스크립터 로직의 구현체 
##### 디스크립터는 디스크립터 프로토콜을 구현한 클래스의 인스턴스임.
##### 클래스는 다음 매직 메서드 중에 최소 한 개 이상을 포함해야 함
###### __ get__ 메서드 
###### →  클래스의 인스턴스를 읽으려고 할 때
###### __ set__ 메서드
###### → 클래스의 인스턴스를 설정하려고 할 때 
###### __ delete__ 메서드
###### → 인스턴스를 속성 중에서 없애려고 할 때
###### __ set_name__ 메서드
##### ★ 프로토콜이 동작하려면 디스크립터 객체가 항상 클래스 속성으로 정의되어야 한다는 것
##### 객체를 인스턴스 속성으로 생성하면 동작하지 않음, 따라서 init 메서드가 아니라 클래스 본문에 있어야 함
##### 디스크립터 프로토콜의 일부만 구현해도 된다는 것에 유의 (원하는 일부 메서드만 구현해도 됨)
#####  ClientClass의 인스턴스에서 descriptor 속성을 호출 시, 디스크립터 프로토콜이 사용됨. 

```python
# 일반적인 클래스의 속성 or 프로퍼티에 접근할 때
>>> class Attribute:
... value = 42
...
>>> class Client:
... attribute = Attribute()
>>> Client().attribute
<__main__.Attribute object at 0x7ff37ea90940>
>>> Client().attribute.value
42
```
##### 클래스 속성을 객체로 선언하면 디스크립터로 인식됨
##### 클라이언트에서 해당 속성을 호출하면 객체 자체를 반환하는 것이 아니라 __ get__ 을 반환

``` python
# 디스크립터의 경우
# 호출당시의 문맥 정보를 로깅하고 클라이언트 인스턴트를 그대로 반환
class DescriptorClass:
	def __get__(self, instance, owner):
		if instance is None:
			return self
			logger.info("Call: %s.__get__(%r, %r)",
			self.__class__.__name__,instance, owner)
			return instancee

class ClientClass:
	descriptor = DescriptorClass()
```
##### ClientClass 인스턴스의 descriptor 속성에 접근해보면  DescriptorClass 인스턴스를 반환하지 않고
##### __ get__() 메서드의 반환 값을 사용한다는 것을 알 수 있음.

```python
>>> client = ClientClass()
>>> client.descriptor
INFO:Call: DescriptorClass.__get__(<ClientClass object at 0x...>, <class'ClientClass'>)
<ClientClass object at 0x...>
>>> client.descriptor is client
INFO:Call: DescriptorClass.__get__(ClientClass object at 0x...>, <class'ClientClass'>)
True
```
##### 이 도구를 사용해 __ get__ 메서드 뒤쪽으로 모든 종류의 논리를 추상화 할 수 있으며
##### 클라이언트에게 내용을 숨긴 채로 모든 유형의 변환을 투명하게 실행할 수 있음
##### 이는 새로운 레벨의 캡슐화임.

#### 1.2 디스크립터 프로토콜의 메서드 탐색
##### 디스크립터는 단지 객체이기 때문에 이러한 메서드들은 self를 첫 번째 파라미터로 사용
##### self는 디스크립터 객체 자신을 의미
---
#### 1.2.1 __ get__(self, instance, owner)
##### instance : 디스크립터를 호출한 객체를 의미
##### owner : 해당 객체의 클래스를 의미
##### owner 파라마터를 추가한 이유 : ClientClass에서 descriptor를 호출하는 특별한 경우이기 때문
``` python
# 디스크립터가 클래스에서 호출될 때와 인스턴스에서 호출될 때 차이
# descriptors_methods_1. py
class DescriptorClass:
	def __get__(self, instance, owner):
		if instance is None:
			return f"{self.__class__.__name__}.{owner.__name__}"
		return f"value for {instance}"

class ClientClass:
	descriptor = DescriptorClass()
```
##### ClientClass에서 직접 호출하면 네임스페이스와 함께 클래스 이름을 출력
```python
>>> ClientClass.descriptor
'DescriptorClass.ClientClass'
```
##### 객체에서 호출하면 다른 메세지를 출력한다.

```python
>>> ClientClass().descriptor
'value for <descriptors_methods_1.ClientClass object at 0x...>'
```
#####  owner 파라미터를 사용하는 경우가 아니라면, 인스턴스가 None일 때는 단순히 디스크립터 자체를 반환
#### 1.2.2 __ set__(self, instance, value)
##### 디스크립터에 값을 할당하려고 할 때 호출
``` python
# 데이터를 저장할 때 사용
# 속성의 유효성을 검사하는 객체
class Validation:

	def __init__(self, validation_function, error_msg: str):
		self.validation_function = validation_function
		self.error_msg = error_msg

	def __call__(self, value):
		if not self.validation_function(value):
			raise ValueError(f"{value!r} {self.error_msg}")
		
class Field:

	def __init__(self, *validations):
		self._name = None
		self.validations = validations
	
	def __set_name__(self, owner, name):
		self._name = name
	
	def __get__(self, instance, owner):
		if instance is None:
			return self
		return instance.__dict__[self._name]
	
	def validate(self, value):
		for validation in self.validations:
			validation(value)

	def __set__(self, instance, value):
		self.validate(value)
		instance.__dict__[self._name] = value

class ClientClass:
	descriptor = Field(
		Validation(lambda x: isinstance(x, (int, float)), "는 숫자가 아님"),
		Validation(lambda x: x >= 0, "는 0보다 작음"),
	)
```
##### 다음과 같은 결과가 나옴
```python
>>> client = ClientClass()
>>> client.descriptor = 42
>>> client.descriptor
42
>>> client.descriptor = -42
Traceback (most recent call last):
...
ValueError: -42는 0보다 작음
>>> client.descriptor = "invalid value"
...
ValueError: 'invalid value'는 숫자가 아님
```
##### __ set()__ 메서드가 @property.setter 의 역할을 대신함

#### 1.2.3 __ delete__(self, instance)
##### self : descriptor 속성
##### instance : client
```python
# __delete__ 메서드를 사용하여 관리자 권한이 없는 객체에서 
# 속성을 제거하지 못하도록 하는 디스크립터

class ProtectedAttribute:
	def __init__(self, requires_role = None) -> None:
		self.permission_required = required_role
		self._name = None

	def __set_name__(self, owner, name):
		self._name = name

	def __set__(self, user, value):
		if value is None:
			raise ValueError(f"{self._name}를 None으로 설정할 수 없음")
		user.__dict__[self._name] = value

	def __delete__(self, user):
		if self.permission_required in user.permissions:
			user.__dict__[self._name] = None
		else:
			raise ValueError(
				f"{user!s} 사용자는 {self.permission_required} 권한이 없음"
			)

class User:
	"""admin 권한을 가진 사용자만 이메일 주소를 삭제할 수 있음"""
	email = ProtectedAttribute(requires_role="admin")
	
	def __init__(self, username: str, email: str, permission_list: list = None) -> None:
		self. username = username
		self. email = email
		self. permissions = permission_list or []

	def __str__(self):
		return self.username
```
##### User를 사용하고자 하는 객체는 email 속성이 있는 것으로 기대함
##### 따라서 email의 "삭제"는 단순히 None으로 설정하는 것으로 함
##### 그렇지 않으면 __ delete__ 메서드의 메커니즘을 우회해버리기 때문
```python
# admin 권한을 가진 사용자만 email 주소를 제거할 수 있음

>>> admin = User("root", "root@d.com", ["admin"])
>>> user - User("user", "user1@d.com", ["email", "helpdesk"])
>>> admin.email
'root@d.com'
>>> del admin.email
>>> admin.email is None
True
>>> user.email
'user1@d.com'
>>> user.email = None
...
ValueError: email를 None으로 설정할 수 없음
>>> del user.email
...
ValueError: user 사용자는 admin 권한이 없음
```
#### 1.2.4 __ set_name__(self, owner, name)
##### 클래스에 디스크립터 객체를 만들 때, 디스크립터가 처리하려는 속성의 이름을 알아야 함
##### 속성의 이름은 __ dict__에서  __ get__과 __ set__ 메서드로 읽고 쓸 때 사용됨
```python
# __set_name__이 없을 때의 전형적인 디스크립터 코드

class DescriptorWithName:
	def __init__(self, name):
		self.name = name
	
	def __get__(self, instance, value):
		if instance is None:
			return self
		logger.info("%r에서 %r 속성 가져오기", instance, self.name)
		return instnace.__dict__[self.name]
	
	def __set__(self, instance, value):
		instance.__dict__[self.name] = value

class ClientClass:
	descriptor = DescriptorWithName("descriptor")

>>> client = ClientClass()
>>> client.descriptor = "value"
>>> client.descriptor
INFO:<ClientClass object at 0x...>에서 'descriptor' 속성 가져오기
'value'
```
##### __ set_name__ 은 파라미터로 디스크립터를 소유한 클래스와 디스크립터의 이름을 받음.
##### 하위 호환을 위해 __ init__ 메서드에 기본값을 지정하고 __ set_name__을 함께 사용하는 것이 좋음
``` python
class DescriptorWithName:
	def __init__(self, name=None):
		self.name = name

	def __set_name__(self, owner, name):
		self.name = name
	...
class ClientClass:
	descriptor = DescriptorWithName()
# DescriptorWithName에 변수 이름을 복사하여 파라미터에 전달하지 않아도 됨.
```

### 02. 디스크립터의 유형
##### 디스크립터의 작동방식에 따라 디스크립터를 구분할 수 있음.
#####  데이터 디스크립터 (data descriptor) : __ set __ 이나  __ delete__ 메서드를 구현한 디스크립터
##### 비데이터 디스크립터 (non-data descriptor): __ get_만을 구현한 디스크립터
##### __ set_name__은 분류에 영향을 미치지 않음

#### 객체의 속성을 결정할 때
##### 데이터 디스크립터 > 객체의 사전 > 비데이터 디스크립터
---
#### 2.1 비데이터(non-data) 디스크립터
```python
# __get__ 메서드만을 구현
class NonDataDescriptor:
	def __get__(self, instance, owner):
		if instance is None:
			return self
		return 42

class ClientClass:
	descriptor = NonDataDescriptor()

# descriptor 호출하면 __get__ 메서드의 결과를 얻을 수 있음
>>> client = ClientClas()
>>> client.descriptor
42

# descriptor 속성을 다른 값으로 바꾸면 이전의 값을 잃고 대신에 새로운 값을 얻음
>>> client.descriptor = 43
>>> client.descriptor
43

# descriptor를 지우고 다시 물으면 ?
>>> del client.descriptor
>>> client.descriptor
42
```
##### client 객체를 만들었을 때 descriptor 속성은 인스턴트가 아니라 클래스 안에 있음
##### 따라서, client 객체의 사전을 조회하면 값은 비어 있게 됨
```python
>>> vars(client)
{}
# 여기서 .descriptor 속성을 조회하면 client.__dict__에서
# "descriptor"라는 이름의 키를 찾지 못하고 클래스에서 디스크립터를 찾게 됨
# 이는 __get__ 메서드의 결과가 반환되는 이유가 됨
```
##### 그러나  .descriptor 속성에 다른 값을 설정하면 인스턴스의 사전이 변경됨
##### client.__dict__는 비어있지 않게 됨
```python
>>> client.descriptor = 99
>>> vars(client)
{'descriptor': 99}

# 다시 del을 호출해 속성을 지우면 이전 시나리오로 돌아가게 됨
>>> del client.descriptor
>>> vars(client)
{}
>>> client.descriptor
42
```
#### 2.2 데이터 디스크립터
```python
class Descriptor:

	def __get__(self, instance, owner):
		if instance is None:
			return self
		return 42

	def __set__(self, instance, value):
		logger.debug("%s.descriptor를 %s 값으로 설정", instance, value)
		instance.__dict__["descriptor"] = value

class ClientClass:
	descriptor = DataDescriptor()

# descriptor의 반환 값을 확인
>>> client = ClientClass()
>>> client.descriptor
42

# 값을 변경하고 반환 값을 확인
>>> client.descriptor = 99
>>> client.descriptor
42
```
##### descriptor의 반환 값이 변경되지는 않음
##### 그러나, 다른 값으로 할당하면 객체의 __ dict__ 사전에는 업데이트가 되어야 함
```python
>>> vars(client)
{'descriptor':99}

>>> client.__dict__["descriptor"]
99

# __ set__() 메서드가 호출되면 객체의 사전에 값을 설정하기 때문임
```
##### 데이터 디스크립터에서 속성을 조회하면 객체의 __ dict__에서 조회하는 대신 클래스의 descriptor를 먼저 조회함
```python
>>> del client.descriptor
Traceback (most recent call last):
...
AttributeError: __delete__
```
##### 삭제가 되지 않는 이유 
##### 인스턴스의 __ dict__에서 속성을 지우려고 시도하는 것이 아닌
##### descriptor에서 __ delete()__ 메서드를 호출을 하게 되는데, 구현하지 않았기 때문에 삭제가 되지 않음.
### 03. 디스크립터 실전
##### 디스크립터를 통해 처리할 수 있는 몇 가지 상황
---
#### 3.1 디스크립터를 사용한 애플리케이션
##### 코드 중복을 디스크립터로 추상화하는 방법
#### 3.1.1 디스크립터를 사용하지 않은 예
```python
# 속성을 가진 일반적인 클래스인데 속성의 값이 달라질 때마다 추적하는 예제

class Traveller:
	def __init__(self, name, current_city):
		self.name = name
		self._current_city = current_city
		self._cities_visited = [current_city]
	
	@property
	def current_city(self):
		return self._current_city
	
	@current_city.setter
	def current_city(self, new_city):
		if new_city != self._current_city:
			self._cities_visited.append(new_city)
			self._current_city = new_city

	@property
	def cities_visited(self):
		return self._cities_visited

>>> alice = Traveller("Alice", "Barcelona")
>>> alice.current_city = 'Paris'
>>> alice.current_city = 'Brussels'
>>> alice.current_city = 'Amsterdam'
>>> alice.cities_visited
['Barcelona', 'Paris', 'Brussels', 'Amsterdam']
```
##### 위의 예제에 한해서 프로퍼티를 사용하는 것만으로도 충분
##### 애플리케이션의 여러 곳에서 똑같은 로직을 사용한다면 ?
#### 3.1.2 이상적인 구현방법
##### 모든 클래스에 적용할 수 있도록 디스크립터를 사용
```python
# 속성에 대해 이름을 가진 일반적인 디스크립터를 만들 것
class HistoryTracedAttribute:
	def __init__(self, trace_attribute_name) -> None:
		self.trace_attribute_name = trace_attribute_name #[1]
		self._name = None

	def __set_name_(self, owner, name):
		self._name = name

	def __get__(self, instance, owner):
		if instance is None:
			return self
		return instance.__dict__[self._name]

	def __set__(self, instance, value):
		self._track_change_in_value_for_instance(instance, value)
		instance.__dict__[self.name] = value

	def _track_change_in_value_for_instance(self, instance, value):
		self._set_default(instance) #[2]
		if self._needs_to_track_change(instance, value):
			instance.__dict__[self.trace._attribute_name].append(value)
	
	def _needs_to_track_change(self, instance, value) -> bool:
		try:
			current_value = instance.__dict__[self._name]
		except KeyError: #[3]
			return True
		return value != current_value #[4]

	def _set_default(self, instance):
		instance.__dict__.setdefault(self.trace_attribute_name, []) #[6]

class Traveller:
	
	current_city = HistoryTracedAttribute("cities_visited") #[1]

	def __init__(self, name, current_city):
		self.name = name
		self.current_city = current_city #[5]
```
###### [1] : 속성의 이름은 디스크립터에 할당된 변수 중 하나 (current_city) 
######      이에 대한 추적을 저장할 변수의 이름을 디스크립터에 전달 (cities_visited ← current_city)
###### [2] : 디스크립터를 처음으로 호출할 때, 추적 값이 존재하지 않음 
###### 따라서 나중에 추가할 수 있도록 비어있는 배열로 초기화
###### [3] : 처음 Traveller를 호출할 때 방문지가 없으므로 인스턴스 사전에서 current_city의 키도 존재하지 않음
###### 이런 경우 또한 새로운 여행지가 생긴 것이므로 추적의 대상이 됨
###### [4] : 새 값이 현재 설정된 값과 다른 경우에만 변경 사항을 추가
###### [5] : Traveller의 __ init__ 메서드에서 디스크립터가 이미 생성된 단계
###### 할당 명령은 2단계 값을 추적하기 위한 빈 리스트 만들기를 실행
###### 3단계를 실행하여 리스트에 값을 추가하고 나중에 검색하기 위한 키를 설정
###### [6] : 사전의 setdefault 메서드는 KeyError를 피하기 위해 사용
###### setdefault는 두 개의 파라미터를 받음 
###### 첫 번째 파라미터의 키가 있으면 해당 값을 반환 없으면 두 번째 파라미터 반환
##### → 디스크립터로 클라이언트 클래스의 코드를 상당히 간단하게 만듬
#### 3.2 다른 형태의 디스크립터
##### 전역 상태 공유(global shared state) 문제
#### 3.2.1 전역 상태 공유 이슈
##### 디스크립터는 클래스 속성으로 설정해야 함
##### 문제점 : 해당 클래스의 모든 인스턴스에서 공유됨
##### 디스크립터 객체에 데이터를 보관하면 모든 객체가 동일한 값에 접근 할 수 있음

``` python
# 각 객체에 데이터를 정의하는 대신 디스크립터가 데이터 자체를 유지하도록 하는 예제

class SharedDataDescriptor:
	def __init__(self, initial_value):
		self.value = initial_value
	
	def __get__(self, instance, owner):
		if instance is None:
			return self
		return self.value

	def __set__(self, instance, value):
		self.value = value

class ClientClass:
	descriptor = SharedDataDescriptor("첫 번째 값")

# 디스크립터 객체는 데이터 자체를 바로 저장.
# 이는 인스턴스의 값을 수정하면 같은 클래스의 다른 모든 인스턴스의 값도 수정됨을 의미

>>> client1 = ClientClass()
>>> client1.descriptor
'첫 번째 값'
>>> client2 = ClientClass()
>>> client2.descriptor
'첫 번째 값'
>>> client2.descriptor = "client2를 위한 값"
>>> client2.descriptor
'client2를 위한 값'
>>> client1.descriptor
'client2를 위한 값'

# ClientClass.descriptor가 고유하기 때문
```
##### 이를 해결하기 위해서 디스크립터는 각 인스턴스의 값을 보관했다가 반환해야 함

#### 3.2.2 객체의 사전에 접근하기
##### 디스크립터는 항상 객체의 사전 __ dict__에 값을 저장하고 조회함.
#### 3.2.3 약한 참조 사용
##### __ dict__를 사용하지 않으려는 경우
##### 디스크립터 객체가 직접 내부 매핑을 통해 각 인스턴스의 값을 보관하고 반환하는 것
