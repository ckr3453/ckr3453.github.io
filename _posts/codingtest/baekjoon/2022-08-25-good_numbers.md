---
title: "1253번 좋다 (Java)"
categories: 
    - baekjoon
date: 2022-08-25
last_modified_at: 2022-08-25
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "1253번 좋다 (Java)"
---
## 문제 설명

N개의 수 중에서 어떤 수가 다른 수 두 개의 합으로 나타낼 수 있다면 그 수를 “좋다(GOOD)”고 한다.

N개의 수가 주어지면 그 중에서 좋은 수의 개수는 몇 개인지 출력하라.

수의 위치가 다르면 값이 같아도 다른 수이다.

## 제한 조건

- 시간제한 : 2초
- 메모리제한 : 256MB

## 입력 설명

첫째 줄에는 수의 개수 N(1 ≤ N ≤ 2,000), 두 번째 줄에는 i번째 수를 나타내는 A<sub>i</sub>가 N개 주어진다. (A<sub>i</sub> ≤ 1,000,000,000, A<sub>i</sub>는 정수)

## 출력 설명

좋은 수의 개수를 첫 번째 줄에 출력한다.

## 예제 입력 및 출력1

    10
    1 2 3 4 5 6 7 8 9 10
<hr>

    8

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.StringTokenizer;
import java.util.Arrays;

public class Main {
    public static void main(String[] args) throws Exception{
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int numberLength = Integer.parseInt(br.readLine());
        long[] numbersArr = new long[numberLength];
        
        StringTokenizer st = new StringTokenizer(br.readLine());
        
        for(int i=0; i<numberLength; i++){
            numbersArr[i] = Long.parseLong(st.nextToken());
        }
        
        Arrays.sort(numbersArr);
        
        int count = 0;
        for(int k=0; k<numberLength; k++){
            long targetNum = numbersArr[k];
            int startIdx = 0;
            int endIdx = numberLength-1;
            while(startIdx < endIdx){
                if(numbersArr[startIdx] + numbersArr[endIdx] > targetNum){
                    endIdx--; 
                } else if(numbersArr[startIdx] + numbersArr[endIdx] < targetNum){
                    startIdx++;
                } else {
                    // 값이 맞을 때
                    // "다른 두 수"의 합이므로 서로 값이 같을 수 없음
                    // 또한 자기 자신(targetNum)을 포함하면 안됨
                    if(startIdx != k && endIdx != k){
                        // 자기 자신을 포함하지 않으며 두 수가 다름 (조건 충족)
                        count++;
                        break;
                    } else if (startIdx == k){
                        // 자기 자신은 포함하면 안되니 인덱스 증감 후 다음 루프 진행
                        startIdx++;
                    } else if (endIdx == k) {
                        // 자기 자신은 포함하면 안되니 인덱스 증감 후 다음 루프 진행
                        endIdx--;
                    }
                }
            }
        }
        
        System.out.println(count);
        br.close();
    }
}
```