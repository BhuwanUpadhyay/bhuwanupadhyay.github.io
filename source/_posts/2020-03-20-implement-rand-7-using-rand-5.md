---
title: Implement rand7 using rand5
date: 2020-03-20 14:38:00 Z
categories: [Coding Problems]
tags: [interview, algorithms]
cover: /images/fsqs-tips-tricks-notes.png
---

Using a function rand5() that returns an integer from 1 to 5 (inclusive) with uniform probability, implement a function rand7() that returns an integer from 1 to 7 (inclusive).

<!-- more -->


Do NOT use system's Math.random()

## Example

```
Input: 2
Output: [7,4]

Input: 3
Output: [3,1,7]
```
## Solution - by Rejection Sampling

This solution is based upon Rejection Sampling.

The main idea is when you generate a number in the desired range, output
that number immediately. If the number is out of the desired range, reject it and
re-sample again. As each number in the desired range has the same
probability of being chosen, a uniform distribution is produced.

Obviously, we have to run rand5() function at least twice,
as there are not enough numbers in the range of 1 to 7.
By running rand5() twice, we can get integers from 1 to 25 uniformly.

Since 25 is not a multiple of 7, we have to use rejection sampling.
Our desired range is integers from 1 to 21, which we can return the answer immediately.
If not (the integer falls between 22 to 25), we reject it and repeat the whole process again.

```java
class Solution {

    /**
     * @return random integer 1 to 5 (inclusive) with uniform (or equal) probability
     */
    private int rand5() {
        int min = 1;
        int max = 5;
        Random r = new Random();
        return r.nextInt((max - min) + 1) + min;
    }

    int rand7() {
        int row, col, idx;
        do {
            row = rand5();
            col = rand5();
            idx = col + (row - 1) * 5;
        } while (idx > 21);
        return 1 + (idx - 1) % 7;
    }

}
```

## Test Cases

```java
class SolutionTest {

    private Solution solution;

    @BeforeEach
    void setUp() {
        this.solution = new Solution();
    }

    @Test
    void testSolution() {
        IntStream.range(0, 1000)
                .mapToObj(value -> this.solution.rand7())
                .forEach(random -> {
                    assertTrue(
                            random > 0 && random <= 7,
                            String.format("random number: %d -> is not in range {1, 7} inclusive", random)
                    );
                });
    }
}
```

## Analysis

- Time Complexity: O(1) average, but O(∞) worst case.
- Space Complexity: O(1)
