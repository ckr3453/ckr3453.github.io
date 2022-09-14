---
title: "1874번 스택 수열 (Java)"
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
excerpt: "1874번 스택 수열 (Java)"
---
## 문제 설명

스택 (stack)은 기본적인 자료구조 중 하나로, 컴퓨터 프로그램을 작성할 때 자주 이용되는 개념이다. 스택은 자료를 넣는 (push) 입구와 자료를 뽑는 (pop) 입구가 같아 제일 나중에 들어간 자료가 제일 먼저 나오는 (LIFO, Last in First out) 특성을 가지고 있다.

1부터 n까지의 수를 스택에 넣었다가 뽑아 늘어놓음으로써, 하나의 수열을 만들 수 있다. 이때, 스택에 push하는 순서는 반드시 오름차순을 지키도록 한다고 하자. 임의의 수열이 주어졌을 때 스택을 이용해 그 수열을 만들 수 있는지 없는지, 있다면 어떤 순서로 push와 pop 연산을 수행해야 하는지를 알아낼 수 있다. 이를 계산하는 프로그램을 작성하라.

## 제한 조건

- 시간제한 : 2초
- 메모리제한 : 128MB

## 입력 설명

첫 줄에 n (1 ≤ n ≤ 100,000)이 주어진다. 둘째 줄부터 n개의 줄에는 수열을 이루는 1이상 n이하의 정수가 하나씩 순서대로 주어진다. 물론 같은 정수가 두 번 나오는 일은 없다.

## 출력 설명

입력된 수열을 만들기 위해 필요한 연산을 한 줄에 한 개씩 출력한다. push연산은 +로, pop 연산은 -로 표현하도록 한다. 불가능한 경우 NO를 출력한다.

## 예제 입력 및 출력1

1부터 n까지에 수에 대해 차례로 [push, push, push, push, pop, pop, push, push, pop, push, push, pop, pop, pop, pop, pop] 연산을 수행하면 수열 [4, 3, 6, 8, 7, 5, 2, 1]을 얻을 수 있다.

    8
    4
    3
    6
    8
    7
    5
    2
    1
<hr>

    +
    +
    +
    +
    -
    -
    +
    +
    -
    +
    +
    -
    -
    -
    -
    -

## 예제 입력 및 출력2

    5
    1
    2
    5
    3
    4
<hr>

    NO

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Stack;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int arrLength = Integer.parseInt(br.readLine());
        int[] numArr = new int[arrLength];
        Stack<Integer> stack = new Stack<>();
        StringBuilder sb = new StringBuilder();
        
        for(int i=0; i<arrLength; i++){
            numArr[i] = Integer.parseInt(br.readLine());
        }
        
        int inputNum = 1;
        for(int i=0; i<arrLength; i++){
            int targetNum = numArr[i];
            if(targetNum >= inputNum){
                while(targetNum >= inputNum){
                    stack.push(inputNum);
                    sb.append("+\n");
                    inputNum++;
                }
                stack.pop();
                sb.append("-\n");
            } else {
                int n = stack.pop();
                if(n != targetNum){
                    sb.setLength(0);
                    sb.append("NO");
                    break;
                } else {
                    sb.append("-\n");
                }
            }
        }
        
        System.out.println(sb.toString());
    }
}
```

1씩 증가하는 자연수를 입력받은 수열과 값이 동일해질때까지 스택에 push(+)하며 비교한다.

자연수(top)가 수열보다 클 경우 pop(-)한다.

이때 pop한 자연수가 수열과 같지않은경우 스택으로 표현할 수 없는 수열이므로 NO를 출력하며 종료한다.
