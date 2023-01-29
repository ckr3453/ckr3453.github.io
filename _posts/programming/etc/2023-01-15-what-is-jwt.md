---
title: "JWT(Json Web Token)"
categories: 
    - etc
date: 2023-01-15
last_modified_at: 2023-01-30
toc: true
toc_sticky: true
excerpt: "JWT란 무엇인가?"
---

## JWT 란
JWT는 RFC7519 웹 표준으로 지정되어있고 JSON 객체를 사용해서 토큰 자체에 정보들을 저장하고 있는 웹 토큰이라고 정의할 수 있다. 특히 JWT를 이용하는 방식은 간편하고 쉽게 적용할수있다.

<center><img src="https://user-images.githubusercontent.com/36228833/215335585-1cb33f15-c89c-4ba0-a82e-a2faed609851.png"></center><br/>

JWT는 .을 기준으로 `Header`, `Payload`, `Signature` 3개의 부분으로 구성되어있다.

- `Header`
  - `Signature`를 암호화하기 위한 알고리즘 정보가 담겨있다.
  ```json
  {
       "alg" : "HS512", // Signature 암호화 알고리즘 종류 (HS256, HS384, SHA256,..)
       "typ" : "JWT" // 토큰 유형
  }
  ```
- `Payload`
  - 서버와 클라이언트가 주고받을수 있는, 시스템에서 실제로 사용될 정보에 대한 내용을 담고있다.
  - key-value로 이루어진 한쌍의 정보를 Claim 이라고 한다.
  - `Payload`는 용도에 맞게 이미 정해져있는 Registered Claim과 그 외 사용자 지정 클레임으로 구분된다.
  ```json
  {
      // Registered Claim
      "iss": "ckr", // 토큰 발급자 (issuer)
      "sub": "jwt-test", // 토큰 제목 (subject)
      "aud": "user", // 토큰 대상자 (audience)
      "exp": "1485270000000", // 토큰의 만료시간 (expiraton), 시간은 NumericDate 형식으로 되어있어야 하며 (예: 1480849147370) 언제나 현재 시간보다 이후로 설정되어있어야합니다.
      "nbf": "1185270003000", // Not Before 를 의미하며, 토큰의 활성 날짜와 비슷한 개념입니다. 여기에도 NumericDate 형식으로 날짜를 지정하며, 이 날짜가 지나기 전까지는 토큰이 처리되지 않습니다.
      "iat": "1482533320000", // 토큰이 발급된 시간 (issued at), 이 값을 사용하여 토큰의 age 가 얼마나 되었는지 판단 할 수 있습니다.
      "jti": "identifier1252#@%12a", // JWT의 고유 식별자로서, 주로 중복적인 처리를 방지하기 위하여 사용됩니다. 일회용 토큰에 사용하면 유용합니다.

      // 사용자가 임의로 등록한 클레임
      "test123" : "test111" 
  }
  ```
- `Signature`
  - `Header`와 `Signature`의 인코딩된 값 + secret-key 값으로 조합한 값을 `Header`에서 정의한 알고리즘으로 암호화를 한다.
  - 여기서 사용되는 secret-key 값은 보통 임의의 값을 특정 알고리즘으로 암호화한 값을 사용한다.
  - 토큰의 위/변조 여부를 확인 및 유효성 검증을 하는데 사용한다.
  
## JWT 활용

주로 JWT는 클라이언트에 대한 인증 및 권한 확인을 위해 주로 사용된다.

<center><img src="https://user-images.githubusercontent.com/36228833/215338648-b976c1b3-26aa-4703-8c16-feed7f2d3c43.png"></center>

### jwt가 없는 경우
1. 클라이언트는 jwt 발급을 위해 필요한 정보(ex. 아이디/패스워드)들을 서버에 던지며 요청을 한다.
2. 서버는 클라이언트가 던진 정보들의 유효성을 검증한다.
  - 유효한 값인경우 jwt를 header에 담아서 보낸다.
  - 유효하지 않을경우 예외처리를 한다.

### jwt가 있는 경우
1. 클라이언트가 서버에서 발급받은 jwt를 header에 담아서 서버에 api를 요청한다.
2. 서버는 클라이언트가 던진 jwt를 검증한다.
  - 유효한 경우 클라이언트가 요청한 api를 수행한다.
  - 유효하지 않을경우 토큰 값에 따라 401 혹은 403 response를 보낸다.

## 장점 및 단점
JWT는 다음과 같은 장점, 단점들을 가진다.

장점
- 중앙의 인증서버와 데이터 스토어에 대한 의존성이 없기 때문에 시스템 수평확장에 유리하다.
- Base64 URL Safe Encoding을 이용하고 있기때문에 URL, Cookie, Header에 모두 사용 가능하다.

단점
- Payload의 정보가 많아지면 네트워크 사용량이 증가하므로 이를 고려하여 설계해야 한다.
- 토큰이 클라이언트에 저장되므로 서버에서 클라이언트의 토큰을 조작할 수 없다.

## 📣 Reference
[JSON 웹 토큰](https://ko.wikipedia.org/wiki/JSON_%EC%9B%B9_%ED%86%A0%ED%81%B0)
[jwt.io](https://jwt.io/)