---
title: "[Level 2] 스킬트리 (Java)"
categories: 
    - programmers
date: 2022-08-03
last_modified_at: 2022-08-03
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "[Level 2] 스킬트리 (Java)"
---
#### **문제 설명**

선행 스킬이란 어떤 스킬을 배우기 전에 먼저 배워야 하는 스킬을 뜻합니다.

예를 들어 선행 스킬 순서가 스파크 → 라이트닝 볼트 → 썬더일때, 썬더를 배우려면 먼저 라이트닝 볼트를 배워야 하고, 라이트닝 볼트를 배우려면 먼저 스파크를 배워야 합니다.

위 순서에 없는 다른 스킬(힐링 등)은 순서에 상관없이 배울 수 있습니다. 따라서 스파크 → 힐링 → 라이트닝 볼트 → 썬더와 같은 스킬트리는 가능하지만, 썬더 → 스파크나 라이트닝 볼트 → 스파크 → 힐링 → 썬더와 같은 스킬트리는 불가능합니다.

선행 스킬 순서 skill과 유저들이 만든 스킬트리를 담은 배열 skill_trees가 매개변수로 주어질 때, 가능한 스킬트리 개수를 return 하는 solution 함수를 작성해주세요.

#### **제한사항**

- 스킬은 알파벳 대문자로 표기하며, 모든 문자열은 알파벳 대문자로만 이루어져 있습니다.
- 스킬 순서와 스킬트리는 문자열로 표기합니다.
  - 예를 들어, C → B → D 라면 "CBD"로 표기합니다
- 선행 스킬 순서 skill의 길이는 1 이상 26 이하이며, 스킬은 중복해 주어지지 않습니다.
- skill_trees는 길이 1 이상 20 이하인 배열입니다.
- skill_trees의 원소는 스킬을 나타내는 문자열입니다.
  - skill_trees의 원소는 길이가 2 이상 26 이하인 문자열이며, 스킬이 중복해 주어지지 않습니다.

#### **입출력 예**

|skill|skill_trees|return|
|"CBD"|["BACDE", "CBADF", "AECB", "BDA"]|2|

#### **입출력 예 설명**

- "BACDE": B 스킬을 배우기 전에 C 스킬을 먼저 배워야 합니다. 불가능한 스킬트립니다.
- "CBADF": 가능한 스킬트리입니다.
- "AECB": 가능한 스킬트리입니다.
- "BDA": B 스킬을 배우기 전에 C 스킬을 먼저 배워야 합니다. 불가능한 스킬트리입니다.


#### **나의 풀이**

```java
class Solution {
    public int solution(String skill, String[] skill_trees) {
        int answer = 0;
        int treeLength = skill_trees.length;

        for(int i=0; i<treeLength; i++){
            boolean flag = true;
            
            // 스킬트리를 한글자씩 배열에 원소로 다시담음 (skills[])
            // ex) ["BACDE"] -> ["B","A","C","D","E"]
            String[] skills = skill_trees[i].split("");
            
            // 인덱스를 순차적으로 올려가며 비교할 변수
            int cnt=0;

            // 스킬순서(skill)에 스킬트리의 원소들을 하나씩 넣어보면서
            // cnt와 비교 (순서대로 스킬이 적용되야 하기 때문에 0부터 시작)
            // (필요한 선행 스킬보다 인덱스가 높으면 flag -> false후 break)
            // (cnt와 일치한다면 cnt를 증가시키며 계속 검사수행)
            // 최종적으로 flag가 true면 정답에 담음.
            for(int j=0; j<skills.length; j++){
                if(cnt < skill.indexOf(skills[j])){
                    flag = false;
                    break;
                }else if(cnt == skill.indexOf(skills[j])){
                    cnt++;
                }
            }

            if(flag){
                answer++;
            }
        }
        return answer;
    }
}
```
