##이진 검색(Binary Search)
- 이진 검색은 요소가 오름차순 또는 내림차순으로 정렬된 배열에서 검색하는 알고리즘이다.
- 검색을 반복할 때마다 검색 범위가 절반이 되므로 검색에 필요한 비교 횟수의 평균은 log n이고 시간 복잡도는 O(log n)이다.
- 오름차순으로 정렬된 데이터인 아래의 그림에서 39를 검색하는 과정은 먼저 배열의 중앙에 위치한 a[5]부터 검색을 시작한다.
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Binary_Search_1.png)
- 검색하려는 값인 39는 a[5]인 31보다 큰 값이므로 검색 대상을 a[5]보다 큰 값으로 좁힐 수 있다. 그 다음 검색 범위의 중아엥서 위치한 요소인 a[8]과 39를 비교한다.
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Binary_Search_2.png)
- 검색하려는 값인 39는 a[8]인 68보다 작으므로 검색 대상을 작은 값이 a[6]과 a[7]로 줄이고 중앙값((6+7)/2)인 a[6]과 비교했을 때 원하는 값인 39와 일치하므로 검색을 마친다.
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Binary_Search_3.png)
<br>

**이진 검색 예시 코드**
```Java
import java.util.Scanner;
// 이진 검색
class BinSearch {
	static int binSearch(int[] a, int n, int key) {
		int pl = 0; // 검색 범위의 첫 인덱스
		int pr = n - 1; // 검색 범위의 끝 인덱스

		do {
			int pc = (pl + pr) / 2; // 중앙 요소의 인덱스
			if (a[pc] == key)
				return pc; // 검색 성공
			else if (a[pc] < key)
				pl = pc + 1; // 검색 범위를 뒤쪽 절반으로 좁힘
			else
				pr = pc - 1; // 검색 범위를 앞쪽 절반으로 좁힘
		} while (pl <= pr);

		return -1; // 검색 실패
	}

	public static void main(String[] args) {
		Scanner stdIn = new Scanner(System.in);

		System.out.print("요솟수：");
		int num = stdIn.nextInt();
		int[] x = new int[num];

		System.out.println("오름차순으로 입력하세요.");

		System.out.print("x[0]：");
		x[0] = stdIn.nextInt();

		for (int i = 1; i < num; i++) {
			do {
				System.out.print("x[" + i + "]：");
				x[i] = stdIn.nextInt();
			} while (x[i] < x[i - 1]); // 바로 앞의 요소보다 작으면 다시 입력
		}

		System.out.print("검색할 값：");
		int ky = stdIn.nextInt();
		int idx = binSearch(x, num, ky);

		if (idx == -1) {
			System.out.println("그 값의 요소가 없습니다.");
		} else {
			System.out.println(ky + "은(는) x[" + idx + "]에 있습니다.");
    }
  }
}
/*
실행결과
요솟수：7
오름차순으로 입력하세요.
x[0]：15
x[1]：27
x[2]：39
x[3]：77
x[4]：92
x[5]：108
x[6]：121
검색할 값：39
39은(는) x[2]에 있습니다.
*/
```
<br>

### Arrays.binarySearch
- Java에서는 배열에서 이진 검색을 하는 메서드를 표준 라이브러리고 제공한다.
- 이진 검색 표준 라이브러리의 메서드로는 java.util.Arrays 클래스의 binarySearch 메서드가 있다.
- binarySearch 메서드는 오름차순으로 정렬된 배열 a를 가정하고, 키 값이 key인 요소를 이진 검색한다.
- 검색에 성공한 경우 요소의 인덱스를 반환하고, 일치하는 요소가 여러 개인 경우 무작위의 인덱스를 반환한다.
- 검색에 실패한 경우 삽입 포인트를 x라고 할 때 -x-1을 반환한다.
- 사용 예시 : ```java int idx = Arrays.binarySearch(x, ky); ```
