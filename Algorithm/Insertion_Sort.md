# 삽입 정렬 (Insertion Sort) 

## 단순 삽입 정렬
![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Insertion_Sort_1.jpg)
- 선택한 요소를 그보다 더 앞쪽의 알맞은 위치에 삽입하는 작업을 반복하여 정렬하는 알고리즘이다.
- 단순 선택 정렬과 비슷하게 보일 수 있지만 단순 선택 정렬은 값이 가장 작은 요소를 선택해 알맞은 위치로 옮긴다는 점이 다르다. 
- 카드를 한 줄로 늘어놓을 때 사용하는 방법과 유사한 알고리즘이다. 
- 서로 떨어져 있는 데이터를 교환하는 것이기 때문에 안정적이지 않다.
- 시간복잡도 : O($n^{2}$)

### 구현
```java
static void insertionSort(int arr[]) {
    for (int i = 1; i < arr.length; i++) {
        int tmp = arr[i];
        for (int j = i; j > 0 && arr[j - 1] > tmp; j--) {
            arr[j] = arr[j - 1];
        }
        arr[j] = tmp;
    }
}
```