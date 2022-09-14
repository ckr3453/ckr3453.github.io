---
title: "[Level 1] 두 개 뽑아서 더하기 (Python)"
categories: 
    - programmers
date: 2020-10-01
last_modified_at: 2022-07-28
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "[Level 1] 두 개 뽑아서 더하기 (Python)"
---
#### **문제설명**
정수 배열 numbers가 주어집니다. numbers에서 서로 다른 인덱스에 있는 두 개의 수를 뽑아 더해서 만들 수 있는 모든 수를 배열에 오름차순으로 담아 return 하도록 solution 함수를 완성해주세요.

#### **제한사항**
> - numbers의 길이는 2 이상 100 이하입니다.
- numbers의 모든 수는 0 이상 100 이하입니다.

#### **입출력 예**
> |**numbers**|**result**|
|---|---|
|[2,1,3,4,1]|[2,3,4,5,6,7]|
|[5,0,2,7]|[2,5,7,9,12]|

#### **입출력 예 설명**

> 입출력 예 #1
- 2 = 1 + 1 입니다. (1이 numbers에 두 개 있습니다.)
- 3 = 2 + 1 입니다.
- 4 = 1 + 3 입니다.
- 5 = 1 + 4 = 2 + 3 입니다.
- 6 = 2 + 4 입니다.
- 7 = 3 + 4 입니다.
- 따라서 [2,3,4,5,6,7] 을 return 해야 합니다.

> 입출력 예 #2
- 2 = 0 + 2 입니다.
- 5 = 5 + 0 입니다.
- 7 = 0 + 7 = 5 + 2 입니다.
- 9 = 2 + 7 입니다.
- 12 = 5 + 7 입니다.
- 따라서 [2,5,7,9,12] 를 return 해야 합니다.

#### **나의 풀이**
서로 다른 index에 있는 값을 뽑아서 더해야 한다고 해서 이미 더해진 index를 체크하고 넘어가야하나 고민했는데 다시 생각해보니 그냥 모든 요소에 대해 더하면서 나온 결과값에 중복을 배제하면 될 것 같았다.

```python
# 첫번째 풀이
def solution(numbers):
    answer = list()
    for i in range(len(numbers)):
        for j in range(i+1, len(numbers)):
            if numbers[i] + numbers[j] not in answer:
                answer.append(numbers[i] + numbers[j])
    answer.sort()
    return answer
```
![](https://images.velog.io/images/ckr3453/post/a7e294f9-2921-47ad-bfe7-30acfc273641/image.png)

통과는 했지만 테스트 케이스 7번과 8번에서 많이 느려짐을 알 수 있었다. 요소를 더할때마다 if를 거쳐서 느려진건가 싶어서 set을 활용해 수정했다.
```python
# 두번째 풀이
def solution(numbers):
    answer = set()
    for i in range(len(numbers)):
        for j in range(i+1, len(numbers)):
            answer.add(numbers[i] + numbers[j])
    answer = list(answer)
    answer.sort()
    return answer
```
![](https://images.velog.io/images/ckr3453/post/ce570b23-f75e-426b-a077-0afb46b3cd27/image.png)

#### **다른 사람의 풀이**
> ```python
from itertools import combinations
def solution(numbers):
    answer = set()
    for i in list(combinations(numbers,2)):
        answer.add(sum(i))
    return sorted(answer)
```
- 파이썬 기본 라이브러리인 itertools의 combinations 라는 내장함수를 사용하여 인자값에 따라 해당 요소로 구할수 있는 모든 조합을 리턴한다.
- ex) combinations(numbers, 2)는 numbers 리스트 안에 2개의 요소로 구할 수 있는 모든 조합을 반환 

**결론**
찾아보니 itertools 라이브러리의 permutations(리스트의 순열을 리턴하는 함수), product(두개 이상의 리스트에서 조합을 리턴하는 함수)가 있었다. 경우에 따라 유용하게 사용할 수 있을 것 같다.
pythonic하게 코드를 짤 수 있도록 노력해야겠다.

** 추가) yanemone님의 풀이**
>```python
from itertools import combinations
def solution(numbers):
    return sorted(list(set([sum([i,j]) for i,j in combinations(numbers,2)])))
```

세상에... 해당 문제는 한줄로도 풀이가 가능했었다! comprehension를 잘 활용하면 같은 내용이어도 훨씬 우아하게 코드를 짤 수 있다.