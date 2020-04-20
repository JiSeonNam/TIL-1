# 선택정렬 (Selection Sort) 

## 단순 선택 정렬
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Selection_Sort_1.jpg)
- 가장 작은 요소부터 선택해 알맞은 위치로 옮겨서 순서대로 정렬하는 알고리즘이다.
- 서로 떨어져 있는 데이터를 교환하는 것이기 때문에 안정적이지 않다.
- 비교 횟수 : ($$ n^{2} $$ - n) / 2 회 <br>
시간복잡도 : O($$ n^{2} $$)

### 구현
```java
static void SelctionSort(int arr[]) {
    for(int i = 0; i < arr.length; i++) {
        int min = i;
        for(int j = i + 1; j < n; j++) {
            if(arr[j] < arr[min]){
                min = j;
            }
        }
        swap(arr, i, min);
    }
}
static void swap(int[] arr, int index1, int index2) {
	int temp = arr[index1];
	arr[index1] = arr[index2];
	arr[index2] = temp;
}
```