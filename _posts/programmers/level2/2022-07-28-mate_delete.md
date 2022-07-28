---
title: "[Level 2] 짝지어 제거하기 (Java)"
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
---
#### **문제설명**
짝지어 제거하기는, 알파벳 소문자로 이루어진 문자열을 가지고 시작합니다. 먼저 문자열에서 같은 알파벳이 2개 붙어 있는 짝을 찾습니다. 그다음, 그 둘을 제거한 뒤, 앞뒤로 문자열을 이어 붙입니다. 이 과정을 반복해서 문자열을 모두 제거한다면 짝지어 제거하기가 종료됩니다. 문자열 S가 주어졌을 때, 짝지어 제거하기를 성공적으로 수행할 수 있는지 반환하는 함수를 완성해 주세요. 성공적으로 수행할 수 있으면 1을, 아닐 경우 0을 리턴해주면 됩니다.

예를 들어, 문자열 S = baabaa 라면

b aa baa → bb aa → aa →

의 순서로 문자열을 모두 제거할 수 있으므로 1을 반환합니다.


#### **제한사항**
> - 문자열의 길이 : 1,000,000이하의 자연수
- 문자열은 모두 소문자로 이루어져 있습니다.


#### **입출력 예**
>|s|result|
|---|---|
|baabaa|1|
|cdcd|0|

#### **입출력 예 설명**
> - 입출력 예 #1
위의 예시와 같습니다.
- 입출력 예 #2
문자열이 남아있지만 짝지어 제거할 수 있는 문자열이 더 이상 존재하지 않기 때문에 0을 반환합니다.

---

#### **나의 풀이**
문자열을 스택에 넣어 하나하나 비교하는 방식으로 풀었다. 다음 문자열과 스택 top에 문자열이 동일하면 제거를 하는 방식으로 스택 내에 문자열이 없을 경우 전부 제거되었다고 판단한다.


```java
import java.util.Stack;


class Solution {
    public int solution(String s) {
        int answer = 0;
        String[] words = s.split("");
        Stack<String> stack = new Stack<>();
        
        for (String word : words) {
            if (stack.isEmpty()) {
                stack.push(word);
                continue;
            }
            if (word.equals(stack.peek())) {
                stack.pop();
            } else {
                stack.push(word);
            }
        }

        if(stack.size() == 0) answer = 1;
        return answer;
    }
}
```

문자열의 처음부터 순서대로 접근하는 방식을 보고 스택 자료구조를 떠올렸다면 쉽게 풀어낼수 있다.