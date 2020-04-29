# 도수 정렬(Frequency Sort)

- 도수를 이용하여 데이터의 대소 관계를 판단하지 않고 빠르게 정렬하는 알고리즘
- 시간복잡도 : O(n)
- 정렬과정
    * 도수분포표 만들기<br>
    <p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Frequency_Sort_1.jpg" width="600px"></p>
    * 누적도수분포표 만들기<br>
    <p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Frequency_Sort_2.jpg" width="600px"></p>
    * 목적 배열 만들기<br>
    <p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Frequency_Sort_3.jpg" width="600px"></p>
    * 배열 복사하기
<br>

## 구현
```java
static void fSort(int[] a, int n, int max) {
	int[] f = new int[max + 1];		// 누적 도수
	int[] b = new int[n];			// 작업용 목적 배열
    
    for (int i = 0; i < n; i++) f[a[i]]++;				// 도수분포표 만들기
	for (int i = 1; i <= max; i++) f[i] += f[i - 1];		// 누적도수분포표 만들기
	for (int i = n - 1; i >= 0; i--) b[--f[a[i]]] = a[i];		// 목적 배열 만들기
	for (int i = 0; i < n; i++) a[i] = b[i];				// 배열 복사하기
}
```