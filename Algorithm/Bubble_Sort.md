# 버블 정렬(Bubble Sort)

## 버블 정렬 알고리즘
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Bubble_Sort_1.jpg)
- 서로 인접한 두 요소의 대소 관계를 비교하여 교환을 반복하는 알고리즘
- 구현이 간단하지만 구현할 때 데이터의 교환 작업이 있기 떄문에 효율이 좋지 않다.
- 비교 횟수 : n-1 + n-2 + ... + 2 + 1 = n(n-1)/2<br>
교환 횟수 : 교환 작업을 위해 3번의 이동이 필요하므로 3n(n-1)/2
시간 복잡도 : O($n^{2}$)

## 구현
```java
static void bubbleSort(int arr[]) {
    for(int i = 0; i < arr.length; i++) {
        for(int j = 0; j < arr.length - i - 1; j++) {
            if(arr[j] > arr[j+1]) {
                swap(arr, j, j+1);
            }
        }
    }
}
static void swap(int[] arr, int index1, int index2) {
	int temp = arr[index1];
	arr[index1] = arr[index2];
	arr[index2] = temp;
}
```