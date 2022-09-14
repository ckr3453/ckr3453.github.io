---
title: "[Level 2] 문자열 압축 (Java)"
categories: 
    - programmers
date: 2022-06-06
last_modified_at: 2022-07-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "[Level 2] 문자열 압축 (Java)"
---
#### **문제설명**
데이터 처리 전문가가 되고 싶은 "어피치"는 문자열을 압축하는 방법에 대해 공부를 하고 있습니다. 최근에 대량의 데이터 처리를 위한 간단한 비손실 압축 방법에 대해 공부를 하고 있는데, 문자열에서 같은 값이 연속해서 나타나는 것을 그 문자의 개수와 반복되는 값으로 표현하여 더 짧은 문자열로 줄여서 표현하는 알고리즘을 공부하고 있습니다.
간단한 예로 "aabbaccc"의 경우 "2a2ba3c"(문자가 반복되지 않아 한번만 나타난 경우 1은 생략함)와 같이 표현할 수 있는데, 이러한 방식은 반복되는 문자가 적은 경우 압축률이 낮다는 단점이 있습니다. 예를 들면, "abcabcdede"와 같은 문자열은 전혀 압축되지 않습니다. "어피치"는 이러한 단점을 해결하기 위해 문자열을 1개 이상의 단위로 잘라서 압축하여 더 짧은 문자열로 표현할 수 있는지 방법을 찾아보려고 합니다.

예를 들어, "ababcdcdababcdcd"의 경우 문자를 1개 단위로 자르면 전혀 압축되지 않지만, 2개 단위로 잘라서 압축한다면 "2ab2cd2ab2cd"로 표현할 수 있습니다. 다른 방법으로 8개 단위로 잘라서 압축한다면 "2ababcdcd"로 표현할 수 있으며, 이때가 가장 짧게 압축하여 표현할 수 있는 방법입니다.

다른 예로, "abcabcdede"와 같은 경우, 문자를 2개 단위로 잘라서 압축하면 "abcabc2de"가 되지만, 3개 단위로 자른다면 "2abcdede"가 되어 3개 단위가 가장 짧은 압축 방법이 됩니다. 이때 3개 단위로 자르고 마지막에 남는 문자열은 그대로 붙여주면 됩니다.

압축할 문자열 s가 매개변수로 주어질 때, 위에 설명한 방법으로 1개 이상 단위로 문자열을 잘라 압축하여 표현한 문자열 중 가장 짧은 것의 길이를 return 하도록 solution 함수를 완성해주세요.


#### **제한사항**
> - s의 길이는 1 이상 1,000 이하입니다.
- s는 알파벳 소문자로만 이루어져 있습니다.


#### **입출력 예**
>|s|result|
|---|---|
|"aabbaccc"|7|
|"ababcdcdababcdcd"|9|
|"abcabcdede"|8|
|"abcabcabcabcdededededede"|14|
|"xababcdcdababcdcd"|17|

#### **입출력 예 설명**
- 입출력 예 #1

문자열을 1개 단위로 잘라 압축했을 때 가장 짧습니다.

- 입출력 예 #2

문자열을 8개 단위로 잘라 압축했을 때 가장 짧습니다.

- 입출력 예 #3

문자열을 3개 단위로 잘라 압축했을 때 가장 짧습니다.

- 입출력 예 #4

문자열을 2개 단위로 자르면 "abcabcabcabc6de" 가 됩니다.
문자열을 3개 단위로 자르면 "4abcdededededede" 가 됩니다.
문자열을 4개 단위로 자르면 "abcabcabcabc3dede" 가 됩니다.
문자열을 6개 단위로 자를 경우 "2abcabc2dedede"가 되며, 이때의 길이가 14로 가장 짧습니다.

- 입출력 예 #5

문자열은 제일 앞부터 정해진 길이만큼 잘라야 합니다.
따라서 주어진 문자열을 x / ababcdcd / ababcdcd 로 자르는 것은 불가능 합니다.
이 경우 어떻게 문자열을 잘라도 압축되지 않으므로 가장 짧은 길이는 17이 됩니다.



---

#### **나의 풀이**
반복되는 규칙은 주어진 문자열의 절반의 길이를 넘을수 없다. (절반을 넘게되면 2번이상 반복될 수 없음) 때문에 주어진 문자열의 절반까지만 loop를 돌도록 설정하였다. 

반복되는 규칙 횟수를 저장하는 count는 0이아닌 1부터 시작했는데 그 이유는 안쪽 for문에 있다. 안쪽 for문은 문자열을 substring 함수를 통해 가져오고 비교한다. 

순차적으로 index를 옮겨가며 비교를 해야했기 때문에 i를 기준(비교를 할 기준 문자열의 길이)으로 동적으로 움직여야 했다. 하지만 저렇게 두면 안쪽 for문은 0부터 시작하지 못하는 문제가 생긴다.

첫 시작은 어차피 기준 문자열을 가져옴과 동시에 한번 반복 되었다는 의미 이기도 하므로 count를 1로 두고 사실상 2번째 반복부터 count를 시작했다고 생각하면 된다.

그리고 규칙에 벗어나는 새로운 문자열이 등장했을 경우, 저장된 횟수와 문자열을 조합하고 다음 기준 문자열을 새로운 문자열로 변경 및 count를 초기화 하여 반복 진행한다.

마지막으로 기준 문자열의 길이가 총 문자열의 길이의 배수가 아닌 경우 딱 떨어지지 않아 나머지 문자열이 남게된다. 해당 문자열의 경우도 고려하여 마지막에 붙여준다.

```java
class Solution {
    public int solution(String s) {
        int answer = s.length();
        int count = 1;
        for(int i=1; i<=s.length()/2; i++){
            StringBuilder result = new StringBuilder();
            String base = s.substring(0, i);
            for(int j=i; j<=s.length(); j+=i){
                // 이미 base에서 하나는 count 했음 (j=i)
                int endIdx = Math.min(j + i, s.length());   // 인덱스는 길이를 넘을수 없음
                String compare = s.substring(j, endIdx);
                if(base.equals(compare)){
                    count++;
                } else {
                    if(count >= 2){
                        result.append(count);
                    }
                    result.append(base);
                    base = compare;     // 마지막 인덱스일때 한번 더 더해야함 (딱 안떨어지는 경우 있음)
                    count = 1;
                }
            }
            result.append(base);    // 마지막 문자 붙이기
            answer = Math.min(answer, result.length());
        }
        return answer;
    }
}
```

1. String.substring(startIndex, endIndex) 함수를 활용하자.
2. 문자열의 연산이 자주 이뤄지는경우, String 보단 StringBuilder를 활용하여 부하를 줄이자.
3. 단순하게 기준 문자열에 따라 순차적으로 count 한다는 것을 잊지말자. (너무 복잡하게 생각할 경우 어려워짐)