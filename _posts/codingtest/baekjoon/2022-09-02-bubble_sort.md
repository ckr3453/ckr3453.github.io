---
title: "1377번 버블 소트 (Java)"
categories: 
    - baekjoon
date: 2022-09-02
last_modified_at: 2022-09-02
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "1377번 버블 소트 (Java)"
---
## 문제 설명

버블 소트 알고리즘을 다음과 같이 C++로 작성했다.

    bool changed = false;
    for (int i=1; i<=N+1; i++) {
        changed = false;
        for (int j=1; j<=N-i; j++) {
            if (A[j] > A[j+1]) {
                changed = true;
                swap(A[j], A[j+1]);
            }
        }
        if (changed == false) {
            cout << i << '\n';
            break;
        }
    }

위 소스에서 N은 배열의 크기이고, A는 정렬해야 하는 배열이다. 배열은 A[1]부터 사용한다.

위와 같은 소스를 실행시켰을 때, 어떤 값이 출력되는지 구해보자.

## 제한 조건

- 시간제한 : 2초
- 메모리제한 : 128MB

## 입력 설명

첫째 줄에 N이 주어진다. N은 500,000보다 작거나 같은 자연수이다. 둘째 줄부터 N개의 줄에 A[1]부터 A[N]까지 하나씩 주어진다. A에 들어있는 수는 1,000,000보다 작거나 같은 자연수 또는 0이다.

## 출력 설명

정답을 출력한다.

## 예제 입력 및 출력1

    5
    10
    1
    5
    2
    3
<hr>

    3

## 예제 입력 및 출력2

    5
    1
    3
    5
    7
    9
<hr>

    1

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int arrLength = Integer.parseInt(br.readLine());
        // 값이 중복될수있어서 map은 사용불가
        // comparable을 상속받은 클래스를 활용하여 정렬 조건을 커스텀
        // Map<Integer, Integer> map = new HashMap<>(); 
        Data[] arr = new Data[arrLength];
        
        for(int i=0; i<arrLength; i++){
            arr[i] = new Data(i, Integer.parseInt(br.readLine()));
        }
        
        Arrays.sort(arr);
        int max = 0;
        for(int i=0; i<arrLength; i++){
            max = Math.max(max, arr[i].index-i);
        }
        
        // 정답은 인덱스가 아닌 횟수이므로 + 1
        System.out.println(max + 1);
    }
    
    static class Data implements Comparable<Data> {
        int index;
        int value;
        
        public Data(int index, int value){
            this.index = index;
            this.value = value;
        }
        
        @Override
        public int compareTo(Data d){
            return this.value - d.value;
        }
    }
}
```
