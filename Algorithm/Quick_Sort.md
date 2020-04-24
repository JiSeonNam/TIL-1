# 퀵 정렬(Quick Sort)
- 분할 정복 알고리즘의 하나로, 매우 빠른 정렬 알고리즘이다. 
- 불안정 정렬에 속하며, 비교만으로 정렬을 수행하는 비교 정렬이다.
- 시간복잡도
    * 시간복잡도는 pivot을 어느 것으로 선택하는지에 따라 영향을 많이 받는다.
    <br>
    
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Quick_Sort_1.jpg)
- 정렬 과정
    * 요소들의 그룹을 나누는 기준인 pivot을 결정한다.
    * pivot을 기준으로 pivot보다 작은 데이터들은 pivot 왼쪽으로, 큰 데이터들은 pivot 오른쪽으로 교환한다.
    * pivot을 제외한 왼쪽 그룹과 오른쪽 그룹을 각각 다시 Quick Sort 한다.
    * 그룹의 크기가 0 또는 1이 될 때까지 반복한다.<br>

![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Quick_Sort_2.jpg)

## 구현
### 재귀를 사용한 구현
```java
static void swap(int[] arr, int index1, int index2) {
	int temp = arr[index1];  
    arr[index1] = arr[index2];
    arr[index2] = temp;
}
// 퀵 정렬
static void quickSort(int[] arr, int left, int right) {
	int pl = left;					// 왼쪽 커서
	int pr = right;					// 오른쪽 커서
	int x = a[(pl + pr) / 2];		// pivot
	do {
		while (a[pl] < x) pl++;
	    while (a[pr] > x) pr--;
		if (pl <= pr)
			swap(arr, pl++, pr--);
	} while (pl <= pr);
		if (left < pr)  quickSort(arr, left, pr);
		if (pl < right) quickSort(arr, pl, right);
}
```

### 재귀를 사용하지 않은 구현
```java
static void swap(int[] arr, int index1, int index2) {
	int temp = arr[index1];
    arr[index1] = arr[index2];
    arr[index2] = temp;
}

// 퀵정렬
static void quickSort(int[] arr, int left, int right) {
	IntStack lstack = new IntStack(right - left + 1);
	IntStack rstack = new IntStack(right - left + 1);

	lstack.push(left);
	rstack.push(right);

	while (lstack.isEmpty() != true) {
		int pl = left  = lstack.pop();		// 왼쪽 커서
		int pr = right = rstack.pop();		// 오른쪽 커서
		int x = arr[(left + right) / 2];	// pivot

		do {
			while (arr[pl] < x) pl++;
			while (arr[pr] > x) pr--;
			if (pl <= pr)
				swap(arr, pl++, pr--);
		} while (pl <= pr);

		if (left < pr) {
			lstack.push(left);				// 왼쪽 그룹 범위의 
			rstack.push(pr);				// 인덱스를 푸시합니다.
		}
		if (pl < right) {
			lstack.push(pl);				// 오른쪽 그룹 범위의 
			rstack.push(right);				// 인덱스를 푸시합니다.
		}
	}
}
```