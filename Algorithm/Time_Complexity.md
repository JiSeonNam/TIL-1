## 시간복잡도

알고리즘의 성능 분석의 주된 요소는 2가지가 있다.

- 공간 복잡도(Space Complexity) : 알고리즘에 사용되는 메모리 공간의 총량
- 시간 복잡도(Time Complexity) : 알고리즘에 사용되는 연산 횟수의 총 횟수
- 보통 공간 복잡도와 시간 복잡도는 반비례한다.

저장매체의 발전 덕분에 메모리의 용량이 점차 커지면서 보통 평가기준을 시간 복잡도를 기준으로 한다.

시간 복잡도는 절대적인 수행 시간이 아닌 연산 횟수를 기준으로 측정하며 연산 횟수를 카운팅 할 때 3가지 경우가 있다.

- 최선의 경우(Best Case) : 오메가 표기법(Big-Ω)
- 평균적인 경우(Average Case) : 세타 표기법(Big-Θ)
- 최악의 경우(Worst Case) : 빅오 표기법(Big-O)

3가지 경우 중 평균적인 경우가 가장 좋아 보이지만 평균을 구하기 까다롭고 어려워 최악의 경우(Big-O)를 기준으로 한다.
<br>

## Big-O
**정의**

> 두 함수 f(n), g(n)이 있을 때,  $$n_{1}$$ ≤ n, f(n) ≤ C*g(n)이 성립하는 상수 C, $$n_{1}$$이 존재하면 f(n)=O(g(n))이다.

![Big-O Complexity](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Time_Complexity_1.png)

그림과 같이 Big-O의 연산 횟수는 O(1) → O(logn) → O(n) → O(nlogn) → O($$n^{2}$$) → O($$2^{n}$$) → O(n!)으로 갈수록 증가한다. 
- 일반적으로 입력의 개수와 시간 복잡도 함수의 관계는 상당히 복잡하다. 그러나 자료의 개수가 많을 경우 차수가 가장 큰 항이 가장 영행을 미치고 다른 항들은 상대적으로 무시될 수 있다.
- 예를 들어 f(n) = $$n^{2}$$ + n + 1이라고 가정하면, n이 1000일 때, n의 제곱 값은 전체의 99.9%를 차지하고 나머지는 0.01%를 차지한다. 
- 따라서 Big-O는 가장 큰 차수에 의해서 결정된다.
ex) $$n^{2}$$ + n + 1 의 시간 복잡도 : O($$n^{2}$$)
ex) $$n^{3}$$ + $$n^{2}$$ + n + 1 의 시간 복잡도 : O($$n^{3}$$)
<br>
 
## Java에서의 실제 Big-O 표기법 예
- O(1)
    * 코드에서 n의 값과 출력하는 횟수는 무관하다.
    * O(1) = O(2) = O(3) = O(1000) 이다.
```java
int n = 1000;
System.out.println("n = " + n);
System.out.println("n = " + n);
System.out.println("n = " + n);
/*
n = 1000
n = 1000
n = 1000
*/
```
<br>

- O(logn)
    * 로그 시간 알고리즘의 일반적인 예는 이진 검색 알고리즘이다.
    * logn에 비례하여 증가한다.
    * 아래의 코드에서 n이 1000일 때 실행 횟수는 log1000 = 3 회 이다.
```java
int n = 1000;

for (int i = 1; i < n; i *= 10){
    System.out.println("i = " + i);
}
/*
i = 1
i = 10
i = 100
*/
```
<br>

- O(n)
    * 실행 시간은 n의 크기에 비례하여 선형적으로 증가하며 출력문의 횟수는 무관하다.
    * 따라서 O(n) = O(2n) = O(2n+1)
```java
int n = 1000;

for (int i = 1; i <= n; i++) {
    System.out.println("i = " + i);
    System.out.println("i = " + i);
    System.out.println("i = " + i);
}
/*
i = 1
i = 2
  .
  .
  .
i = 1000
*/
```
<br>

- O(nlogn)
    * nlogn 크기에 비례하여 증가한다.
    * 아래의 코드에서 n이 1000이면 실행 횟수는 1000 * log(1000) = 1000 * 3 = 3000 회 실행된다.
```java
int n = 1000;

for (int i = 1; i <= n; i++) {
	for (int j = 1; j < 1000; j *= 10) {
		System.out.println("i = " + i + ", j = " + j);
	}
}

/*
i = 1, j = 1
i = 1, j = 10
i = 1, j = 100
	.
	.
	.
i = 1000, j = 1
i = 1000, j = 10
i = 1000, j = 100
*/
```
<br>

- O($$n^{2}$$)
    * 이중 루프 안에서 입력 자료를 처리할 경우 시간 복잡도는 O($$n^{2}$$)이다.
    * 만약 삼중 루프문이라면 시간 복잡도는 O($$n^{3}$$)이다.
    * ($$n^{2}$$)에 비례하여 증가하며 아래의 코드의 경우 n이 3이라면 $$3^{2}$$ = 9회 실행된다.
```java
int n = 3;

for (int i = 1; i <= n; i++) {
	for(int j = 1; j <= n; j++) {
	System.out.println("i = " + i + ", j = " + j);
	}
}
/*
i = 1, j = 1
i = 1, j = 2
i = 1, j = 3
 	 .
	 .
	 .
i = 3, j = 1
i = 3, j = 2
i = 3, j = 3
*/
```
<br>

- O($$2^{n}$$)
    * 시간 복잡도 O($$2^{n}$$)은 입력마다 2배로 증가한다. 
    * 즉, O($$2^{n}$$)에 비례하여 증가하며 아래의 코드와 같이 n이 8이면라면 $$2^{8}$$ = 256회 실행된다.
```java
int n = 8;

for (int i = 1; i <= Math.pow(2, n); i++){
	System.out.println("i = " + i);
}
/*
i = 1
i = 2
  .
  .
  .
i = 256
*/
```

- O(n!)
    * n!에 비례하여 증가하며 아래의 코드와 같이 n이 8이라면 8! = 40320회 실행된다.
```java
int n = 8;

//factorial() 메서드가 구현되어있다고 가정
for (int i = 1; i <= factorial(n); i++){
	System.out.println("i = " + i);
}
/*
i = 1
i = 2
  .
  .
  .
i = 40320
*/
```
<br>

## 자료구조에서의 시간 복잡도

- 정렬 알고리즘에서의 시간 복잡도
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Time_Complexity_2.png)
<br>

- 자료구조에서의 시간 복잡도
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Time_Complexity_3.png)
<br>

## Reference
- C언어로 쉽게 풀어쓴 자료구조 (천인국, 공용해, 하상호)
- [Practical Java Example of the Big O Notation](https://www.baeldung.com/java-algorithm-complexity){:target="_blank"}
- [알고리즘의 시간 복잡도와 Big-O 쉽게 이해하기](https://blog.chulgil.me/algorithm/){: target="_blank"}
