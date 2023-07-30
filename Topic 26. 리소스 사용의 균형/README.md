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



























