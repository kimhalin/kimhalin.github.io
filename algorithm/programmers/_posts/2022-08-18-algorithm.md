---
layout: post
title: "[programmers] 숫자 문자열과 영단어"
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.

categories: [algorithm, programmers]
tags:       [algorithm, programmers]
sitemap: false
hide_last_modified: true
---
숫자 문자열과 영단어






## 문제

숫자의 일부 자릿수를 영단어로 바뀌어졌거나, 혹은 바뀌지 않고 그대로인 문자열 s가 매개변수로 주어지고, s가 의미하는 원래 숫자를 return 하도록 만들어라!

```java
public int solution(String s) {
    String [] english = {"zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine"};
    for (int i = 0; i < english.length; ++i) {
        s = s.replace(english[i], Integer.toString(i));
    }
    int answer = 0;
    answer = Integer.parseInt(s);
    return answer;
}
```

처음에는 영단어로 바뀐 부분을 하나 하나 찾아서 index 값을 받은 다음, 그 이후에 바꿔주려고 했다.. ~~멍청한 생각~~

근데 `replace` 라는 함수가 있다는 것을 깨닫고 바로 replace함수를 도입해 짧은 코드 완성!

그래서 제출 후 채점해보니, 통과라고 했지만,,,,

다른 사람 코드를 보니 `replaceAll`  이라는 함수를 사용하고 있었다.

생각해보니 replace는 하나만 바꿔주므로 만약 `1two32two` 이런 숫자가 있었으면 `1232two` 이렇게 밖에 바뀌지 않았을 것이다 ..!!

프로그래머스 테스트 왜 허술해 ..

암튼 그래서 다시 replaceAll로 바꿔치기해서 제출 성공 ~ ~ ~

## My code

- 숫자를 영단어로 바꾼 String 배열을 만들어 놓는다
- 그리고 String 배열길이만큼 for문을 돌면서 replaceAll함수를 실행해 숫자로 바꿔치기 해준다.