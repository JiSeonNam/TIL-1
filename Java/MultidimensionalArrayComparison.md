자바에서는 보통 Object 클래스의 equals 메서드를 오버라이딩하여 객체를 비교한다.

다차원 배열을 비교하기 위해서는 equals( )와 반복문을 함께 써야하지만

Object 클래스의 보조 클래스인 **Objects 클래스(java.util.Objects)** 를 사용하여 다차원 배열을 쉽게 비교할 수 있다.

Objects 클래스는 **deepEquals( )** 메서드를 제공하여 객체를 재귀적으로 비교하기 때문에 다차원 배열의 비교도 가능하다.
```java
String[][] 2Dstr1 = new String[][] {{"aaa", "bbb"}, {"AAA", "BBB"}};
String[][] 2Dstr2 = new String[][] {{"aaa", "bbb"}, {"AAA", "BBB"}};

System.out.println(Objects.equals(2Dstr1, 2Dstr2));		// false
System.out.println(Objects.deepEquals(2Dstr1, 2Dstr2));		// true
```	
