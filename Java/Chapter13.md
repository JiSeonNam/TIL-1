> 자바의 정석(남궁성 저) 학습내용 정리

### 1. 쓰레드의 구현과 실행
- 쓰레드를 구현하는 방법에는 Thread 클래스를 상속받는 방법과 Runnable 인터페이스를 구현하는 방법이 있다.
- Thread 클래스를 상속받으면 다른 클래스를 상속받을 수 없기 때문에, Runnable 인터페이스를 구현하는 방법이 일반적이다.
- Runnable 인터페이스를 구현한 경우, 인터페이스를 구현한 클래스의 인스턴스를 생성한 다음 인스턴스를 Thread 클래스 생성자의 매개변수로 제공해야 한다.
- Thread 클래스를 상속받은 경우, 자손 클래스에서 조상인 Thread 클래스의 메서드를 직접 호출할 수 있지만, Runnable을 구현하면 Thread 클래스의 static 매서드인 currentThread( )를 호출하여 쓰레드에 대한 참조를 얻어 와야만 호출이 가능하다.
- **Thread 클래스를 상속**
```java
class MyThread extends Thread {
	public void run() {
    	...
    }
}
// Thread 클래스의 run()을 오버라이딩
```
- **Runnable 인터페이스를 구현**
```java
class MyThread implements Runnable {
	public void run() {
    	...
    }
}
// Runnable 인터페이스의 run()을 구현
```

### 2. start( )와 run( )
- main 메서드에서 run( )을 호출하는 것은 생성된 쓰레드를 실행시키는 것이 아니라 단순히 클래스에 선언된 메서드를 호출하는 것이다.
- start( )는 새로운 쓰레드가 작업을 실행하는데 필요한 호출스택을 생성한 다음에 run( )을 호출해서, 생성된 호출스택에 run( )이 첫 번째로 올라가게 한다.
- 실행 중인 사용자 쓰레드가 하나도 없을 때 프로그램은 종료된다.

### 3. 쓰레드의 우선순위
- 쓰레드는 우선순위라는 속성을 가지고 있어 이 우선순위에 따라 쓰레드가 얻는 실행시간이 달라진다.
- 쓰레드가 가질 수 있는 우선순위 범위는 1~10이며 숫자가 높을수록 우선순위가 높다.
- 쓰레드의 우선순위는 쓰레드를 생성한 쓰레드로부터 상속받는다.
(main 메서드를 수행하는 쓰레드는 우선순위가 5이므로 main 메서드 내에서 생성하는 쓰레드의 우선순위는 5이다.)
```java
void setPriority(int newPriority)	// 쓰레드의 우선순위를 지정한 값으로 변경
int getPriority()			// 쓰레드의 우선순위를 반환

public static final int MAX_PRIORITY = 10;		// 최대우선순위
public static final int MIN_PRIORITY = 1; 		// 최소우선순위
public static final int NORM_PRIORITY = 5;		// 보통우선순위
```

### 4. 쓰레드 그룹
- 쓰레드 그룹은 서로 관련된 쓰레드를 그룹으로 다루기 위한 것으로, 폴더를 생성해서 관련된 파일들을 함께 넣어서 관리하는 것처럼 쓰레드 그룹을 생성해서 쓰레드를 그룹으로 묶어서 관리할 수 있다.
- 보안상의 이유로 도입된 개념으로, 자신이 속한 쓰레드 그룹이나 하위 쓰레드 그룹은 변경할 수 있지만 다른 쓰레드 그룹의 쓰레드를 변경할 수 없다.
- 모든 쓰레드는 반드시 쓰레드 그룹에 포함되어 있어야 하기 때문에, 그룹을 지정하는 생성자를 사용하지 않은 쓰레드는 기본적으로 자신을 생성한 쓰레드와 같은 그룹에 속하게 된다.
- 자바 어플리케이션이 실행되면, JVM은 main과 system이라는 쓰레드 그룹을 만들고 JVM운영에 필요한 쓰레드들을 생성해서 이 쓰레드 그룹에 포함시킨다.

### 5. 데몬 쓰레드
- 데몬 쓰레드는 다른 일반 쓰레드의 작업을 돋는 보조적인 역할을 수행하는 쓰레드이다.
<br>
- 일반 쓰레드가 모두 종료되면 데몬 쓰레드는 자동 종료된다.
- 무한루프와 조건문을 이용해서 실행 후 대기하고 있다가 특정 조건이 만족되면 작업을 수행하고 다시 대기하도록 작성한다.
- 일반 쓰레드의 작성방법과 실행 방법이 같으며 생성 후 실행하기 전 setDaemon(true)를 호출하기만 하면 된다.
- 데몬 쓰레드가 생성한 쓰레드는 자동적으로 데몬 쓰레드가 된다.
```java
boolean isDaemon()		// 쓰레드가 데몬 쓰레드인지 확인. 데몬쓰레드이면 true를 반환
void setDaemon(boolean on) 	// 쓰레드를 데몬 쓰레드로 또는 사용자 쓰레드로 변경. on이 true이면 데몬 쓰레드
```

### 6. 쓰레드의 실행제어

##### 6.1 쓰레드의 상태
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/13_1.png)
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/13_2.png)

##### 6.2 쓰레드의 스케쥴링
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/13_3.png)
- sleep(long millis) : 일정 시간 동안 쓰레드를 멈추게 한다.
```java
static void sleep(long millis)
static void sleep(long millis, int nanos)
```
- interrupt( )와 interrupted( ) : 쓰레드의 작업을 취소한다.
```java
void interrupt()		// 쓰레드의 interrupted상태를 false에서 true로 변경.
boolean isInterrupted()		// 쓰레드의 interrupted상태를 반환.
static boolean interrupted()	// 현재 쓰레드의 interrupted상태를 반환 후, false로 변경.
```
- suspend( ), resume( ), stop( )
	* suspends( )는 sleep( )처럼 쓰레드를 멈추게하며, resume( )을 호출해야 다시 실행 대기 상태가 된다.
	* stop( )은 호출되는 즉시 쓰레드가 종료된다.
	* suspend( )와 stop( )은 교착상태(deadlock)를 일으키기 쉽게 작성되어있으므로 사용이 권장되지 않아 deprecated되었다.
- yield( ) : 다른 쓰레드에게 자신에게 주어진 실행시간을 양보한다.
- join( ) : 다른 쓰레드의 작업을 기다린다.
```java
void join()
void join(long millis)
void join(long millis, int nanos)
```

### 7. 쓰레드의 동기화
- 한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 쓰레드의 동기화라고 한다.
- synchronized를 이용한 동기화 : 가장 간단한 방법이며 두 가지 방식이 있다.
```java
// 메서드 전체를 임계 영역으로 지정
public sysnchronized void calcSum() {
	...
}

// 특정한 영역을 임계 영역으로 지정
synchronized(객체의 참조변수) {
	...
}
```
