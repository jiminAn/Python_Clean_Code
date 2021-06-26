> 해당 게시글은 <파이썬 클린 코드 : 유지보수가 쉬운 파이썬 코드를 만드는 비결, 마리아노 아나야 지음> 책의 7장을 참고하여 작성되었습니다



# Chapter 07. 제너레이터 사용하기

제너레이터는 파이썬의 특징적인 기능 중 하나로 해당 장에서는 제너레이터의 이론적 근거와 소개 배경, 이를 통한 문제 해결 사례를 살펴보도록 하자

### 7장의 목표

- 프로그램의 성능을 향상시키는 제너레이터 만들기
- 이터레이터가 파이썬에 어떻게 완전히 통합되었는가
- 이터레이션 문제의 이상적 해결 방법
- 제너레이터가 어떻게 코루틴과 비동기 프로그래밍의 기반이 되는 역할을 하는지
- 코루틴을 지원하기 위한 문법의 세부 기능 확인



## 기술적 요구사항

------

해당 장의 예제는 Python 3.6 버전에서 정상 작동하며 코드는 [여기](https://github.com/PacktPublishing/Clean-Code-in-Python/tree/master/Chapter07)를 참고



## 제너레이터 만들기

------

### 제너레이터(Generator)란?

한 번에 하나씩 구성요소를 반환해주는 이터러블을 생성해주는 객체로, 주 목적은 메모리를 절약하는 것이다. 이는, 구성요소 전체를 한꺼번에 메모리에 저장하는 대신 특정 요소를 어떻게 만드는지 아는 객체를 만들어서 필요할 때마다 하나씩만 가져오는 방식으로 실현할 수 있다.

이 기능은, 다른 함수형 프로그래밍 언어가 제공하는 것과 비슷한 방식으로 게으른 연산을 통해 무거운 객체를 사용할 수 있도록 한다

**게으른 연산(Lazy Computation)이란?** [참고 사이트 : 하스켈, 왜 이렇게 어려운거야?](https://panty.run/why-haskell-is-so-difficult/) 



### 제너레이터 개요

대규모의 구매 정보에서 최저 판매가, 최고 판매가, 평균 판매가를 구한다고 해보자. 파일은 아래와 같이 두 개의 필드만 가진 csv file이다

```
<purchase_data>, <price>
```

구매 상태 클래스는 아래와 같다. 해당 객체는 구매 정보를 받아서 필요한 계산을 하게 된다

```python
class PurchasesStats:
    def __init__(self, purchases):
        self.purchases = iter(purchases)
        self.min_price: float = None
        self.max_price: float = None
        self._total_purchases_price: float = 0.0
        self._total_purchases = 0
        self._initialize()

    def _initialize(self):
        try:
            first_value = next(self.purchases)
        except StopIteration:
            raise ValueError("no values provided")

        self.min_price = self.max_price = first_value
        self._update_avg(first_value)

    def process(self):
        for purchase_value in self.purchases:
            self._update_min(purchase_value)
            self._update_max(purchase_value)
            self._update_avg(purchase_value)
        return self

    def _update_min(self, new_value: float):
        if new_value < self.min_price:
            self.min_price = new_value

    def _update_max(self, new_value: float):
        if new_value > self.max_price:
            self.max_price = new_value

    @property
    def avg_price(self):
        return self._total_purchases_price / self._total_purchases

    def _update_avg(self, new_value: float):
        self._total_purchases_price += new_value
        self._total_purchases += 1

    def __str__(self):
        return (
            f"{self.__class__.__name__}({self.min_price}, "
            f"{self.max_price}, {self.avg_price})"
        )


```

이 모든 정보를 로드해서 어딘가에 담아서 반환해주는 함수를 만들어보자

**Ver1. list에 모든 데이터를 append하여 사용하는 방법**

- 정상적인 결과를 반환하나 파일에서 모든 정보를 읽어서 리스트에 저장하므로 성능상 문제가 있음
- 파일에 많은 데이터가 있을 경우 로드하는데 시간이 오래 걸리고 메인 메모리에 담지 못할 만큼 큰 데이터일 수 있음

```python
def _load_purchases(filename):
  purchases = []
  with open(filename) as f:
    for line in f:
      *_, price_raw = line.partition(",")
      purchases.append(float(price_raw))
  return purchases
```

**Ver2. 제너레이터 사용**

- 파일의 전체 내용을 리스트에 보관하는 대신에 필요한 값만 그때그때 가져오는 것
- 함수에서 `yield` 키워드 사용시 제너레이터 함수가 되며, 해당 함수 호출 시 제너레이터의 인스턴스를 만들게 됨
  - 모든 제너레이터 객체는 이터러블임(for 루프와 함께 사용할 수 있음)
  - 이터러블을 사용하면 for 루프의 다형성을 보장하는 강력한 추상화가 가능하며, 투명하게 객체의 요소 반복이 가능함

```python
def load_purchases(filename):
  with open(filename) as f:
    for line in f:
      *_, price_raw = line.partition(',')
      yield float(price_raw)
```



### 제너레이터 표현식

제너레이터 사용시 많은 메모리를 절약할 수 있으며, List, Tuple, Set 처럼 많은 메모리를 필요로 하는 이터러블이나 컨테이너의 대안이 될 수 있다.

컴프리헨션(comprehension)에 정의될 수 있는 List, Set, Dictionary처럼 제너레이터도 표현식으로 정의할 수 있다.

```python
>>> [x**2 for x in range(5)]
[0,1,4,9,16]

>>> (x**2 for x in range(10))
<generator object <genexpr> at 0x...>

>>> sum(x**2 for x in range(10)) 
285
```



## 이상적인 반복

------

 파이썬에서 반복 시 유용하게 사용할 수 있는 관용적인 코드를 살펴보도록 하자 

아래는 시작 값을 입력하면 무한 시퀀스를 만드는 단순한 역할을 하는 객체를 두 가지 버전으로 작성한 코드이다

### 관용적인 반복 코드

**Ver1. next() 호출 시 다음 값을 리턴해주는 class**

- 이 경우 NumberSequence는 반복을 지원하지 않는다. 즉, 이터러블 형태의 파라미터로는 사용할 수 없다

```python
class NumberSequence:
	def __init__(self, start = 0):
    self.current = start
  def next(self):
    current = self.current
    self.current += 1
    return current
```

**Ver1 . 매직 메서드를 구현한 class**

- 요소를 반복할 수 있어 이터러블 형태의 파라미터로 사용할 수 있다
  - e.g. `zip(arg1, agr2)` 의 파라미터로 사용 가능

```python
class SequenceOfNumbers:
  def __init__(self, start = 0):
    self.current = start
  def __next__(self):
    current = self.current
    self.current += 1
    return current
  def __iter__(self):
    return self
```

1. #### next() 함수

   - `next()` 내장 함수는 이터러블을 다음 요소로 이동시키고 기존의 값을 반환함
   - 만약, 다음 값이 없다면 StopIteration 예외가 발생함
     - 예외 캐치 혹은 `next()` 함수의 두 번째 파라미터에 default value 설정하여 기본 값을 반환하도록 설정 가능

   

2. #### 제너레이터 사용하기

   - 클래스를 만드는 대신 `yield`  키워드를 사용하여 함수를 제너레이터로 만들어 줄 수 있음
   - 제너레이터는 무한 루프를 사용해도 안전함
     - 함수 호출 시,  `yield`  문장을 만나기 전까지 실행되고, 그 값을 생성하고 그 자리에서 멈추기 때문

3. #### itertools

   - Itertools 모듈 사용 시 이터러블 기능을 온전히 활용할 수 있음

   - 예제 : 1000개 넘게 구매한 이력의 처음 10개만 처리하려고 할 경우

     ```python
     from itertools import islice
     purchases = islice(filter(lambda p : p > 1000.0 , purchase), 10) # Lazy evaluation
     stats = PurchaeStats(purchases.process())
     ```

4. #### 이터레이터를 사용한 코드 간소화

   지금까지 이터레이터/itertools 모듈을 활용해 코드를 개선하는 것을 살펴보았는데, 아래의 사례의 최적화 방법에 대한 결론을 추론해보자

   1. 여러 번 반복하기

      반복을 여러번 해야 되는 경우 `itertools.tee`를 사용하자

   2. 중첩 루프

      1차원 이상의 중첩 루프가 필요한 경우 **플래그/예외/코드를 나누어 함수에서 반환하는 방법**을 사용하는 방법은 권장되지 않는다.

      가장 좋은 방법은 중첩을 풀어서 1차원 루프로 만드는 것이다

      - 보조 제너레이터를 사용해 반복을 추상화한다

        ```python
        def _iterator_array2d(array2d):
          for i, row in enumerate(array2d):
            for j, cell in enumerate(row):
              yield (i,j), cell
              
        def search_nested(array, desired_value):
          try:
            coord = next(
              coord
              for (coord, cell) in _iterator_array2d(array)
              if cell == desired_value
            )
          except StopIteration:
            raise ValueError("{desired_value} not found")
            
          logger.info("found Value %r in [%i, %i]", desired_value, *coords)
          return coord
        ```

        

### 파이썬의 이터레이터 패턴

이터레이터는  `__iter__()` 와 `__next__()` 매직 메서드를 구현한 객체이다. 일반적으로 이렇게 구현을 하나, 항상 이 두 가지를 구현해야하는 것은 아니다. 해당 장에서는   `__iter__()` 를 구현한 이터러블 객체와 `__next__()` 를 구현한 이터레이터 객체를 비교해보도록 하자 또한 시퀀스와 컨테이너 객체의 반복에 대해서 알아보자

#### 1. 이터레이션의 인터페이스

- 이터러블 vs 이터레이터? 이터러블은 반복을 지원하는 객체로, 이를 실제 반복할 때 이터레이터를 사용한다
  - 즉,  `__iter__()` 를 통해 이터레이터를 반환,  `__next__()` 를 통해 반복 로직을 구현

- 모즌 제너레이터는 이터레이터이다 
  - 이터레이터는 단지 next() 함수 호출 시 일련의 값에 대해 한 번에 하나씩만 어떻게 생성하는지 알고 있는지 객체임

- 이터레이션의 인터페이스

  | 파이썬 개념 | 매직 메서드  | 비고                                                         |
  | ----------- | ------------ | ------------------------------------------------------------ |
  | Iterable    | `__iter__()` | 이터레이터와 함께 반복 로직 생성, 이터러블은 `for ... in ...` 구문에서 사용 가능 |
  | Iterator    | `__next__()` | 한 번에 하나씩 값을 생산하는 로직을 정의, 더 이상 생산값이 없을 경우 `StopIteration` 예외 발생, 내장 next( ) 함수 사용해 하나씩 값 읽어올 수 있음 |

- 이터러블 하지 않은 이터레이터 객체의 예

  - 시퀀스에서 하나씩 값을 가져올 수는 있으나 무한 루프를 유발하므로 반복은 불가

  ```python
  class SequenceIterator:
    def __init__(self, start = 0, step = 1):
      self.current = start
      self.strp = step
      
    def __next__(self):
      value = self.current
      self.current += self.step 
      return value
  ```

#### 2. 이터러블이 가능한 시퀀스 객체

- `for` loop에서 사용 가능한 형태

  -  `__iter__()` 메서드를 구현한 객체
  - 객체가 시퀀스인 경우
    -  즉, `__getitem__()`,   `__len__()` 매직 메서드를 구현한 경우 
    - 이 경우 인터프리터는 IndexError 예외가 발생할 때까지 순서대로 값을 제공함

- map() 을 구현한 시퀀스 객체 예제

  ```python
  class MappedRange:
    """특정 숫자 범위에 대해 맵으로 변환"""
    def __init__(self, transformation, start, end):
      self._transformation = transformation
      self._wrapped = range(start, end)
      
    # 인스턴스 변수에 직접 접근하지 않고 객체 자체를 통해서 슬라이싱을 구현할 수 있음
    def __getitem__(self, idx): 
      value = self._wrapped.__getitem__(idx)
      result = self._transformation(value)
      logger.info("Index %d: %s", idx, result)
      return result
    
    def __len__(self):
      return len(self._wrapped)
    
  ```

- 위와 같은 방법으로, `__iter__()` 가 구현되어 있지 않아도 for  루프를 사용해 반복가능하나, 대부분의 경우는 단순히 반복 가능한 객체를 만드는 것이 아니라 적절한 시퀀스를 만들어 해결하는 것이 바람직하다

































