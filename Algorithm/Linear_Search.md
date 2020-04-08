# 선형검색
<br>

## 선형검색(Linear Search)이란?
- 선형검색은 배열에서 검색하는 방법 가운데 가장 기본적인 알고리즘이다.
- 요소가 직선 모양으로 늘어선 배열에서의 검색은 원하는 키 값을 갖는 요소를 만날 때까지 맨 앞부터 순서대로 요소를 검색한다.
- 순차검색(Sequential Search)라고도 한다.
- 선형 검색의 예시
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Linear_Search_1.png)
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Linear_Search_2.png)
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Linear_Search_3.png)
- 예시를 통해 알 수 있는 선형검색의 종료 조건은 2개이다.
  1. 검색할 값과 같은 요소를 발견한 경우
  2. 검색할 값을 발견하지 못하고 배열의 끝을 지나간 경우
- 따라서 조건 1, 2를 판단하는 횟수는 평균 n/2회이다.
<br>

**예시 코드**
```java
import java.util.Scanner;

// 선형 검색 (for문)
class SeqSearchFor {
	// 배열 a의 앞쪽 n개의 요소에서 key와 같은 요소를 선형 검색
	static int seqSearch(int[] a, int n, int key) {
		for (int i = 0; i < n; i++)
			if (a[i] == key) { // 검색 성공!(인덱스를 반환)
				return i;
			}
		return -1; // 검색 실패!(-1을 반환)
	}

	public static void main(String[] args) {
		Scanner stdIn = new Scanner(System.in);

		System.out.print("요솟수：");
		int num = stdIn.nextInt();
		int[] x = new int[num]; // 요솟수 num인 배열

		for (int i = 0; i < num; i++) {
			System.out.print("x[" + i + "]：");
			x[i] = stdIn.nextInt();
		}

		System.out.print("찾는 값："); // 킷값을 읽어 들임
		int ky = stdIn.nextInt();
		int idx = seqSearch(x, num, ky); // 배열x에서 값이 ky인 요소를 검색

		if (idx == -1) {
			System.out.println("그 값의 요소가 없습니다.");
		} else {
			System.out.println("그 값은 x[" + idx + "]에 있습니다.");
		}
	}
}
/* 실행 결과
-----------------------
요솟수：7
x[0]：22
x[1]：8
x[2]：55
x[3]：32
x[4]：120
x[5]：55
x[6]：70
찾는 값：55
그 값은 x[2]에 있습니다.
-----------------------
*/
```
<br><br>

## 보초법(Sentinel Method)
- 선형검색은 위의 내용과 같이 조건 1, 2를 모두 판단한다. 이 때 검색할 데이터가 많으면 많을 수록 조건을 검사하는 비용은 결코 무시할 수 없는데 이 비용을 50%로 줄이는 방법이 보초법이다.
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Linear_Search_4.png)
- 그림에서 배열의 요소를 검색하기 전에 검색하고자 하는 키 값을 맨 끝 요소에 저장하면 원하는 데이터가 존재하지 않아도 배열의 끝에서 키 값이 찾아지기 때문에 종료 판단 횟수를 2회에서 1회로 줄일 수 있다.
<br>

**예시코드**
```java
import java.util.Scanner;
// 선형 검색(보초법)
class SeqSearchSen {
	// 요솟수가 n인 배열 a에서 key와 같은 요소를 보초법으로 선형 검색합니다.
	static int seqSearchSen(int[] a, int n, int key) {
		int i = 0;

		a[n] = key; // 보초를 추가

		while (true) {
			if (a[i] == key) { // 검색 성공!
				break;
      }
			i++;
		}
		return i == n ? -1 : i;
	}

	public static void main(String[] args) {
		Scanner stdIn = new Scanner(System.in);

		System.out.print("요솟수：");
		int num = stdIn.nextInt();
		int[] x = new int[num + 1]; // 요솟수 num + 1

		for (int i = 0; i < num; i++) {
			System.out.print("x[" + i + "]：");
			x[i] = stdIn.nextInt();
		}

		System.out.print("검색할 값："); // 키값을 입력
		int ky = stdIn.nextInt();

		int idx = seqSearchSen(x, num, ky); // 배열x에서 값이 ky인 요소를 검색

		if (idx == -1) {
			System.out.println("그 값의 요소가 없습니다.");
		} else {
			System.out.println(ky + "은(는) x[" + idx + "]에 있습니다.");
    }
  }
}
/*
실행결과
---------------------------
요솟수：7
x[0]：22
x[1]：8
x[2]：55
x[3]：32
x[4]：120
x[5]：55
x[6]：70
검색할 값：120
120은(는) x[4]에 있습니다.
---------------------------
*/
```
