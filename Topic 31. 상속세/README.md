# Topic 31. 상속세

객체 지향 언어로 프로그래밍을 하는가? 상속을 사용하는가? 여러분에게 필요한 것은 상속이 아닐것이다.

## 코드를 공유하기 위해 상속을 쓸 때의 문제

```Ruby
class Vehicle
    def initialize
        @speed = 0
    end
    def stop
        @speed = 0
    end
    def move_at(speed)
        @speed = speed
    end
end

class Car < Vehicle
    def info
        "#{@speed}의 속도로 주행 중인 차입니다."
    end
end

# 최상위 코드
my_car = Car.new
my_car.move_at(30)
```
Vehicle클래스를 개발자가 메소드의 이름을 바꾸면 Car클래스를 사용하는 개발자가 당황해함. 작동이 안되서..  
결합이 너무 많다.  

## 타입을 정의하기 위해 상속을 쓸 떄의 문제
![KakaoTalk_Photo_2023-08-27-17-35-42](https://github.com/WBBookStudy/ProgramProgrammingProgrammer/assets/60125719/3307c2fe-8a12-474c-af5d-f2ffea2593cc)
상속을 사용하다 보면 이렇게 됨..  
그런데 더 나쁜것은 다중 상속 문제이다.

## 더 나은 대안
 1. 인터페이스와 프로토콜
 2. 위임
 3. 믹스인과 트레이트

### 인터페이스와 프로토콜
```Java
public class Car implements Drivable, Locatable {
    // Car 클래스의 코드. 이 코드는 Drivable과 Locatable이 
    // 요구하는 기능을 모두 구현해야 한다.
}
```
인터페이스 혹은 프로토콜 혹은 트레이트(뒤에서 나오는 트레이트와는 다른 개념)  
```Java
public interface Drivable {
    double getSpeed();
    void stop();
}
public interface Locatable {
    Coordinate getLocation();
    boolean locationIsValid();
}
```

이런것도 가능함
```Java
List<Locatable> items = new ArrayList<>();

items.add(new Car(...));
items.add(new Phone(...));
items.add(new Car(...));

void printLocation(Locatable item) {
    if (item.locationIsValid()) {
        print(item.getLocation().asString());
    }
}
```
> 인터페이스와 프로토콜은 상속 없이도 다형성을 가져다준다.

### 위임
```Ruby
class Account
    def initializa(. . .)
        @repo = Persister.for(self)
    end

    def save
        @repo.save()
    end
end
```
> Account 클래스는 클라이언트에게 프레임워크 API를 전혀 노출하지 않는다. 결합이 사라진 것  
  
서비스에 위임하라. Has-A가 Is-A보다 낫다.  

### 믹스인, 트레이트, 카테고리, 프로토콜 확장 등
클래스나 객체에 상속을 사용하지 않고 새로운 기능을 추가하여 확장하고 싶다.  
이 기능을 구현하는건 언어마다 제각각이다.  

```Java
mixin CommonFinders {
    def find(id) { ... }
    def findAll() { ... }
}

class AccountRecord extends BasicRecord with CommonFinders
class OrderRecord extends BasicRecord with CommonFinders
```










































