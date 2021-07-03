> 해당 게시글은 <파이썬 클린 코드: 유지보수가 쉬운 파이썬 코드를 만드는 비결, 마리아노 아나야 지음> 책의 6장을 참고하여 작성되었습니다

# Chapter 06. 디스크립터로 더 멋진 객체 만들기

해당 장에서는 다른 언어에서는 생소한, 파이썬의 새로운 고급기능인 디스크립터를 소개한다. 디스크립터는 파이썬의 객체지향 수준을 한 단계 더 높여주는 기능으로, 보다 견고하고 재사용성이 높은 추상화를 가능하게 한다

### 6장의 목표

- 디스크립터란 무엇이며 어떻게 동작하며 효율적으로 구현하는지 이해하자

- 두 가지 유형의 디스크립터의 개념적 차이와 세부 구현의 차이를 분석한다

  1. data discriptor : 데이터 디스크립터
  2. non-data discriptor : 비데이터 디스크립터

- 디스크립터를 활용한 코드 재사용 방법

- 디스크립터의 좋은 사용 예를 보고, 자체 라이브러리의 API에 어떻게 활용될 수 있는지 살펴보자

  

## 디스크립터 개요

------

디스크립터를 사용하면 완전히 새롭게 프로그램의 제어 흐름을 변경할 수 있다. 먼저 디스크립터의 내부 동작 메커니즘을 이해하기 위해 배경에 있는 주요 개념을 먼저 알아보도록 하자

### 디스크립터 메커니즘

동작방식은 그리 복잡하지 않으나 세부 구현 시의 주의사항이 많으며, 디스크립터 구현을 위해서는 아래와 같이 최소 두 개의 클래스가 필요하다

1. 클라이언트 클래스 : 디스크립터 구현의 기능을 활용할 도메인 모델로서 솔루션을 위해 생성한 일반적인 추상화 객채
2. 디스크립터 클래스 : 디스크립터 로직의 구현체 

디스크립터는 단지 **디스크립터 프로토콜을 구현한 클래스의 인스턴스**이며 이 클래스는 다음 매직 메서드 중 최소 한 개 이상을 포함해야 한다(Python 3.6 기준의 Discriptor protocol)

- `__get__`
- `__set__`
- `__delete__`
- `__set_name__`

다음과 같은 네이밍 컨벤션을 사용한다

| NAME            | MEAN                                                         |
| --------------- | ------------------------------------------------------------ |
| ClientClass     | 디스크립터 구현체의 기능을 활용할 도메인 추상화 객체로 디스크립터 클라이언트다. 클래스 속성으로 디스크립터를 가진다 |
| DescriptorClass | 디스크립터 클래스로, 디스크립터 프로토콜을 따르는 매직 메서드를 구현해야 함 |
| Client          | ClientClass의 인스턴스                                       |
| Descriptor      | DescriptorClass의 인스턴스로 이 객체는 클래스 속성으로서 ClientClass에 위치한다 |

프로토콜이 동작하기 위해서는 반드시 **디스크립터 객체가 클래스 속성으로 정의되어야 함**

디스크립터의 동작 방식 예제를 살펴보자

```python
class DecriptorClass:
  def __get__(self, instance, owner):
    if instance is None:
      return self
    logger.info("Call: %s.__get__(%r, %r)", self.__class__.name__, instance, owner)
    return instance
  
class ClientClass:
  descriptor = DescriptorClass()
    
```

- `client = ClientClass()` : ClientClass 인스턴스 생성

  - `client.descriptor` : descriptor 속성에 접근시 `__get__()` 메서드의 반환 값 사용

  

### 디스크립터 프로토콜의 메서드 탐색

디스크립터는 단지 객체이므로 `self`를 첫 번째 파라미터로 사용한다.  디스크립터 프로토콜의 각 메서드에 사용되는 파라미터와 사용 방법에 대해서 알아보도록 하자(`self`는 디스크립터 객체 자신을 의미함)

**1. `__get__(self, instance, owner)`**

- instance : 디스크립터를 호출한 객체
  - client 객체에 해당
- owner : 해당 객체의 클래스
  - ClientClass에 해당

예제

```python
class DecriptorClass:
  def __get__(self, instance, owner):
    if instance is None:
      return f"{self.__class__.name__}.{owner.__name__}"
    return f"value for{instance}"
  
class ClientClass:
  descriptor = DescriptorClass()
```

- `ClientClass.descriptor` -> `DescriptorClass.ClientClass`

- `ClientClass().descriptor` -> `value for {...contents of instance infor...}`

  

**2. `__self__(self, instance, value)`** 

기본적으로 데이터를 저장할 때 사용되는 메서드지만, 유효성 검사 객체를 생성할 때도 활용 가능

- instance : 디스크립터를 호출한 객체
  - client 객체에 해당
- value : "value"라는 문자열에 해당

**3. `__delete__(self, instance)`**

```python
del client.descriptor
```

- self : descriptor 의 속성을 나타냄
- instance : cliente를 나타냄

**4. `__set_name__(self, owner, name)`**

python 3.6 이상 버전 부터 추가되어 파라미터로 디스크립터를 소유한 클래스와 디스크립터의 이름을 받을 수 있게 됨.

```python
class DescriptorWithName:
  def __init__(self, name = None):
    self.name = name
  def __set_name__(self, owner, name):
    self.name = name
 
# ... 
 
class ClientClass:
  descriptor = DescriptorWithName()
```



## 디스크립터의 유형

### 비데이터(non-data) 디스크립터

### 데이터 디스크립터

