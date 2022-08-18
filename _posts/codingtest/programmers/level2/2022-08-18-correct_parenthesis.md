---
title: "[Level 2] 올바른 괄호 (Java)"
categories: 
    - programmers
date: 2022-08-18
last_modified_at: 2022-08-18
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---
## 문제 설명
괄호가 바르게 짝지어졌다는 것은 '(' 문자로 열렸으면 반드시 짝지어서 ')' 문자로 닫혀야 한다는 뜻입니다. 예를 들어

- "()()" 또는 "(())()" 는 올바른 괄호입니다.
- ")()(" 또는 "(()(" 는 올바르지 않은 괄호입니다.

'(' 또는 ')' 로만 이루어진 문자열 s가 주어졌을 때, 문자열 s가 올바른 괄호이면 true를 return 하고, 올바르지 않은 괄호이면 false를 return 하는 solution 함수를 완성해 주세요.

## 제한 조건
- 문자열 s의 길이 : 100,000 이하의 자연수
- 문자열 s는 '(' 또는 ')' 로만 이루어져 있습니다.

## 입출력 예

|s|answer|
|"()()"|true|
|"(())()"|true|
|")()("|false|
|"(()("|false|

## 입출력 예 설명
입출력 예 #1,2,3,4<br/>
문제의 예시와 같습니다.

## 나의 기존 풀이 (효율성 통과 실패)
```java
import java.util.Stack;

class Solution {
    boolean solution(String s) {
        boolean answer = true;
        int sLen = s.length();
        Stack<Character> stack = new Stack<>();
        
        for(int i=0; i<sLen; i++){
            char c = s.charAt(i);
            if(stack.isEmpty()){ 
                // ')' 로 시작하거나 길이가 짝수가 아니면 true 불가능
                if(c == ')' || sLen % 2 != 0) return false;
                stack.push(c);
            } else {
                if(stack.peek() == c){
                    stack.push(c);
                    answer = false;
                } else {
                    stack.pop();
                    answer = true;
                }
            }
        }
        return answer;
    }
}
```

스택의 top을 다음 요소와 비교하여 올바른 괄호 -> () 를 충족하는지 확인 후 만족 할 시 해당 요소를 pop한다.

그러나 해당 방식으로 진행 시 정확성 테스트는 모두 통과하였으나 효율성 테스트를 통과하지 못했다.

![image](https://user-images.githubusercontent.com/36228833/185336867-957e9095-9f88-4e3d-85aa-3e178b3e5d38.png)

속도를 개선하기 위해 방법을 찾다가 Java에서 제공하는 Stack 인터페이스를 사용하지 않고 구현했다.

## 나의 개선된 풀이

```java
class Solution {
    boolean solution(String s) {
        int cnt = 0;    // 스택 역할
        
        for(int i=0; i<s.length(); i++){
            if(cnt < 0){
              return false;  
            } else if(s.charAt(i) == '(') {
                cnt++;
            } else {
                cnt--;
            }
        }
        return cnt == 0 ? true : false;
    }
}
```

![image](https://user-images.githubusercontent.com/36228833/185337762-96c2d24a-ee24-42a9-bfea-7a145976c153.png)

count를 통해 Stack 역할을 대신하여 속도가 훨씬 개선되었다.