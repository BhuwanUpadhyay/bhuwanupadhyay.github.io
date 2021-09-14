---
title: Find the longest palindromic contiguous substring
date: 2020-03-20 10:57:00 Z
categories: [Coding Problems]
tags: [interview, algorithms]
cover: /images/coding-problems.png
---

Given a string, find the longest palindromic contiguous substring.If there are more than one with the maximum length, return any one.
<!-- more -->

## Example

For example, the longest palindromic substring of `aabcdcb` is `bcdcb`. 
The longest palindromic substring of `bananas` is `anana`.

## Solution - by Manacher Algorithm

```java
class Solution {

    String longestPalindrome(String s) {
        char[] t = preprocess(s);
        int[] p = new int[t.length];

        int center = 0, right = 0;
        for (int i = 1; i < t.length - 1; i++) {
            int mirror = 2 * center - i;

            if (right > i) {
                p[i] = Math.min(right - i, p[mirror]);
            }

            // attempt to expand palindrome centered at i
            while (t[i + (1 + p[i])] == t[i - (1 + p[i])]) {
                p[i]++;
            }

            // if palindrome centered at i expands past right,
            // adjust center based on expanded palindrome.
            if (i + p[i] > right) {
                center = i;
                right = i + p[i];
            }
        }
        return longestPalindromicSubstring(p, s);
    }

    private String longestPalindromicSubstring(int[] p, String s) {
        int length = 0; // length of longest palindromic substring
        int center = 0; // center of longest palindromic substring
        for (int i = 1; i < p.length - 1; i++) {
            if (p[i] > length) {
                length = p[i];
                center = i;
            }
        }
        return s.substring((center - 1 - length) / 2, (center - 1 + length) / 2);
    }

    // Transform s into t.
    // For example, if s = "abba", then t = "$#a#b#b#a#@"
    // the # are interleaved to avoid even/odd-length palindromes uniformly
    // $ and @ are prepended and appended to each end to avoid bounds checking
    private char[] preprocess(String s) {
        char[] t = new char[s.length() * 2 + 3];
        t[0] = '$';
        t[s.length() * 2 + 2] = '@';
        for (int i = 0; i < s.length(); i++) {
            t[2 * i + 1] = '#';
            t[2 * i + 2] = s.charAt(i);
        }
        t[s.length() * 2 + 1] = '#';
        return t;
    }

}
```

## Test Cases

```java
class SolutionTest {

    private Solution solution;

    static Collection<Object[]> data() {
        return Arrays.asList(
                new Object[][]{
                        {"aabcdcb", "bcdcb"},
                        {"bananas", "anana"},
                }
        );
    }

    @BeforeEach
    void setUp() {
        this.solution = new Solution();
    }


    @ParameterizedTest
    @MethodSource("data")
    void testSolution(String input, String expected) {
        String actual = this.solution.longestPalindrome(input);

        assertEquals(expected, actual);

    }

}
```

## Analysis

- Time Complexity: O(n)
- Space Complexity: O(n)
