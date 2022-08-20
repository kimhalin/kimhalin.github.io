---
layout: post
title: "[programmers] 수포자 수학 문제 찍기"
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.

categories: [algorithm, programmers]
tags:       [algorithm, programmers]
sitemap: false
hide_last_modified: true
---
수포자 수학 문제 찍기






## 문제

1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...

1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.

### 제한 조건

- 시험은 최대 10,000 문제로 구성되어있습니다.
- 문제의 정답은 1, 2, 3, 4, 5중 하나입니다.
- 가장 높은 점수를 받은 사람이 여럿일 경우, return하는 값을 오름차순 정렬해주세요.

```java
public int[] solution(int[] answers) {
    List<Integer> answerList = new ArrayList<Integer>();
    int [] count = {0,0,0};
    int [][] rule = {{1,2,3,4,5}, {2,1,2,3,2,4,2,5}, {3, 3, 1, 1, 2, 2, 4, 4, 5, 5}};
    int [] index = {0, 0, 0};
    int [] answer = {};
    for (int i =0; i< answers.length; i++) {
        for(int j =0; j< 3; j++){
            if(answers[i] == rule[j][index[j]])
                count[j]++;
            if(index[j] >= rule[j].length - 1)
                index[j] = 0;
            else
                index[j]++;
        }
    }
    int [] temp = count.clone();
    Arrays.sort(temp);
    for (int i = 0; i < 3; i++) {
        if (temp[2] == count[i])
            answerList.add(i+1);
    }
    answer = answerList.stream().mapToInt(i->i).toArray();
    return answer;
}
```

## My Code

누가 더 많이 맞췄는지에 대한 배열 count, 수포자들의 찍기 룰인 rule 배열, 그리고 rule배열 인덱스를 저장할 index 이렇게 선언한다.

- for문을 입력으로 받은 answers의 길이만큼 돌면서, 각 수포자들의 찍은 답과 비교해 답을 맞췄는지 본다.
    
    맞췄을 시 count 배열 값 증가
    
- count 배열을 복사해 정렬하여 최대값을 구한다.
- 최대값과 같은 값을 가지고 있는 count배열의 인덱스값을 ArrayList인 answerList에 add한다.
- ArrayList를 list로 변환해 준 후 리턴한다!

다른 사람들 코드를 봤는데 다 거기서 거기인 것 같아 생략!