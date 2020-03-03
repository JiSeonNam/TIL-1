> 자바의 정석(남궁성 저) 학습내용 정리

## Collections Framework
- 컬렉션(다수의 객체)을 다루기 위한 표준화된 프로그래밍 방식이다.
- 컬렉션을 쉽고 편리하게 다룰 수 있는 다양한 클래스를 제공한다
- java.util패키지에 포함되어 있으며 JDK1.2부터 제공되었다
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/11_1.png)

### 1. List 인터페이스
- 중복을 허용하면서 저장 순서가 유지되는 컬렉션을 구현하는 데 사용된다.
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/11_2.png)

### 2. ArrayList
- 기존의 Vector를 개선한 것으로 Vector와 구현원리와 기능적인 측면에서 거의 동일하다.
- Vector는 자체적으로 동기화처리가 되어 있으나 ArrayList는 그렇지 않다.
- Object배열을 이용해서 데이터를 순차적으로 저장한다.

### 3. LinkedList
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/11_3.png)
- 실제로 LinkedList클래스는 이름과 달리 linked list가 아닌 doubly linked list로 구현되어 있다.
- 순차적으로 추가/삭제하는 경우에는 ArrayList가 LinkedList보다 빠르다
- 중간 데이터를 추가/삭제하는 경우에는 LinkedList가 ArrayList보다 빠르다.
- 데이터의 개수가 변하지 않는 경우에는 ArrayList, 변경이 잦다면 LinkedList를 사용하는 것이 좋다.
- 처음에 작업하기 전에 데이터를 저장할 때는 ArrayList를 사용한 다음, 작업할 때는 LinkedList로 데이터를 옮겨서 작업하면 효율이 좋다.
**※ 인덱스가 n인 데이터의 주소 = 배열의 주소 + n * 데이터 타입의 크기**

### 4. Iterator, ListIterator, Enumeration
- 컬렉션에 저장된 데이터를 접근하는 데 사용되는 인터페이스
- Enumeration은 Iterator의 구버전으로 더 이상 사용되지 않는다.
- ListIterator는 Iterator의 접근성을 향상 시킨 것으로 다음 요소뿐 아니라 이전 요소까지도 접근이 가능하다.
- ListIterator는 List 인터페이스를 구현한 컬렉션에만 사용할 수 있고 Iterator는 List와 Set 모두 사용 가능하다.
- Iterator는 일회용으로 한번 얻어오면 한번 사용 가능하며 컬렉션의 요소를 얻어올 때 값이 변하면 예외가 발생한다.
```java
// ArrayList에 있는 모든 요소들을 출력하는 예제
Collection c = new ArrayList();
	...
Iterator it = c.iterator();

while(it.hasNext()) {
	System.out.println(it.next());
}
```

### 5. Arrays
- 1차원 배열의 비교와 출력 : equals( ) , toString( )
- 다차원 배열의 비교와 출력 : deepEquals( ) , deepToString( )
- 배열의 복사 : 배열 전체 - copyOf( ) , 배열의 일부 - copyOfRange( ) 는 배열을 복사하여 새로운 배열을 만들어 반환한다.
- 배열의 정렬과 검색 : 배열의 정렬 - sort( ) , 배열의 검색 - binarySearch( ). 이때 binarySearch( )를 사용하려면 배열이 sort( )를 통해 정렬되어 있어야 한다.
- 배열을 List로 변환 : asList( ). 읽기 전용으로 List 컬렉션을 반환한다.

### 6. Comparator와 Comparable
- 객체를 정렬하는데 필요한 메서드를 정의한 인터페이스로 정렬 기준을 정할 때 사용한다.
- Comparable은 기본 정렬 기준을 구현하는데 사용하고 Comparator는 기본 정렬기준 외에  다른 기준으로 정렬하고자 할 때 사용한다.
- Arrays.sort( )는 자동으로 정렬되어 있는 것이 아니라 메서드 안에 compareTo( )를 호출해서 정렬에 사용한다.
- 따라서 원하는 기준으로 compareTo( ) 메서드만 작성하면 Arrays.sort( )가 알아서 정렬해준다.

### 7. HashSet
- Set 인터페이스를 구현한 대표적인 컬렉션으로 순서와 중복이 없고 순서를 유지하고 싶으면 LinkedHashSet을 사용해야 한다.
- boolean add(Object o) 메서드는 중복을 판별하기 위해 추가하려는 요소의 equals( )와 hashCode( )를 호출하기 때문에 equals( )와 hashCode( )를 목적에 맞게 오버라이딩해야 한다.

### 8. TreeSet
- 중복과 순서를 유지하지 않으며 범위 검색과 정렬에 유리한 이진 탐색 트리로 구현되어 있다.
- 부모보다 작은 값을 왼쪽에, 큰 값을 오른쪽에 저장한다.
- HashSet보다 데이터 추가, 삭제에 시간이 더 걸린다.

### 9. HashMap
- Map 인터페이스를 구현하여 key와 value를 하나로 묶어서 데이터로 저장한다.
- Hasing기법을 사용하기 때문에 많은 양의 데이터를 검색하는데 성능이 좋다.

### 10. Properties
- 내부적으로 HashMap의 구버전인 Hashtable을 상속받아 사용하며, key와 value를 String으로 저장한다.
- 주로 애플리케이션의 환경 설정에 관련된 속성을 저장하는 데 사용되며 파일로부터 편리하게 값을 읽고 쓸 수 있는 메서드를 제공한다.

### 11. Collections
- 컬렉션과 관련된 메서드를 제공한다. fill( ), copy( ), sort( ), binarySearch( ) 등의 메서드도 포함되어 있다.
- 그 밖에도 컬렉션의 동기화, 변경 불가(readOnly) 컬렉션 만들기, singleton 컬렉션 만들기, 한 종류의 객체만 저장하는 컬렉션 만들기 등의 기능을 제공한다.

### 12. 컬렉션 클래스 정리
![](https://github.com/qlalzl9/TIL/blob/master/Java/img/11_4.png)
