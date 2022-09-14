---
title: "10989번 수 정렬하기3 (Java)"
categories: 
    - baekjoon
date: 2022-09-07
last_modified_at: 2022-09-07
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "10989번 수 정렬하기3 (Java)"
---
## 문제 설명

N개의 수가 주어졌을 때, 이를 오름차순으로 정렬하는 프로그램을 작성하시오.

## 제한 조건

- 시간제한 : 5초 (Java 11의 경우 3초)
- 메모리제한 : 8MB (Java 11의 경우 512MB)

## 입력 설명

첫째 줄에 수의 개수 N(1 ≤ N ≤ 10,000,000)이 주어진다. 둘째 줄부터 N개의 줄에는 수가 주어진다. 이 수는 10,000보다 작거나 같은 자연수이다.

## 출력 설명

첫째 줄부터 N개의 줄에 오름차순으로 정렬한 결과를 한 줄에 하나씩 출력한다.

## 예제 입력 및 출력1

    10
    5
    2
    3
    1
    4
    2
    3
    5
    1
    7
<hr>

    1
    1
    2
    2
    3
    3
    4
    5
    5
    7

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class Main {
    public static long result;
    
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        
        int arrLength = Integer.parseInt(br.readLine());
        int[] arr = new int[arrLength];
        
        for(int i=0; i<arrLength; i++){
            arr[i] = Integer.parseInt(br.readLine());
        }
        
        // 주어지는 숫자는 10000 보다 작거나 같으므로 5 자리까지
        radixSort(arr, 5);
        
        for(int i=0; i<arrLength; i++){
            bw.write(arr[i]+"\n");
        }
        
        bw.flush();
        bw.close();
        br.close();
    }
    
    public static void radixSort(int[] target, int maxDigit){
        int[] temp = new int[target.length];
        int digit = 1;
        int count = 0;
        
        while(count != maxDigit){
            int[] bucket = new int[10];    // 자릿수 별 0~9 까지 저장
            
            for(int i=0; i<target.length; i++){
                // (대상숫자 / 자릿수) % 10 = 해당 자릿수의 0~9 사이 숫자 return
                // 자릿수 에 해당하는 숫자들을 count
                int idx = target[i] / digit % 10;
                bucket[idx]++;
            }
            
            // 누적합 정렬로 변환
            // 갯수를 누적합 한것이므로 각 배열의 값은 원래 배열길이의 값을 넘을 수 없다.
            // 결과값이 들어갈 위치를 미리 계산하는 셈
            for(int i=1; i<bucket.length; i++){
                bucket[i] += bucket[i-1];
            }
            
            for(int i=target.length-1; i>=0; i--){
                int idx = target[i] / digit % 10;    // 자릿수
                temp[bucket[idx] - 1] = target[i];    // 인덱스로 접근해야하므로 -1
                bucket[idx]--;    // 같은 자릿수가 여러개 나오더라도 해당 자릿수의 값을 줄여서 저장하여 문제되지않음
            }
            
            for(int i=0; i<target.length; i++){
                target[i] = temp[i];
            }
            
            digit *= 10;    // 자릿수를 증가하여 다음 loop 대비
            count++;
        }
    }
}
```
데이터의 길이와 각 데이터의 범위를 봤을 때 내장 라이브러리의 sort를 이용시 아슬아슬하게 통과 혹은 불가능하다.

따라서 상대적으로 빠른 시간복잡도를 가진 radix sort로 구현 (radix sort는 O(kn), k=데이터의 최대 자릿수)

radix sort는 O(nlogn) 시간 복잡도를 가진 merge sort보다 빠르다. 

대신 데이터 타입이 일정해야하고 데이터의 자릿수를 저장하는 bucket의 메모리가 추가적으로 소모된다.
