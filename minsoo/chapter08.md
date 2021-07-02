## Chapter 08. 단위 테스트와 리팩토링


**단위 테스트(Unit Test)**는  하나의 모듈을 기준으로 독립적으로 진행되는 가장 작은 단위의 테스트이다. 

여기서 모듈은 애플리케이션에서 작동하는  하나의 기능 또는 메소드로 이해할 수 있다. 

예를 들어 웹 애플리케이션에서 로그인 메소드에 대한 독립적인 테스트가 1개의 단위테스트가 될 수 있다.

즉, 단위 테스트는 애플리케이션을 구성하는 하나의 기능이 올바르게 동작하는지를 독립적으로 테스트하는 것으로, "어떤 기능이 실행되면 어떤 결과가 나온다" 정도로 테스트를 진행한다.

출처: [https://mangkyu.tistory.com/143](https://mangkyu.tistory.com/143) 


### 01. 디자인 원칙과 단위 테스트
---
**단위 테스트**는 다른 코드의 일부분이 유효한 지를 검사하는 코드이고 소프트웨어의 핵심이 되는 필수적인 기능임.

단위 테스트 특징
1. 격리
2. 성능
3. 자체 검증

단위 테스트는 단위 테스트를 작성한 `.py` 파일을 만들고 도구에서 호출하는 것

`.py` 파일에는 
1. 비즈니스 로직에서 필요한 것을 가져오기 위한 import 구문
2. 테스트하기 위한 프로그램

이 있다.

테스트 도구에서 파일의 내용을 호출하면 테스트가 실행되는데, 

> 성공 : `점(.)`  /  실패 : `F`   /  예외 발생 : `E`

를 출력

#### 1.1 자동화된 테스트의 다른 형태

단위 테스트는 함수 또는 메서드와 같은 작은 단위를,
클래스를 테스트하려면 단위 테스트의 집합인 **테스트 스위트( test suite )** 를 사용

단위 테스트는 여러 방법으로 할 수 있으나 모든 오류를 잡을 수 있는 것은 아님.

---
**통합 테스트**는 한 번에 여러 컴포넌트를 테스트하고 종합적으로 예상대로 잘 동작하는지 검증하는 테스트.

**인수 테스트**는 유스케이스 ( use case )를 활용하여 사용자의 관점에서 시스템의 유효성을 검사하는 자동화된 테스트.

위의 두 가지 테스트를 하면 **속도**라는 단위 테스트의 중요한 특성을 잃게 됨

---

좋은 개발 환경을 구축했을 때 전체 테스트 스위트를 만들고 코드에 수정이 생길 때마다 **반복적으로 단위 테스트와 리팩토링** 을 할 수 있어야 함.
 
**풀 요청** ( Pull Request )은 나와 리포지토리의 다른 사용자가 검토하고, 주석을 추가하며, 한 브랜치에서 다른 브랜치로 코드를 변경하기 위한 기본적인 방법. 

풀 요청을 사용하여 중요하지 않은 변경 내용이나 수정 사항, 중요 기능 추가 또는 릴리스된 소프트웨어의 새 버전에 대한 코드 변경 내용을 공동으로 검토할 수 있음

**CI** (Continuous Integration) 는 개발자를 위한 자동화 프로세스인 지속적인 통합을 의미 CI를 성공적으로 구현할 경우 애플리케이션에 대한 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 공유 리포지토리에 통합되므로 여러 명의 개발자가 동시에 애플리케이션 개발과 관련된 코드 작업을 할 경우 서로 충돌할 수 있는 문제를 해결할 수 있음.

**브랜치**란 독립적으로 어떤 작업을 진행하기 위한 개념. 필요에 의해 만들어지는 각각의 브랜치는 다른 브랜치의 영향을 받지 않기 때문에, 여러 작업을 동시에 진행할 수 있음.

---
**실용성이 이상보다 우선임을 기억**해야 함.

#### 1.2 단위 테스트와 애자일 소프트웨어 개발

**애자일 소프트웨어 개발**은 반복적이고 점진적인 접근 방식을 기반으로 하고 신속한 변화를 위한 옵션은 필요할 때 언제든지 변경 및 반복을 수행할 수 있는 자유롭고 유연한 접근 방식임.

단위 테스트로 프로그램이 정확하게 동작한다는 공식적인 증거가 될 수 있음. 

따라서 좋은 단위 테스트를 가질수록 애자일 소프트웨어 개발에 도움이 될 수 있음.

#### 1.3 단위 테스트와 소프트웨어 디자인

좋은 소프트웨어는 **테스트 가능한** 소프트웨어임. 

테스트의 용이성 ( 소프트웨어를 얼마나 쉽게 테스트 할 수 있는지 ) 은 클린 코드의 핵심 가치.

단위 테스트는 기본 코드를 보완하기 위한 것이 아니라 **실제 코드의 작성 방식에 직접적인 영향을 미치는 것**

단위 테스트의 여러 단계
 1.  특정 코드에 단위 테스트를 해야겠다고 발견하는 단계
 2.  더 나은 코드를 작성하는 단계
 3.  모든 코드가 테스트에 의해 작성되는 단계

```python
# 특정 작업에서 얻은 지표를 외부 시스템에 보내는 프로세스
# 테스트가 어떻게 코드의 개선으로 이어지는지 확인
class MetricsClient:
	"""타사 지표 전송 클라이언트"""

	def send(self, metric_name, metric_value):
		if not isinstance(metric_name, str): # metric_name이 str인지 확인
			raise TypeError("metric_name으로 문자열 타입을 사용해야 함")
		
		if not isinstance(metric_value, str): # metric_value이 str인지 확인
			raise TypeError("metric_value로 문자열 타입을 사용해야 함")

		logger.info("%s 전송 값 = %s", metric_name, metric_value)

class Process:

	def __init__(self):
		self.client = MetricsClient() # 타사 지표 전송 클라이언트

	def process_iterations(self, n_iterations):
		for i in range(n_iterations);
			result = self.run_process() # result가 문자열이 아니면 전송에 실패
			self.client.send("iteration.".format(i), result)
```
타사의 라이브러리는 직접 제어할 수 없으므로 실행 전에 정확한 타입을 제공해야 함.

단위 테스트를 통해 리팩토링을 여러 번 하더라도 이후에 재현되지 않는다는 것을 증명할 수 있음.

```python

class WrappedClient:
	def __init__(self):
		self.client = MetricsClient()
	
	def send(self, metric_name, metirc_value):
		return self.client.send(str(metric_name), str(metric_value))

class Process:
	def __init__(self):
		self.client = WrappedClient()
		
	def process_iterations(self, n_iterations):
		for i in range(n_iterations);
			result = self.run_process() # result가 문자열이 아니면 전송에 실패
			self.client.send("iteration".format(i), result)

... # 나머지 코드는 그대로 유지
```
타사 라이브러리를 사용하는 대신 자체적으로 만든 클래스를 지표 전송 `client`로 사용

```python
# 단위 테스트
import unittest
from unittest.mock import Mock

class TestWrappedClient(unittest.TestCase):
	def test_send_converts_types(self):
		wrapped_client =  WrappedClient()
		wrapped_client.client = Mock()
		wrapped_client.send("value", 1)
		wrapped_client.client.send.assert_called_with("value", "1")
```

#### 1.4 테스트의 경계 정하기

무엇을 테스트할지 주의하지 않으면 끝없이 테스트를 해야 하고 뚜렷한 결실도 없이 시간만 낭비하게 됨

따라서, 테스트의 범위는 **작성한 코드의 범위**로 한정해야 함.

외부 의존성에 대해서는 올바른 파라미터를 사용해 호출하면 정상적으로 실행된다는 것만 확인해도 충분함.


### 02.  테스트를 위한 프레임워크와 도구
---
#### 2.1 단위 테스트 프레임워크와 라이브러리
단위 테스트를 작성하고 실행하기 위한 두 가지 프레임워크
1. unittest
2. pytest 

테스트 시나리오를 다루는 것은 unittest 만으로도 충분. 그러나, 외부 시스템에 연결하는 등의 의존성이 많은 경우 pytest가 적합

```python 
# 머지 리퀘스트 ( Merge Request = MR )에 대해 코드 리뷰를 도와주는 간단한 버전 제어 도구

1. 한 명 이상의 사용자가 변경 내용에 동의하지 않은 경우 머지 리퀘스트가 거절된다.
2. 아무도 반대하지 않은 상태에서 두 명 이상의 개발자가 동의하면 해당 머지 리퀘스트는 승인된다.
3. 이외의 상태는 보류상태이다.

from enum import Enum

class MergeRequestStatus(Enum):
	APPROVED = "approved"
	REJECTED = "rejected"
	PENDING = "pending"

class MergeRequest:
	def __init__(self):
		self._context =
			"upvotes": set(),
			"downvotes": set(),

	@property
	def status(self):
		if self._context["downvotes"]:
		return MergeRequestStatus.REJECTED
		elif len(self.context["upvotes"]) >= 2:
		return MergeRequestStatus.APPROVED
		return MergeRequestStatus.PENDING

def upvote(self, by_user):
	self._context["downvotes"].discard(by_user)
	self._context["upvotes"].add(by_user)

def downvote(self, by_user):
	self._context["upvotes"].discard(by_user)
	self._context["downvotes"].add(by_user)
```
#### 2.1.1 unittest

`unittest`모듈은 모든 종류의 테스트를 작성할 수 있는 풍부한 API를 제공함. `Junit`을 기반으로 만들어져 객체 지향적임

단위 테스트를 만들려면 `unittest.TestCase`를 상속하여 테스트 클래스를 만들고 메서드에 테스트할 조건을 정의하면 됨. 

메서드는 `test_`로 시작해야하고 본문에서는 `unittest.TestCase`에서 상속받은 메서드를 사용하여 체크하려는 조건이 참인지 확인.

```python
class TestMergeRequestStatus(unittest.TestCase):

	def test_simple_rejected(self):
		merge_request = MergeRequest()
		merge_request.downvote("maintainer")
		self.assertEqual(merge_request.status, MergeRequestStatus.REJECTED)

	def test_just_created_is_pending(self):
		self.assertEqual(MergeRequest().status, MergeRequestStatus.PENDING)

	def test_pending_awaiting_review(self):
		merge_request = MergeRequest()
		merge_request.upvote("core-dev")
		self.assertEqual(merge_request.status, MergeRequestStatus.PENDING)

	def test_approved(self):
		merge_request = MergeRequest()
		merge_request.upvote("dev1")
		merge_request.upvote("dev2")
		self.assertEqual(merge_request.status, MergeRequestStatus.APPROVED)
```

`assertEquals( <actual>, <expected>[, message])` 는 실제 실행 값과 예상 값을 비교하는 단위 테스트 API 이고 에러가 발생한 경우를 대비해 메세지를 지정할 수 있음.

```python
# 머지 리퀘스트 추가 기능 구현
# 사용자가 머지 리퀘스트를 종료
# 병합이 종료되면 투표 불가능 ( 종료된 머지 리퀘스트에 투표 시도 시, 예외 발생 )
class MergeRequestStatus(Enum):
	APPROVED = "approved"
	REJECTED = "rejected"
	PENDING = "pending"

class MergeRequest:
	def __init__(self):
		self._context = {
			"upvotes": set(),
			"downvotes": set(),
			}
		self._status = MergeRequestStatus.OPEN
	
	def close(self):
		self._status = MergeRequestStatus.CLOSED

	'''
	def _cannot_vote_if_closed(self):
		if self._status == MergeRequestStatus.CLOSED:
			raise MergeRequestException("종료된 머지 리퀘스트에 투표할 수 없음")
			
	def upvote(self, by_user):
		self._cannot_vote_if_closed()
		
		self._context["downvotes"].discard(by_user)
		self._context["upvotes"].add(by_user)

	def downvote(self, by_user):
		self._cannot_vote_if_closed()
		
		self._context["upvotes"].discard(by_user)
		self._context["downvotes"].add(by_user)

```
유효성 검사가 실제로 작동하는지 확인. `assertRaises`와 `assertRaisesRegex` 메서드를 사용

```python
def test_cannot_upvote_on_closed_merge_request(self):
	self.merge_request.close()
	self.assertRaises(
		MergeRequestException, self.merge_request.upvote, "dev1"
	)
def test_cannot_downvote_on_closed_merge_request(self):
	self.merge_request.close()
	self.assertRaisesRegex(
		"종료된 머지 리퀘스트에 투표할 수 없음",
		self.merge_request.downvote,
		"dev1",
	)
```
`assertRaises` 는 제공한 **예외가 실제로 발생하는지** 확인. 두 번째 파라미터로 호출 가능한 객체를 전달, 나머지 파라미터에 호출에 필요한 파라미터 (*args, **kwargs)를 전달

`assertRaisesRegex`는 `assertRaises`와 동일한 방식으로 처리하고 **발생된 예외의 메세지가 제공된 정규식과 일치하는지** 확인. 

발생한 예외가 정확하게 원했던 예외인지 확인하기 위해서 오류 메세지도 확인

#### 2.1.1.1 테스트 파라미터화

데이터에 따라 머지 리퀘스트가 정상적으로 동작하는지 확인하기 위해 임계값을 변경하며 테스트

컴포넌트를 다른 클래스로 분리하고 컴포지션을 사용하여 다시 가져오는 것.

 그리고 분리된 클래스에 대해서는 자체 테스트 스위트를 가진 새로운 추상화 객체를 만들고 이것에 대해서 테스트 한다.

```python
class AcceptanceThreshold:
	def __init__(self, merge_request_context: dict) -> None:
		self._context = merge_request_context

	def status(self):
		if self._context["downvotes"]:
			return MergeRequestStatus.REJECTED
		elif len (self._context["upvotes"]) >= 2:
			return MergeRequestStatus.APPROVED
		return MergeRequestStatus.PENDING
class MergeRequest:
...
	@property
	def status(self):
		if self._status == MergeRequestStatus.CLOSED:
			return self._status

		return AcceptanceThreshold(self._context).status()
```

```python
class TestAcceptanceThreshold(unittest.TestCase):
	def setUp(self):
		self.fixture_data = (
			(
				{"downvotes": set(), "upvotes": set()},
				MergeRequestStatus.PENDING
			),
			(
				{"downvotes": set(), "upvotes": {"dev1"}},
				MergeRequestStatus.PENDING
			),
			(
				{"downvotes": {"dev1"}, "upvotes": set()},
				MergeRequestStatus.REJECTED
			),
			(
				{"downvotes": set(), "upvotes": {"dev1","dev2"}},
				MergeRequestStatus.APPROVED
			),
		)

	def test_status_resolution(self):
		for context, expected in self.fixture_data:
			with self.subTest(context=context):
				status = AcceptanceThreshold(context).status()
				self.assertEqual(status, expected)	
```

`subTest`는 헬퍼로써 호출되는 테스트 조건을 표시하는데 사용됨. 반복 중 하나가 실패하면 `unittest`는 `subTest`에 전달된 변수의 값을 보고함

#### 2.1.2 pytest

`pytest`는 `pip install pytest` 명령어를 통해 설치할 수 있음

`unittest`와 차이점은 

테스트 시나리오를 클래스로 만들고 객체 지향 모델을 생성하는 것이 가능하지만 필수는 아니며, 단순히 `assert` 구문을 사용해 조건을 검사하는 것이 가능함.

또한 `assert` 비교만으로 단위 테스트를 식별하고 결과를 보고하는 것이 가능함.

`pytest` 명령어를 통해 탐색 가능한 모든 테스트를 한번에 실행할 수 있음. 

`unittest`로 작성한 테스트도 실행할 수 있어서, `unittest`에서 `pytest`로 점진적으로 전환하기도 가능함.
#### 2.1.2.1 기초적인 pytest 사용 예
```python
# 이전 테스트를 pytest를 사용해 작성한 코드

def test_simple_rejected():
	merge_request = MergeRequest()
	merge_request.downvote("maintainer")
	assert merge_request.status == MergeRequestStatus.REJECTED

def test_just_created_is_pending():
	assert MergeRequest().status == MergeRequestStatus.PENDING

def test_pending_awaiting_review():
	merge_request = MergeRequest()
	merge_request.upvote("core-dev")
	assert merge_request.status == MergeRequestStatus.PENDING
```
결과가 참인지를 비교하는 것은 `assert`구문만 사용하면 되지만, 예외의 발생 유무 검사와 같은 것은 일부 함수를 사용해야 함.

```python
def test_invalid_types():
	merge_request = MergeRequest()
	pytest.raises(TypeError, merge_request.upvote, {"invalid-object"})

def test_cannot_vote_on_closed_merge_request():
	merge_request = MergeRequest()
	merge_request.close()
	pytest.raises(MergeRequestException, merge_request.upvote, "dev1")
	with pytest.raises(
		MergeRequestException,
		match = " 종료된 머지 리퀘스트에 투표할 수 없음",
	):
		merge_request.downvote("dev1")
```

`pytest.raises`는 `unittest.TestCase.assertRaises`와 동일하고 메서드 형태 또는 컨텍스트 관리자 형태로 호출될 수 있음.

예외의 메세지를 검사할 때 `match`파라미터에 확인하려는 표현식을 전달하면 됨.

#### 2.1.2.2 테스트 파라미터화

`pytest`로 **파라미터화 된 테스트**를 하는 것은 테스트 조합마다 새로운 테스트 케이스를 생성하기 때문에 더 좋음

그러기 위해서는 `pytest.mark.parametrize` 데코레이터를 사용해야 함

`pytest.mark.parametrize` 데코레이터의 파라미터
1. 테스트 함수에 전달할 파라미터의 이름을 나타내는 문자열
2. 해당 파라미터에 대한 각각의 값으로 반복이 가능해야 함.

```python
@pytest.mark.parametrize("context,expected_status", (
	( 
		{"downvotes": set(), "upvotes": set()},
		MergeRequestStatus.PENDING
	),
	(
		{"downvotes": set(), "upvotes": {"dev1"},
		MergeRequestStatus.PENDING,
	),
	( 
		{"downvotes": "dev1", "upvotes": set()},
		MergeRequestStatus.REJECTED
	),
	( 
		{"downvotes": set(), "upvotes": {"dev1", "dev2"},
		MergeRequestStatus.APPROVED
	),
)) 

def test_acceptance_threshold_status_resolution(context, expected_status):
	assert AcceptanceThreshold(context).status() == expected_status
```

#### 2.1.2.3 픽스처(Fixture)

픽스처로 생성한 데이터나 객체를 재 사용해 보다 효율적으로 테스트를 할 수 있음.

`fixture`는 테스트 사전/사후에 사용 가능한 리소스 또는 모듈을 뜻함.

픽스처를 정의하려면 
1. 함수를 만든다.
2. @pytest.fixture 데코레이터를 적용한다.

픽스처를 사용하기 원하는 테스트에는 파라미터로 픽스처의 이름을 전달하면 됨.

```python
@pytest.fixture
def rejected_mr():
	merge_request = MergeRequest()

	merge_request.downvote("dev1")
	merge_request.upvote("dev2")
	merge_request.upvote("dev3")
	merge_request.downvote("dev4")

	return merge_request

def test_simple_rejected(rejected_mr):
	assert rejected_mr.status == MergeRequestStatus.REJECTED

def test_rejected_with_approvals(rejected_mr):
	rejected_mr.upvote("dev2")
	rejected_mr.upvote("dev3")
	assert rejected_mr.status == MergeRequestStatus.REJECTED

def test_rejected_to_pending(rejected_mr):
	rejected_mr.upvote("dev1")
	assert rejected_mr.status == MergeRequestStatus.PENDING

def test_rejected_to_approved(rejected_mr):
	rejected_mr.upvote("dev1")
	rejected_mr.upvote("dev2")
	assert rejected_mr.status == MergeRequestStatus.APPROVED
	
```
테스트는 메인 코드에도 영향을 미치므로 클린 코드의 원칙이 테스트에도 적용되므로 **DRY 원칙** 을 적용할 수 있음.
