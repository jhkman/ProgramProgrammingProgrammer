# Topic 26. 리소스 사용의 균형
자신이 시작한 것은 자신이 끝내라.  
리소스를 할당하는 함수나 객체가 리소스를 해제하는 책임 역시 져야 한다는 뜻이다.  
```Ruby

def read_costomer
    @customer_file = File.open(@name + ".rec", "r+")
    @balance = BigDecimal(@customer_file.gets)
end

def write_costomer
    @customer.file.rewind
    @customer.file.puts @balance.to_s
    @customer.file.close
end

def update_customer(transaction_amount)
    read_costomer
    @balance += transaction_amount
    write_costomer
end

```
-> 명세가 바뀌었다. 잔액은 새로운 값이 음수가 아닌 경우에만 갱신되어야 한다.
```Ruby
def read_costomer
    @customer_file = File.open(@name + ".rec", "r+")
    @balance = BigDecimal(@customer_file.gets)
end

def write_costomer
    @customer.file.rewind
    @customer.file.puts @balance.to_s
    @customer.file.close
end

def update_customer(transaction_amount)
    read_costomer
    @balance += transaction_amount
    # write_costomer
    if (new_balance >= 0.00)
        @balance = new_balance
        write_costomer
    end
end
```
> 이렇게 수정하고 배포했음. 곧 파일이 너무 많이 얼려있다는 오류와 함께 프로그램이 죽었다.. 알고보니 wirte_customer가 몇몇 상황에서 호출되지 않았고, 그런 경우 파일이 닫히지 않았다.
```Ruby
def update_customer(transaction_amount)
    read_costomer
    @balance += transaction_amount

    if (new_balance >= 0.00)
        @balance = new_balance
        write_costomer
    else 
        @customer_file.close # 나쁜 방법!
    end
end
```
> 이렇게 수정하면 될까? 잔액에 상관없이 파일이 닫히긴 함. 하지만 이제 공유되는 변수 customer_file로 세 개의 루틴이 결합되었고 파일이 열렸느지 닫혔는지를 추적하기가 점점 복잡해진다.
```Ruby
# def read_costomer
#     @customer_file = File.open(@name + ".rec", "r+")
#     @balance = BigDecimal(@customer_file.gets)
# end
def read_costomer(file)
    @balance = BigDecimal(file.gets)
end

# def write_costomer
#     @customer.file.rewind
#     @customer.file.puts @balance.to_s
#     @customer.file.close
# end
def write_costomer(file)
    file.rewind
    file.puts @balance.to_s
end

# def update_customer(transaction_amount)
#     read_costomer
#     @balance += transaction_amount
#     # write_costomer
#     if (new_balance >= 0.00)
#         @balance = new_balance
#         write_costomer
#     end
# end
def update_customer(transaction_amount)
    file = File.open(@name + ".rec", "r+")
    read_costomer(file)
    @balance += transaction_amount
    write_customer(file)
    file.close
end
```
> 자신이 시작한 것은 자신이 끝내라. 원칙을 적용해서 수정. 이제 해당 파일에 대한 모든 책임은 update_customer 루틴에 있다.  
> 이 루틴은 파일을 열고 루틴을 끝내기 전에 닫는다.  
> 지역적으로 행동하라.

## 중첩 할당

리로스 할당의 기본 패턴을 확장해서 한 번에 여러 리소스를 사용하는 루틴에 적용 가능
 - 리소소를 할당한 순서의 역순으로 해제하라. 이렇게 해야 한 리소스가 다른 리소스를 참조하는 경우에도 참조를 망가트리지 않는다.
 - 코드의 여러 곳에서 동일한 구성의 리소스들을 할당하는 경우에는 언제나 같은 순서로 할당해야 교착 가능성을 줄일 수 있다.
  
## 객체와 예외
리소스를 클래스 안에서 캡슐화하자. 그면 생성자 소멸자에서 할당 해제가 보장됨.

## 균형 잡기와 예외
try catch 를 사용한다면 finally에서 리소스를 할당 해제 하자.  
### 나쁜 예외 처리 방식
```Ruby
begin
    thing = allocate_resource()
    process(thing)
finally
    deallocate(thing)
end
```
> 리소스 할당이 실패해서 예외가 발생하면 finally절이 실행되면서 할당된 적이 없는 thing을 할당해제 하려한다.
  ->
```Ruby
thing = allocate_resource()
begin
    process(thing)
finally
    deallocate(thing)
end
```

## 리소스 사용의 균형을 잡을 수 없는 경우
한 루틴에서 메모리의 일정 영역을 할당한 다음 어떤 더 큰 구조에 그것을 연결한 후, 한동안 그대로 쓰는 경우  
->  메모리 할당에 대한 의미론적 불변식을 정해놓자. 한 군데 모은 자료 구조 안의 자료를 누가 책임지는지 정해놓아야 한다.  
최상위 구조의 메모리 할당을 해제할 경우
 - 최상위 구조가 자기 안에 들어있는 하우 구조들을 해제할 책임을 진다.
 - 최상위 구조가 드냥 할당 해제된다. -> 댕글포인터 들 많이 생겨서 메모리 릭 남
 - 최상위 구조가 하나라도 하위 구조를 가지고 있으면 자신의 할당 해제를 거부한다.
-> 상황에 따라 선택하자.

## 균형을 점검하기
실용주의 프로그래머는 자신을 포함해서 아무도 믿지 않는다.  
정말로 리소스가 적절하게 해제되었는지 실제로 점검하는 코드를 작성하자.  
리소스의 종류별로 래퍼를 만들고 그 래퍼들이 모든 할당과 해제 기록을 보관하는 방법도 있음  
혹은 메모리 릭 체크 도구를 이용하자.
