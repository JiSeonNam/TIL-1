> 자바의 정석(남궁성 저) 학습내용 정리

### 1. 에러의 구분
- 컴파일 에러 : 컴파일 시에 발생하는 에러
- 런타임 에러 : 실행 시에 발생하는 에러
- 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것
<br>

#### 자바에서의 에러
- 런타임 에러를 에러(error)와 예외(exception)로 구분
- 에러(error) : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
- 예외(exception) : 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/8_1.png)

- Exception 클래스들(checked exception) : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외 (예외처리 필수)
- RuntimeException 클래스들(unchecked exception) : 프로그래머의 실수로 발생하는 예외 (예외처리 선택)
<br>

### 2. 예외처리 하기 : try-catch
- 예외처리의 정의 : 프로그램 실행 시 발생할 수 있는 예외의 발생에 대비한 코드를 작성하는 것
- 예외처리의 목적 : 프로그램의 비정상 종료를 막고, 정상적인 실행상태를 유지하는 것
```java
try {
	// 예외가 발생할 가능성이 있는 문장들을 넣는다.
} catch (Exception1 e1) {
	// Exception1이 발생했을 경우, 이를 처리하기 위한 문장을 적는다.
} catch (Exception2 e2) {
	// Exception2이 발생했을 경우, 이를 처리하기 위한 문장을 적는다.
} catch (ExceptionN eN) {
	// ExceptionN이 발생했을 경우, 이를 처리하기 위한 문장을 적는다.
}
// if문과들 달리 문장이 하나여도 { } 생략이 불가능
```
- try-catch문 내에 try-catch문을 쓸 수 있지만 변수가 중복되서는 안 된다.
- try블록 내에서 예외가 발생한 경우 : 발생한 예외와 일치하는 catch블록을 확인 --> 일치하는 catch블록을 찾으면 문장들을 수행하고 전체 try-catch문을 빠져나가 다음 문장을 수행한다. 일치하는 catch블록을 찾지 못하면, 예외는 처리되지 못한다.
- try블록 내에서 예외가 발생하지 않은 경우 : catch블록을 거치지 않고 전체 try-catch문을 빠져나가서 수행을 계속한다.
- catch블록의 ( ) 안에 Exception클래스 타입의 참조 변수를 선언해 놓으면 어떤 종류의 예외가 발생하더라도 이 catch블록에 의해서 처리된다. 따라서 항상 마지막 catch문에 사용해야 한다. 그렇지 않으면 밑의 블록까지 내려가기 전에 다 예외처리를 해버린다.
예외 클래스의 인스턴스에 담겨있는 정보를 얻을 수 있는 방법
- printStackTrace( ) : 예외 발생 당시의 호출 스택에 있었던 메서드의 정보와 예외 메시지를 화면에 출력한다.
- getMessage( ) : 발생한 예외 클래스의 인스턴스에 저장된 메시지를 얻을 수 있다.
<br>

##### finally 블록
```java
try {
	// 예외가 발생할 가능성이 있는 문장들을 넣는다.
} catch (Exception1 e1) {
	// 예외처리를 위한 문장을 적는다.
} finally {
	// 예외 발생여부에 관계없이 항상 수행되어야하는 문장들을 넣는다.
    // finally블럭은 try-catch문의 맨 마지막에 위치해야한다.
}
```
- 예를 들어 예외가 없어 try문만 실행되어도, 예외가 있어 try문이 끝까지 수행하지 못하고 catch문으로 실행되어도 실행되어야 하는 문장들을 finally에 넣어 반드시 실행시킬 수 있게 하고 코드의 중복을 줄인다.
- catch문에서 예외처리를 하고 return 하여 반환해도 finally문은 실행된다.
<br>

### 3. 예외 발생시키기
-  발생시키려는 예외 클래스의 객체를 만든 다음 throw를 이용해서 예외를 발생시킨다.
```java
public class ExceptionEx9 {

	public static void main(String[] args) {
		try {
			throw new Exception("고의로 발생시켰음");
		} catch (Exception e) {
			System.out.println("에러 메시지 : " + e.getMessage());
			e.printStackTrace();
		}
		System.out.println("프로그램 정상종료");
	}
}
```
<br>

### 4. 메서드에 예외 선언하기
```java
void method() throws Exception1, Exception2, ... ExceptionN {
	// 메서드의 내용
}
// 예외를 발생시킬때는 throw, 예외를 메서드에 선언할 때는 throws
```
<br>

#### 예외 되던지기
- 한 메서드에서 발생할 수 있는 예외가 여럿인 경우, 몇 개는 try-catch문을 통해서 메서드 내에서 자체적으로 처리하고, 나머지는 선언부에 지정하여 호출한 메서드에서 처리하도록 함으로써, 양쪽에서 나눠서 처리되도록 할 수 있다.
- 하나의 예외에 대해서 예외가 발생한 메서드와 이를 호출한 메서드 양쪽 모두 처리해줘야 할 작업이 있을 때도 활용 가능하다.
```java
public class Exception {

	public static void main(String[] args) {
		try {
			method1();
		} catch (Exception e) {
			System.out.println("main 메서드에 의해 예외 처리");
		}
	}

	static void method1() throws Exception {
		try {
			throw new Exception();
		} catch (Exception e) {
			System.out.println("method1 메서드에 의해 예외 처리");
			throw e;
		}
	}
}
```