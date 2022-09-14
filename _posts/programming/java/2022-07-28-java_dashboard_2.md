---
title: "JMX를 활용한 자바 애플리케이션 대시보드 구축 1단계(jmx-exporter + Prometheus + Grafana)"
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
excerpt: "JMX를 활용한 자바 애플리케이션 대시보드 구축에 대해 알아보자"
---
#### 개요
`metric` 데이터를 수집하기 위해 `Tomcat`과 `jmx-exporter`를 활용하여 `metric`를 추출하는 작업을 진행한다.

#### `jmx-exporter`의 설정 파일 생성

- `jmx-exporter`를 실행하려면 `yaml` 포맷의 config파일이 필요하다. config 파일의 내용을 다음과 같이 작성하자.

`jmx_config.yaml`
```
---
startDelaySeconds: 5
ssl: false
lowercaseOutputName: false
lowercaseOutputLabelNames: false
```
(옵션에 대한 내용들은 https://github.com/prometheus/jmx_exporter#configuration 를 참고)

#### `Tomcat` 설정을 통한 프로젝트 배포 시 `metric` 추출

- 이제 `Tomcat` 에 Java Option을 추가하자. `tomcat[version]w.exe` 파일 실행 혹은 `cmd`에 //ES 명령어를 통해 `Tomcat` GUI 환경을 실행하자.

- `Java` 탭 > `Java Options` 안에 다음과 같은 내용을 추가한다.
  - `javaagent:[실제파일경로]/jmx_prometheus_javaagent-0.17.0.jar=[port]:[실제파일경로]/jmx_config.yaml`
  - 여기서 입력한 `[port]`에 `metric` 추출을 위한 `HTTP endpoint`가 자동 설정된다.
  - ![](https://velog.velcdn.com/images/ckr3453/post/65b26e71-6182-4f4b-b51b-049694575788/image.png)

- `General` 탭으로 돌아와서 적용을 누른뒤 `Start`를 클릭하여 `Tomcat` 서버를 구동한다.
  - ![](https://velog.velcdn.com/images/ckr3453/post/837cc89e-725b-4601-934b-bc0596b90814/image.png)
  - (이때 정상적으로 시작되지 않으면 jmx_exporter의 파일경로를 다시 확인해보거나 설정파일 내용을 다시 확인해보자. 필요없는 공백이 포함되는경우 정상 실행되지 않는다.)
  
- 정상적으로 구동되면 `http://localhost:[port]/metrics`로 접속하여 확인해본다.
  - 이때 `[port]`는 위에 Java Options에서 입력한 `[port]`를 입력한다.
  - ![](https://velog.velcdn.com/images/ckr3453/post/29d115c7-2b18-4eb9-b54f-cfc51c365e57/image.png)
  - 해당 화면과 같은 내용이 출력 된다면 `metric` 추출에 성공한 것이다.
  
추출을 완료하였으니 다음장에서 `Prometheus` 서버로 추출한 `metric` 수집을 해보자.