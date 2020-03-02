> 자바의 정석(남궁성 저) 학습내용 정리

### 1. 날짜와 시간
##### 1.1 Date클래스와 Calendar클래스

- java.util.Date : 날짜와 시간을 다룰 목적으로 만들어진 클래스(JDK1.0), 거의 deprecated 되었지만, 여전히 쓰이고 있다.
- java.util.Calendar : Date클래스를 개선한 새로운 클래스(JDK1.1), 여전히 단점이 존재한다.
- Calendar를 Date로 변환
```java
Calendar cal = Calendar.getInstance();
	...
Date d = new Date(cal.getTimeInMillis());
```
- Date를 Calendar로 변환
```java
Date d = new Date();
	...
Calendar cal = Calendar.getInstance();
cal.setTime(d)
```
- Calendar의 get(Calendar.MONTH)는 1~12가 아닌 0~11이다. (0 : 1월, 11 : 12월)
- Calendar의 get(Calendar.DAY_OF_WEEK)에서 요일은 1부터 시작하여 차례로 일, 월, 화, 수, 목, 금, 토이다.
- set( )으로 날짜와 시간 지정
```java
void set(int field, int value)
void set(int year, int month, int date)
void set(int year, int month, int date, int hourOfDay, int minute)
void set(int year, int month, int date, int hourOfDay, int minute, int second)
```


### 2. 형식화 클래스
##### 2.1 DecimalFormat
- 숫자를 다양한 형식(패턴)으로 출력할 수 있게 해 준다.
- DecimalFormat을 사용하여 소수점 자리를 원하는 데로 쉽게 구현할 수 있다.
```java
double number  = 123.456789;

DecimalFormat df1 = new DecimalFormat("0000.000");
DecimalFormat df2 = new DecimalFormat("####.##");

System.out.println(df1.format(number));
System.out.println(df2.format(number));

/*  실행결과
 *  0123.457
 *  123.46
 */
```

##### 2.2 SimpleDateFormat
- 날짜와 시간을 다양한 형식으로 출력할 수 있게 해 준다..
```java
DateFormat df1 = new SimpleDateFormat("yyyy년 MM월 dd일");
DateFormat df2 = new SimpleDateFormat("yyyy/MM/dd");

Date d = df1.parse("2020년 1월 6일");		// 문자열을 Date로 변환
String result = df2.format(d));
```
##### 2.3 ChoiceFormat
- 특정 범위에 속하는 값을 문자열로 변환해준다.
- 반복문으로 처리하기 복잡한 경우 유용하다.
- 패턴 구분자 #은 경곗값을 포함하고 <는 포함하지 않는다.
```java
import java.util.*;
import java.text.*;

class ChoiceFormat {
	public static void main(String[] args) {
		double[] limits = {60, 70, 80, 90};	// 낮은 값부터 큰 값의 순서로 적어야한다.

		String[] grades = {"D", "C", "B", "A"};	// limits, grades간의 순서와 개수를 맞추어야 한다.

		int[] scores = { 100, 95, 88, 70, 52, 60, 70};

		ChoiceFormat form = new ChoiceFormat(limits, grades);

		for(int i=0;i<scores.length;i++) {
			System.out.println(scores[i]+":"+form.format(scores[i]));		
		}
	}
}
```
##### 2.4 MessageFormat
- 데이터를 정해진 양식에 맞게 출력할 수 있도록 도와준다.
- 데이터가 들어갈 자리를 마련해 놓은 양식을 미리 작성하고 프로그램을 이용해서 다수의 데이터를 같은 양식으로 출력할 때 사용하면 좋다.
```java
import java.util.*;
import java.text.*;

class MessageFormat {
	public static void main(String[] args) {
		String msg = "Name: {0} \nTel: {1} \nAge:{2} \nBirthday:{3}";

		Object[] arguments = {
			"홍길동","02-123-1234", "26", "03-17"
		};

		String result = MessageFormat.format(msg, arguments);
		System.out.println(result);
	}
}
```

### 3. java.time 패키지
- Date와 Calendar의 단점을 개선한 새로운 클래스들을 제공한다. (JDK1.8)
- 시간은 LocalTime클래스, 날짜는 LocalDate클래스, 날짜와 시간 모두는 LocalDateTime클래스를 사용한다.

##### 3.1 LocalDate와 LocalTime
- java.time 패키지의 가장 기본이 되는 클래스이며, 나머지 클래스들은 이들의 확장이다.
- now( )는 현재 날짜 시간을, of( )는 특정 날짜 시간을 지정할 때 사용한다.
```java
LocalDate today = LocalDate.now();		//오늘의 날짜
LocalDate now = LocalTime.now();		//현재 시간

LocalDate birthDate = LocalDate.of(1999, 12, 31);		// 1999년 12월 31일
LocalTime birthTime = LocalTime.of(23, 59, 59);			//23tl 59분 59초
```
- parse( )로 문자열을 LocalDate나 LocalTime으로 변환할 수 있다.
```java
LocalDate birthDate = LocalDate.parse("1999-12-31");		// 1999년 12월 31일
LocalTime birthTime = LocalTime.parse("23:59:59");		// 23시 59분 59초
```
##### 3.2 Period와 Duration
- Period는 날짜의 차이를, Duration은 시간의 차이를 계산하기 위한 것이다.
- 두 날짜 또는 시간의 차이를 구할 때는 between( )을 사용한다.
```java
// 두 날짜의 차이 구하기
LocalDate date1 = LocalDate.of(2020, 1, 6);
LocalDate date2 = LocalDate.of(2019, 12, 25);

Period pe = Period.between(date1, date2);

// 두 시각의 차이 구하기
LocalTime time1 = LocalTime.of(00,00,00);
LocalTime time2 = LocalTime.of(12,34,56);

Duration du = Duration.between(time1, time2);
```
