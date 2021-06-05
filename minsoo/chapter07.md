## Chapter 07. 제너레이터 사용하기
### 01. 제너레이터 만들기 
---
##### 한 번에 하나씩 구성요소를 반환해주는 이터러블을 생성해주는 객체
##### 거대한 요소를 한꺼번에 메모리에 저장하는 대신 특정 요소를 어떻게 만드는지 아는 객체를 만들어서 
##### 주요 목적인 메모리 절약을 달성할 수 있음.
#### 1.1 제너레이터 개요
##### 
```python
# 모든 구매 정보를 받아 필요한 지표를 구해주는 객체
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

```python
# 모든 정보를 로드해서 어딘가에 담아서 반환해주는 함수
	def _load_purchases(filename);
		purchases = []
		with open(filename) as f:
			for line in f:
				*_, price_raw = line.partition(",")
				purchases.append(float(price_raw))
	
	return purchases
```
##### 파일에서 모든 정보를 읽어서 리스트에 저장하는 방법은 성능에 문제가 있음
##### 제네레이터를 만들어서 필요한 값만 그때그때 가져오는 것
```python
# 제너레이터를 이용해서 수정
	def _load_purchases(filename);
		with open(filename) as f:
			for line in f:
				*_, price_raw = line.partition(",")
				yield float(price_raw)
```
##### yield 키워드를 사용하면 제너레이터 함수가 됨
##### yield가 포함된 함수를 호출하면 제너레이터의 인스턴스를 만듬
```python
>>> load_purchases("file")
<generator object load_purchases at 0x...> 
```
##### 모든 제너레이터 객체는 이터러블이다
##### 따라서 for 루프와 함께 사용할 수 있는데 이는 for 루프의 다형성을 보장하는 강력한 추상화가 가능함
#### 1.2 제너레이터 표현식
##### 리스트, 세트, 사전 컴프리헨션 처럼 제너레이터도 제너레이터 표현식으로 정의할 수 있음
###### 컴프리헨션 : 이터러블 객체를 쉽게 생성하기 위한 기법
```python
>>> [x**2 for x in range(10)] #리스트 컴프리헨션
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> (x**2 for x in range(10)) # 제너레이터 표현식
<generator object <genexpr> at 0x...>
>>> sum (x**2 for x in range(10)) # 이터러블 연산이 가능한 함수에 직접 전달 가능
285
```
##### 대괄호를 소괄호로 교체하면 표현식의 결과로부터 제너레이터가 생성됨
### 02. 이상적인 반복
---
#### 2.1 관용적인 반복 코드
##### enumerate() 는 이터러블을 입력 받아서 인덱스 번호와 원본의 원소를 튜플 형태로 변환 함
```python
>>> list(enumerate("abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]

# 시작 값을 입력하면 무한 시퀀스를 만드는 객체 만들기
# next 함수를 호출할 때마다 다음 시퀀스 값을 무제한 출력해 줌
class NumberSequence:
	def __init__(self, start=0):
		self.current = start
	
	def next(self):
		current = self.current
		self.current += 1
		return current
```
##### 위와 같이 하면 명시적으로 next() 함수를 호출해야함
```python
>>> seq = NumberSequence()
>>> seq.next()
0
>>> seq.next()
1
>>> seq2 = NumberSequence(10)
>>> seq2.next()
10
>>> seq2.next()
11

>>> list(zip(NumberSequence(), "abcdef"))
Traceback (most recent call last):
TypeError: zip argument #1 must support iteration
```
##### NumberSequence가 반복을 지원하지 않음
##### 해결하기 위해서는 __ iter__() 매직 메서드를 구현하여 객체가 반복 가능하게 만들어야 함
##### 또한, next() 메서드를 수정하여 __ next__ 매직 메서드를 구현해야 함
```python 
class SequenceOfNumbers:

	def __init__(self, start=0):
		self.current = start
	
	def __next__(self):
		current = self.current
		self.current += 1
		return current

	def __iter__(self):
		return self
```
##### 위와 같이 코드를 구현하면 요소를 반복할 수 있을 뿐 아니라, next() 메서드를 호출할 필요도 없음
```python
>>> list(zip(SequenceOfNumbers(), "abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
>>> seq = SequenceOfNumbers(100)
>>> next(seq)
100
>>> next(seq)
101
```
#### 2.1.1 next() 함수
##### next() 내장 함수는 이터러블을 다음 요소로 이동시키고 기존의 값 반환
```python
>>> word = iter("hello")
>>> next(word)
'h'
>>> next(word)
'e'
```
##### 이터레이터가 더 이상의 값을 가지고 있지 않다면 StopIteration 예외가 발생
```python
...
>>> next(word)
'o'
>>> next(word)
Traceback (most recent call last):
StopIteration
```
##### 위 문제를 해결하고 싶으면 next() 함수의 두 번째 파라미터에 기본 값을 제공할 수도 있음.
```python
>>> next(word, "default value")
'default value'
```
#### 2.1.2 제너레이터 사용하기
```python
# 제너레이터를 사용하면 클래스를 만드는 대신 
# 필요한 값을 yield하는 함수를 만들면 됨
def sequence(start=0):
	while True:
		yield start
		start += 1
```
##### 제너레이터 함수가 호출되면 yield 문장을 만나기 전까지 실행됨
##### 값을 생성하고 그 자리에서 멈춰있음
#### 2.1.3 Itertools
##### 이터레이터, 제너레이터, itertools 을 서로 연결하여 새로운 객체를 만들 수 있음
##### 구매 이력에서 지표를 계산하는 과정에서 '특정 기준을 넘은 값'에 대해서만 연산을 하려면 ?
##### while 문 안에 조건문을 추가하는 것 → 파이썬스럽지 않고 너무 엄격함
```python
# 1000개 넘게 구매한 이력의 처음 10개만 처리하려고 할 때
from itertools import islice
purchases = islice(filter(lambda p: p > 1000.0, purchases), 10)
stats = PurchasesStats(purchases).process() 
```
##### 전체에서 필터링한 값으로 연산한 것 처럼 보이지만, 실제로는 하나씩 가져온 것임
#### 2.1.4 이터레이터를 사용한 코드 간소화
##### 1) 여러 번 반복하기
```python
def process_purchases(purchases):
	min_, max_, avg = itertools.tee(purchases, 3) # 반복을 여러 번 해야되는 경우
	return min(min_),max(max_), median(avg)
```
##### itertools.tee 는 원래의 이터러블을 세 개의 새로운 이터러블로 분할해서 오직 한번만 순환함
##### 2) 중첩 루프
##### 1차원 이상을 반복해서 값을 찾을 경우 
##### 값을 찾은 후 순환을 멈추고 break 키워드를 호출해서 루프를 빠져나와야 함
##### 그러나 두 단계 이상을 벗어나야 하므로 정상적으로 작동하지 않음
##### 가장 좋은 방법 → 중첩을 풀어서 1차원 루프로 만드는 것
``` python
# 피해야 할 코드

def search_nested_bad(array, desired_value):
	coords = None
	for i, row in enumerate(array):
		for j, cell in enumerate(row):
		if cell == desired_value:
			coords = (i,j)
			break
		
		if coords is not None:
			break

	if coords is None:
		raise ValueError(f"{desired_value} not found")

	logger.info("[%i, %i]에서 값 %r 찾음", *coords, desired_value)
	return coords
```
##### 종료 플래그를 사용하지 않은 예
```python
def _iterate_array2d(array2d):
	for i, row in enumerate(array2d):
		for j, cell in enumerate(row):
			yield (i,j), cell

def search_nested(array, desired_value):
	try:
		coord = next(
			coord
			for (coord, cell) in _iterate_array2d(array)
			if cell == desired_value
		)
	except StopIteration:
		raise ValueError("{desired_value} not found")
	
	logger.info("[%i, %i]에서 값 %r 찾음", *coords, desired_value)
	return coord 
```
##### 최대한 중첩 루프를 제거하고 추상화하여 반복을 단순화한다
#### 2.2 파이썬의 이터레이터 패턴
##### 이터레이터 : __ iter__() 와 __ next__ () 매직 메서드를 구현한 객체
#### 2.2.1 이터레이터 인터페이스
##### 이터러블은 반복할 수 있는 어떤 것으로 실제 반복을 할 때는 이터레이터를 사용
