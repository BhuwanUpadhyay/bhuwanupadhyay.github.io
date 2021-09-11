---
title: Implement a URL shortener with alphanumeric string
date: 2020-03-29 02:11:00 Z
categories: [CodingProblems]
tags: [alphanumeric-random, algorithms]
cover: /images/coding-problems.png
---

Implement a URL shortener with six-character alphanumeric string.

<!-- more -->

## Example

Implement a URL shortener with the following methods:

- `shorten(url)`, which shortens the url into a six-character alphanumeric string, such as `zLg6wl`.
- `restore(short)`, which expands the shortened string into the original url.
- If no such shortened string exists, return `null`.

<span class="text-primary">
<i class="fa fa-info mr-1"></i>
</span>
What if we enter the same URL twice?

## Solution

```java
class Solution {

    public static final int MAX = 6;
    private final Map<URL, URL> urlCache = new HashMap<>();
    private final AlphanumericRandomizer randomizer = new AlphanumericRandomizer();

    URL shorten(String url) {
        try {
            URL longUrl = URI.create(url).toURL();
            String shortValue = randomizer.next(MAX);
            URL shortUrl = new URL(longUrl.getProtocol(), longUrl.getHost(), "/" + shortValue);
            this.urlCache.put(shortUrl, longUrl);
            return shortUrl;
        } catch (MalformedURLException e) {
            throw new RuntimeException(e);
        }
    }

    URL restore(URL shortValue) {
        return this.urlCache.get(shortValue);
    }

    /**
     * Alphanumeric characters are A to Z, a to z and 0 to 9
     */
    static class AlphanumericRandomizer {

        private static final String CHARS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        private final Random r;
        private final String alphaNumeric;

        public AlphanumericRandomizer() {
            r = new Random();
            this.alphaNumeric = CHARS + CHARS.toLowerCase() + "0123456789";
        }

        public String next(int size) {
            final int length = this.alphaNumeric.length();
            return IntStream.range(0, size)
                    .mapToObj(value -> r.nextInt(length))
                    .map(index -> String.valueOf(alphaNumeric.charAt(index)))
                    .collect(Collectors.joining());
        }

    }
}
```

## Test Cases

```java
class SolutionTest {

    private Solution solution;

    static Collection<Object[]> data() {
        return Arrays.asList(new Object[][]{
                {"http://hello.com/test", "http", "hello.com", "/[A-Za-z0-9]{6}"},
                {"http://hello.com/test", "http", "hello.com", "/[A-Za-z0-9]{6}"},
                {"https://hello.com/test", "https", "hello.com", "/[A-Za-z0-9]{6}"},
        });
    }

    @BeforeEach
    void setUp() {
        this.solution = new Solution();
    }

    @ParameterizedTest
    @MethodSource("data")
    void testSolution(String url, String expectedProtocol, String expectedHost, String expectedPathRegEx) {
        URL actual = this.solution.shorten(url);
        assertEquals(expectedProtocol, actual.getProtocol());
        assertEquals(expectedHost, actual.getHost());
        assertTrue(actual.getPath().matches(expectedPathRegEx), "The shorten url " + actual + " is not six characters alphanumeric.");
    }

    @ParameterizedTest
    @MethodSource("data")
    void testSolution(String url) throws MalformedURLException {
        URL shorten = this.solution.shorten(url);

        URL actual = this.solution.restore(shorten);

        URL expected = URI.create(url).toURL();
        assertEquals(expected, actual);
    }

}
```
