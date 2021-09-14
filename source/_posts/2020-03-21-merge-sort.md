---
title: Merge sort
date: 2020-03-21 02:26:00 Z
categories: [Coding Problems]
tags: [merge-sort, algorithms]
cover: /images/fsqs-tips-tricks-notes.png
---

Merge sort is a divide-and-conquer algorithm based on the idea of breaking down a list into several sub-lists until each sublist consists of a single element and merging those sublists in a manner that results into a sorted list.
<!-- more -->

## Main Idea

- Divide the unsorted list into `N` sublists, each containing  element.
- Take adjacent pairs of two singleton lists and merge them to form a list of `2` elements. `N` will now convert into `N/2` lists of size `2`.
- Repeat the process till a single sorted list of obtained.

## Example

![](/images/merge-sort/solution.jpg)

## Implementation in Java

```java
class MergeSort {

    private void merge(int[] A, int start, int mid, int end) {
        //stores the starting position of both parts in temporary variables.
        int p = start, q = mid + 1;

        int[] Arr = new int[end - start + 1];
        int k = 0;

        for (int i = start; i <= end; i++) {
            if (p > mid)      //checks if first part comes to an end or not .
                Arr[k++] = A[q++];

            else if (q > end)   //checks if second part comes to an end or not
                Arr[k++] = A[p++];

            else if (A[p] < A[q])     //checks which part has smaller element.
                Arr[k++] = A[p++];

            else
                Arr[k++] = A[q++];
        }

        for (int i = 0; i < k; i++) {
            /*
                Now the real array has elements in sorted manner including both parts.
            */
            A[start++] = Arr[i];
        }
    }

    private void mergeSort(int[] A, int start, int end) {
        if (start < end) {
            int mid = (start + end) / 2;           // defines the current array in 2 parts .
            mergeSort(A, start, mid);                 // sort the 1st part of array .
            mergeSort(A, mid + 1, end);              // sort the 2nd part of array.

            // merge the both parts by comparing elements of both the parts.
            merge(A, start, mid, end);
        }
    }

    void mergeSort(int[] A) {
        this.mergeSort(A, 0, A.length - 1);
    }
}
```

## Test Cases

```java
class MergeSortTest {

    private MergeSort solution;

    static Collection<Object[]> data() {
        return Arrays.asList(
                new Object[][]{
                        {new int[]{5, 7, 8, 9, 2}, new int[]{2, 5, 7, 8, 9}},
                        {new int[]{9, 7, 8, 3, 2, 1}, new int[]{1, 2, 3, 7, 8, 9}},
                }
        );
    }

    @BeforeEach
    void setUp() {
        this.solution = new MergeSort();
    }


    @ParameterizedTest
    @MethodSource("data")
    void testSolution(int[] input, int[] expected) {
        this.solution.mergeSort(input);
        assertArrayEquals(expected, input);
    }

}
```

## Analysis

- Time Complexity: O(n log(n))
- Space Complexity: O(n)

Merge Sort is a stable sort which means that the same element in an array maintain their original positions with respect to each other. Overall time complexity of Merge sort is O(nLogn). It is more efficient as it is in the worst case also the runtime is O(nlogn).
