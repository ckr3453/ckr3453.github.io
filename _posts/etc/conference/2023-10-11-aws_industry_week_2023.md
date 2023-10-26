---
title: "AWS Industry Week 2023 정리 및 후기"
categories: 
    - conference
date: 2023-10-11
last_modified_at: 2023-10-26
toc: true
toc_sticky: true
excerpt: "AWS Industry Week 2023 참여 후기"
---

## 코엑스 도착

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/be5f3d4e-526e-49d2-8fcb-8e7af8a15be8">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c854d344-1d69-4262-90df-fca596874d12">

코엑스 들어오자마자 참가자 등록부터 했다. 일찍부터 온사람들로 이미 바글바글.. 

사전 등록한 사람만 등록 가능한줄 알았는데 옆에서 보니 사전등록을 안했어도 참가자 등록이 되는것같다.

## 오프닝

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/86739765-0866-4774-ba44-9f6b90380087">
<img width="400" height="250" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/ae605c1b-2a7f-4a5a-8ca3-ac8385056206"><br/>

오프닝 연사를 들으려고 제일큰 홀에 입장했는데 입구에서 무슨 에어컨리모콘? 같은걸 나눠줘서 당황;

이날 오프닝에 AWS의 Chief Technologist인 Olivier Klein라는 분이 오프닝 연설을 하셨는데 

영어라 들을수 없는 나같은 머글들을 위해 영어로 연설하는 내용을 실시간으로 번역가분이 번역해서 알려주는, 일종의 라디오 수신기? 같은 장치였다.

<center><img width="400" height="250" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/a79f650d-261f-407d-836c-2c94460beb6a"></center><br/>

오프닝 연사중 제일 인상깊었던 내용인데 마트 내부 천장에 수십개에 달하는 AI 카메라를 설치해서 AI 카메라가 마트 내 고객들과 상품 모양, 위치 등을 학습하여

물건을 구매하고 싶은 사람이 굳이 계산대로 계산하지 않고도 카메라들이 인식해서 고객이 물건을 집고, 마트를 나가는 순간 자동 결제될 수 있도록 서비스를 준비하고 있다고 한다. (구겨진 포장지의 상품도 인식할 수 있게끔 학습하는 중이란다.)

이게 현실화 되어 전세계 마트에 보급이되면 더이상 알바는 필요없겠다 라는 생각이..

## 참여 기업 부스 돌아다니기

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/93144564-f531-4188-9dcd-122dab22b35b">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4763758a-1a65-438e-840c-1c3c1094f824"><br/>

(사람들 얼굴은 모자이크 처리)

트랙(리테일, 금융, 통신, 제조)별로 참여한 기업들의 부스도 돌아다녔다. 

눈에 익숙한 대기업들도 있었고, 처음 본 회사들도 다수 있었다. 중간중간 세션 비는시간에 열심히 돌아다님.

## 점심

<center><img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e5ffa2a0-6d01-45e1-b185-25dacdd1cb3f"></center><br/>

밥 안줄줄 알고 식당 어디가야하나.. 고민하고 있었는데 점심 밥을 준다 ㄷㄷ

심지어 구성도 매우 좋음.. 자취하면서 거의 정크푸드만 먹었는데 이렇게 건강을 하나더 챙긴다. 역시 대기업은 다르다


## 세션1. 대범하게, 꼼꼼하게 - KB의 클라우드 최적화 여정

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/582fa818-91ea-42ba-9154-b496c81fbd43">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/b19135ea-d213-40f6-a433-81c00d28fcf0">
<br/>

- FinOps의 필요성
  - 재무 관리 원칙과 클라우드 엔지니어링 및 운영을 결합하여 조직이 클라우드 지출을 더 잘 이해할 수 있도록 하는 분야이자 방법론
  - 클라우드로 인한 비용을 효율화하고 관리하여 조직의 비즈니스 가치를 극대화 하기 위함

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/ed480ef7-1c09-46b5-8c49-5df688b1fbd4"></center><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/5b6804e8-ae71-4898-8a64-7202c3149a15">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2d0d2430-21a7-4769-8c4e-e2ee85d169d8">
<br/>

- Downsizing
  - 의미, 목적성없이 스펙을 크게잡은 aws 서비스들에 대하여 다운사이징을 하여 비용을 절감
  - Ec2와 rds 등 스펙을 상황과 니즈에 맞게 적절한것을 사용하자. (무리한 스펙사용x)

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/8e38a83d-fd7d-4d3c-9744-d93bd183ef89">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/37c92e8e-8cfc-46bb-a92f-c9f61b2dd2b5">
<br/>

- 불필요한 백업/파일 삭제
  - 오래된 Snapshot, 로그, s3 데이터 등 불필요한 리소스들을 정리 및 삭제 함으로써 비용 절감
  - Amazon CloudWatch를 통해 필요한 로그들만 적재하고 모니터링을 최소화하자

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c172c209-862f-49cd-8441-cf42afe76a98"></center><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/1bb2afdf-b81d-4beb-8223-cc3034b4fa92">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4951ad58-9855-4606-bf5e-64018de3f581">
<br/>

- Modernizing
  - 최신세대 타입으로 자원을 변경하여 동일/인하된 비용으로 향상된 성능의 자원을 이용하자.
  - 데이터 생애주기에 따라서 알맞은 s3 타입을 사용 및 변경하자 (각 타입별 가격이 다름)

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/50e41b77-316b-4987-846e-8f6c38748caa">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/a8687222-ee0a-4b64-966f-698d470967ac">
<br/>

- 스케줄링 정책 적용
  - (운영서버 제외) 개발 서버의 경우 워크데이를 제외한 나머지 시간에 가동될 필요가 없음
  - 스케줄링 정책을 통해 퇴근이후, 주말에는 서버를 내리도록 하여 비용을 절감하자.
  - 중앙에서 관리/통제하는 계정을 하나 구성하고 tag policy를 통해 스케줄링 정책을 적용하지않으면 서버를 생성하지 못하도록 막음으로써 비용 최적화 실현
  - Auto Scaling 그룹의 Scheduled Action을 활용하여 스케줄링 적용

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/033fe658-d8f1-4ed7-b10c-f1abfdc838d3">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e4848d88-33a7-42db-86e4-13465b8ae969">
<br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/24997aaf-578d-4664-a04d-1da7e7111e02">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/043ab356-c4e6-4410-b565-719a374c7f4a">
<br/>

- 약정할인 정책 적용
  - aws 서비스는 온디맨드(on-demand), 약정상품(RI/SP)으로 나뉨
    - on-demand: 약정없이 사용한 인스턴스에 대한 비용만 지불
    - RI/SP: 1년또는 3년 기간약정, 시간당 사용량 약정, 온디맨드 대비 최대 70% 절약 가능
  - 운영 서버는 온디맨드보다 약정상품이 적합하다
    - 개발 서버는 워크데이에만 사용하기 때문에 약정보다는 온디맨드-스케줄링 조합이 적
  - aws에서 제공하는 약정 상품용 약정 할인 프로그램들을 적극 활용하자
    - Compute Savings Plan (컴퓨팅 비용 절감계획)
    - EC2 Instance Savings Plan (EC2 인스턴스 비용 절감계획)
    - 기존의 Reserved Instance 보다 유연하게 서비스들에 적용 가능하다.
  - 약정 커버리지 100%가 무조건 좋은건 아니다.
    - 자원변경 계획과 aws 사용비용을 추이하여 약정구입시기를 판단하자.
  - 중앙에서 약정관리 정책기준을 수립하고 관리하자.

- 지속적인 일별 사용량 모니터링을 통한 자원 최적화 관리
  - 특정 시간대에 급격히 코스트가 상승한다던지 발생여부 등 주기적 관리/체크

## 세션2. 기술부채가 뭐야? GS SHOP Builder들의 현대화를 위한 도전

<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c010abee-37e2-4eb8-b4d5-7e185d330ccc"></center><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/56254596-c0ab-4305-8aae-8750186e2fed">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/1610a285-f45e-4e3d-97bb-f289f6cedf9b">
<br/>

- 레거시 시스템은 기술부채를 가지고 있다.
  - 코드레벨, 아키텍처, 문서화/주석 등
- 기존 모놀리식 아키텍처가 마이크로 서비스로 적은 리스크로 전환되려면?
  - 기존 애플리케이션을 중심으로 새로운 애플리케이션을 점진적으로 개발하여 전환해야함

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/13ca390d-7e2c-4fca-8705-6a94346e9c43">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/3d0d8815-8187-4d24-8861-ebeac8f17024">
<br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/df90a3da-db83-441f-980d-7c103fffb444">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/f6a51731-c8d4-4dfb-91c0-44a83adb14f6">
<br/>

- The Strangler Fig Pattern을 활용
  - 소프트웨어 시스템을 점진적으로 재구축하기 위한 디자인패턴이다. 즉, 기존에 구축된 레거시 시스템을 한 번에 완전히 대체하는 대신, 시스템의 일부를 점진적으로 대체하면서 새로운 시스템으로 전환하는 방식 이라는 의미
  - 기존 모놀리식 아키텍처 앞에 프록시 서버를 배치하여 개발이 완료된 신규 서비스부터 하나하나 점진적인 우회(전환)
  - 기존 기능을 전환할때 빈틈 최소화 기법
    - ACL(부패방지계층)을 활용
      - 레거시(legacy) 시스템을 새로운 시스템으로 여러 단계에 걸쳐 이전할 때 사용할 수 있는 패턴(pattern)
      - [Anti-Corruption Layer Pattern](https://junhyunny.github.io/architecture/pattern/anti-corruption-layer-pattern/)
    - 큐/토픽을 활용하여 DB 동기화 적용

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/31003b1f-e3d0-4b4f-b307-00b418ae2f72"></center>

- GS SHOP 주문서비스팀의 주 미션
  - 영업매출 증대를 위해 → 비즈니스 민첩성 향상이 필요
  - 고객 경험 개선을 위해 → 쾌적한 주문속도를 보장해야함
  - 즉, 양질의 코드를 빠르게 배포해야함.

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9bbd7434-c7b4-481b-a366-6fc0d726062a">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/ce40ee3b-d8fc-4a18-8685-f4a7531e21ea">
<br/>

- 페인 포인트 (Pain Point)
  - 복잡하게 얽힌 레거시 프로젝트의 구조상 특정 서비스 배포의 실패 우려로 인한 두려움
  - 마치 빠르게 달리는 열차에 탄것처럼 항상 긴장상태를 유지하게됨

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/d870c84e-fb09-4d8d-ac21-2d7dcb16e23e"></center>

- 접근 전략
  - 한번에 완벽하게 하려 하지말고 0.1v 부터 시작하여 작게작게 점진적으로 실행해보자

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/cd325105-dcba-4b84-bf96-a8ecca9fb053">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e58c7ab5-e50b-4972-9735-bfe38c8e2575">
<br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9e81b85c-1de2-40eb-9e31-426fa11f33dc">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/b2e7a024-57cc-4585-81a4-6461b6febd9c">
<br/>

- 달리는 기차의 바퀴를 갈아 끼우기
  - 특정 서비스를 분리하고 기존연결을 유지하기 위해 위에서 언급한 Strangler Fig Pattern를 활용
  - 기존에는 배치를 통해 주문상태를 변경 하였으나 성격에 따라 DB를 분리하고 실시간 이벤트 발행과 검증테이블 활용을 통해 주문상태를 공유하는 형태로 전환

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/30184fa9-c866-44ff-bece-63824e4ed852"></center>

- 비즈니스 재사용 하기에 집중
  - 누구나 사용할수 있게 기존 레이어 아키텍처를 표준화
  - 데이터 구조에 대한 설계 정의
  - 트랜잭션 분리에 따른 보상처리 적용

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/cfb96961-6a90-44fa-84e2-4984b1928ca6"></center>

- 서비스 아키텍처가 변화함에 따라 일하는 방식 또한 변화
  - CI/CD 파이프라인의 변화
    - 레거시 시스템을 마이크로 서비스로 개선함과 동시에 Blue-Green 배포 전략을 활용하여 무중단 배포가 가능하게끔 변화
  - 모니터링 및 알람 방식 개선
    - 매번 터미널에 들어가서 tail 명령어를 확인하지 않고 웹 콘솔을 통하여 관리가 가능하게끔 개선
    - 문자 및 Teams를 통해 실시간으로 알람을 받도록 개선

## 세션3. 샛별 배송 컬리, 완전 관리형 데이터베이스로 빠르고 안전한 결제 서비스 구축기

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/461c22a3-658e-48b1-a0e6-5f804392fe73">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/e26e6986-cea1-4a06-99c6-8f7b370ab7c5">
<br/>

- payment 서비스의 수요가 점점 증가함에 따라 확장성, 신뢰성이 있는 플랫폼 및 애플리케이션을 구축하는것이 중요함
  - aws는 이러한 플랫폼을 구축할 때 필요한 다양하고 폭넓은 서비스들을 가지고 있음

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/4d64a0fe-a77a-46ad-ab40-c20a081cab51">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2c6d42ce-a152-4e62-9e14-92a1d792ad9d">
<br/>

- AWS RDS로 Blue-Green 배포 전략을 통해 무중단 배포 환경을 구축할 수 있음
  - Blue-Green 배포전략? 애플리케이션 또는 마이크로서비스의 이전 버전에 있던 사용자 트래픽을 이전 버전(blue)과 거의 동일한 새 버전(green)으로 점진적으로 이전하는 애플리케이션 릴리스 모델
  - AWS RDS의 논리적 복제를 통해 현재 product(Blue), 새 product(Green)를 각각 구성
  - 스위치 오버 전에 green 환경에서의 변경사항 테스트
  - 로드 밸런서를 통해 각 사용자들의 다음 트랜잭션을 green 으로 리디렉션 하게끔 구성
  - 모든 트래픽이 정상적으로 green 으로 넘어가면 blue는 오프라인/대기 상태로 진입하며 구성 완료

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2a9d8d93-c93e-4f01-92d5-bed71eee35f7"></center><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/9900f97d-4988-443b-9665-0d0631fda4f6">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/a4243abd-4478-4ebb-bfc5-f65ccc31fb9b">
<br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/fe3ffba3-26b0-4251-b044-cb6f68305493"></center><br/>

- 컬리페이의 간편결제를 위한 도전
  - 분산 환경에서의 데이터 → 트랜잭션이 발생했을때 데이터 정합성 보장을 어떻게 할것인지에 대한 고민
  - 트랜잭션이 하나의 마이크로 서비스에서 실패했을경우 보상 트랜잭션을 적용
  - 데이터 중심의 아키텍처를 구성

## 세션4. 글로벌 K-POP 시대 끊김없는 ‘K팝 덕질’을 위하여 - 위버스의 대용량 트래픽 처리 사례

<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c4847f21-6215-43ae-90f9-5a20193233fc">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/43e0a2b4-604b-40f2-aa55-93dcb951aebf">
<br/>

- 위버스는 전 세계적으로 최대 규모의 팬 커뮤니티 플랫폼으로 성장

<br/><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2a5e2dd9-4dfe-4ba0-99a5-80656ff2b62a">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c7a0927f-4f9b-4737-949c-4bc87b6a4a96">
<br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/6dbb831f-6053-4f97-9ca2-d90651ffffc6">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/daad87f0-92a0-4b24-935e-a7e49ac7d05f">
<br/>

- 스파이크 트래픽 핸들링하기
  - 위버스의 팬이벤트 관련 (ex. 방송 참여 등 선착순 이벤트) 서비스에 스파이크 트래픽이 발생함
  - 트래픽을 핸들링 하기위해 다음과 같은 전략들을 실행함
    - 서버 증설 전략
      - 트래픽 과부하가 예상되는 서비스 시작 30분전에 미리 서버를 증설
    - 캐시 전략
      - Redis with ElasticCache를 활용
    - DB 부하 방지 전략
      - AWS SQS를 통해 트랜잭션을 관리하여 DB 부하 방지
- AWS CloudWatch를 통해 모니터링

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/ed182f40-546c-44c5-b9f1-fca9ca1dda0c"></center>

- TroubleShoot
  - 예상보다 많은 request가 몰려서 넉넉하게 람다 프로비저닝을 구성했음에도 람다 스로틀링이 발생했었음.
  - 프로비저닝된 람다에서만 오류가 발생했음을 확인함. 
  - 로그를 확인해보니 람다가 init(warm) 상태일때 ioredis 또한 생애주기를 같이하여 connect time out 에러가 발생한 문제
  - ioredis 의 Connection 방법을 lazyConnect로 변경하여 해결

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/842fe2aa-fb7b-4af7-b5e2-92bb4f201827"></center><br/>
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/23a60441-2e7a-4dff-951f-8b7d88b45bbd">
<img width="400" height="400" src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/c3d94bf1-040f-48c8-84e8-2b4b7444e3ea">
<br/>

- Architecture Scalability
  - 아키텍처의 확장성을 위해 Serverless 한 Lambda Function들을 적극활용
  - 람다는 버스트 동시성을 통해 한번에 최대 500~3,000개의 실행 환경 인스턴스로 즉시 scale-out 하여 트래픽을 받아낼 수 있음
    - 이후 계정 동시성 한도까지 1분마다 500단위로 scale-out을 하게되며 그동안 500단위를 넘는 요청에 대해선 throttle이 발생함. 
  - 또한 프로비저닝 동시성을 통해 람다를 **미리 scale-out**을 적용하여 대비할수 있음.
  - 보통 필요한 동시성의 경우는 **초당 평균 요청수 x 평균 응답시간**에 해당함
    - ex. 초당 평균요청수 (10000) x 평균 응답시간 (0.15s) = 필요한 동시성 (1500)
  - 캘린더와 연동하여 팬이벤트가 있는 날짜 30분전에 프로비저닝 동시성으로 미리 scale-out 하고 이벤트 종료 시간에 맞춰 scale-in 하여 유연성을 높임

<br/><br/>
<center><img src="https://github.com/ckr3453/ckr3453.github.io/assets/36228833/2dfe8be4-c2c1-491f-bc51-f8d5d4a5136a"></center>

- Alarm & Monitoring
  - 슬랙을 통해 scale-out, in 할때, 예외 발생 등 알람을 받도록 설정
  - 노션 도큐먼트를 활용하여 모니터링 수집지표 관리, 분석

## 후기

재밌었다!

원래 정리로 올린거 보다 더 많은 세션을 들었었는데 뺀 이유는, 너무 회사자랑 느낌이 강한 발표여서 빼버렸다.

세션 내용이 이해가 되지않으면 어떡하나 싶었는데, 다행히 대부분 이해 가능한 내용들이어서 더 와닿은것 같다.

다음은 어느 컨퍼런스를 참여해야할지 고민좀 해봐야겠다 ㅎㅎ

