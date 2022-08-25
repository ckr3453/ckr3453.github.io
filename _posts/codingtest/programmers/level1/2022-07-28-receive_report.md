---
title: "[Level 1] 신고 결과 받기 (Java)"
categories: 
    - programmers
date: 2022-03-17
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
신입사원 무지는 게시판 불량 이용자를 신고하고 처리 결과를 메일로 발송하는 시스템을 개발하려 합니다. 무지가 개발하려는 시스템은 다음과 같습니다.
> - 각 유저는 한 번에 한 명의 유저를 신고할 수 있습니다.
- 신고 횟수에 제한은 없습니다. 서로 다른 유저를 계속해서 신고할 수 있습니다.
- 한 유저를 여러 번 신고할 수도 있지만, 동일한 유저에 대한 신고 횟수는 1회로 처리됩니다.
- k번 이상 신고된 유저는 게시판 이용이 정지되며, 해당 유저를 신고한 모든 유저에게 정지 사실을 메일로 발송합니다.
- 유저가 신고한 모든 내용을 취합하여 마지막에 한꺼번에 게시판 이용 정지를 시키면서 정지 메일을 발송합니다.      

다음은 전체 유저 목록이 ["muzi", "frodo", "apeach", "neo"]이고, k = 2(즉, 2번 이상 신고당하면 이용 정지)인 경우의 예시입니다.

> |유저 ID|유저가 신고한 ID|설명|
|---|---|---|
|"muzi"|"frodo"|"muzi"가 "frodo"를 신고했습니다.|
|"apeach"|"frodo"|"apeach"가 "frodo"를 신고했습니다.|
|"frodo"|"neo"|"frodo"가 "neo"를 신고했습니다.|
|"muzi"|"neo"|"muzi"가 "neo"를 신고했습니다.|
|"apeach"|"muzi"|"apeach"가 "muzi"를 신고했습니다.|

각 유저별로 신고당한 횟수는 다음과 같습니다.

> |유저 ID|신고당한 횟수|
|---|---|
|"muzi"|1|
|"frodo"|2|
|"apeach"|0|
|"neo"|2|

위 예시에서는 2번 이상 신고당한 "frodo"와 "neo"의 게시판 이용이 정지됩니다. 이때, 각 유저별로 신고한 아이디와 정지된 아이디를 정리하면 다음과 같습니다.

>|유저 ID|유저가 신고한 ID|정지된 ID|
|---|---|---|
|"muzi"|["frodo", "neo"]|["frodo", "neo"]|
|"frodo"|["neo"]|["neo"]|
|"apeach"|["muzi", "frodo"]|["frodo"]|
|"neo"|없음|없음|

따라서 "muzi"는 처리 결과 메일을 2회, "frodo"와 "apeach"는 각각 처리 결과 메일을 1회 받게 됩니다.

이용자의 ID가 담긴 문자열 배열 id_list, 각 이용자가 신고한 이용자의 ID 정보가 담긴 문자열 배열 report, 정지 기준이 되는 신고 횟수 k가 매개변수로 주어질 때, 각 유저별로 처리 결과 메일을 받은 횟수를 배열에 담아 return 하도록 solution 함수를 완성해주세요.

#### **제한사항**
> - 2 ≤ id_list의 길이 ≤ 1,000
  - 1 ≤ id_list의 원소 길이 ≤ 10
  - id_list의 원소는 이용자의 id를 나타내는 문자열이며 알파벳 소문자로만 이루어져 있습니다.
  - id_list에는 같은 아이디가 중복해서 들어있지 않습니다.
- 1 ≤ report의 길이 ≤ 200,000
  - 3 ≤ report의 원소 길이 ≤ 21
  - report의 원소는 "이용자id 신고한id"형태의 문자열입니다.
  - 예를 들어 "muzi frodo"의 경우 "muzi"가 "frodo"를 신고했다는 의미입니다.
  - id는 알파벳 소문자로만 이루어져 있습니다.
  - 이용자id와 신고한id는 공백(스페이스)하나로 구분되어 있습니다.
  - 자기 자신을 신고하는 경우는 없습니다.
- 1 ≤ k ≤ 200, k는 자연수입니다.
- return 하는 배열은 id_list에 담긴 id 순서대로 각 유저가 받은 결과 메일 수를 담으면 됩니다.


#### **입출력 예**
>|id_list|report|k|result|
|---|---|---|---|
|["muzi", "frodo", "apeach", "neo"]|["muzi frodo","apeach frodo","frodo neo","muzi neo","apeach muzi"]|2|[2,1,1,0]|
|["con", "ryan"]|["ryan con", "ryan con", "ryan con", "ryan con"]|3|[0,0]|

#### **입출력 예 설명**
> - 입출력 예 #1
문제의 예시와 같습니다.
- 입출력 예 #2
"ryan"이 "con"을 4번 신고했으나, 주어진 조건에 따라 한 유저가 같은 유저를 여러 번 신고한 경우는 신고 횟수 1회로 처리합니다. 따라서 "con"은 1회 신고당했습니다. 3번 이상 신고당한 이용자는 없으며, "con"과 "ryan"은 결과 메일을 받지 않습니다. 따라서 [0, 0]을 return 합니다.

#### **제한시간 안내**
> - 정확성 테스트 : 10초

---

#### **나의 풀이**
신고자가 누구를 신고했는지를 저장하는 reporterInfoMap과 신고당한사람이 몇번이나 신고 당했는지 저장하는 reportedCountInfoMap을 선언하여 풀었다.
reportInfoMap는 _**동일한 유저에 대한 신고 횟수는 1회**_로 처리되기 때문에 중복처리를 위해 Set으로 저장하였고, reportedCountInfoMap는 신고당한사람이 정지기준 이상이 되면 해당 유저를 신고한 신고자에게 정지메일을 발송해야하기 때문에 신고횟수를 저장하는 방식으로 진행했다. 

테스트 케이스는 전부 통과했으나, **신고자가 이미 신고한 유저를 또 신고했을 경우**에 대한 처리가 너무 지저분한것 같아 다시 수정했다.

#### **수정 전**

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;

class Solution {
    public int[] solution(String[] idList, String[] report, int k){
        // @param  idList : 이용자의 ID를 담은 배열.
        // @param  report : 신고한 이용자와 신고당한 이용자의 정보를 담은 배열. ex) "a b",.. -> a가 b를 신고
        // @param  k      : 신고 횟수에 따른 정지 기준 정수값.
        // @return answer : id_list에 담긴 id 순서대로 각 유저가 받은 결과 메일 배열.
      int[] answer = new int[idList.length];
      HashMap<String, HashSet<String>> reporterInfoMap = new HashMap<>();
      HashMap<String, Integer> reportedCountInfoMap = new HashMap<>();
        
      for(String reportInfo : report){
          String reporter = reportInfo.split(" ")[0];  // 신고 한 사람
          String reported = reportInfo.split(" ")[1];  // 신고 당한 사람
          boolean flag = false;
            
          if(reporterInfoMap.containsKey(reporter)){
              if(reporterInfoMap.get(reporter).contains(reported)){
                  // 이미 신고한 유저를 또 신고했을 경우
                  flag = true;
              } else {
                  reporterInfoMap.get(reporter).add(reported);    
              }
          } else {
            {% raw %}
              reporterInfoMap.put(reporter, new HashSet<String>(){{
                  add(reported);
              }});
            {% endraw %}
          }
            
          if (flag) {
              continue;
          } else if (reportedCountInfoMap.containsKey(reported)){
              reportedCountInfoMap.put(reported, reportedCountInfoMap.get(reported)+1);
          } else {
              reportedCountInfoMap.put(reported, 1);
          }
      }
        
      for (String reported : reportedCountInfoMap.keySet()){
          int reportedCount = reportedCountInfoMap.get(reported);
          if(reportedCount >= k){
              // 메일 발송 대상
              for(int i=0; i<idList.length; i++){
                  if(reporterInfoMap.get(idList[i]) == null){
                      answer[i] = 0;
                  } else if(reporterInfoMap.get(idList[i]).contains(reported)){
                      answer[i]++;
                  }
              }
          }
      }
      return answer;
   }
}
```

#### **수정 후**
```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;

class Solution {
    public int[] solution(String[] idList, String[] report, int k){
        // @param idList : 이용자의 ID를 담은 배열.
        // @param report : 신고한 이용자와 신고당한 이용자의 정보를 담은 배열. ex) "a b",.. -> a가 b를 신고
        // @param k      : 신고 횟수에 따른 정지 기준 정수값.
        // @return answer : id_list에 담긴 id 순서대로 각 유저가 받은 신고 결과 메일 개수 배열.
        int[] answer = new int[idList.length];
        HashMap<String, HashSet<String>> reporterInfoMap = new HashMap<>();
        HashMap<String, Integer> reportedCountInfoMap = new HashMap<>();
        HashSet<String> reportSet = new HashSet<>(Arrays.asList(report));
        
        for(String reportInfo : reportSet){
            String reporter = reportInfo.split(" ")[0];  // 신고 한 사람
            String reported = reportInfo.split(" ")[1];  // 신고 당한 사람
            {% raw %}
            reporterInfoMap.putIfAbsent(reporter, new HashSet<String>(){{
                add(reported);
            }});
            {% endraw %}
            reporterInfoMap.get(reporter).add(reported);
            reportedCountInfoMap.put(reported, reportedCountInfoMap.getOrDefault(reported, 0)+1);
        }

        for (String reported : reportedCountInfoMap.keySet()){
            int reportedCount = reportedCountInfoMap.get(reported);
            if(reportedCount >= k){
                // 메일 발송 대상
                for(int i=0; i<idList.length; i++){
                    if(reporterInfoMap.containsKey(idList[i]) && reporterInfoMap.get(idList[i]).contains(reported)) {
                        answer[i]++;
                    }
                }
            }
        }
        return answer;
   }
}
```


#### **다른 사람의 풀이** (최석현, progg님)
```java
import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Solution {
    public int[] solution(String[] id_list, String[] report, int k) {
        Users users = IntStream.range(0, id_list.length)
                .mapToObj(i -> new User(i, new UserId(id_list[i])))
                .collect(Collectors.collectingAndThen(Collectors.toList(), Users::new));

        Reports reports = Arrays.stream(report)
                .map(rawReport -> ReportParser.parse(users, rawReport))
                .collect(Collectors.collectingAndThen(Collectors.toList(), Reports::new));

        Mails mails = reports.generateMails(users, k);

        return users.findSortedAll().stream()
                .mapToInt(user -> mails.findAllByUser(user).size())
                .toArray();
    }


    private static class Mails {
        private final List<Mail> mails;

        public Mails(List<Mail> mails) {
            this.mails = mails;
        }

        public List<Mail> findAllByUser(User user) {
            return mails.stream()
                    .filter(mail -> mail.isSameUser(user))
                    .collect(Collectors.toList());
        }
    }

    private static class Mail {
        private final User recipient;

        public Mail(User recipient) {
            this.recipient = recipient;
        }

        public boolean isSameUser(User user) {
            return Objects.equals(recipient, user);
        }
    }

    private static class ReportParser {
        private static final String DELIMITER = " ";

        public static Report parse(Users users, String report) {
            String[] splitted = report.split(DELIMITER);
            User reporter = users.findUser(new UserId(splitted[0]));
            User reported = users.findUser(new UserId(splitted[1]));

            return new Report(reporter, reported);
        }
    }

    private static class Reports {
        private final List<Report> reports;

        public Reports(List<Report> reports) {
            this.reports = reports;
        }

        public Mails generateMails(Users users, int mailThreshold) {
            return users.findSortedAll().stream()
                    .map(user -> generateMailsOf(user, mailThreshold))
                    .flatMap(Collection::stream)
                    .collect(Collectors.collectingAndThen(Collectors.toList(), Mails::new));
        }

        private List<Mail> generateMailsOf(User user, int mailThreshold) {
            List<Report> userReports = reports.stream()
                    .filter(report -> report.isReported(user))
                    .distinct()
                    .collect(Collectors.toList());

            if (userReports.size() >= mailThreshold) {
                return userReports.stream()
                        .map(Report::getReporter)
                        .map(Mail::new)
                        .collect(Collectors.toList());
            }

            return Collections.emptyList();
        }
    }

    private static class Report {
        private final User reporter;
        private final User reported;

        public Report(User reporter, User reported) {
            this.reporter = reporter;
            this.reported = reported;
        }

        public boolean isReported(User user) {
            return reported.equals(user);
        }

        public User getReporter() {
            return reporter;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Report report = (Report) o;
            return Objects.equals(reporter, report.reporter) && Objects.equals(reported, report.reported);
        }

        @Override
        public int hashCode() {
            return Objects.hash(reporter, reported);
        }
    }

    private static class Users {
        private final List<User> users;

        public Users(List<User> users) {
            this.users = users;
        }

        public User findUser(UserId userId) {
            return users.stream()
                    .filter(user -> user.getId().equals(userId))
                    .findAny()
                    .orElseThrow(() -> new IllegalArgumentException("User Not Found. id: " + userId));
        }

        public List<User> findSortedAll() {
            return users.stream()
                    .sorted()
                    .collect(Collectors.toList());
        }
    }

    private static class User implements Comparable<User> {
        private final Integer sequence;
        private final UserId userId;

        public User(Integer sequence, UserId userId) {
            this.sequence = sequence;
            this.userId = userId;
        }

        public UserId getId() {
            return userId;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            User user = (User) o;
            return Objects.equals(userId, user.userId);
        }

        @Override
        public int hashCode() {
            return Objects.hash(userId);
        }

        @Override
        public int compareTo(User other) {
            return this.sequence.compareTo(other.sequence);
        }
    }

    private static class UserId {
        private final String id;

        public UserId(String id) {
            this.id = id;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            UserId userId1 = (UserId) o;
            return Objects.equals(id, userId1.id);
        }

        @Override
        public int hashCode() {
            return Objects.hash(id);
        }

        @Override
        public String toString() {
            return "Id{" +
                    "id='" + id + '\'' +
                    '}';
        }
    }
}
```
다른 분들은 최대한 코드를 짧게쓰려고 노력한 흔적이 보였는데 이분은 해당 문제를 객체지향적으로 제일 잘 풀어내신것 같아서 가져왔다. (대단...)

이번 문제는 **신고자가 이미 신고한 유저를 또 신고했을 경우**를 어떻게 처리하는지가 관건이었던것 같다. Set으로 해결 하신분들이 대부분 이었고 간혹 stream api를 사용하여 해결하신 분들도 볼 수 있었다. 문제 자체의 난이도는 높지 않아서 문제의 지문을 잘 읽고 접근한다면 금방 풀어낼 수 있을 것이다.