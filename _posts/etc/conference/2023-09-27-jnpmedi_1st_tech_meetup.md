---
title: "JNPMEDI 1st Tech Meetup 정리 및 후기"
categories: 
    - conference
date: 2023-09-27
last_modified_at: 2023-09-27
toc: true
toc_sticky: true
excerpt: "JNPMEDI 에서 처음으로 주최한 Tech Meetup 참여 후기"
---
## JNPMEDI 1st Tech Meetup 참여 신청

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/37f4ab81-9037-47e7-aae3-f661fc152e39"></center><br/>

festa를 통해 괜찮은 컨퍼런스가 없나 훑어보던 도중 흥미로운 주제들을 다루는 컨퍼런스를 발견했다. 

임상시험 데이터를 다루는 JNPMEDI 라는 스타트업에서 주최한 테크 밋업이며 크게 기술세션과 패널토크로 나눠져 있었다.

기술세션 주제들이 평소 관심 있어하던 주제여서 망설이지 않고 바로 신청했다.

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/b3d4fb1e-91a4-4545-b29b-eb34cd455f2e"></center><br/>

몇 주뒤 JNPMEDI 측으로 부터 최종 참여 여부를 확인하는 전화와 행사 하루전 디테일한 행사 안내 메일을 통해 JNPMEDI가 이번행사에 얼마나 진심인지 느낄수 있었다.

## 포스코 타워 도착

오전 11시쯤 출발했는데 솔직히 여유롭게 도착.. 할줄 알았으나 차가 상당히 막혀서 아슬아슬하게 도착했다 ㅜㅜ

<center><img width="600" height="600" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/cd20189b-c031-4fbd-960f-c5d1cb683bb4"></center><br/>

1층에 대기하고 계신 직원분들의 안내를 받고 들어갔는데 탁 트인 공간의 사무실이 있었고 통유리로 바깥을 내다볼수 있는 구조였다. 

이런곳에서 일할 수 있다니.. 너무 부러웠다. (사무실 내부 풍경 사진을 더 찍었어야했는데 구경하느라 못찍음..)

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c0ec88f2-50d5-490c-ac90-ce43212fae82">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/0a6f3a96-fb85-4310-8f2a-0809ab38e3c6">

참가 등록을 하면서 키링, 노트, 펜 등 굿즈도 받고 간단하게 먹을수 있는 음식들도 비치되어 있어서 접시에 담아왔다. 

지금보니 참.. 사진을 너무 안찍은 것 같다. 다음부터는 사진을 마구마구 찍는걸로..


<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9e279513-acbe-47fd-9815-fa403705262b">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/01110b3a-d6cb-40b7-addf-6308df02cfa8">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2e091b9f-86f7-4f21-aea1-0965c1306acb">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/31a951a4-96f0-430d-98b8-3ddd345276ba">

기술 세션 이후에 패널토크도 이어졌는데 주로 창업에 관련한 이야기들을 주로 하셔서 아직까진 창업에 관심이 없기 때문에.. 잘 귀담아 듣진 않았다 ㅜㅜ

아래부터는 기술 세션을 정리한 내용들이다.

## 세션1. Serverless Event Driven Architecture - Amazon SNS와 SQS를 활용한 Data Pipeline

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/50bebd57-7e99-442b-8646-88e3cdf9af8c"></center>

- JNPMEDI의 서비스들(Maven Builder, Maven CDMS 등)은 Microservice Architecture 기반으로 구성되어있다.
  - Maven Builder: 전자 임상시험 양식 구성을 더 빠르고 정확하게 setup하는 소프트웨어 서비스
  - Maven CDMS: 임상시험 설문지에 입력될 데이터를 전자적으로 수집/관리하는 서비스
<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/01b5d39c-2c25-466c-a5a2-85609849a1c4"></center>

- Maven Builder에서 생성한 전자 설문지를 Maven CDMS에서 사용하려면?
  - MSA 구조에서는 분산된 서비스들 간에 서로 http api로 통신
  - 데이터 전달을 internal api로 하면 상황에 따라 데이터 정합성, 무결성 문제가 발생할 요지가 있음.
  - MSA의 구조적 문제점
    - 동기적인 처리방식인 REST 통신이 중간에 끊기거나 응답이 늦어질 경우 관련 서비스들이 멈출수 있음
    - 분산된 서비스 마다 분리된 DB들 간의 트랜잭션 관리가 어려움
  - 이러한 문제점들은 임상시험 데이터를 다루는 JNPMEDI 서비스에서 매우 치명적 (데이터 무결성, 정합성이 중요함)

<br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/1a832968-b9db-47b6-a7e9-177b0e50c311">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e4ffb849-23d0-42c2-a94c-2418c2ed6cfb">

- 이를 보완하고자 Event Driven Architecture + S3 Event Notification를 도입
  - Event Driven Architecture → 분산된 시스템에서 이벤트를 생성(발행)하고 발행된 이벤트를 수신자에게 전송하는 구조로 수신자는 그 이벤트를 처리하는 방식의 아키텍처이다. 분산 아키텍처 환경에서 상호 간 결합도를 낮추기 위해 비동기 방식으로 이벤트를 전달
  - Event Producers와 Event Consumers 사이에 Event Broker 역할을 AWS S3가 수행
  - S3 Event에 trigger 된 AWS Lambda Function 을 통해 이벤트를 수신

- 그러나 여전히 다음과 같은 문제점이 존재함.
  - Bucket당 100개 제한
  - S3 Event는 1개의 lambda function만 trigger 가능
  - Event가 유실되어 사라지는 경우 Event 실행 보장이 불가함
  - Event 재시도 처리의 자동화 불가
  - Event 관리의 효율성 저하
<br/><br/>

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/73ab9a96-a070-4621-92c1-b38144e2c113">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/5a762a52-6ad1-4cfd-ab46-7b62e2184ad9">

- S3의 문제점을 보완하고자 SNS와 SQS 를 도입
  - AWS Simple Notification Service(SNS)
    - 구독 중인 서비스나 사용자에게 메세지 전달 및 전송을 해주는 서비스. 즉, 특정 topic에 대한 알림 서비스를 의미함
  - AWS Simple Queue Service(SQS)
    - 마이크로 서비스, 분산시스템 및 서버리스 애플리케이션을 쉽게 분리하고 확장할 수 있도록 지원하는 완전 관리형 메세지 대기열 서비스

- Event Driven Architecture + SNS, SQS
  - Maven Builder(Event Producer)가 전자 설문지를 생성할 때 SNS로 topic을 publish함
  - SNS를 구독중인 SQS가 topic을 대기열에 추가함
  - Maven CDMS(Event Consumer)가 SQS로부터 topic(전자 설문지)를 전달받음

## 세션2. 주니어 개발자 관점에서 본 코드 리뷰를 통한 협업과 성장

- 코드 리뷰! 왜 안(못)할까?
  - 타인이 바라보는 나의 이미지 → 내 코드가 이상해서 날 이상하게 보면 어떡하지? 등
  - 코드리뷰가 무엇인지 모름
  - 코드리뷰를 할 시간이 없음 (그 시간에 개발 해야함)

- 코드 리뷰 이후 긍정적인 변화
  - 질문을 통해 팀 내 의사소통능력 향상
  - 코드 일관성을 유지하여 팀원간 혼란을 방지
  - PR에 리뷰 내용을 Approve 하기 전 놓친 부분을 다시 검토하는 습관이 생김
  - 결과적으로 코드 작성능력, 피력능력, 코드분석 능력이 향상된다.

- 나만의 코드리뷰 규칙 만들기
  - 개선이 필요한 이유를 최대한 적어주기
    - 단순히 여기는 이렇게 수정해야 한다 라기보다는 수정해야 하는 이유에 대해서 자세히 적어줘서 코드 리뷰 효율성을 극대화하자
  - 코드 컨벤션은 일관되고, 클린하게 코드리뷰 작성하도록 리뷰하기
    - 코드 컨벤션이 일관되지 않으면 코드 분석에 시간이 더 소요되고 복잡도가 증가한다.
  - 코드 리뷰 작성시 숙제가 아닌 학습으로 느낄 수 있게 리뷰하기
    - 긍정적인 피드백, 물어보기 대신 제안하기, 리소스 공유 등
  - 피드백 할 사항이 없다면 칭찬하기
    - 많은 개발자들이 코드리뷰를 할때 지적만하고 칭찬에 인색하다. 자주 칭찬하자
  - 개발자의 성향을 리뷰하지 않기
    - ex. 왜 삼항연산자를 안쓰고 if-else 문을 썼어요?

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/22e5139d-5f1b-4205-acf7-34aed509272e"></center>

- PR용 템플릿을 만들어서 효율적인 PR을 작성하자

## 후기

너무 유익하고 재밌는 시간이었다. 패널 토크이후 경품 추첨도 했는데 아쉽게도 당첨 되지는 못했다..

그리고 이번 행사에서 세션 별 참여자들 질문 취합용으로 padlet 이라는 툴을 사용했는데 실시간으로 질문들이 올라오고 좋아요 표시도 가능하고.. 상당히 괜찮은 툴인것 같다.

추가로 마지막에 채용설명회도 진행하였는데 백엔드 포지션에는 Node.js(Typescript)를 쓰는것 같다. 나는 Java, kotlin 원툴이라 지원 하기는 힘들 것 같다 ㅜㅜ

이번 행사를 통해 JNPMEDI가 헬스케어/바이오 산업에서 나아가고자 하는 방향을 조금이나마 엿볼수 있었다.

이번이 첫 행사라고 했으니.. JNPMEDI 에서 두번째 행사를 열게 되면 다음에도 참여해봐야겠다.