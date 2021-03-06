# Enum

## 열거형이란? 
 - 컴퓨터 프로그래밍에서 열거형(enumerated type, enumeration), 이넘(enum)은  요소, 멤버라 불리는 명명된 값의 집합을 이루는 자료형 
 - 열거자 이름들은 일반적으로 해당 언어의 상수 역할을 하는 식별자

## 장점
- 문자열과 비교해, IDE의 적극적인 지원을 받을 수 있다.
    - 자동완성, 오타검증, 텍스트 리팩토링 등등 
- 허용 가능한 값들을 제한할 수 있다.
- 리팩토링시 변경 범위가 최소화 된다.
   - 내용 추가가 필요하더라도, Enum 코드외에 수정할 필요가 없다.
- 확실한 부분과 불확실한 부분을 분리할 수 있습니다.
   - A값과 B값이 실제로 동일한 것인지, 전혀 다른 의미인지
   - 코드를 사용하기 위해 추가로 필요한 메서드들은 무엇인지
   - 변경되면 어디까지 변경해야 하는 것인지
- 문맥(Context)을 담을 수 있습니다.
  - A라는 상황에서 "a"와 B라는 상황에서 "a"는 똑같은 문자열 "a" 지만 전혀 다른 의미입니다.
  - 문자열은 이를 표현할 수 없지만, Enum은 이를 표현할 수 있습니다.
  - 이로인해 실행되는 코드를 이해하기 위해 추가로 무언가를 찾아보는 행위를 최소화 할 수 있습니다.

## 정의하는 방법
- 아래와 같이 '{}' 안에 상수의 이름을 나열하기만 하면 된다.
```
enum 열거형 이름 { 상수명1, 상수명2, ... 상수명n }
```
- 열거형에 정의된 상수를 사용하는 방법은 '열거형이름.상수명' 이다. 마치 클래스의 static변수를 참조하는것과 같다.
- 열거형 상수는 필드를 가질 수 있다. 필드를 추가하는 방법은 추가하고 싶은 필드를 정의하고 생성자의 인자로 추가한 뒤 각각의 상수에 값을 입력하면 된다.
```java
enum Direction {
    EAST(1), SOUTH(5), WEST(-1), NORTH(10);

    pricvate int value;
    Direction (int value) {
        this.value = value; 
    }

    public int getValue() { return value; }
}
```
- 열거형의 생성자는 접근제어자가 묵시적으로 private이기 때문에 외부에서는 열거형의 생성자를 호출할 수 없다.

## 비교
 - 열거형 상수간의 비교에는 '=='을 사용할 수 있다.
 -  '<', '>'와 같은 비교연산자는 사용할 수 없고 compareTo()는 사용가능
```java
Direction dir;

if (dir == Direction.WEST) {
	...
} else if (dir > Direction.WEST) {
	// 에러. 열거형 상수에 비교연산자는 사용 불가능
} else if (dir.compareTo(Direction.WEST) > 0) {
	// compareTo()는 가능
}
```
 - switch문의 조건식에도 열거형을 사용할 수 있다.
 - switch 문에 열거형을 사용할 때 주의할 점은 case문에 열거형 타입의 이름은 적지 않고 상수의 이름만 적어야 한다는 제약이 있다. ('case Direction.EAST' 가 아닌 'case EAST' 만 적어야 한다.)

## 내부 구현 모습

```
enum Direction { EAST, SOUTH, WEST, NORTH }
```
- 위의 열거형 상수 하나하나(EAST, SOUTH, WEST, NORTH)는 Direction의 객체이다.
- 클래스로 구현한다면 아래와 같은 모습이 된다.
```java
public class Direction {
    static final Direction EAST = new Direction("EAST");
    static final Direction SOUTH = new Direction("SOUTH");
    static final Direction WEST = new Direction("WEST");
    static final Direction NORTH = new Direction("NORTH");

    private String name;

    private Direction(String name) {
        this.name = name;
    }
}
```
- Direction 클래스의 static 상수 EAST, SOUTH, WEST, NORTH의 값은 객체의 주소고, 이 값은 바뀌지 않는 정적(static) 값이므로 '=='으로 비교가 가능한 것

## 메소드
- values() : 열거형의 모든 상수를 배열에 담아 반환한다.
```
  Direction[] arr = Direction.values(); 
```
- valueOf(String name) : 열거형 상수의 이름으로 문자열 상수에 대한 참조를 얻을 수 있게 해준다.
```
Direction d = Direction.valueOf("WEST");
Direction.WEST == Direction.valueOf("WEST"); // true 반환
```
- String name() : 열거형 선언에 선언된대로 정확하게 열거형 상수의 이름을 문자열로 반환한다. 
- int ordinal() : 열거형 상수가 정의된 순서를 반환한다.(0부터 시작)
