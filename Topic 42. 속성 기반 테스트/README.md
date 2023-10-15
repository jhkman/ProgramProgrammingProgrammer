# Topic 42. 속성 기반 테스트
믿으라, 하지만 확인하라.  
  
단위테스트를 작성할 때 테스트 대상에 대한 지식을 바탕으로 일반적으로 문제가 될 만한 상황들을 떠올린다.  
이 때 잘못된 가정이 들어갈 수 있지 않을까?(테스트 할 떄도)  
이 문제의 해결 방법은 서로 다른 사람 둘이 한명은 테스트코드를 작성하고 한명은 테스트를 하는것. 
우리의 대안은 컴퓨터에게 테스트를 맡기는 것.  

## 계약, 불변식, 속성
 - 계약: 선행 조건에 맞추어 입력을 넣으면 코드가 생산하는 출력이 주어진 후행 조건에 맞음을 보장해 준다.
 - 불변식: 함수 실행 전후로 계속 어떤 부분의 상태에 대하여 참이 되는 조건. ex) 리스트를 정렬했을 때의 결과는 정렬하기 전 리스트와 원소의 수가 같음. 그러면 리스트의 길이가 불변식임
 - 속성: 계약과 불변식을 뭉뚱그려서 속성이라고 함. 
  
코드에서 속성을 찾아내서 테스트 자동화에 사용할 수 있음 -> 이것을 속성 기반 테스트라고 한다.  

```Python
@given(some.lists(some.integers()))
def test_list_size_is_invariant_across_sorting(a_list):
  original_length = len(a_list)
  a_list.sort()
  assert len(a_list) == original_length

@given(some.lists(some.text()))
def test_sorted_result_is_ordered(a_list):
  a_list.sort()
  for i in range(len(a_list) - 1):
    assert a_list[i] <= a_list[i + 1]
```
-> 실행결과
```
...
plugins: hypothesis-4.14.0
sort.py ..   [100%]
```
보이지 않는 곳에서 hypothesis는 우리 테스트를 백 번씩 돌렸다. 리스트의 길이도 바꾸고 내용도 바꿔가면서 매번 다른 리스트를 입력으로 넘겼다.  
무작위로 만든 200개의 리스트로 200개의 테스트를 만들어 수행한 셈


## 잘못된 가정 찾기
우리는 간단한 주문 처리 및 재고 관리 시스템을 만들고 있다. 창고 객체의 재고 정보를 모델링 하려 한다.  
특정 물품의 재고가 있는지 확인하거나 물품을 재고에서 꺼낼수 있고, 아니면 현재 남아있는 재고량을 조회할 수도 있다.

```Python
# 코드
class Warehouse:
  def __init__(self, stock):
    self.stock = stock

  def in_stock(self, item_name):
    return (item_name in self.stock) and (self.stock[item_name] > 0)

  def take_from_stock(self, item_name, quantity):
    if quantity <= self.stock[item_name]:
      self.stock[item_name] -= quantity
    else:
      raise Exception("{}를 재고보다 많이 팔았음".format(item_name))

  def stock_count(self, item_name):
    return self.stock[item_name]

```
```Python
# 테스트

def test_warehouse():
  wh = Warehouse({"신발": 10, "모자": 2, "우산": 0})
  assert wh.in_stock("신발")
  assert wh.in_stock("모자")
  assert not wh.in_stock("우산")

  wh.take_from_stock("신발", 2)
  assert wh.in_stock("신발")

  wh.take_from_stock("모자", 2)
  assert not wh.in_stock("모자")

```
일단은 통과함...
```Python
# stock.py

def order(Warehouse, item, quantity):
  if warehouse.in_stock(item):
    warehouse.take_from_stock(item, quantity)
    return ( "완료", item, quantity )
  else:
    return ( "재고 없음", item, quantity )

```

```Python

def test_order_in_stock():
  wh = Warehouse({"신발": 10, "모자": 2, "우산": 0 })
  status, item, quantity = order(wh, "모자", 1)
  assert status == "완료"
  assert item == "모자"
  assert quantity == 1
  assert wh.stock_count("모자") == 1

def test_order_not_in_stock():
  wh = Warehouse({"신발": 10, "모자": 2, "우산": 0 })
  status, item, quantity = order(wh, "우산", 1)
  assert status == "재고 없음"
  assert item == "우산"
  assert quantity == 1
  assert wh.stock_count("우산") == 0

def test_order_unknown_stock():
  wh = Warehouse({"신발": 10, "모자": 2, "우산": 0 })
  status, item, quantity = order(wh, "베이글", 1)
  assert status == "재고 없음"
  assert item == "베이글"
  assert quantity == 1

```
여기까지도 일단은 모두 괜찮아보인다.  
속성기반 테스트를 더 추가해보자.  
거래가 이루어지는 동안 상품이 생겨나거나 사라질 수 없다. 즉, 창고에서 어떤 상품을 꺼내 오면 꺼낸 상품 수와 남은 재고 숫자를 더 했을 때 원래 창고에 있던 재고 숫자가 되어야한다.  
```Python

@given(item = some.sampled_from(["신발", "모자"]),
       quantity = some.integers(min_value=1, max_value=4))
def test_stock_level_plus_quantity_equals_original_stock_level(item, quantity):
  wh = Warehouse({"신발": 10, "모자": 2, "우산": 0})
  initial_stock_level = wh.stock_count(item)
  (status, item, quantity) = order(wh, item, quantity)
  if status == "완료"
    assert wh.stock_count(item) + quantity == initial_stock_level

```
-> 실행 결과
```
rais Exception("제고보다 많이 팔았음")
````
터짐  
속성 기반 테스트가 잘못된 가정을 찾아냈다.
-> 

```Python
class Warehouse:
  def __init__(self, stock):
    self.stock = stock

  def in_stock(self, item_name):
    # return (item_name in self.stock) and (self.stock[item_name] > 0)
    return (item_name in self.stock) and (self.stock[item_name] >= quantity)

  def take_from_stock(self, item_name, quantity):
    if quantity <= self.stock[item_name]:
      self.stock[item_name] -= quantity
    else:
      raise Exception("{}를 재고보다 많이 팔았음".format(item_name))

  def stock_count(self, item_name):
    return self.stock[item_name]

  def order(Warehouse, item, quantity):
    # if warehouse.in_stock(item):
    if warehouse.in_stock(item, quantity):
      warehouse.take_from_stock(item, quantity)
    return ( "완료", item, quantity )
  else:
    return ( "재고 없음", item, quantity )
```
> 이렇게 수정함

## 속성 기반 테스트는 우리를 자주 놀래킨다.
속성 기반 테스트가 강력한 까닭은 그저 입력을 생성하는 규칙과 출력을 검증하는 단정문만 설정한 채 제멋대로 작동하도록 놔두기 때문이다. 정확히 어떤 일이 일어날지 절대 알 수 없다.  
속성 기반 테스트가 실패했다면 테스트 함수가 어떤 매개 변수를 사용했는지 알아낸 다음 그 값을 이용하여 별도의 단위 테스트를 정식으로 추가하는 것이 좋다.  
  
이 단위 테스트의 하는 일은 두가지임
1. 속성 기반 테스트의 여러 가지 다른 수행 결과와 상관 없이 문제가 발생하는 상황에 집중할 수 있게 해줌
2. 단위 테스트가 회귀테스트 열할을 함. 임의의 값을 생성하여 사용하기 떄문에 다음번에 실행헀을 때 똑같은 값을 테스트 함수에 넘긴다는 보장이 없다.

## 속성 기반 테스트는 설계에도 도움을 준다.
속성 테스트는 코드를 불변식과 계약이라는 관점으로 바라보게 한다.  
무엇이 변하지 않아야 하고, 어떤 조건을 만족해야 하는지 생각하게 된다.  



















