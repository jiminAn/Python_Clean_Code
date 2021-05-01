# Chapter 04. SOLID 원칙

### SOLID란?

이해하기 쉽고 유연하며 유지 보수가 쉬운 SW 개발을 위한 다섯가지 SW 설계 원칙

| 약어 | 원칙                            | 한글 명칭            |
| ---- | ------------------------------- | -------------------- |
| SRP  | Single Responsibility Principle | 단일 책임 원칙       |
| OCP  | Open-Closed Principle           | 개방-폐쇄 원칙       |
| LSP  | Liskov Substitution Principle   | 리스코프 치환 원칙   |
| ISP  | Interface Segregation Principle | 인터페이스 분리 원칙 |
| DIP  | Dependency Inversion Principle  | 의존 역전 원칙       |

### 4장의 목표

- SW 디자인에서 SOLID 원칙을 익힌다
- SRP을 따르는 컴포넌트를 디자인한다
- OCP을 통해 유지보수성을 뛰어나게 한다
- LSP을 준수하여 객체지향 디자인에서 적절한 클래스 계층을 설계한다
- ISP와 DIP을 활용해 설계하기



## 단일 책임 원칙(Single Responsibility Principle - SRP)

------

### SRP이란? 

- SW 컴포넌트(클래스)가 단 하나의 책임을 져야한다는 원칙
- 클래스의 메서드는 상호 배타적이며 서로 관련이 없어야 함
  - 따라서 서로 다른 책임을 가지고 있으므로 더 작은 클래스로 분해할 수 있어야 함
- 클래스에 있는 프로퍼티와 속성이 항상 메서드를 통해서 사용되도록 하는 것
  - 프로퍼티 / 속성  : ??
  - 이는 응집력과 밀접한 관련이 있음

### 너무 많은 책임을 가진 클래스

아래의 예제는 독립적인 동작을 하는 메서드를 하나의 인터페이스에 정의했다는 문제점을 가짐 

```python
class SystemMonitior:
  def load_activity(self):
    	"""소스에서 처리할 이벤트 가져오기"""
  def identify_events(self):
    	"""가져온 데이터를 파싱하여 도메인 객체 이벤트로 전환"""
  def stream_events(self):
    	"""파싱한 이벤트를 외부 에이전트로 전송"""
```

클래스가 너무 많은 책임을 가질 경우 발생하는 문제점

- 클래스가 손상되기 쉽고 유지보수가 어려움
- 클래스가 경직되고 융통성 없으며 오류가 발생하기 쉬움

-> 외부 요소에 의한 영향을 최소화하고 싶을 때 해결책은 보다 **작고 응집력 있는 추상화를 하는 것**

### 책임 분산

솔루션을 보다 관리하기 쉽게 하기 위해 모든 메소드를 다른 클래스로 분리하여 각 클래스마다 단일 책임을 갖게 하자

```python
#class AlertSystem(SystemMonitor, ActivityReader, Output): # 상속 관계 확인 필요
class AlertSystem:
  def run(self):
    """SystemMonitor의 indentify_events() 호출"""
    """ActivityReader의 load() 호출"""
    """Output의 stream() 호출"""
```

```python
class SystemMonitior:
  def identify_events(self):
    	"""가져온 데이터를 파싱하여 도메인 객체 이벤트로 전환"""
```

```python
class ActivityReader:
  def load(self):
    	"""소스에서 처리할 이벤트 가져오기"""
```

```python
class Output:
  def stream(self):
    	"""파싱한 이벤트를 외부 에이전트로 전송"""
```

- 각각의 객체들은 특정한 기능을 캡슐화하여 나머지 객체들에게 영향을 미치지 않음
- 각 클래스들은 명확하고 구체적인 의미를 가짐



## 개방/폐쇄 원칙(Open/Close Principle - OCP)

------

### OCP란?

- 클래스를 디자인할 때는 유지보수가 쉽도록 로직을 캡슐화하여 확장에는 개방되어야 하고 수정에는 폐쇄되도록 해야 한다
- 즉, 기존 코드를 변경하지 않고 확장할 수 있도록 만들어야 함

### OCP 원칙을 따르지 않을 경우 유지보수의 어려움

```python
class Event:
  def __init(self, raw_data):
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
    if( 
      ...
    ):
    	return LoginEvent(self.event_data)
   elif(
   		...
   ):
  		return LogoutEvent(self.event_data)
	return UnKownEvent(self.event_data)
```

- 이벤트 유형을 결정하는 논리가 일체형으로 중앙 집중화됨
  - 지원하려는 이벤트가 늘어날수록 메서드가 커짐
- 메서드가 수정을 위해 닫히지 않았음
  - 새로운 유형의 이벤트를 시스템에 추가할 때마다 메서드를 수정해야한다
  - elif명령문 체인은 가독성 측면에서 최악

**지향점**

1. 메서드를 변경하지 않고도 새로운 유형의 이벤트 추가(폐쇄 원칙)
2. 새로운 이벤트가 추가될 때 이미 존재하는 코드를 변경하지 않고 코드를 확장하여 새로운 유형의 이벤트 추가(개방 원칙)

### 확장성을 가진 이벤트 시스템으로 리펙토링

- SystemMonitor클래스를 추상적인 이벤트와 협력하도록 변경
- 이벤트에 대응하는 개별 로직은 각 이벤트 클래스에 위임
- 각각의 이벤트에 다형성을 가진 새로운 메서드를 추가 :  `meets_condition(event_data:dict)`
- [@staticmethod](https://dojang.io/mod/page/view.php?id=2379) : 참고

```python
class Event:
  def __init(self, raw_data):
    self.raw_data = raw_data
    
  @staticmethod
  def meets_condition(event_data:dict):
    return False

class UnknownEvent(Event):
  """데이터만으로 식별할 수 없는 이벤트"""
  
class LoginEvent(Event):
  @staticmethod
  def meets_condition(event_data:dict):
    return(
    	...
    )
  
  
class LogoutEvent(Event):
  """로그아웃 사용자에 의한 이벤트"""
  @staticmethod
  def meets_condition(event_data:dict):
    return(
    	...
    )
  
class SystemMonitor:
  """시스템에서 발생한 이벤트 분류"""
  def __init__(self, event_data):
    self.event_data = event_data
  def identify_event(self):
    """이벤트 분류"""
    for event_cls in Event.__subclasses__():
      try:
        if event_cls.meets_condition(self.event_data):
          return event_cls(self.event_data)
      except KeyError:
        continue
	return UnKownEvent(self.event_data)
```

### OCP 최종 정리

- 다형성의 효과적인 사용과 밀접하게 관련

  : 다형성을 따르는 형태의 계약을 만들고 모델을 쉽게 확장할 수 있는 일반적인 구조로 디자인하는 것

- OCP는 SW엔지니어링의 중요한 문제인 유지보수성에 대한 문제를 해결

- 코드를 변경하지 않고 기능을 확장하기 위해서는 보호하려는 추상화에 대해서 적절한 폐쇄를 해야 함

  - 단, 일부 추상화의 경우 충돌이 발생할 수 있으므로 항상 해당 원칙이 적용가능한 것은 아님



## 리스코프 치환 원칙(Liskov substitution principle-LSP)

------

### LSP란?

- 설계 시 안정성을 유지하기 위해 객체 타입이 유지해야하는 일련의 특성
- 어떤 클래스에서든 클라이언트는 하위 타입을 사용할 수 있어야 함
  - 즉 클라이언트는 완전히 분리되어 있으며 클래스 변경 사항과 독립되어야 함
  - ex:) 자식 클래스는 부모 클래스를 대체할 수 있어야 함

### 도구를 사용해 LSP 문제 검사하기

1장에서 소개한 `MyPy`나 `Pylint` 같은 도구를 사용해 쉽게 검출할 수 있음

- [메서드 서명 참고](https://www.netinbag.com/ko/internet/what-is-a-method-signature.html)

1. 메서드 서명의 잘못된 데이터타입 검사

   사전조건  : 코드 전체에 타입 어노테이션을 사용, MyPy를 설정

   이 경우 초기에 기본오류 여부와 LSP 준수 여부를 빠르게 확인 할 수 있음

2. Pylint로 호환되지 않는 서명 검사

   메서드의 서명 자체가 완전히 다른 경우를 검출 할 수 있음

### 애매한 LSP 위반 사례

자동화된 도구로 검사하기 애매한 경우는 코드 리뷰를 통해 확인할 수 있다

```python
class Event:
  def __init(self, raw_data):
    self.raw_data = raw_data
  @staticmethod
  def meets_condition(event_data:dict):
    return False
  @staticmethod
  def meets_condition_pre(event_data:dict):
    """인터페이스 계약의 사전조건 event_data 파라미터가 적절한 형태인지 유효성 검사"""
    assert ...
```

- 올바른 이벤트 유형을 탐지하기 전에 사전조건을 먼저 검사

```python
  class SystemMonitor:
  """시스템에서 발생한 이벤트 분류"""
      def __init__(self, event_data):
        self.event_data = event_data
      def identify_event(self):
        """이벤트 분류"""
        Event.meets_condition_pre(self.event_data)
        event_cls = next(...)
        return event_cls(self.event_data)
```

### LSP 최종 정리

- 인터페이스의 메서드가 올바른 계층구조를 갖도록 하여 상속된 클래스가 부모클래스와 다형성을 유지하도록 함
- LSP에서 제안하는 방식으로 신중하게 클래스를 디자인하면 계층을 올바르게 확장하는데 도움이 된다
  - 즉, LSP가 OCP에 기여함



























