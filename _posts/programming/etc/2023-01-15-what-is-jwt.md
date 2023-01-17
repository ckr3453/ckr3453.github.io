---
title: "JWT(Json Web Token) - 작성중"
categories: 
    - etc
date: 2023-01-15
last_modified_at: 2023-01-15
toc: true
toc_sticky: true
excerpt: "JWT란 무엇인가?"
---

## JWT 란
JWT는 RFC7519 웹 표준으로 지정되어있고 JSON 객체를 사용해서 토큰 자체에 정보들을 저장하고 있는 웹 토큰이라고 정의할 수 있다. 특히 JWT를 이용하는 방식은 간편하고 쉽게 적용할수있다.

JWT는 Header, Payload, Signature 3개의 부분으로 구성되어있다.
- Header
  - Signature를 해싱하기 위한 알고리즘 정보가 담겨있다.
- Payload
  - 서버와 클라이언트가 주고받을수 있는, 시스템에서 실제로 사용될 정보에 대한 내용을 담고있다.
- Signature
  - 토큰의 유효성 검증을 위한 문자열이다.
  - 이 문자열을 통해 서버에서는 이 토큰이 유효한 토큰인지를 검증할 수 있다.

### 장점 및 단점
JWT는 다음과 같은 장점, 단점들을 가진다.

장점
- 중앙의 인증서버와 데이터 스토어에 대한 의존성이 없기 때문에 시스템 수평확장에 유리하다.
- Base64 URL Safe Encoding을 이용하고 있기때문에 URL, Cookie, Header에 모두 사용 가능하다.

단점
- Payload의 정보가 많아지면 네트워크 사용량이 증가하므로 이를 고려하여 설계해야 한다.
- 토큰이 클라이언트에 저장되므로 서버에서 클라이언트의 토큰을 조작할 수 없다.


## 📣 Reference
