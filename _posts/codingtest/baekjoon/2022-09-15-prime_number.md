---
title: "2023번 신기한 소수 (Java)"
categories: 
    - baekjoon
date: 2022-09-15
last_modified_at: 2022-09-15
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "2023번 신기한 소수 (Java)"
---
## 문제 설명

수빈이가 세상에서 가장 좋아하는 것은 소수이고, 취미는 소수를 가지고 노는 것이다. 요즘 수빈이가 가장 관심있어 하는 소수는 7331이다.

7331은 소수인데, 신기하게도 733도 소수이고, 73도 소수이고, 7도 소수이다. 즉, 왼쪽부터 1자리, 2자리, 3자리, 4자리 수 모두 소수이다! 수빈이는 이런 숫자를 신기한 소수라고 이름 붙였다.

수빈이는 N자리의 숫자 중에서 어떤 수들이 신기한 소수인지 궁금해졌다. N이 주어졌을 때, 수빈이를 위해 N자리 신기한 소수를 모두 찾아보자.

## 제한 조건

- 시간제한 : 2초
- 메모리제한 : 4MB

## 입력 설명

첫째 줄에 N(1 ≤ N ≤ 8)이 주어진다.

## 출력 설명

N자리 수 중에서 신기한 소수를 오름차순으로 정렬해서 한 줄에 하나씩 출력한다.

## 예제 입력 및 출력1

    4
<hr>

    2333
    2339
    2393
    2399
    2939
    3119
    3137
    3733
    3739
    3793
    3797
    5939
    7193
    7331
    7333
    7393

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    static int maxDigit;
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());

        maxDigit = Integer.parseInt(st.nextToken());

        // 1의 자릿수에서 소수인 숫자는 2, 3, 5, 7 이므로 먼저 진행
        dfs(2,1);
        dfs(3,1);
        dfs(5,1);
        dfs(7,1);
    }

    static void dfs(int number, int digit){
        if(digit == maxDigit) {
            if(isPrimeNumber(number)) System.out.println(number);
            return;
        }
        
        // 자릿수를 늘리면서(number * 10) 1부터 9(i)까지 확인
        // 뒤에 추가되는 숫자가 짝수가 아니면서 (소수여야함)
        // 다음 비교할 숫자가 소수면 dfs 탐색
        for(int i=1; i<=9; i++){
            if(i % 2 != 0 && isPrimeNumber(number * 10 + i))
                dfs(number * 10 + i, digit + 1);
        }
    }

    // 소수 인지 판별
    static boolean isPrimeNumber(int number){
        for(int i=2; i<number; i++){
            if(number % i == 0) return false;
        }
        return true;
    }
}
```