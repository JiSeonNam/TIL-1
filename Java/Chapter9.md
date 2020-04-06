> 자바의 정석(남궁성 저) 학습내용 정리

### 1. Object 클래스
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/9_1.png)

#### 1-1 equals(Object obj)

```java
public boolean equals(Object obj) {
	return (this==obj);
}
```

- 매개변수로 객체의 참조 변수를 받아서 비교하여 그 결과를 boolean값으로 알려주는 역할을 한다.
- 두 참조 변수에 저장된 값(주소 값)이 같은지를 판단하는 기능
- 인스턴스가 가지고 있는 내용을 비교하기 위해서는 equals 메서드를 오버라이딩하여 주소가 아닌 객체에 저장된 내용을 비교하도록 변경해야 한다.

#### 1-2 hashCode( )
- 해싱 기법에 사용되는 '해시함수'를 구현한 것이다.
- 값이 저장된 위치를 알려주는 hashcode를 반환한다.
- 문자열 내용이 같으면 동일한 해시코드를 반환하도록 오버라이딩하여 사용할 수 있다.
- equals( )를 오버라이딩 하면, equals( )의 결과가 true인 두 객체의 hashcode는 같아야 하기 때문에 hashCode( )도 같이 오버라이딩 해야한다.
- System.identityHashCode(Object x)는 Object클래스의 hashCode매서드처럼 객체의 주소 값으로 해시코드를 생성하기 때문에 모든 객체에 대해 항상 다른 해시코드값을 반환한다

#### 1-3 toString( )

```java
public String toString() {
	return getCalss().getName() +"@"+ Integer.toHexString(hashCode());
}
```

- 인스턴스에 대한 정보를 문자열로 반환한다. (클래스@해시코드)
- 일반적으로 인스턴스나 클래스에 대한 정보 또는 인스턴스 변수들의 값을 문자열로 변환하여 반환하도록 오버라이딩하여 사용한다.
- Object클래스에 정의된 toString( )의 접근 제어자가 public 이므로 이를 오버라이딩할 때 접근제어자를 public으로 선언해야 한다.

#### 1-4 clone
- 자신을 복제하여 새로운 인스턴스를 생성한다.
- Object클래스에 정의된 clone( )은 인스턴스 변수의 값만을 복사하기 때문에 참조 타입의 작업이 원래의 인스턴스에 영향을 미치게 된다. 따라서 clone메서드를 오버라이딩해서 사용해야 한다.
- clone( )을 사용하기 위해 복제할 클래스가 Cloneable인터페이스를 구현해야 하고 clone( )을 오버라이딩하면서 접근제어자를 protected에서 public으로 변경해야 한다.
- Cloneable인터페이스를 구현한 클래스의 인스턴스만 clone( )을 통한 복제가 가능한데, 이는 인스턴스의 데이터를 보호하기 위해서이다. Cloneable인터페이스가 구현되었다는 것은 클래스 작성자가 복제를 허용한다는 의미이다.

#### 1-5 getClass( )

```java
public final class Class implements ... {
	...
}
```

- 자신의 속한 클래스의 Class객체를 반환하는 메서드
- Class객체는 클래스의 모든 정보를 담고 있으며, 클래스 당 1개만 존재한다. 클래스 파일이 클래스 로더에 의해서 메모리에 올라갈 때, 자동으로 생성된다.
- 클래스 로더는 실행 시에 필요한 클래스를 동적으로 메모리에 로드하는 역할

**※ Class 객체를 얻는 방법**

```java
Class cobj = new Card().getClass();		// 생성된 객체로부터 얻는 방법
Class cobj = Card.class;		// 클래스 리터럴(.class)로부터 얻는 방법
Class cobj = Class.forName("Card");		// 클래스 이름으로부터 얻는 방법
```

### 2. String 클래스
- 문자열 배열(char[ ])과 그에 관련된 메서드들이 정의되어 있다.
- String 인스턴스의 내용은 바꿀 수 없다. (immutable)

#### 2-1 빈 문자열

- 자바는 내용이 없는 문자열. 즉 크기가 0인 char 배열을 저장하는 문자열을 사용하는 것이 가능하다.
- String을 참조형의 기본값인 null로 초기화하고 char를 기본값인 '\u0000'으로 초기화하는 것보다
String을 빈 문자열(" ")로 초기화하고 char를 공백(' ')으로 초기화하는 것이 바람직하다.
- String은 참조형의 기본값인 null 보다 빈 문자열로 초기화하고 char형은 기본값인 ‘\u0000’보다 공백으로 초기화하자.

![](https://github.com/qlalzl9/TIL/blob/master/Java/img/9_2.png)

#### 2-2 문자열과 기본형 간의 변환
- 기본형 값을 문자열로 바꾸는 방법
```java
int i = 100;
String str1 = i + "";					// 방법1 : 100을 "100"으로 변환
String str2 = String.valueOf(i);			//방법2 : 100을 "100"으로 변환
// 방법2가 더 빠르다
```

- 문자열을 기본형 값으로 변환하는 방법
```java
int i = Integer.parseInt("100");		// 방법1 : "100"을 100으로 변환
int i2 = Integer.valueOf("100");		// 방법2 : "100"을 100으로 변환(JDK1.5 이후)
char c = "A".charAt(0);				// 문자열 "A"를 문자 'A'로 변환하는 방법
```

![](https://github.com/qlalzl9/TIL/blob/master/Java/img/9_3.png)

### 3. StringBuffer 클래스
- String처럼 문자형 배열(char[ ])을 내부적으로 가지고 있다. 그러나 String클래스와 달리 내용을 변경할 수 있다.(mutable)
- 인스턴스를 생성할 때 버터(배열)의 크기를 충분히 지정해주는 것이 좋다.
- String 클래스와는 달리 equals( )를 오버라이딩 하지 않는다.


### 4. Math 클래스
- 기본적인 수학계산에 유용한 메서드로 구성되어 있다.
- Math.round() 메서드를 사용하면 소수점 첫째 자리에서 반올림한 정수 값을 반환한다.
	- ※ 소수점 두 자리까지의 값만을 얻고 싶으면
	-	원래 값에 100을 곱하고     ex) 90.7552 * 100 -> 9075.52
	- Math.round( ) 사용한 뒤       Math.round(9075.52) -> 9076
	- 결과를 100.0으로 나누면 된다.  9076/100.0 -> 90.76
	- 셋째 자리는 1000, 넷째 자리는  10000이다.



### 5. wrapper 클래스
- 기본형을 클래스로 정의한 것으로 기본형 값도 객체로 다루어야 할 때가 있다.
- 내부적으로 기본형(primitive type) 변수를 가지고 있다.
- 값을 비교하도록 equals( )가 오버라이딩되어 있다.
