# 힙 정렬(Heap Sort)
- 힙(heap)을 사용하여 정렬하는 알고리즘
- 힙이란?
    * 최댓값 및 최솟값을 찾아내는 연산을 빠르게 하기 위해 고안된 완전이진트리(Complete binary tree)를 기본으로 한 자료구조.
- 최대 힙 트리 또는 최소 힙 트리를 구성하고 root에 있는 데이터를 배열의 마지막 index부터 저장하여 정렬하는 알고리즘이다.
- 시간복잡도<br>

![](https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Heap_Sort_1.jpg)
- 정렬 과정
    1. 정렬해야 할 n개의 데이터로 최대 또는 최소 힙을 만든다.
    2. root의 값을 배열의 뒤부터 저장한다.
    3. root와 힙의 마지막 데이터(트리의 오른쪽 아래 끝 데이터)를 교환한다.
    4. 바뀐 root의 값을 비교하며 알맞은 위치로 옮긴다.
    5. 1~4 반복

<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Heap_Sort_2.jpg" width="600px"></p>

## 구현
```java
static void swap(int[]arr, int index1, int index2) {
	int t =arr[index1];  
	a[index1] =arr[index2];  
	a[index2] = t;
}
//arr[left] ~arr[right]를 힙으로 만들기
static void downHeap(int[]arr, int left, int right) {
	int temp =arr[left];			// 루트
	int child;						// 큰 값을 가진 노드
	int parent;						// 부모
		
    for (parent = left; parent < (right + 1) / 2; parent = child) {
	    int cl = parent * 2 + 1;							// 왼쪽 자식
	    int cr = cl + 1;									// 오른쪽 자식
	    child = (cr <= right &&arr[cr] >arr[cl]) ? cr : cl;	// 큰 값을 가진 노드를 자식에 대입 
	    if (temp >=arr[child]) {
		    break;
        }
		a[parent] =arr[child];
	}
	a[parent] = temp;
}
// 힙 정렬
static void heapSort(int[]arr, int n) {
	for (int i = (n - 1) / 2; i >= 0; i--) {	//arr[i] ~arr[n-1]를 힙으로 만들기
		downHeap(a, i, n - 1);
    }
	for (int i = n - 1; i > 0; i--) {
		swap(a, 0, i);				// 가장 큰 요소와 아직 정렬되지 않은 부분의 마지막 요소를 교환
		downHeap(a, 0, i - 1);		//arr[0] ~arr[i-1]을 힙으로 만들기
	}
}
```