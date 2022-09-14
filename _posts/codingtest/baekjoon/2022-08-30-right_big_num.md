---
title: "17298번 오큰수 (Java)"
categories: 
    - baekjoon
date: 2022-08-30
last_modified_at: 2022-08-30
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "17298번 오큰수 (Java)"
---
## 문제 설명

크기가 N인 수열 A = A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>N</sub>이 있다. 수열의 각 원소 Ai에 대해서 오큰수 NGE(i)를 구하려고 한다. Ai의 오큰수는 오른쪽에 있으면서 Ai보다 큰 수 중에서 가장 왼쪽에 있는 수를 의미한다. 그러한 수가 없는 경우에 오큰수는 -1이다.

예를 들어, A = [3, 5, 2, 7]인 경우 NGE(1) = 5, NGE(2) = 7, NGE(3) = 7, NGE(4) = -1이다. A = [9, 5, 4, 8]인 경우에는 NGE(1) = -1, NGE(2) = 8, NGE(3) = 8, NGE(4) = -1이다.

## 제한 조건

- 시간제한 : 1초
- 메모리제한 : 512MB

## 입력 설명

첫째 줄에 수열 A의 크기 N (1 ≤ N ≤ 1,000,000)이 주어진다. 둘째 줄에 수열 A의 원소 A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>N</sub> (1 ≤ A<sub>i</sub> ≤ 1,000,000)이 주어진다.

## 출력 설명

총 N개의 수 NGE(1), NGE(2), ..., NGE(N)을 공백으로 구분해 출력한다.

## 예제 입력 및 출력1

    4
    3 5 2 7
<hr>

    5 7 7 -1

## 예제 입력 및 출력2

    4
    9 5 4 8
<hr>

    -1 8 8 -1

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.BufferedWriter;
import java.io.OutputStreamWriter;
import java.util.StringTokenizer;
import java.util.Stack;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int arrLength = Integer.parseInt(br.readLine());
        int[] numArr = new int[arrLength];
        StringTokenizer st = new StringTokenizer(br.readLine());
        
        for(int i=0; i<arrLength; i++){
            numArr[i] = Integer.parseInt(st.nextToken());
        }
        
        Stack<Integer> stack = new Stack<>();
        int[] answerArr = new int[arrLength];
        
        for(int i=0; i<arrLength; i++){
            if (!stack.isEmpty()) {
                while (!stack.isEmpty() && numArr[stack.peek()] < numArr[i]) {
                    // 스택 top에 있는 numArr의 index를 기준으로 우측 값과 비교
                    // 우측 값이 클 경우 오큰수
                    answerArr[stack.pop()] = numArr[i];
                }
            }
            // 다음 값 비교를 위해 스택에 push
            stack.push(i);
        }

        while(!stack.isEmpty()){
            // 스택에 남아있는 index는 오큰수가 없다는 의미
            answerArr[stack.pop()] = -1;
        }

        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        for (int j : answerArr) {
            bw.write(j + " ");
        }
        bw.flush();
    }
}
```
