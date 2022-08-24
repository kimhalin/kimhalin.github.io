---
layout: post
title: "[programmers] 성격 유형 검사하기"
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.

categories: [algorithm, programmers]
tags:       [algorithm, programmers]
sitemap: false
hide_last_modified: true
---
성격 유형 검사하기






## 문제

나만의 카카오 성격 유형 검사지를 만들려고 합니다.성격 유형 검사는 다음과 같은 4개 지표로 성격 유형을 구분합니다. 성격은 각 지표에서 두 유형 중 하나로 결정됩니다.

라이언형(R), 튜브형(T) / 콘형(C), 프로도형(F) / 제이지형(J), 무지형(M) / 어피치형(A), 네오형(N)

검사지에는 총 `n`개의 질문이 있고, 각 질문에는 아래와 같은 7개의 선택지가 있습니다.

- `매우 비동의` = 1
- `비동의` = 2
- `약간 비동의` = 3
- `모르겠음` = 4
- `약간 동의` = 5
- `동의` = 6
- `매우 동의` = 7

각 질문은 1가지 지표로 성격 유형 점수를 판단합니다.

각 선택지는 고정적인 크기의 점수를 가지고 있습니다.

- `매우 동의`나 `매우 비동의` 선택지를 선택하면 3점을 얻습니다.
- `동의`나 `비동의` 선택지를 선택하면 2점을 얻습니다.
- `약간 동의`나 `약간 비동의` 선택지를 선택하면 1점을 얻습니다
- `모르겠음` 선택지를 선택하면 점수를 얻지 않습니다.

검사 결과는 모든 질문의 성격 유형 점수를 더하여 각 지표에서 더 높은 점수를 받은 성격 유형이 검사자의 성격 유형이라고 판단

단, 하나의 지표에서 각 성격 유형 점수가 같으면, 두 성격 유형 중 사전 순으로 빠른 성격 유형을 검사자의 성격 유형이라고 판단

질문마다 판단하는 지표를 담은 1차원 문자열 배열 `survey`

검사자가 각 질문마다 선택한 선택지를 담은 1차원 정수 배열 `choices`가 매개변수

---

### 제한사항

- 1 ≤ `survey`의 길이 ( = `n`) ≤ 1,000
    - `survey`의 원소는 `"RT", "TR", "FC", "CF", "MJ", "JM", "AN", "NA"` 중 하나입니다.
    - `survey[i]`의 첫 번째 캐릭터는 i+1번 질문의 비동의 관련 선택지를 선택하면 받는 성격 유형을 의미합니다.
    - `survey[i]`의 두 번째 캐릭터는 i+1번 질문의 동의 관련 선택지를 선택하면 받는 성격 유형을 의미합니다.
- `choices`의 길이 = `survey`의 길이
    - `choices[i]`는 검사자가 선택한 i+1번째 질문의 선택지를 의미합니다.
    - 1 ≤ `choices`의 원소 ≤ 7

## My Code

- 성격유형 String 배열 mbti, 각 유형의 점수를 저장할 HashMap map
- for 문을 실행하며 각 유형별 점수 더하기
- mbti 배열을 두 칸씩 보면서 get으로 map에 저장되어 있던 유형의 점수 비교 후 알맞은 유형을 answer에 추가

```java
public static String solution(String[] survey, int[] choices) {
    String answer = "";
    String[] mbti = {"R", "T", "C", "F", "J", "M", "A", "N"};
    HashMap<String, Integer> map = new HashMap<>();
    map.put("R", 0);
    map.put("T", 0);
    map.put("C", 0);
    map.put("F", 0);
    map.put("J", 0);
    map.put("M", 0);
    map.put("A", 0);
    map.put("N", 0);

    for (int i = 0; i < survey.length; i++) {
        // "모르겠음" 은 그냥 넘어가기
        if(choices[i] == 4)
            continue;

        // "AN" -> ["A", "N"]
        String[] strArr = survey[i].split("");

        if (choices[i] < 4) {
            map.replace(strArr[0], map.get(strArr[0]) + (4 - choices[i]));
        } else {
            map.replace(strArr[1], map.get(strArr[1]) + (choices[i]) - 4);
        }
    }

    for (int i = 0; i < mbti.length; i += 2) {
        String subStr = mbti[i];
        if(map.get(mbti[i]) < map.get(mbti[i + 1])){
            subStr = mbti[i + 1];
        }

        answer += subStr;
    }

    return answer;
}
```

다른 사람 풀이 딱히 좋아 보이는 게 안 보여서 생략 !