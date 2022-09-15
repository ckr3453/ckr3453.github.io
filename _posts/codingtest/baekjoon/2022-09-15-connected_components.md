---
title: "11724번 연결 요소의 개수 (Java)"
categories: 
    - baekjoon
date: 2022-09-15
last_modified_at: 2022-09-15
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
excerpt: "11724번 연결 요소의 개수 (Java)"
---
## 문제 설명

방향 없는 그래프가 주어졌을 때, 연결 요소 (Connected Component)의 개수를 구하는 프로그램을 작성하시오.

## 제한 조건

- 시간제한 : 3초
- 메모리제한 : 512MB

## 입력 설명

첫째 줄에 정점의 개수 N과 간선의 개수 M이 주어진다. (1 ≤ N ≤ 1,000, 0 ≤ M ≤ N×(N-1)/2) 둘째 줄부터 M개의 줄에 간선의 양 끝점 u와 v가 주어진다. (1 ≤ u, v ≤ N, u ≠ v) 같은 간선은 한 번만 주어진다.

## 출력 설명

첫째 줄에 연결 요소의 개수를 출력한다.

## 예제 입력 및 출력1

    6 5
    1 2
    2 5
    5 1
    3 4
    4 6
<hr>

    2

## 예제 입력 및 출력2

    6 8
    1 2
    2 5
    5 1
    3 4
    4 6
    5 4
    2 4
    2 3
<hr>

    1

## 나의 풀이

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.StringTokenizer;

public class Main {
    public static void main(String[] args) throws Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st = new StringTokenizer(br.readLine());

        int nodes = Integer.parseInt(st.nextToken());
        int edges = Integer.parseInt(st.nextToken());

        // 인덱스가 노드가 된다.
        // 노드가 1부터 시작하기 때문에 +1
        ArrayList<Integer>[] arrayList = new ArrayList[nodes+1];

        // 해당 노드에 방문 했는지
        boolean[] visited = new boolean[nodes+1];

        for(int i=1; i<nodes+1; i++) {
            arrayList[i] = new ArrayList<>();   // 인접 노드 저장할 공간 초기화
        }

        for(int i=0; i<edges; i++) {
            st = new StringTokenizer(br.readLine());
            int node1 = Integer.parseInt(st.nextToken());
            int node2 = Integer.parseInt(st.nextToken());

            // 방향이 없는 그래프 즉, 양방향 이므로 엣지가 가리키는 노드 둘다에 엣지 더하기
            arrayList[node1].add(node2);
            arrayList[node2].add(node1);
        }

        // 연결 요소 개수
        int connectedComponentCount = 0;
        
        for(int i=1; i<nodes+1; i++){
            if(!visited[i]){
                connectedComponentCount++;
                dfs(arrayList, visited, i);
            }
        }

        System.out.println(connectedComponentCount);
    }
    
    static void dfs(ArrayList<Integer>[] arrayList, boolean[] visited, int node){
        // 이미 방문한 노드면 return
        if(visited[node]) return;
        
        // 현재 노드 탐색
        visited[node] = true;
        
        for(int i : arrayList[node]){
            // 현재 노드의 연결된 노드 중 방문하지 않은 노드의 경우 재탐색 
            if(visited[i] == false){
                dfs(arrayList, visited, i);
            }
        }
    }
}
```

dfs를 활용해서 각 연결요소 갯수를 구하기.