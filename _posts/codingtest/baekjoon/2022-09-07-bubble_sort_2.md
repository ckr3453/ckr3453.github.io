---
title: "1517번 버블 소트2 (Java)"
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
---
## 문제 설명

N개의 수로 이루어진 수열 A[1], A[2], …, A[N]이 있다. 이 수열에 대해서 버블 소트를 수행할 때, Swap이 총 몇 번 발생하는지 알아내는 프로그램을 작성하시오.

버블 소트는 서로 인접해 있는 두 수를 바꿔가며 정렬하는 방법이다. 예를 들어 수열이 3 2 1 이었다고 하자. 이 경우에는 인접해 있는 3, 2가 바뀌어야 하므로 2 3 1 이 된다. 다음으로는 3, 1이 바뀌어야 하므로 2 1 3 이 된다. 다음에는 2, 1이 바뀌어야 하므로 1 2 3 이 된다. 그러면 더 이상 바꿔야 할 경우가 없으므로 정렬이 완료된다.

## 제한 조건

- 시간제한 : 1초
- 메모리제한 : 512MB

## 입력 설명

첫째 줄에 N(1 ≤ N ≤ 500,000)이 주어진다. 다음 줄에는 N개의 정수로 A[1], A[2], …, A[N]이 주어진다. 각각의 A[i]는 0 ≤ A[i] ≤ 1,000,000,000의 범위에 들어있다.

## 출력 설명

첫째 줄에 Swap 횟수를 출력한다

## 예제 입력 및 출력1

    3
    3 2 1
<hr>

    3

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {
    public static int[] arr, temp;
    public static long result;
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int arrLength = Integer.parseInt(br.readLine());
        arr = new int[arrLength + 1];
        temp = new int[arrLength + 1];
        
        StringTokenizer st = new StringTokenizer(br.readLine());
        for(int i=1; i<=arrLength; i++){
            arr[i] = Integer.parseInt(st.nextToken());
        }
        
        result = 0;
        mergeSort(1, arrLength);
        System.out.println(result);
    }
    
    public static void mergeSort(int start, int end){
        if(end - start < 1) return;
        int median = start + (end - start) / 2; // 중간 인덱스
        
        mergeSort(start, median);
        mergeSort(median+1, end);
        
        for(int i=start; i<=end; i++){
            temp[i] = arr[i];
        }
        
        int resultArrIdx = start; // 병합용 인덱스
        int left = start;
        int right = median+1;
        while(left <= median && right <= end){    // 두개의 그룹(배열) 병합
            if(temp[left] > temp[right]){
                // 왼쪽 값이 크므로 오른쪽 값을 삽입 (오름차순)
                arr[resultArrIdx] = temp[right];
                result += right - resultArrIdx;    // 앞으로 이동(스왑)했으므로 이동한 idx 만큼 결과에 추가
                resultArrIdx++;
                right++;
            } else {
                // 오른쪽 값이 크므로 왼쪽 값을 삽입 (오름차순)
                arr[resultArrIdx] = temp[left];
                resultArrIdx++;
                left++;
            }
        }
        
        // 왼쪽과 우측 그룹의 인덱스가 아직 남아있을 경우 전부 소진
        while(left <= median){
            arr[resultArrIdx] = temp[left];
            left++;
            resultArrIdx++;
        }
    
        while(right <= end){
            arr[resultArrIdx] = temp[right];
            right++;
            resultArrIdx++;
        }
    }
}
```

제목은 버블 소트지만 숫자의 범위가 크므로 버블 소트로 구현시 시간초과 발생.

버블 소트의 경우, idx 1번의 이동 = swap 을 의미하므로 swap 구간을 캐치하여 이동한 idx 만큼을 저장하는 방식으로 구현해야한다.

O(nlogn) 시간복잡도를 가진 merge sort로 구현