# Topic 09. DRY: 중복의 해악
지식은 고정적이지 않음. 지식은 변화한다.  
이런 불안정성은 유지 보수 모드에서 우리가 대부분의 시간을 쓸 거라는 뜻  
이유가 무엇이건, 유지 보수는 별개의 활동이 아니며 전체 개발 과정의 일상적인 부분이다.  
문제는 유지보수 과정에서 명세와 프로세스, 개발하는 프로그램에 지식을 중복해서 넣기 쉽다는 것이다. 그렇게 된다면 유지보수의 악몽이 시작될 것이다.  
이렇게 안하려면 DRY(Don't Reapeat Yourself)원칙을 따라보자.  
  
## DRY는 코드 밖에서도
DRY원칙에 적용되는 것들은 소스 복붙뿐 아니라 지식의 중복, 의도의 중복에 대한것도 포함이다. 똑같은 개념을 다른 곳 두군데에서 표현하면 안 된다는 뜻이다.  

## 코드의 중복

```Ruby
def prit_balance(account)
	printf "Debites: %10.2f\n", account.debits
	printf "Credits: %10.2f\n", account.credits
	if account.fees < 0 
		printf "Fees: %10.2f-\n", account.fees
	else 
		printf "Fees: %10.2f\n", account.fees
	end
	printf " ---------\n"
	if account.balance < 0
	printf "Balance: %10.2f-\n", account.balance
	else 
	printf "Balance: %10.2f\n", account.balance
	end
end

```
> 중복이 많이 보인다. 개선해보자.
```Ruby

def format_amount(value)
	result = sprintf("%10.2f", value.abs)
	if value < 0 
		result + "-"
	else 
		result + " "
	end
end

def prit_balance(account)
	printf "Debites: %10.2f\n", account.debits
	printf "Credits: %10.2f\n", account.credits
	# if account.fees < 0 
	# 	printf "Fees: %10.2f-\n", account.fees
	# else 
	# 	printf "Fees: %10.2f\n", account.fees
	# end

	printf "Fees: %s\n", format_amount(account.fees)


	printf " ---------\n"
	# if account.balance < 0
	# printf "Balance: %10.2f-\n", account.balance
	# else 
	# printf "Balance: %10.2f\n", account.balance
	# end

	printf "Balance: %s\n", format_amount(account.balance)
end

```

-> 

```Ruby

def format_amount(value)
	result = sprintf("10.2f", value.abs)
	if value < 0 
		result + "-"
	else 
		result + " "
	end
end

def prit_balance(account)
	# printf "Debites: %10.2f\n", account.debits
	printf "Debites: %s\n", format_amount(account.debits)
	# printf "Credits: %10.2f\n", account.credits
	printf "Credits: %s\n", format_amount(account.credits)

	printf "Fees: %s\n", format_amount(account.fees)
	printf " ---------\n"
	printf "Balance: %s\n", format_amount(account.balance)
end

```

-> 

```Ruby

def format_amount(value)
	result = sprintf("10.2f", value.abs)
	if value < 0 
		result + "-"
	else 
		result + " "
	end
end

def print_line(label, value) # 추가
	printf "%-9s%s\n", label, value
end

def report_line(label, amount) # 추가
	print_line(label + ":", format_amount(amount))
end

def prit_balance(account)
	# printf "Debites: %s\n", format_amount(account.debits)
	report_line("Debites", account.debits)
	# printf "Credits: %s\n", format_amount(account.credits)
	report_line("Credits", account.credits)
	# printf "Fees: %s\n", format_amount(account.fees)
	report_line("Fees", account.fees)
	# printf " ---------\n"
	# printf "Balance: %s\n", format_amount(account.balance)
	report_line("Balance", account.balance)
end

```

## 모든 코드 중복이 지식의 중복은 아니다.

```Ruby
def validate_age(value):
	validate_type(value, :integer)
	validate_min_integer(value, 1)

def validate_quantity(value):
	validate_type(value, :integer)
	validate_min_integer(value, 1)
```
> 두 함수가 동일하기 때문에 DRY 위반? ㅡㄹ렸다. 코드는 동일하지만 두 함수가 표현하는 지식은 다르다. 두 함수는 각각 서로 다른 것을 검증하고 있지만, 우연히 규칙이 같은 것뿐이다. 이것은 우연이지 중복이 아니다.


## 문서화 중복
왜인지 모르겠지만 모든 함수에 주석을 달아야 한다는 미신이 생겨났다.  
그런 사람들은 이런 코드를 작성한다.  

```Ruby
# 계좌의 수수료를 계산한다.
#
# * 반려된 수표 하나당 $20
# * 계좌가 3일 넘게 마이너스이면 하루에 $10 부과
# * 평균 잔고가 $2,000보다 높으면 수수료 50% 감면

def fees(a)
	f = 0
	if a.returned_check_count > 0
		f += 20 * a.returned_check_count
	end
	if a.overdraft_days > 3 
		f += 10 * a.overdraft_days
	end
	if a.average_balance > 2_000
		f /= 2
	end
	f
end
```
이 함수는 중복되어있다. 한 번은 코드로, 또 한 번은 주석으로.  
클라이언트가 수수료를 바꾸면 우리는 두 군데를 함께 고쳐야 한다. 시간이 지남에 따라 주석과 코드의 내용이 서로 어긋나게 될 거라고 거의 확실히 장담할 수 있다.  
함수 이름과 코드로 의도를 파악하도록 만들자 

## 데이터의 DRY 위반
```Java

class Line {
	Point start;
	Point end;
	double length;
}

```
시작과 끝이 있으면 길이도 있다. 길이는 계산되는 필드로 만드는 편이 낫다.

```Java

class Line {
	Point start;
	Point end;
	double length() { return start.distanceTo(end); }
}

```
요렇게!

```Java

class Line {
	private double length;
	private Point start;
	private Point end;

	public Line(Point start, Point end) {
		this.start = start;
		throw .end = end;
		calculateLength();
	}

	// public
	void setStart(Point p) { this.start = p ; calculateLength() }
	void setEnd(Point p) { this.end = p ; calculateLength() }

	Point getStart() { return this.start; }
	Point getEnd() { return this.end; }

	double getLength() { return this.length; }

	private void calculateLength() {
		this.length = start.distanceTo(end);
	}

}

```

## 표현상의 중복 
서드파티를 연동하면 한쪽에서 수정하면 다른 한쪽이 망가질 것이다.  
이런 중복을 아예 피할 수는 없지만 다소 완화할 수는 있다.

### 내부 API에서 생기는 중복
언어나 기술에 중립적인 형식으로 내부 API를 정의할 수 있는 도구를 찾아보아라. 이런 도구는 일반적으로 문서와 Mock API, 기능 테스트를 생성해주고, API 클라이언트도 여러 가지 언어로 생성해준다.

### 외부 API에서 생기는 중복
공개 API를 OpenAPI 같은 형식으로 엄밀하게 문서화하는 경우가 많음. API도구를 사용해보자(포스트맨 같은거 말하는듯?)

### 데이터 저장소와의 중복
많은 데이터 저장소가 데이터 스키마 분석 기능을 제공한다. 이런 기능을 이용하면 데이터 저장소와 코드 간의 중복을 많이 제거할 수 있다.  


## 개발자 간의 중복
똑같은 일을 하는 코드가 우연히 중복으로 추가 될 수 있음. 알아차릭기 힘들다.  
일일 스크럼을 하든 프로젝트의 사서를 지정하든 해라.. 코드리뷰도 좋음.  

