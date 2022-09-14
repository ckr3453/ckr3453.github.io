---
title: "11003번 최솟값 찾기 (Java)"
categories: 
    - baekjoon
date: 2022-08-26
last_modified_at: 2022-08-26
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "11003번 최솟값 찾기 (Java)"
---
## 문제 설명

N개의 수 A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>N</sub>과 L이 주어진다.

D<sub>i</sub> = A<sub>i-L+1</sub> ~ A<sub>i</sub> 중의 최솟값이라고 할 때, D에 저장된 수를 출력하는 프로그램을 작성하시오. 이때, i ≤ 0 인 A<sub>i</sub>는 무시하고 D를 구해야 한다.

## 제한 조건

- 시간제한 : 2.4초 (java 11은 2.6초)
- 메모리제한 : 512MB

## 입력 설명

첫째 줄에 N과 L이 주어진다. (1 ≤ L ≤ N ≤ 5,000,000)

둘째 줄에는 N개의 수 A<sub>i</sub>가 주어진다. (-10<sup>9</sup> ≤ A<sub>i</sub> ≤ 10<sup>9</sup>)

## 출력 설명

첫째 줄에 D<sub>i</sub>를 공백으로 구분하여 순서대로 출력한다.

## 예제 입력 및 출력 1

    12 3
    1 5 2 3 6 2 3 7 3 5 2 6
<hr>

    1 1 1 2 2 2 2 2 3 3 2 2

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.util.StringTokenizer;
import java.util.LinkedList;
import java.util.Deque;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        
        StringTokenizer st = new StringTokenizer(br.readLine());
        int numLength = Integer.parseInt(st.nextToken());
        int L = Integer.parseInt(st.nextToken());
        st = new StringTokenizer(br.readLine());
        Deque<Node> deque = new LinkedList<>();
        
        for(int i=0; i<numLength; i++){
            int currentValue = Integer.parseInt(st.nextToken());
            
            // 덱의 마지막 값이 새로운 값보다 크면 삭제 (오름차순 정렬 역할)
            while(!deque.isEmpty() && deque.getLast().value > currentValue){
                deque.removeLast();
            }
            deque.addLast(new Node(currentValue, i));
            
            // 인덱스 범위에서 벗어난 값 제거
            if(deque.getFirst().index <= i - L) deque.removeFirst();
            bw.write(deque.getFirst().value+" ");
        }
        
        br.close();
        bw.flush();
        bw.close();
    }
    
    static class Node {
        int value;
        int index;
        
        Node(int value, int index){
            this.value = value;
            this.index = index;
        }
    }
}
```

새로운 값과 Deque의 last값 비교를 통해 Deque의 first에 최소값이 오도록 유지한다. 

이때 first의 인덱스가 정해진 범위를 벗어나면 제거한다. (슬라이딩 윈도우)

양방향 큐 구조인 Deque를 활용하여 제한된 범위(A<sub>i-L+1</sub> ~ A<sub>i</sub>) 내에서 슬라이딩 윈도우 알고리즘을 적용하면 풀 수 있다.