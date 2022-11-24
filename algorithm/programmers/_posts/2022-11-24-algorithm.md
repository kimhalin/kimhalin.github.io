---
layout: post
title: "[programmers] 정수 제곱근 판별"
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.

categories: [algorithm, programmers]
tags:       [algorithm, programmers]
sitemap: false
hide_last_modified: true
---
정수 제곱근 판별






임의의 양의 정수 n에 대해, n이 어떤 양의 정수 x의 제곱인지 아닌지 판단하려 합니다.n이 양의 정수 x의 제곱이라면 x+1의 제곱을 리턴하고, n이 양의 정수 x의 제곱이 아니라면 -1을 리턴하는 함수를 완성하세요.

### 제한 사항

- n은 1이상, 50000000000000 이하인 양의 정수입니다.

### 입출력 예

| n | return |
| --- | --- |
| 121 | 144 |
| 3 | -1 |

### ✏️ 나의 풀이

```java
class Solution {
    public long solution(long n) {
        for (long i = 1; i * i <= n; i++) {
            if(i * i == n) {
                return (i + 1) * (i+1);
            }
        }
        return -1;
    }
}
```

처음에는 아무 생각 없이 for문에 사용되는 i를 int형으로 선언했었는데, 시간 초과가 났다.. 바로 long으로 바꾸니 통과!! 형변환 시간이 길어서 시간초과가 났나보다.

### 💡다른 풀이

```java
class Solution {
  public long solution(long n) {
      if (Math.pow((int)Math.sqrt(n), 2) == n) {
            return (long) Math.pow(Math.sqrt(n) + 1, 2);
        }

        return -1;
  }
}
```

**Math**를 사용하니 한 번에 해결 👀

Math.sqrt도 double형이라 int 변환은 괜찮다.

`**Math.pow` 에 대해**

특정 값의 제곱을 구하기 위해 사용되는 함수이다.

```java
Math.pow(5, 2) // 5의 제곱인 25 return (double)
```