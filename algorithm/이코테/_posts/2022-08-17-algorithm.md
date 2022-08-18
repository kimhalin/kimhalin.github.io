---
layout: post
title: "[이코테] 큰 수의 법칙"
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.

categories: [algorithm, 이코테]
tags:       [algorithm, 이코테]
sitemap: false
hide_last_modified: true
---
큰 수의 법칙 - 이코테
Chapter 2 - Greedy 알고리즘





## 큰 수의 법칙

동빈이의 큰 수의 법칙에 따라 더해진 답을 출력하자!

동빈이의 큰 수의 법칙은 다양한 수로 이루어진 배열이 있을 때, 주어진 수들을 M번 더하여 가장 큰 수를 만드는 법칙이다. 단, 배열의 특정한 인덱스에 해당하는 수가 연속해서 K번을 초과하여 더해질 수 없다.

단, 서로 다른 인덱스에 해당하는 수가 같은 경우에도 서로 다른 것으로 간주 !!

💡 예시
배열: 2, 4, 5, 4, 6
M: 8
K: 3
6 + 6 + 6 + 5 + 6 + 6 + 6 + 5 = 46


아래는 처음에 문제를 보고 쌩으로 작성해본 코드이다. 

처음에는 살짝 헤매다가 나중에는 결국에는 제일 큰 수랑 그 다음 큰 수(제일 큰 수랑 같을 수도 있음) 수 2개만 사용한다는 것을 깨달았다.

그래서 내가 짠 코드는

- 입력받은 배열을 정렬(오름차순)
- M번만큼 for문을 돌게 만들기
    - 배열 가장 끝 부분에 있는 수를 먼저 K번 더하기
    - K번 더한 후에는 index를 앞으로 한 칸 옮겨서 그 다음 큰 수를 더하기, 이때 그 다음 큰 수가 제일 큰 수와 값이 같다면 K번을 더하고, 아니라면 한 번 더하기
        
        (근데 사실 이렇게 안하고 그냥 무조건 한 번만 더했어도 됐다 ,,,)
        

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int N = scanner.nextInt();
    int M = scanner.nextInt();
    int K = scanner.nextInt();
    int[] arr = new int[N];

    for (int i = 0; i < N; i++) {
        arr[i] = scanner.nextInt();
    }
    Arrays.sort(arr);

    int sum = 0;
    int countK = 0;
    int index = arr.length - 1;
    int max = arr[index];
    for (int i = 0; i < M; i++) {
        if(countK >= K){
            countK = 0;
            if(index >= arr.length - 1)
                index--;
            else
                index++;
        }
        sum += arr[index];
        if(arr[index] < max)
            countK = K;
        else
            ++countK;
    }
    System.out.println(sum);
}
```

## 핵심 아이디어

이 문제의 핵심 아이디어는

> ‘가장 큰 수를 K번 더하고 두 번째로 큰 수를 한 번 더하는 연산'을 반복하면 되는 것
> 

책에는 파이썬으로 해설이 나와 있지만, 참고해서 내가 짰던 코드를 수정해 보았다. 코드가 깔끔해졌다. 

애초에 가장 큰 수랑 두 번째로 큰 수만 사용하기 때문에 그 수들만 따로 빼서 계산하면 된다.

- while true로 반복문을 실행(입력 받았던 M이 0이 될 떄까지)
    - 가장 큰 수를 k번 반복해서 더한 후,
    - 두 번째로 큰 수를 1번 더한다.
    - 수를 더할 때 마다 M을 1씩 감소시킨다

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int N = scanner.nextInt();
    int M = scanner.nextInt();
    int K = scanner.nextInt();
    int[] arr = new int[N];

    for (int i = 0; i < N; i++) {
        arr[i] = scanner.nextInt();
    }
    Arrays.sort(arr);

    int first = arr[N - 1];
    int second = arr[N - 2];
    int sum = 0;

    while (true) {
        for(int i = 0; i < K; ++i){
            if(M == 0)
                break;
            sum += first;
            M--;
        }
        if(M == 0)
            break;
        sum += second;
        M--;
    }
    System.out.println("sum = " + sum);
}
```

## 반복되는 수열 파악

가장 큰 수와 두 번째로 큰 수가 더해질 때는 특정한 수열 형태로 일정하게 반복해서 더해지는 특징이 있다. 위의 예시에서는 수열 [6, 6, 6, 5]가 반복된다. 

반복되는 수열의 길이는? 바로 K + 1

따라서 가장 큰 수가 등장하는 횟수는 (M / K + 1) * K이고,

만약 M이 나누어 떨어지지 않는다면? (M / K + 1) * K + **M % (K + 1)**

이것을 활용해 다시 코드를 짜보자

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int N = scanner.nextInt();
    int M = scanner.nextInt();
    int K = scanner.nextInt();
    int[] arr = new int[N];

    for (int i = 0; i < N; i++) {
        arr[i] = scanner.nextInt();
    }
    Arrays.sort(arr);

    int first = arr[N - 1];
    int second = arr[N - 2];

    int count = (M / (K + 1)) * K;
    count += M % (K + 1);

    int sum = 0;
    sum += (count) * first;
    sum += (M - count) * second;
    System.out.println(sum);
}
```

훨씬 코드가 짧아졌고, 아마 실행시간도 짧을 것이다. M번만큼 반복문을 돌던 것을 이제 반복문 없이 처리가 가능하므로!