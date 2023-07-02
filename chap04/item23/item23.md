## **태그달린 클래스**

태그달린 클래스 라는 건 무엇일까? 책에 나오는 예시는 다음과 같다.

```java
// Tagged class - vastly inferior to a class hierarchy!
class Figure {

	enum Shape { RECTANGLE, CIRCLE };

	// Tag field - the shape of this figure
	final Shape shape;

	// These fields are used only if shape is RECTANGLE
	double length;
	double width;
	// This field is used only if shape is CIRCLE
	double radius;

	// Constructor for circle
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// Constructor for rectangle
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch(shape) {
			case RECTANGLE:
			return length * width;

			case CIRCLE:
			return Math.PI * (radius * radius);

			default:
			throw new AssertionError(shape);
		}
	}
}
```

## **태그달린 클래스의 단점**

1. enum타입 선언, 태그 필드, 스위치 문 등 쓸데없는 코드가 많아진다.
2. 여러 구현이 한 클래스에 몰려 있으니 가독성이 나쁘고 메모리를 많이 사용한다.
3. 필드들을 final로 사용하려면 쓰이지 않는 필드까지 생성자에서 초기화 해야한다.
4. 또 다른 의미를 추가하려면 코드 전체를 수정해야 한다. (모든 switch 분기에 하나를 추가해야 한다)
5. 인스턴스의 타입만으로는 현재 어떤 의미를 가지는지 알 길이 없다.

## **태그달린 클래스의 대안**

자바에는 클래스 계층구조를 활용하는 서브타이핑을 지원한다. 계층 클래스의 루트root가 되어줄 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들은 루트 클래스의 추상 메서드로 선언한다.위 코드에서 area()가 이런 경우에 해당한다.

Figure 클래스에는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 그 결과로 루트 클래스에는 추상 메서드인 area만 남게 된다.

다음으로 루트 클래스를 확장하면 되는데, 원 클래스와 사각형 클래스를 만들고, 각자에 의미에 맞는 데이터 필드를 넣자. 그 후 추상메서드 area()를 알맞게 구현하면 된다.

```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;

	Circle(double radius) { this.radius = radius; }

	@Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
	final double width;

	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override double area() { return length * width; }
}
```

이런 구조를 활용하면, 태그달린 클래스의 단점을 모두 해결할 수 있다.

## **번외 ) 상태 패턴State Pattern**

상태패턴과 비슷하다고 느꼈는데 글에서 의미하는 태그는 특정 상태를 int나 enum으로 표현하고, 해당 값에 따라 달리 표현하는 방식을 의미하는 방식이라 상태패턴과는 차이점이 있다.

상태 패턴에 대해 알아보고, 본문 내용을 상태 패턴으로 구현해보자.

<img src = "/Users/jeonghwan/Desktop/Study/Effective Java/my study/item23/state pattern.png">

상태패턴은 **각 상태별로 분기가 나뉘는 행위를 인터페이스화**하고, 각 상태는 구현체로써 표현하는 디자인 패턴이다.

상태를 가진 객체는, 그저 상태 구현체의 동작을 호출하기만 하면 된다. 위 Figure를 상태패턴을 사용하도록 리팩토링 해보자.

```java
// Shape마다 분기가 나뉘는 area 행위를 인터페이스화
interface Shape {

  double area();
}

// 직사각형 상태
class Rectangle implements Shape {

  private final double length;
  private final double width;

  public Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }

  @Override public double area() { return length * width;}
}

// 원형 상태
class Circle implements Shape {

  private final double radius;

  public Circle(double radius) {
    this.radius = radius;
  }

  @Override public double area() { return Math.PI * (radius * radius);  }
}

public class Figure {

  final Shape shape;

  // Constructor for circle
  Figure(double radius) {
    this.shape = new Circle(radius);
  }

  // Constructor for rectangle
  Figure(double length, double width) {
    this.shape = new Rectangle(length, width);
  }

  double area() {
    return shape.area();
  }
}
```

저자가 활용한 상속과 비교한다면, 상태패턴을 이용해 구현한 Figure 객체는 상태를 변경해(setState) 인스턴스의 행위를 변화시킬 수 있다는 장점이 있을 것이다.