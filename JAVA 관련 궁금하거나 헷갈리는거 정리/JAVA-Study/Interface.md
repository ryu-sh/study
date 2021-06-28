# 인터페이스

## 인터페이스 기본 메소드 (default method), 자바 8
- 인터페이스는 기능에 대한 선언만 가능하기 때문에, 실제 코드를 구현한 로직은 포함될 수 없다.
- 하지만 자바8에서 이러한 룰을 깨트리는 기능이 나오게 되었는데, 그것이 default method이다.
- 메소드 선언시에 default를 명시하게 되면 인터페이스 내부에서도 코드가 포함된 메소드를 선언할 수 있다.
- 접근제어자에서 사용하는 default와 같은 키워드이지만, 접근제어자는 아무것도 명시하지 않은 접근제어자를 default라 하며 인터페이스의 default method는 'default'라는 키워드를 명시해야 한다.
- default라는 키워드를 메소드에 명시하게 되면 인터페이스 내부라도 코드를 작성 할 수 있다.

```java
interface MyInterface { 
  default void printHello() { System.out.println("Hello World!"); } 
}
```

### 왜 사용할까?
```
...(중략)... 바로 "하위 호환성"때문이다. 예를 들어 설명하자면, 여러분들이 만약 오픈 소스코드를 만들었다고 가정하자. 
그 오픈소스가 엄청 유명해져서 전 세계 사람들이 다 사용하고 있는데, 인터페이스에 새로운 메소드를 만들어야 하는 상황이 발생했다. 
자칫 잘못하면 내가 만든 오픈소스를 사용한 사람들은 전부 오류가 발생하고 수정을 해야 하는 일이 발생할 수도 있다. 
이럴 때 사용하는 것이 바로 default 메소드다. (자바의 신 2권)
```

기존에 존재하던 인터페이스를 이용하여서 구현된 클래스를 만들고 사용하고 있는데,

인터페이스를 보완하는 과정에서 추가적으로 구현해야 할, 혹은 필수적으로 존재해야 할 메소드가 있다면,

이미 이 인터페이스를 구현한 클래스와의 호환성이 떨어지게 된다. 이러한 경우 default 메소드를 추가하게 된다면

하위 호환성은 유지되고 인터페이스의 보완을 진행 할 수 있다.

### 정리
-interface에서도 메소드 구현이 가능하다.
-참조 변수로 함수를 호출할 수 있다.
-implements한 클래스에서 재정의가 가능하다.

## 인터페이스의 static 메소드, 자바 8
- 인스턴스 생성과 상관없이 인터페이스 타입으로 호출하는 메소드이다.
- static 예약어를 사용하고, 접근제어자는 항상 public이며 생략 할 수 있다.
- static method는 일반적으로 우리가 정의하는 메소드와는 다르다.
- body가 있어야 한다.
- implements 한 곳에서 override가 불가능하다.
- static 메소드를 사용하는 데 주의해야할 점은 기존 클래스의 static 메소드처럼 class이름.메소드로 호출하는게 아니라 interface이름.메소드로 호출해야 한다.

```java
public interface ICalculator { 
  static void print(int value) { System.out.println(value); } 
}
```

```java
public class CalcTest { 
  public static void main(String[] args) { 
    ICalculator cal = new Calculator(); // cal.print(100); error 
    
    ICalculator.print(100); // interface의 static 메소드는 반드시 interface명.메소드 형식으로 호출 
  } 
}
```

## 인터페이스의 private 메소드, 자바 9
- java8 에서는 default method와 static method가 추가 되었고, java9 에서는 private method와 private static method가 추가 되었다.
- java8의 default method와 static method는 여전히 불편하게 만든다. 단지 특정 기능을 처리하는 내부 method일 뿐인데도, 외부에 공개되는public method로 만들어야하기 때문이다.
interface를 구현하는 다른 interface 혹은 class가 해당 method에 엑세스 하거나 상속할 수 있는 것을 원하지 않아도, 그렇게 될 수 있는 것이다.
- java9 에서는 위와 같은 사항으로 인해 private method와 private static method라는 새로운 기능을 제공해준다. → 코드의 중복을 피하고 interface에 대한 캡슐화를 유지 할 수 있게 되었다.

```java
public interface Car { 
  void carMethod(); 
  default void defaultCarMethod() { 
    System.out.println("Default Car Method"); 
    privateCarMethod(); 
    privateStaticCarMethod(); } 
    
  private void privateCarMethod() { 
    System.out.println("private car method"); } 
    
  private static void privateStaticCarMethod() { 
    System.out.println("private static car method"); } 
}
```

```java
public class DefaultCar implements Car{ 
  @Override 
  public void carMethod() { 
  System.out.println("car method by DefaultCar"); } 
}
```

```java
public class Main { 
  public static void main(String[] args) { 
    DefaultCar car = new DefaultCar(); 
    car.carMethod(); 
    car.defaultCarMethod(); } 
}
```

```
car method by DefaultCar 
Default Car Method 
private car method 
private static car method 

Process finished with exit code 0
```
