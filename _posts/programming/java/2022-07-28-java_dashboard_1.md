---
title: "JMX를 활용한 자바 애플리케이션 대시보드 구축 개요(jmx-exporter + Prometheus + Grafana)"
categories: 
    - java
date: 2022-07-02
last_modified_at: 2022-07-30
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---
#### 계기

어느 날 발주처의 요청으로 기존 ```Spring Framework```기반으로 구축된 웹 애플리케이션의 자원을 모니터링 할수있게끔 해달라는 요청이 들어왔다. (커스터마이징이 가능한 대시보드 형태로)

근데 이걸 웹소켓으로 대시보드를 일일이 구현하기에는 주어진 시간이 너무 촉박했다.

그래서 직접 소스로 개발하지 않고 모니터링 및 데이터 시각화를 지원하는 오픈소스 툴킷들을 이용하여 간단하게 구축하는 방법을 알아본다.

#### 준비
- OS : Window
- jdk : 1.6 이상
  - 현재 나의 프로젝트 jdk 버전은 1.8
- `Tomcat` 서버 설치 및 배포 (이 단계는 설명 생략)
  - 현재 Window OS에 war로 빌드된 `Spring Framework` (v4.3.22) 웹앱이 `Tomcat` (v8.5.31)에 배포되고 있는 상황
- ##### `Prometheus`
  - **설명**
    - SoundCloud사에서 만든 오픈소스 모니터링 툴킷이다. 
    - 대상 시스템으로 부터 각종 모니터링 지표를 수집하여 저장하고 검색할 수 있는 시스템이다. 
    - 수집한 정보를 `Prometheus`가 자체적으로 제공하는 간단한 웹 뷰를 통해 조회할 수 있고 그 안에서 테이블 및 그래프 형태로 볼 수 있다.
      - 하지만 `Grafana` 를 통한 데이터 시각화를 지원하기 때문에 `Grafana`를 통해 확인할 것이다. (자체 웹 뷰로 지원하는 기능이 빈약하기도 하다.)
    - ![](https://velog.velcdn.com/images/ckr3453/post/9c2d21f2-9f7b-40ca-a1dc-b3018d22ea32/image.png)


  - **다운로드**
    - https://prometheus.io/download/
    - ![](https://velog.velcdn.com/images/ckr3453/post/f7c51cbe-69f7-4a88-9786-0a780b1739fe/image.png)
    
- ##### `jmx-exporter`
  - **설명**
    - `Prometheus`가 `Java` 어플리케이션에서 `metric`을 수집하기 위해 만든 `JVM` 에서 동작 가능한 `Java` 어플리케이션이다.
      - `metric`을 수집하기 위해서는 `Java` 어플리케이션에서 `jmx` 가 활성화 되어야한다.
    - `Tomcat` 실행 시 `JVM` 옵션으로 추가하면 `jmx-exporter`가 어플리케이션으로부터 `metric`을 수집하고 `Prometheus`가 `HTTP` 통신을 통해 `metric` 데이터를 가져갈 수 있게 /metrics 라는 `HTTP endpoint`를 제공한다.
  - **다운로드**
    - https://github.com/prometheus/jmx_exporter
      - `jmx-exporter`는 jdk 7이상 호환 or jdk 6 호환 가능한 버전으로 구성되어있다.
      - (현재 내가 진행하는 프로젝트는 jdk 8버전 기반이므로 `jmx_prometheus_javaagent-0.17.0.jar`를 다운로드 하였음)
    - ![](https://velog.velcdn.com/images/ckr3453/post/99e657bd-00b2-48f5-b61e-a5985bced5d3/image.png)

- ##### `Grafana`
  - **설명**
    - `metric` 데이터를 시각화 하는데 가장 최적화된 대시보드를 제공해주며 Grafana Labs에서 관리하고 있는 오픈소스 툴킷이다.
    - `Prometheus`는 물론이고 `InfluxDB`, `Elasticsearch` 등 여러 데이터 소스와 통합이 가능하도록 지원하고있다.
    - 또한 일반 사용자들이 만들어놓은 대시보드 구성을 import 하여 사용할 수 있으며 커스터마이징 또한 용이하다.
  - **다운로드**
    - https://grafana.com/grafana/download
    - ![](https://velog.velcdn.com/images/ckr3453/post/dba8ff07-be75-450d-8d74-5ae493828c92/image.png)
    - 다운로드를 받고 설치를 하면 `Grafana`라는 이름으로 서비스 등록이 된다.
    - ![](https://velog.velcdn.com/images/ckr3453/post/4ffb6c7c-c91f-4f06-ae0e-1c2786c7d846/image.png)


#### 단계 구성

단계별로 크게 나눠 보면 다음과 같다.

1. `Tomcat` 서버에 `Java Option`에 `jmx-exporter`를 추가하여 `Java` 어플리케이션의 `jmx`를 기반으로 `metric`를 추출한다. (/metrics 라는 `HTTP endpoint`를 제공)

2. `Prometheus`를 이용하여 `HTTP endpoint`로 `GET` 요청을 날려 `metric` 정보를 수집한다.

3. `Prometheus`가 수집한 `metric`을 `Grafana`를 이용하여 데이터 시각화하여 구성한다. (대시보드 구성)


다음장부터 해당 단계별로 어떻게 구성되는지 상세히 설명하겠다.