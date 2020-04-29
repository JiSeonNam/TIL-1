# 셸 정렬(Shell Sort)
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Shell_Sort_1" width="600px"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Shell_Sort_2" width="600px"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Shell_Sort_3" width="600px"></p>
- 정렬할 배열의 요소를 그룹으로 나눠 각 그룹 별로 단순 삽입 정렬을 수행하고, 그 그룹을 합치면서 정렬을 반복하여 요소의 이동 횟수를 줄이는 알고리즘.
- 삽입 정렬의 장점은 살리고 단점은 보완한 알고리즘이다.
- 시간복잡도 : O($n^{2}$)

## 구현
```java
static void shellSort(int[] arr) {
	for (int i = arr.length / 2; i > 0; i /= 2)
		for (int j = i; j < arr.length; j++) {
			int tmp = arr[j];
			for (k = j - i; k >= 0 && arr[k] > tmp; k -= h) {
				arr[k + i] = arr[k];
            }
			arr[k + i] = tmp;
		}
	}
```