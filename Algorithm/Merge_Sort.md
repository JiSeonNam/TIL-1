# 병합 정렬(Merge Sort)
- 데이터의 앞부분과 뒷부분으로 나누어 각각 정렬한 다음 병합하는 작업을 반복하여 정렬을 수행하는 알고리즘
- 안정 정렬에 속하며, 분할 정복 알고리즘의 하나이다.
- 배열로 구성하는 것보다 연결 리스트로 구성하는 것이 효율적이다.
- 시간복잡도<br>

![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Merge_Sort_1.jpg)
- 정렬 과정
    * 데이터의 길이가 0 또는 1이면 이미 정렬된 것으로 본다.
    * 그렇지 않은 경우, 정렬되지 않은 데이터를 절반으로 잘라 비슷한 크기의 두 부분으로 나눈다.
    * 각 부분 데이터를 재귀적으로 병합 정렬을 이용해 정렬한다.
    * 데이터를 다시 하나의 정렬된 리스트로 병합한다.<br>

![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Merge_Sort_2.jpg)

## 구현
```java
static int[] buff;	// 작업용 배열

// a[left] ~ a[right]를 재귀적으로 병합 정렬 
static void __mergeSort(int[] a, int left, int right) {
	if (left < right) {
		int i;
		int center = (left + right) / 2;
		int p = 0;
		int j = 0;
		int k = left;
		__mergeSort(a, left, center);			// 배열의 앞부분을 병합 정렬
		__mergeSort(a, center + 1, right);		// 배열의 뒷부분을 병합 정렬
		for (i = left; i <= center; i++)
			buff[p++] = a[i];
		while (i <= right && j < p)
			a[k++] = (buff[j] <= a[i]) ? buff[j++] : a[i++];
		while (j < p)
			a[k++] = buff[j++];
	}
}

// 병합 정렬
static void mergeSort(int[] a, int n) {
	buff = new int[n];				// 작업용 배열을 생성
	__mergeSort(a, 0, n - 1);		// 배열 전체를 병합 정렬
	buff = null;					// 작업용 배열을 해제
}
```