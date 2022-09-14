---
title: "JMX를 활용한 자바 애플리케이션 대시보드 구축 3단계(jmx-exporter + Prometheus + Grafana)"
categories: 
    - java
date: 2022-07-04
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
`Prometheus`가 수집한 `metric` 데이터를 `Grafana` 를 이용하여 시각화 한 뒤 대시보드를 구성해본다.

#### Grafana를 이용하여 수집한 metric을 대시보드로 구성
- `Grafana`를 설치했으면 `http://localhost:3000`로 접근이 가능하다.  ID: admin / PW: admin 으로 로그인 진행
  - ![](https://velog.velcdn.com/images/ckr3453/post/9a0db909-d2ed-43f5-91e3-421850e23e74/image.png)
- 로그인 후 첫 화면 중앙에 `Add your first data source` 를 클릭한 뒤 첫번째로 보이는 `Prometheus`를 선택
  - ![](https://velog.velcdn.com/images/ckr3453/post/a400449e-abd0-4088-8542-8a90f4ee4732/image.png)
  - ![](https://velog.velcdn.com/images/ckr3453/post/27bb1830-8e00-47cb-8e91-c9cd3eccece0/image.png)

- 그 후 상세 창에 URL에 `metric`을 수집하고있는 `Prometheus` url를 입력한 뒤 하단의 `Save & Test`를 클릭하여 잘 연결이 되었는지 확인
  - ![](https://velog.velcdn.com/images/ckr3453/post/182077d6-bd4c-42dd-8bc7-ff134e9736fa/image.png)
  - ![](https://velog.velcdn.com/images/ckr3453/post/63fe4470-1848-4883-b4fb-7b6632fdb470/image.png)

- `https://grafana.com/grafana/dashboards/` 에서 원하는 대시보드 템플릿을 선택 후 해당 `ID`를 클립보드에 복사
  - ![](https://velog.velcdn.com/images/ckr3453/post/5d160cbf-2b89-43d0-ad26-1cd75f6ce505/image.png)

- 대시보드 템플릿 import를 위해 좌측 `Dashboard` 메뉴의 `import`를 클릭
  - ![](https://velog.velcdn.com/images/ckr3453/post/fbf06cf5-169b-4a05-a204-148bf2633bbe/image.png)
  - ![](https://velog.velcdn.com/images/ckr3453/post/28940054-8859-46ad-93fb-d55fdafbba3b/image.png)
  - `Load`를 클릭하면 해당 ID의 대시보드 템플릿 내용으로 자동 입력되며 하단의 `Load`를 다시 클릭하여 템플릿을 적용
    
- 해당 템플릿으로 적용된 대시보드를 확인
  - ![](https://velog.velcdn.com/images/ckr3453/post/7755f766-6900-445c-945c-63fdbc08d9c3/image.png)
  
#### 마치며
이로써 `Grafana`에 `Java` 어플리케이션을 모니터링할 수 있는 대시보드를 구축할 수 있게 되었다. 

조금 더 찾아보면 추가적인 설정을 통해 모니터링 내용을 사용자가 원하는 채널(이메일, 디스코드, 슬랙 등등)로 알람을 받을 수 있게 적용할 수 있으니 필요하다면 추가 설정을 통해 적용해보자.


#### Reference

1. https://velog.io/@jsj3282/20.-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-API%EC%9D%B8-JMX (JMX api에 관한 내용)
2. https://youngram5.tistory.com/entry/%ED%94%84%EB%A1%9C%EB%A9%94%ED%85%8C%EC%9A%B0%EC%8A%A4-%EA%B7%B8%EB%9D%BC%ED%8C%8C%EB%82%98-%EC%9E%90%EB%B0%94-%EC%96%B4%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81-1-tomcat-%EC%84%A4%EC%B9%98?category=979783 (prometheus + grafana 구축과정)
3. https://prometheus.io/docs/introduction/overview/ (prometheus 공식 docs)
4. https://grafana.com/docs/ (grafana 공식 docs)