---
title: "[Level 2] 행렬 테두리 회전하기 (Java)"
categories: 
    - programmers
date: 2022-03-31
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
rows x columns 크기인 행렬이 있습니다. 행렬에는 1부터 rows x columns까지의 숫자가 한 줄씩 순서대로 적혀있습니다. 이 행렬에서 직사각형 모양의 범위를 여러 번 선택해, 테두리 부분에 있는 숫자들을 시계방향으로 회전시키려 합니다. 각 회전은 (x1, y1, x2, y2)인 정수 4개로 표현하며, 그 의미는 다음과 같습니다.
> - x1 행 y1 열부터 x2 행 y2 열까지의 영역에 해당하는 직사각형에서 테두리에 있는 숫자들을 한 칸씩 시계방향으로 회전합니다.   

다음은 6 x 6 크기 행렬의 예시입니다.

![](https://images.velog.io/images/ckr3453/post/b4855bec-e994-49a7-b3a8-c2aaf1ebcdb0/grid_example.png)

이 행렬에 (2, 2, 5, 4) 회전을 적용하면, 아래 그림과 같이 2행 2열부터 5행 4열까지 영역의 테두리가 시계방향으로 회전합니다. 이때, 중앙의 15와 21이 있는 영역은 회전하지 않는 것을 주의하세요.

![](https://images.velog.io/images/ckr3453/post/394bd569-dd4f-4522-8afe-98b23eca29a7/rotation_example.png)

행렬의 세로 길이(행 개수) rows, 가로 길이(열 개수) columns, 그리고 회전들의 목록 queries가 주어질 때, 각 회전들을 배열에 적용한 뒤, 그 회전에 의해 위치가 바뀐 숫자들 중 가장 작은 숫자들을 순서대로 배열에 담아 return 하도록 solution 함수를 완성해주세요.

#### **제한사항**
> - rows는 2 이상 100 이하인 자연수입니다.
- columns는 2 이상 100 이하인 자연수입니다.
- 처음에 행렬에는 가로 방향으로 숫자가 1부터 하나씩 증가하면서 적혀있습니다.
  - 즉, 아무 회전도 하지 않았을 때, i 행 j 열에 있는 숫자는 ((i-1) x columns + j)입니다.
- queries의 행의 개수(회전의 개수)는 1 이상 10,000 이하입니다.
- queries의 각 행은 4개의 정수 [x1, y1, x2, y2]입니다.
  - x1 행 y1 열부터 x2 행 y2 열까지 영역의 테두리를 시계방향으로 회전한다는 뜻입니다.
  - 1 ≤ x1 < x2 ≤ rows, 1 ≤ y1 < y2 ≤ columns입니다.
  - 모든 회전은 순서대로 이루어집니다.
  - 예를 들어, 두 번째 회전에 대한 답은 첫 번째 회전을 실행한 다음, 그 상태에서 두 번째 회전을 실행했을 때 이동한 숫자 중 최솟값을 구하면 됩니다.

#### **입출력 예시**
>|rows|columns|queries|result|
|---|---|---|---|
|6|6|[[2,2,5,4],[3,3,6,6],[5,1,6,3]]|[8, 10, 25]|
|3|3|[[1,1,2,2],[1,2,2,3],[2,1,3,2],[2,2,3,3]]|[1, 1, 5, 3]|
|100|97|[[1,1,100,97]]|[1]|

#### **입출력 예 설명**
입출력 예 #1
- 회전을 수행하는 과정을 그림으로 표현하면 다음과 같습니다.
- ![](https://images.velog.io/images/ckr3453/post/b83d9fb3-13a4-40d3-9dc7-a684280c61d9/example1.png)

입출력 예 #2
- 회전을 수행하는 과정을 그림으로 표현하면 다음과 같습니다.
- ![](https://images.velog.io/images/ckr3453/post/168737e0-6602-4da6-96b4-221b8f82d32f/example2.png)

입출력 예 #3
- 이 예시에서는 행렬의 테두리에 위치한 모든 칸들이 움직입니다. 따라서, 행렬의 테두리에 있는 수 중 가장 작은 숫자인 1이 바로 답이 됩니다.


#### **나의 풀이**
각 회전의 최소값을 구하기 위해 queries를 대상으로 for-each 문을 작성했다. 행렬(square)위에서 인덱스로 사용하기 위해 각 요소마다 -1를 하여 진행했다. 이제 중요한 부분은 firstNum 부분이다. 


아래 그림처럼(입출력 예 #1 기준) 회전에 해당하는 요소들을 한칸씩 시계방향으로 밀어서 진행할 것이다.

- <img src="https://images.velog.io/images/ckr3453/post/b345d9e6-34a4-467b-9253-48f7d63ac9ed/pic1.jpg" height="70%" width="70%">


이때 처음으로 밀려서 덮어써진 숫자(여기서는 10)를 따로 빼놓지 않으면 그대로 overwrite 되어서 복구할 수 없게 된다. 그래서 아래처럼 시작 전에 **변수에 따로 저장해놨다가** 회전이 전부 끝나면 적용 하도록 한다.

- <img src="https://images.velog.io/images/ckr3453/post/1580acb9-ca57-4aa9-9745-5594d764790a/pic2.jpg" height="70%" width="70%">

첫번째 순서로 아래 그림처럼 상단의 숫자를 오른쪽으로 밀어서 덮어쓴다.

- <img src="https://images.velog.io/images/ckr3453/post/0fa788c8-ec9f-4b34-98ac-c694ca4d2cec/%EA%B7%B8%EB%A6%BC3.jpg" height="70%" width="70%">

두번째 순서로 아래 그림처럼 좌측의 숫자를 위로 밀어서 덮어쓴다.

- <img src="https://images.velog.io/images/ckr3453/post/55de67ed-ed67-4c44-89b5-99ee5799960e/%EA%B7%B8%EB%A6%BC4.jpg" height="70%" width="70%">

세번째, 네번째도 마찬가지로 진행하면 아래와 같이 형태를 갖추게 된다. 여기서 첫번째 회전 때 따로 저장했던 firstNum을 맨 마지막 회전 위치에 덮어써준다.

- <img src="https://images.velog.io/images/ckr3453/post/5d8bc686-6a55-4647-bfcb-4f23189ab732/%EA%B7%B8%EB%A6%BC5.jpg" height="70%" width="70%">

이렇게 하면 아래의 그림 처럼 한번의 회전이 종료되며, 반복문을 통해 나머지 회전도 적용하면 된다. 최소값은 숫자를 덮어쓸때마다 min 함수를 사용하여 구한다.

- <img src="https://images.velog.io/images/ckr3453/post/d4187206-ba50-4c00-9e19-33d21e5757f8/%EA%B7%B8%EB%A6%BC7.jpg" height="70%" width="70%">

```java
import java.util.*;

class Solution {
    public int[] rotateNums(int[][] square, int[][] queries){
        int[] answer = new int[queries.length];
        int minimalsIdx=0;

        for(int[] query : queries){
            int x1 = query[0]-1;
            int y1 = query[1]-1;
            int x2 = query[2]-1;
            int y2 = query[3]-1;
            int firstNum = square[x1][y2];
            int min = firstNum;
            
            // 숫자를 우로 이동 (상단)
            for(int i=y2-1; i>=y1; i--){
                min = Math.min(min, square[x1][i]);
                square[x1][i+1] = square[x1][i];
            }

            // 숫자를 위로 이동 (좌측)
            for(int i=x1+1; i<=x2; i++){
                min = Math.min(min, square[i][y1]);
                square[i-1][y1] = square[i][y1];
            }

            // 숫자를 좌로 이동 (하단)
            for(int i=y1+1; i<=y2; i++){
                min = Math.min(min, square[x2][i]);
                square[x2][i-1] = square[x2][i];
            }

            // 숫자를 아래로 이동 (우측)
            for(int i=x2-1; i>=x1; i--){
                min = Math.min(min, square[i][y2]);
                square[i+1][y2] = square[i][y2];
            }

            square[x1+1][y2] = firstNum;
            answer[minimalsIdx] = min;
            minimalsIdx++;
        }

        return answer;
    }
    
    public int[][] initSquare(int rows, int columns){
        int[][] square = new int[rows][columns];
        int value = 1;
        for(int i=0; i<rows; i++){
            for(int j=0; j<columns; j++){
                square[i][j] = value;
                value++;
            }
        }
        
        return square;
    }
    
    public int[] solution(int rows, int columns, int[][] queries) {
        int[] answer = {};
        int[][] square = initSquare(rows, columns);
        return rotateNums(square, queries);
    }
}
```

첫 회전 시작 때 덮어써지는 숫자 하나를 신경쓰지못해 애를 먹었다. 복잡한 알고리즘을 요하지는 않았다.