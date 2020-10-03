# 4. Algorithm(알고리즘)

# Sorting Algorithm

## 1. Selection Sort(선택 정렬)

선택 정렬은 처음 제일 작은 값을 찾아서 0번 인덱스부터 차례대로 넣는 방식으로 진행된다. 

```java
public static int[] sort(int[] arr) {
  int minIndex;
  for(int i = 0; i < arr.length-1; i++) {
    minIndex = i;
    for(int j = i+1; j < arr.length; j++) {
      if(arr[minIndex] > arr[j]) {
          minIndex = j;
      }
    }
    Utils.swap(arr,minIndex,i);
  }
  return arr;
}
```

> 시간 복잡도: O(n^2) 
공간 복잡도: O(1)

## 2. Insertion Sort(삽입 정렬)

i번째까지 정렬되어있다고 가정할 때 i+1값을 i부터 0번까지 가면서 해당 위치에 넣어주는 정렬이다. 

```java
public static int[] sort(int[] arr) {
  int key;
  for(int i = 1; i < arr.length; i++) {  
	  key = arr[i];                     //key는 i번째 값
	  int j = i-1;
	  while(j >= 0 && key < arr[j]) {   //i-1부터 앞으로 가면서 들어갈 위치 확인 
      arr[j+1] = arr[j];
      j--;
	  }
	  arr[j+1] = key;                   //들어갈 위치에 삽입
  }
  return arr;
}
```

> 시간 복잡도: O(n^2) 
공간 복잡도: O(1)

## 3. Bubble Sort(거품 정렬)

두개의 값을 비교하면서 계속 swap하여 가장 큰 값을 뒤로 오게 하는 정렬 

 

```java
public int[] bubbleSort(int[] arr) {
  for(int i = arr.length-1; i > 0; i--) {
    for(int j = 0; j < i; j++) {
      if(arr[j] > arr[j+1]) {
        Utils.swap(arr,j,j+1);
      }
    }
  }
  return arr;
}
```

> 시간 복잡도: O(n^2) 
공간 복잡도: O(1)

## 4. Quick Sort(퀵 정렬)

quicksort는 아래의 로직으로 이루어진다. 크게 pivot을 구하고 pivot을 기준으로 재귀적으로 진행된다.

1. 중간 index를 pivot으로 잡고 pivot을 맨 앞에 값과 스왑한다.
2. 그리고 첫번째 값과 끝값 양방향으로 진행해오면서 pivot보다 값이 크면 오른쪽으로 가고 작으면 왼쪽으로 가도록 swap을 한다. 
3. 이 과정을 마친 후 pivot과 중간 값을 swap하고 pivot의 인덱스를 리턴한다. 
4. pivot을 기준으로 좌우로 재귀를 진행한다. 

pivot의 위치부터 정해지고 이 과정이 재귀적으로 진행되기 때문에 시간은 O(logn)의 시간 복잡도를 가지고 pivot을 정하는 시간은 O(n)이 걸린다.

```java
public static void quicksort(int[] array, int left, int right) {
  if(left >= right)
    return;
  int pivot = partition(array,left,right);

  quicksort(array,left,pivot-1);
  quicksort(array,pivot+1,right);
}

public static int partition(int[] array, int left, int right) {
  int pivot = array[left];
  int i = left;
  int j = right;
  while(i< j) {
    while(pivot < array[j])
        j--;
    while (i< j && pivot >= array[i])
        i++;
    Utils.swap(array,i,j);
  }
  array[left] = array[i];
  array[i] = pivot;
  return i;
}
```

> 시간 복잡도: O(nlog(n)) 
최악의 경우 정렬된 배열이 앞에서 pivot을 잡으면 파티션이 되지 않아 n번의 횟수를 반복하여 O(n^2)이 나온다. 
공간 복잡도: O(logn)

## 5. Merge Sort(병합 정렬)

## 6. Heap Sort(힙 정렬)

Heap Sort는 두가지 방법으로 실행할 수 있다. 

1. Heap Sort은 정렬 대상을 힙으로 넣었다가 꺼내는 원리로 Heap Sort를 실행할 수 있다. 
2. 기존의 배열을 heapify를 통해 heap으로 만들어주는 과정을 통해서 Heap Sort를 실행할 수 있다.

2번의 방법을 실행할 때

순서는

1. 있는 배열로 최대 힙을 구성(heapify)한다. 
2. 최대힙의 루트와 가장 뒷 값을 바꾼다. 
3. 배열 크기를 하나 줄이고 다시 heapify를 진행한다. 

```java
//가장 큰 값을 parent에 두도록한다.(O(logn))
public static void heapify(int[] array,int length ,int i) {
  int parent = i;
  int left = 2*i+1;
  int right = 2*i+2;

  //left childNode
  if(left < length && array[left] > array[parent])
    parent = left;
  //right childNode
  if(right < length && array[right] > array[parent])
    parent = right;

  if(parent != i) {
    Utils.swap(array,parent,i);
    heapify(array,length,parent);
  }
}

public static void heapSort(int[] array) {
  int length = array.length;
  for(int i = length/2-1; i>=0; i--) {      //최대 힙 구성
    heapify(array,length,i);
  }
	//O(nlogn)
  for(int i = length-1; i >0; i--) {
    Utils.swap(array,0,i);          // 맨 마지막 값과 root swap
    heapify(array,i,0);             // 최대 힙 구성
  }
}
```

> 시간 복잡도: O(nlogn) 
(heapify: O(logn))
공간 복잡도: O(1)