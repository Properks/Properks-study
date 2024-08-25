# Arrays.sort() vs Collections.sort()

## 목차

1. 각 메소드가 사용하는 정렬 알고리즘
    &emsp; • Arrays.sort()
    &emsp; • Collections.sort()
2. 두 정렬 알고리즘과 자료형의 차이
    &emsp; • 데이터의 저장 방식
    &emsp; • 각 정렬 알고리즘이 가지는 장점

3. 추측
    &emsp; • 기본형 배열에 Dual-pivot Quick Sort를 사용하는 이유
    &emsp;• 참조형 배열에 Tim Sort를 사용하는 이유
4. 직접 확인
    &emsp; • 정렬 시간 결과
    &emsp; • int[]에 대한 추가 결과
5. 결론

## 각 메소드가 사용하는 정렬 알고리즘

### • Arrays.sort()

 Arrays.sort()는 parameter로 무엇을 받느냐에 따라 다른 정렬 알고리즘을 실행한다.
 <b>int[]와 같은 기본형에는 Dual-pivot Quick Sort를 적용하여 정렬을 실행한다.</b> 해당 알고리즘은 기존의 Quick sort보다 빠르고 시간복잡도를 O(nlogn)에 가깝게 정렬할 수 있다.
 반면에 <b>Integer[]와 같은 Object[] 형태로 이루어진 배열에서는 기본적으로 TimSort를 진행한다.</b> 유저의 요청 (`-Djava.util.Arrays.useLegacyMergeSort=true`)으로 legacyMergeSort를 실행할 수도 있다.

Arrays.sort(int[] a) 실행 시 사용하는 Method
```java
    /**
     * Sorts the specified array into ascending numerical order.
     *
     * @implNote The sorting algorithm is a Dual-Pivot Quicksort
     * by Vladimir Yaroslavskiy, Jon Bentley, and Joshua Bloch. This algorithm
     * offers O(n log(n)) performance on all data sets, and is typically
     * faster than traditional (one-pivot) Quicksort implementations.
     *
     * @param a the array to be sorted
     */
    public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, 0, a.length);
    }
```

Arrays.sort(Object[] a)
```java

    public static void sort(Object[] a) {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a);
        else
            ComparableTimSort.sort(a, 0, a.length, null, 0, 0);
    }
```

 ### • Collections.sort()

 Collections.sort()도 Tim sort를 사용한다. 밑의 코드를 보면 `list.sort()`를 사용하는데 이는 List를 Array형태로 변경하여 `Arrays.sort(T[] a, Comparator<? super T> c)`를 사용하고 순서대로 원소를 바꾸는 것을 알 수 있다.

 ```java
    // Collections.sort(List<T> list)
     public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }

    // list.sort(null)
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    // Arrays.sort(a, (Comparator) c)
        public static <T> void sort(T[] a, Comparator<? super T> c) {
        if (c == null) {
            sort(a);
        } else {
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a, c);
            else
                TimSort.sort(a, 0, a.length, c, null, 0, 0);
        }
    }
 ```
 
## 두 정렬 알고리즘과 자료형의 차이

#### • 데이터의 저장 방식

위의 정렬 알고리즘 코드에서 알 수 있듯이 기본형인 int와 참조형인 Integer사이에서 발생하는 차이이다.


1. 기본형의 저장 방식
기본형은 실제 값을 스택 메모리 영역에 저장한다.

<br>

2. 참조형의 저장 방식
참조형은 실제 데이터를 힙 메모리 영역에 저장하고 스택에는 참조(주소) 값을 저장한다.


#### • 각 정렬 알고리즘이 가지는 장점

 Dual-pivot Quick Sort
 1. 참조 지역성이 좋다. 
 2. 정렬되지 않은 배열에서 좋은 성능을 보인다.

 Tim sort
 1. 최악의 경우에도 시간복잠도가 `O(nlogn)`이다.
 2. 부분적으로 정렬된 배열에서 좋은 성능을 보인다.

## 추측

#### • 기본형 배열에 Dual-pivot Quick Sort를 사용하는 이유
Dual-pivot Quick Sort의 장점 중 좋은 참조 지역성으로 Dual-pivot Quick Sort는 실제 값을 나란히 스택 메모리 영역에 저장하여 방금 접근한 메모리의 근처에 접근이 쉬운 기본형 정렬에서 좋은 성능을 보일 것으로 추측된다.

#### • 참조형 배열에 Tim Sort를 사용하는 이유
참조형 배열은 실제 값이 저장된 위치가 연속적으로 있는 것이 아닌 참조값만 저장하므로 참조지역성이 좋지 않다. 즉 Dual-pivot Quick Sort에서 <b><i>캐시 적중율 (hit-rate)</i></b>를 높게 유지할 수 없을 것으로 예상된다. 

따라서 최악의 경우에도 `O(nlogn)`을 유지하고 Merge Sort보다 공간 복잡도가 낮으며 `C × nlogn + α`라는 동작 시간에서 C 값을 작게 유지할 수 있는 Tim Sort를 사용하는 것으로 예상된다.

## 직접 확인

#### • 정렬 시간 결과
아래의 결과는 1,000,000개의 무작위 원소를 100번 정렬한 경우의 평균 시간이다.
```shell
> Task :Main.main()
Dual-pivot Quick Sort
int[]: 0.13876999999999998s
Integer[]: 0.43644000000000005s
List: 0.43065999999999993s

Tim Sort
int[]: 0.11916s
Integer[]: 0.21272s
List: 0.22536000000000006s
```

`Integer[]`와 `List<Integer>`의 경우에는 Dual-pivot Quick Sort에 비해 Tim Sort가 확실히 시간을 적게 소모한다는 것을 알 수 있다.

반면 `int[]`의 경우 추측과는 다르게 Tim Sort에서 더 좋은 성능을 보였다. 하지만 밑의 내용을 읽어보면 알 수 있듯이 실제로 기본형이 Dual-pivot Quick Sort에 최적화되어 있다.

#### • int[]에 대한 추가 결과

100,000개의 무작위 원소를 100번 정렬한 평균
```shell
> Task :Main.main()
Dual-pivot Quick Sort
int[]: 0.017429999999999987s

Tim Sort
int[]: 0.010180000000000007s
```

10,0000,000개의 무작위 원소를 100번 정렬한 평균
```shell
> Task :Main.main()
Dual-pivot Quick Sort
int[]: 2.05719s

Tim Sort
int[]: 7.374059999999998s
```

위는 추가적으로 원소의 개수만 바꾸어 정렬을 실행해본 결과이다. 원소의 개수가 증가하면서 Dual-pivot Quick Sort와 Tim Sort의 차이가 줄어들며 오히려 1,000,000개에서는 Tim Sort가 더 오래걸렸다. 즉, 기본형 배열에서 Dual-pivot Quick Sort가 Tim Sort에 비해 안정적인 성능을 보여준다.


## 결론

1. `Arrays.sort(int[] array)`는 Dual-pivot Quick Sort, `Arrays.sort(Object[] array)`와 `Collections.sort(List<T> list)`는 Tim Sort를 사용한다.


2. 기본형은 스택 메모리에 실제 값을 나란히 저장하는 방식, 참조형은 힙 메모리에 저장하고 참조(주소) 값을 스택 메모리에 저장하여 기본형이 참조 지역성이 더 좋다.


3. 참조 지역성이 안 좋은 `Object[]`와 `List<T>`는 좋은 성능을 보여주는 Tim Sort를 적용한다.

4. 참조 지역성이 좋은 `int[]`는 Dual-pivot Quick Sort가 Tim Sort에 비해 안정적인 성능을 보여준다.

#### 참고자료
정렬 알고리즘 - [The algorithm Github](https://github.com/TheAlgorithms/Java)
Tim Sort - [네이버 D2](https://d2.naver.com/helloworld/0315536)
