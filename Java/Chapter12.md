> 자바의 정석(남궁성 저) 학습내용 정리

## 1. Generics
- Generics는 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입 체크를 해주는 기능이다.
- 즉 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법을 의미하며 다룰 객체의 타입을 미리 명시해줌으로써 타입 안정성을 높이고 번거로운 형 변환을 줄여준다.
<br>

### 1.1  Generic 클래스의 선언
- 클래스를 작성할 때, 타입 대신 T와 같은 타입 변수를 사용한다.
- 참조변수, 생성자 T 대신 실제 타입을 지정하면 형 변환이 생략 가능하다.
```java
//class Box {
//	Object item;
//    
//    void setItem(Object item) { this.item = item; }
//    Object getItem() { return item; }
//}

// generic class 선언
class Box<T> {
	T item;

    void getItem(T item) { this.item = item; }
    T getItem() { return item; }
}
```
- Box클래스의 객체를 생성할 때는 다음과 같이 참조변수와 생성자에 타입 T 대신에 사용될 실제 타입을 지정해주어야 한다.
```java
Box<String> b = new Box<String>();
b.setItem("ABC");
String item = (String) b.getItem();
```
- T대신에 String 타입을 지정해줬으므로 Box<T>는 다음과 같이 정의된 것과 같다.
```java
class Box<String> {
	String item;

    void getItem(String item) { this.item = item; }
    String getItem() { return item; }
}
```
<br>

### 1.2 Generics의 제한
- static 멤버에는 타입 변수 T를 사용할 수 없다.
	* T는 인스턴스 변수로 간주되기 때문
- generic 타입의 배열 T[]를 생성할 수 없다.
<br>

### 1.3 generic 클래스의 객체 생성과 사용
- generic 클래스 Box<T>를 선언하고 Box <T>의 객체를 생성하면 참조 변수와 생성자에 대입된 타입이 일치해야 한다.
- 상속 관계에 있어도 마찬가지이다.
	* 단, 두 generic class가 상속관계이고, 대입된 타입이 일치하는 것은 가능하다.
- 대입된 타입과 다른 타입의 객체는 추가할 수 없다.
```java
import java.util.ArrayList;

class Fruit	{ public String toString() { return "Fruit";}}
class Apple extends Fruit { public String toString() { return "Apple";}}
class Grape extends Fruit { public String toString() { return "Grape";}}
class Toy { public String toString() { return "Toy";}}

class FruitBoxEx1 {
	public static void main(String[] args) {
		Box<Fruit> fruitBox = new Box<Fruit>();
		Box<Apple> appleBox = new Box<Apple>();
		Box<Toy>   toyBox   = new Box<Toy>();
//		Box<Grape> grapeBox = new Box<Apple>(); // 에러. 타입 불일치

		fruitBox.add(new Fruit());
		fruitBox.add(new Apple()); // OK. void add(Fruit item)

		appleBox.add(new Apple());
		appleBox.add(new Apple());
//		appleBox.add(new Toy()); // 에러. Box<Apple>에는 Apple만 담을 수 있음

		toyBox.add(new Toy());
//		toyBox.add(new Apple()); // 에러. Box<Toy>에는 Apple을 담을 수 없음

		System.out.println(fruitBox);
		System.out.println(appleBox);
		System.out.println(toyBox);
	}
}

class Box<T> {
	ArrayList<T> list = new ArrayList<T>();
	void add(T item)  { list.add(item); }
	T get(int i) { return list.get(i); }
	int size() { return list.size(); }
	public String toString() { return list.toString();}
}
```
<br>

### 1.4 제한된 generic 클래스
- generic 타입에 extends를 사용하면, 특정 타입의 자손들만 대입할 수 있게 제한할 수 있다.
- 메서드의 매개변수의 타입 T도 자신과 그 자손 타입이 될 수 있다.
- 인터페이스의 경우에도 implements가 아닌 extends를 사용한다.
```java
import java.util.ArrayList;

class Fruit implements Eatable {
	public String toString() { return "Fruit";}
}
class Apple extends Fruit { public String toString() { return "Apple";}}
class Grape extends Fruit { public String toString() { return "Grape";}}
class Toy { public String toString() { return "Toy"  ;}}

interface Eatable {}

class FruitBoxEx2 {
	public static void main(String[] args) {
		FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
		FruitBox<Apple> appleBox = new FruitBox<Apple>();
		FruitBox<Grape> grapeBox = new FruitBox<Grape>();
//		FruitBox<Grape> grapeBox = new FruitBox<Apple>(); // 에러. 타입 불일치
//		FruitBox<Toy>   toyBox   = new FruitBox<Toy>();   // 에러.

		fruitBox.add(new Fruit());
		fruitBox.add(new Apple());
		fruitBox.add(new Grape());
		appleBox.add(new Apple());
//		appleBox.add(new Grape());  // 에러. Grape는 Apple의 자손이 아님
		grapeBox.add(new Grape());

		System.out.println("fruitBox-"+fruitBox);
		System.out.println("appleBox-"+appleBox);
		System.out.println("grapeBox-"+grapeBox);
	}
}

class FruitBox<T extends Fruit & Eatable> extends Box<T> {}

class Box<T> {
	ArrayList<T> list = new ArrayList<T>();
	void add(T item)  { list.add(item);      }
	T get(int i)      { return list.get(i); }
	int size()        { return list.size();  }
	public String toString() { return list.toString();}
}
```
<br>

### 1.5 와일드 카드 -  '?'
- generic 타입에 와이드 카드를 쓰면, 여러 타입의 대입이 가능하다.
- 와일드카드에는 <? extends T & E>와 같이 &는 사용할 수 없다.
- 와일드카드의 사용으로 다음 코드의 makeJuice( )의 매개변수로 FruitBox <Apple>, FruitBox <Grape>가 가능하다.
```java
import java.util.ArrayList;

class Fruit		          { public String toString() { return "Fruit";}}
class Apple extends Fruit { public String toString() { return "Apple";}}
class Grape extends Fruit { public String toString() { return "Grape";}}

class Juice {
	String name;

	Juice(String name)	     { this.name = name + "Juice"; }
	public String toString() { return name;		 }
}

class Juicer {
	// 와이드카드 사용
	static Juice makeJuice(FruitBox<? extends Fruit> box) {
		String tmp = "";

		for(Fruit f : box.getList())
			tmp += f + " ";
		return new Juice(tmp);
	}
}

class FruitBoxEx3 {
	public static void main(String[] args) {
		FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
		FruitBox<Apple> appleBox = new FruitBox<Apple>();

		fruitBox.add(new Apple());
		fruitBox.add(new Grape());
		appleBox.add(new Apple());
		appleBox.add(new Apple());

		System.out.println(Juicer.makeJuice(fruitBox));
		System.out.println(Juicer.makeJuice(appleBox));
	}  
}

class FruitBox<T extends Fruit> extends Box<T> {}

class Box<T> {
	ArrayList<T> list = new ArrayList<T>();
	void add(T item) { list.add(item);      }
	T get(int i)     { return list.get(i); }
	ArrayList<T> getList() { return list;  }
	int size()       { return list.size(); }
	public String toString() { return list.toString();}
}
```
<br>

### 1.6 generic method
- 반환타입 앞에 generic 타입이 선언된 메서드이다.
- static 멤버에는 타입 매개변수를 사용할 수 없지만, 메서드에 generic 타입을 선언하고 사용하는 것은 가능하다.
- generic 메서드를 호출할 때, 타입 변수에 타입을 대입해야 한다. (대부분 추정 가능해 생략 가능)
- makeJucie()를 다음과 같이 generic method로 바꿀 수 있다.
```java
/*
static Juice makeJuice(FruitBox<? extends Fruit> box) {
	String tmp = "";
	for(Fruit f : box.getList())
		tmp += f + " ";
	return new Juice(tmp);
}
*/

static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
	String tmp = "";
	for(Fruit f : box.getList())
		tmp += f + " ";
	return new Juice(tmp);
}
```
<br>

### 1.7 generic 타입의 형 변환
- generic 타입과 원시 타입 간의 형 변환은 불가능하다.
- 와일드카드가 사용된 generic 타입으로는 형 변환이 가능하다.
- `<? extends Object>`를 줄여서 `<?>`로 쓸 수 있다.
<br>

## 2. 열거형(enums)
- 열거형이란 관련된 상수들을 같이 묶어 놓은 것으로 Java는 JDK1.5부터 타입에 안전한 열거형을 제공하여 타입까지 관리하기 때문에 보다 논리적인 오류를 줄일 수 있다.
<br>

### 2.1 열거형의 정의와 사용
- `enum 열거형이름 { 상수명1, 상수명2, ... }`
- 아래의 코드와 같이 열거형 타입의 변수를 선언하고 사용할 수 있으며 ==와 compareTo( )로 비교 가능하다.
```java
enum Direction { EAST, SOUTH, WEST, NORTH }

class EnumEx1 {
	public static void main(String[] args) {
		Direction d1 = Direction.EAST;
		Direction d2 = Direction.valueOf("WEST");
		Direction d3 = Enum.valueOf(Direction.class, "EAST");

		System.out.println("d1="+d1);
		System.out.println("d2="+d2);
		System.out.println("d3="+d3);

		System.out.println("d1==d2 ? "+ (d1==d2));
		System.out.println("d1==d3 ? "+ (d1==d3));
		System.out.println("d1.equals(d3) ? "+ d1.equals(d3));
//		System.out.println("d2 > d3 ? "+ (d1 > d3)); // 에러
		System.out.println("d1.compareTo(d3) ? "+ (d1.compareTo(d3)));
		System.out.println("d1.compareTo(d2) ? "+ (d1.compareTo(d2)));

		}

		Direction[] dArr = Direction.values();

		for(Direction d : dArr)  // for(Direction d : Direction.values())
			System.out.printf("%s=%d%n", d.name(), d.ordinal());
	}
}
```
<br>

### 2.2 java.lang.Enum
- 모든 열거형은 Enum의 자손이며, 아래의 메서드를 상속받는다.
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/12_1.png)
<br>

### 2.3 열거형에 멤버 추가하기
- 불연속적인 열거형 상수의 경우, 원하는 값을 괄호( ) 안에 적는다.
	* enum Direction { EAST(1), SOUTH(5), WEST(-1), NORTH(10) }
- 괄호( )를 사용하려면, 인스턴스 변수와 생성자를 새로 추가해 줘야 한다.
- 열거형의 생성자는 묵시적으로 private이므로, 외부에서 객체 생성이 불가능하다.
