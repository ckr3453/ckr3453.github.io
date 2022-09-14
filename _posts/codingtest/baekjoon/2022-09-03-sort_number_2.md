---
title: "2751번 수 정렬하기 2 (Java)"
categories: 
    - baekjoon
date: 2022-09-03
last_modified_at: 2022-09-03
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "2751번 수 정렬하기 2 (Java)"
---
## 문제 설명

N개의 수가 주어졌을 때, 이를 오름차순으로 정렬하는 프로그램을 작성하시오.

## 제한 조건

- 시간제한 : 2초
- 메모리제한 : 256MB

## 입력 설명

첫째 줄에 수의 개수 N(1 ≤ N ≤ 1,000,000)이 주어진다. 둘째 줄부터 N개의 줄에는 수가 주어진다. 이 수는 절댓값이 1,000,000보다 작거나 같은 정수이다. 수는 중복되지 않는다.

## 출력 설명

첫째 줄부터 N개의 줄에 오름차순으로 정렬한 결과를 한 줄에 하나씩 출력한다.

## 예제 입력 및 출력1

    5
    5
    4
    3
    2
    1
<hr>

    1
    2
    3
    4
    5

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class Main {
    
    public static int[] arr, tmpArr;
    public static long result;
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        
        int arrLength = Integer.parseInt(br.readLine());
        arr = new int[arrLength+1];
        tmpArr = new int[arrLength+1];
        
        for(int i=1; i <= arrLength; i++){
            arr[i] = Integer.parseInt(br.readLine());
        }
        
        mergeSort(1, arrLength);
        
        for(int i=1; i <= arrLength; i++){
            bw.write(arr[i] + "\n");
        }
        
        bw.flush();
        bw.close();
    }
    
    static void mergeSort(int s, int e){
        if(e - s < 1) return;
        int m = s + (e - s) / 2;
        
        mergeSort(s, m);
        mergeSort(m+1, e);
        
        for(int i=s; i<=e; i++){
            tmpArr[i] = arr[i];
        }
        
        int k = s;
        int idx1 = s;
        int idx2 = m+1;
        
        while(idx1 <= m && idx2 <= e){
            // 두 그룹 병합
            if(tmpArr[idx1] > tmpArr[idx2]){
                arr[k] = tmpArr[idx2];
                k++;
                idx2++;
            } else {
                arr[k] = tmpArr[idx1];
                k++;
                idx1++;
            }
        }
        
        // 남아있는 값 정리
        while(idx1 <= m){
            arr[k] = tmpArr[idx1];
            k++;
            idx1++;
        }
        
        while(idx2 <= m){
            arr[k] = tmpArr[idx2];
            k++;
            idx2++;
        }
    }
}
```

주어지는 숫자가 최대 1,000,000 까지 되므로 Arrays 클래스의 sort 사용 시 시간초과가 발생할 수 있다. (Java 8 기준)
(Arrays.sort() 는 dual pivot quick sort 방식을 사용하며 최악의 경우 O(n<sup>2</sup>))

O(nlogn) 시간복잡도를 가진 merge sort로 구현