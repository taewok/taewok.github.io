---
title: "[JWT] Access Token과 Refresh Token 로그인 흐름 이해하기"
date: 2023-06-10T23:22:00
categories: [backend]
tags: [jwt, authentication, security, nodejs, web-dev]
description: "JWT 기반 인증에서 Access Token과 Refresh Token을 함께 사용하는 이유와 기본 로그인 흐름을 정리했습니다."
custom_style: true
---

## 들어가며

JWT를 처음 사용하면 로그인 후 토큰을 저장하고 API 요청에 붙이는 흐름은 금방 이해할 수 있습니다.

하지만 실제 서비스에 적용하려고 하면 곧바로 이런 질문을 만나게 됩니다.

Access Token이 만료되면 어떻게 해야 할까요?

사용자가 계속 로그인 상태를 유지하게 하려면 어떻게 해야 할까요?

이 문제를 해결하기 위해 보통 Access Token과 Refresh Token을 함께 사용합니다.

---

## JWT는 어떤 구조일까요?

JWT는 세 부분으로 구성됩니다.

```txt
Header.Payload.Signature
```

Header에는 토큰 타입과 서명 알고리즘 정보가 들어갑니다.

Payload에는 사용자 id, 권한, 만료 시간 같은 정보가 들어갈 수 있습니다.

Signature는 토큰이 중간에 변조되지 않았는지 확인하는 데 사용됩니다.

중요한 점은 Payload는 누구나 decode해서 읽을 수 있다는 것입니다.

그래서 비밀번호나 민감한 개인정보는 JWT 안에 넣으면 안 됩니다.

---

## Access Token

Access Token은 API 요청을 보낼 때 인증 수단으로 사용하는 토큰입니다.

```http
Authorization: Bearer access-token
```

보통 만료 시간을 짧게 가져갑니다.

예를 들면 15분, 30분처럼 설정할 수 있습니다.

만료 시간이 짧으면 토큰이 유출되더라도 피해 범위를 줄일 수 있습니다.

---

## Refresh Token

Refresh Token은 새로운 Access Token을 발급받기 위해 사용하는 토큰입니다.

Access Token보다 만료 시간을 길게 가져갑니다.

예를 들면 7일, 14일, 30일처럼 설정할 수 있습니다.

Refresh Token은 유출되면 위험하기 때문에 보통 HttpOnly Cookie에 저장하는 방식을 많이 사용합니다.

---

## 기본 로그인 흐름

Access Token과 Refresh Token을 함께 사용하는 흐름은 보통 다음과 같습니다.

1. 사용자가 로그인합니다.
2. 서버가 Access Token과 Refresh Token을 발급합니다.
3. 클라이언트는 Access Token으로 API 요청을 보냅니다.
4. Access Token이 만료되면 서버가 401 응답을 반환합니다.
5. 클라이언트는 Refresh Token으로 새 Access Token을 요청합니다.
6. 새 Access Token을 받은 뒤 실패했던 요청을 다시 보냅니다.

이 구조를 사용하면 Access Token의 만료 시간을 짧게 가져가면서도, 사용자가 자주 다시 로그인하지 않게 만들 수 있습니다.

---

## Node.js에서 토큰 발급 예시

```js
const jwt = require("jsonwebtoken");

const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET;
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET;

const generateAccessToken = (user) => {
  return jwt.sign(
    {
      id: user.id,
      email: user.email,
      role: user.role,
    },
    ACCESS_TOKEN_SECRET,
    {
      expiresIn: "15m",
    },
  );
};

const generateRefreshToken = (user) => {
  return jwt.sign(
    {
      id: user.id,
    },
    REFRESH_TOKEN_SECRET,
    {
      expiresIn: "7d",
    },
  );
};
```

secret key는 코드에 직접 쓰지 않고 환경 변수로 관리해야 합니다.

---

## Refresh Token은 서버에서도 관리하는 것이 좋아요

Refresh Token을 단순히 발급만 하고 서버에서 따로 관리하지 않으면 문제가 생길 수 있습니다.

예를 들어 사용자가 로그아웃했는데도 이전 Refresh Token으로 계속 새 Access Token을 발급받을 수 있다면 위험합니다.

그래서 실제 서비스에서는 Refresh Token을 DB나 Redis에 저장하고, 갱신 요청이 들어올 때 서버에 저장된 값과 비교하는 방식을 사용합니다.

로그아웃 시에는 저장된 Refresh Token을 제거하거나 무효화합니다.

---

## 저장 위치 고민하기

Access Token은 메모리, 상태 관리 도구, localStorage 등 여러 위치에 저장할 수 있습니다.

Refresh Token은 가능하면 HttpOnly Cookie에 저장하는 방식이 더 안전합니다.

HttpOnly Cookie는 JavaScript에서 직접 읽을 수 없기 때문에 XSS 공격으로 토큰이 탈취될 가능성을 줄일 수 있습니다.

물론 쿠키를 사용하면 CSRF 대응도 함께 고려해야 합니다.

---

## 마무리

JWT 인증은 단순히 토큰 하나를 발급하는 것으로 끝나지 않습니다.

Access Token은 짧게 유지하고, Refresh Token으로 새 Access Token을 발급받는 흐름을 함께 설계해야 사용자 경험과 보안을 균형 있게 가져갈 수 있습니다.

특히 Refresh Token은 서버에서 저장하고 검증할 수 있게 만들어야 로그아웃, 강제 만료, 보안 사고 대응이 쉬워집니다.
