---
title: "[Level 3] 베스트앨범 (Java)"
categories: 
    - programmers
date: 2022-08-17
last_modified_at: 2022-08-17
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---
## **문제 설명**
스트리밍 사이트에서 장르 별로 가장 많이 재생된 노래를 두 개씩 모아 베스트 앨범을 출시하려 합니다. 노래는 고유 번호로 구분하며, 노래를 수록하는 기준은 다음과 같습니다.

1. 속한 노래가 많이 재생된 장르를 먼저 수록합니다.
2. 장르 내에서 많이 재생된 노래를 먼저 수록합니다.
3. 장르 내에서 재생 횟수가 같은 노래 중에서는 고유 번호가 낮은 노래를 먼저 수록합니다.

노래의 장르를 나타내는 문자열 배열 genres와 노래별 재생 횟수를 나타내는 정수 배열 plays가 주어질 때, 베스트 앨범에 들어갈 노래의 고유 번호를 순서대로 return 하도록 solution 함수를 완성하세요.

## **제한 조건**
- genres[i]는 고유번호가 i인 노래의 장르입니다.
- plays[i]는 고유번호가 i인 노래가 재생된 횟수입니다.
- genres와 plays의 길이는 같으며, 이는 1 이상 10,000 이하입니다.
- 장르 종류는 100개 미만입니다.
- 장르에 속한 곡이 하나라면, 하나의 곡만 선택합니다.
- 모든 장르는 재생된 횟수가 다릅니다.

## **입출력 예**

|genres|plays|return|
|["classic", "pop", "classic", "classic", "pop"]|[500, 600, 150, 800, 2500]|[4, 1, 3, 0]|

## **입출력 예 설명**
classic 장르는 1,450회 재생되었으며, classic 노래는 다음과 같습니다.

- 고유 번호 3: 800회 재생
- 고유 번호 0: 500회 재생
- 고유 번호 2: 150회 재생

pop 장르는 3,100회 재생되었으며, pop 노래는 다음과 같습니다.

- 고유 번호 4: 2,500회 재생
- 고유 번호 1: 600회 재생

따라서 pop 장르의 [4, 1]번 노래를 먼저, classic 장르의 [3, 0]번 노래를 그다음에 수록합니다.

장르 별로 가장 많이 재생된 노래를 최대 두 개까지 모아 베스트 앨범을 출시하므로 2번 노래는 수록되지 않습니다.

## **나의 풀이**
```java
import java.util.*;
import java.util.stream.Collectors;

class Solution {
    public int[] solution(String[] genres, int[] plays) {
        List<String> genresList = new ArrayList<>(Arrays.asList(genres));
        List<Integer> answerList = new LinkedList<>();
        Map<String, Integer> genreCountMap = new HashMap<>();
        
        for(String genre : genresList){
            if(genreCountMap.containsKey(genre)){
                genreCountMap.put(genre, genreCountMap.get(genre)+1);
            } else {
                genreCountMap.put(genre, 1);
            }
        }
        
        Map<String, Map<Integer, Integer>> totalMap = new HashMap<>();
        Map<String, Integer> genreTotalPlayMap = new HashMap<>();
        
        for(String genre : genreCountMap.keySet()){
            Map<Integer, Integer> compareMap = new HashMap<>();
            
            while(true){
                int idx = genresList.indexOf(genre);
                if(idx < 0){
                    break;
                } else {       
                    compareMap.put(idx, new Integer(plays[idx]));
                    genresList.set(idx, null);
                }
            }
            totalMap.put(genre, compareMap);
            genreTotalPlayMap.put(genre, compareMap.values().stream().mapToInt(Integer::intValue).sum());
        }
        
        List<Map.Entry<String, Integer>> genreTotalPlayEntryList = new LinkedList<>(genreTotalPlayMap.entrySet());
        genreTotalPlayEntryList.sort(new Comparator<Map.Entry<String, Integer>>() {
            @Override
            public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
                return o2.getValue() - o1.getValue();
            }
        });
                
        for(Map.Entry<String,Integer> entry : genreTotalPlayEntryList){
            
            List<Map.Entry<Integer, Integer>> playsEntryList = new LinkedList<>(totalMap.get(entry.getKey()).entrySet());
            playsEntryList.sort(new Comparator<Map.Entry<Integer, Integer>>() {
                // 음수면 자리바꿈 양수면 그대로
                @Override
                public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
                    int o1Plays = o1.getValue();
                    int o2Plays = o2.getValue();
                    if(o1Plays < o2Plays){
                        return 1;
                    } else if (o1Plays > o2Plays) {
                        return -1;
                    } else {
                        // 재생수가 같을 때 고유 번호(인덱스)로 정렬하기 위한 조건문
                        int o1Index = o1.getKey();
                        int o2Index = o2.getKey();
                        if(o1Index < o2Index) {
                            return -1; // 고유 번호가 낮은 값이 앞으로 오게 설정
                        } else if(o1Index > o2Index) {
                            return 1;
                        } else {
                            return 0;
                        }
                    }
                }
            });
            
            int totalSize = playsEntryList.size() > 2 ? 2 : playsEntryList.size();
            for(int i=0; i<totalSize; i++){
                answerList.add(playsEntryList.get(i).getKey());    
            }
        }
        return answerList.stream().mapToInt(Integer::intValue).toArray();
    }
}
```

Map.Entry 및 Comparator 인터페이스 등을 활용해서 Map 형태의 구조를 정렬해야 하는게 핵심이다.

문제 자체는 어렵지 않았지만 구조 자체가 물고 물리는 살짝 더러운 구조라 잘 생각해서 풀어야한다.

만약 테스트케이스를 통과했는데도 통과가 안되면 아래 테스트 케이스를 추가해서 해보는걸 추천한다.

|genres(string[])|plays(int[])|Return|
|["classic", "pop", "classic", "classic", "pop"]|[500, 600, 150, 800, 600]|[3, 0, 1, 4]|
|["classic", "Newage", "pop", "classic", "classic", "pop", "Newage"]|[500, 1700, 600, 150, 800, 2500, 1500]|[1, 6, 5, 2, 4, 0]|
|["classic", "pop", "classic", "classic", "pop", "zazz", "zazz"]|[500, 600, 150, 800, 2500, 2000, 6000]|[6, 5, 4, 1, 3, 0]|
|["classic", "pop", "classic", "classic", "pop", "test"]|[500, 600, 150, 800, 2500, 100]|[4, 1, 3, 0, 5]|
|["pop", "pop", "pop", "rap", "rap"]|[45, 50, 40, 60, 70]|[1, 0, 4, 3]|
|["a", "b", "b", "c", "c"]|[5, 5, 40, 5, 5]|[2, 1, 3, 4, 0]|
|["A", "B", "A", "B", "A", "C"]|[500, 600, 150, 800, 2500, 5000]|[5, 4, 0, 3, 1]|
|["A", "A", "B", "A"]|[5, 5, 6, 5]|[0, 1, 2]|
