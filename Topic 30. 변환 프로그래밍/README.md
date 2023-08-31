# Topic 30. 변환 프로그래밍

프로그램이란 입력을 출력으로 바꾸는 것.  
1970년대에는..  
디렉토리 안에 있는 파일중 가장 긴 파일 다섯개를 찾는 프로그램을 작성

```
$ find . -type f | xargs wc -l | sort -n | tail -5
```
->
```
find ./ -type f
```
> 현재 티렉토리(.)나 그 하위 디렉토리에 있는 모든 일반 파일(-type f)의 목록을 표준출력에 써라
```
xargs wc -l
```
> 표준 입력에서 여러 줄을 읽은 다음 그 내용을 모아서 wc -l 명령의 인자로 넘겨라. wc프로그램은 -l 옵션을 주면 인자로 받은 파일들의 줄수를 각각 센 후, 그 결과를 "(줄수) (파일명)"형태로 표준 출력에 쓴다.
```
sort -n
```
> 표준 입력을 각 줄이 숫자로 시작한다고 가정(-n)하고 정렬한 후, 결과를 표준 출력에 써라
```
tail -t
```
> 표준 입력을 읽어서 맨 마지막 다섯 줄(-5)만 표준 출력에 써라.

![KakaoTalk_Photo_2023-08-27-16-30-26](https://github.com/WBBookStudy/ProgramProgrammingProgrammer/assets/60125719/29dd6a07-04bc-4d78-970e-e8f0b4ef7df2)
> 도식화 하면 위의 사진과 같음
## 변환 찾기 
때에 따라선 요구 사항에서 시작하는 게 변환을 찾는 가장 쉬운 방법이다.  
요구 사항에서 입력과 출력이 무엇인지 찾으면 전체 프로그램을 나타내는 함수가 정해진다. 이제 입력을 출력으로 바꿔 가는 단계들을 찾으면 된다.  
### ex) 입력: 몇 개의 알파벳. 출력: 세 글자 단어들, 네글자 단어들 등의 목록
```
3 => lvy, lin, nil, yin
4 => inly, liny, viny
5 => vinyl
```
> "lvyin"를 반환하면
## 어떻게 찾았을까. 애너그램 탐색기
| 단계 | 변환 | 예시 데이터 |  
|:---:|:---:|:---:|
| 단계 0: | 초기 입력 | "lvyin" |
| 단계 1: | 세 글자 이상인 모든 조합(combination) | vin, viy, vil, vny, vnl, vyl, iny, inl, iyl, nyl, viny, vinl, viyl, vnyl, inyl, vinyl |
| 단계 2: | 각 조합의 특성값 | inv, ivy, nvy, lnv, lvy, iny, iln, ily, lny, invy, ilnv, ilvy, lnvy, ilny, ilnvy |
| 단계 3: | 주어진 특성값 중 하나와 특성값이 일치하는 사전의 단어 목록 | ivy, yin, nil, lin, viny, liny, inly, vinyl |
| 단계 4: | 길이에 따라 분류한 단어들 | 3 => lvy, lin, nil, yin, 4 => inly, liny, viny, 5 => vinyl |

### 1단계. 더 작은 변환들로 나누기 
1단계를 살펴보자. 단어를 받아서 세 글자 이상인 모든 조합을 만든다. 이 변환 자체도 여러 개의 연결된 변환으로 더 나눌 수 있다. 
| 단계 | 변환 | 예시 데이터 |  
|:---:|:---:|:---:|
| 단계 1.0: | 초기 입력 | "lvyin" |
| 단계 1.1: | 글자들로 변환 | v,i,n,y,l |
| 단계 1.2: | 모든 부분집합을 구함 | (), (v) .... (v,i,n,y,l) |
| 단계 1.3: | 글자가 셋 이상인 것만 | (v,i,n) .... (v,i,n,y,l) |
| 단계 1.4: | 문자열로 다시 변환 | (vin, viy ... (vinyl)) |
-> 코딩으로 해보면
```Elixir
defp all_subsets_longer_than_three_charactors(word) do 
    word
    |> String.codepoints()
    |> Comb.subsets()
    |> Stream.filter(fn subset -> length(subset) >= 3 end)
    |> Stream.map(&List.to_string(&1))
end
```
##### 연산자
|> 연산자는 뭘까?  
-> 그냥 왼쪽에 있는 값을 오른쪽 함수의 첫 번째 인자로 넣는다는 뜻이라고함
```
"vinyl" |> String.codepoints() |> Comb.subsets()
```
==
```
Comb.subsets(String.codepoints("vinyl"))
```

### 2단계
2단계를 보자 
| 단계 | 변환 | 예시 데이터 |  
|:---:|:---:|:---:|
| 단계 2.0: | 초기 입력 | vin, viy, ... inyl, vinyl |
| 단계 2.1: | 특성값으로 바꿈 | inv, ivy, ... inlvy |
코드로 작성해보면
```Elixir
defp as_unique_signatures(subsets) do 
    subsets
    |> Stream.map(&Dictionary.signature_of/1)
end
```

### 3단계
특성값 리스트가 입력이다. 단어가 없으면 nil이 나온다. 그다음 결과에서 nil을 제거한 후, 특성값별 단어 리스트를 이어 붙혀서 하나의 리스트로 만든다.
```Elixir
defp find_in_dictionary(signatures) do 
    signatures
    |> Stream.map(&Dictionay.lookup_by_signature/1)
    |> Stream.reject(&is_nil/1)
    |> Stream.concat()
end
```

### 4단계
단어를 길이별로 분류한다.
```Elixir
defp group_by_length(words) do 
    words
    |> Enum.sort()
    |> Enum.group_by(&String.length/1)
end
```

### 모두 연결하기
```Elixir
defp anagrams_in(words) do 
    words
    |> all_subsets_longer_than_three_charactors()
    |> as_unique_signatures()
    |> find_in_dictionary()
    |> group_by_length()
end
```
> 작동 잘 함 ㅎ;

### 이것이 왜 그리 대단한가?
```
words
    |> all_subsets_longer_than_three_charactors()
    |> as_unique_signatures()
    |> find_in_dictionary()
    |> group_by_length()
```
1. 요구 사항을 달성하기 위해 필요한 것은 하나로 연결된 변환들뿐이다. 각각은 앞의 변환에서 입력을 받아 처리한 결과를 다음 변환으로 넘겨준다.  
이보다 글처럼 읽기 쉬운 코드를 만들긴 어려움  
2. 데이터를 숨기고, 객체 안에 캡슐화해야함. 변환 모델에서는 데이터를 전체 시스템 여기저기 작은 웅덩이에 넣는 대신, 하나의 흐름으로 생각한다.  
이렇게 되면 데이터는 더 이상 클래스를 정의할 때처럼 특정한 함수들과 묶이지 않는다. 결합도가 낮아짐

### 오류 처리는 어떻게?
변환 사이에 값을 절대 날것으로 넘기지 않고 wrapper역할을 하는 자료 구조나 타입으로 값을 싸서 넘긴다. ((Result)[https://velog.io/@un1945/Swift-Result-Type]타입 같은거?)  

#### 변환은 프로그래밍을 변환한다.
코드를 일련의 (중첩된) 변환으로 생각하는 접근 방식은 프로그래밍을 해방시킨다. 코드가 더 명확해지고 함수는 짧아지면 설계는 단순해진다.




















