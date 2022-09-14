---
title: "JMX를 활용한 자바 애플리케이션 대시보드 구축 2단계(jmx-exporter + Prometheus + Grafana)"
categories: 
    - java
date: 2022-07-03
last_modified_at: 2022-07-30
# tags:
#     - 태그1
#     - 태그2
#     - tag_test..
toc: true
toc_sticky: true
# toc_label: "MYSELF"
---

#### 개요
`Prometheus` 를 이용하여 `metric` 데이터를 수집하기 위해 설정파일 수정 및 서버 구동방법을 알아본다.

#### `Prometheus` 설정 파일 수정
- `Prometheus` 를 다운로드한 폴더에서 `prometheus.yml` 파일을 다음과 같이 수정한다.

`prometheus.yml`
```
...
(생략)
    static_configs:
      - targets: ["localhost:7890"]
```
(`targets`에 `jmx-exporter`에서 `metric`을 추출한 "host:port"를 입력)

#### `Prometheus` 서버 구동
- 설정 파일을 저장한 뒤 같은 경로에 있는 `prometheus.exe`를 실행하여 서버를 구동한다.
- 서버를 구동한 후 `http://localhost:9090` 으로 접속 후 `metric` 데이터를 수집하고 있는 대상 시스템 목록을 확인한다.
  - ![](https://velog.velcdn.com/images/ckr3453/post/829deeb1-ab94-4e53-8d5e-59a8c1ad59c7/image.png)
  - ![](https://velog.velcdn.com/images/ckr3453/post/3022be28-6b31-4288-8578-d370984ba813/image.png)
  - `prometheus.yml`에서 등록한 `metric` 데이터를 수집하고있는 대상 시스템의 `endpoint`들을 확인할 수 있으며 `state`가 `up`인 경우에 정상적으로 수집하고 있는 것 이다.

이제 다음장에서 Grafana를 활용해서 `metric` 데이터를 시각화 해보자.