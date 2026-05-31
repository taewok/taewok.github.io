---
title: "[React] jwt-decode로 JWT Payload 읽기"
date: 2023-06-03T23:10:00
categories: [react]
tags: [react, jwt, jwt-decode, frontend]
description: "React에서 jwt-decode를 사용해 JWT payload를 읽고 사용자 정보를 UI에 활용하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

로그인 후 서버에서 access token을 받으면, 그 안에 사용자 id나 권한 같은 기본 정보가 들어 있는 경우가 있습니다.

이 정보를 화면에 바로 보여주고 싶을 때 매번 사용자 정보 API를 다시 호출하는 대신, JWT의 payload를 읽어 활용할 수 있습니다.

React에서는 `jwt-decode` 라이브러리를 사용하면 JWT payload를 간단히 읽을 수 있습니다.

---

## JWT 구조 이해하기

JWT는 보통 세 부분으로 나뉩니다.

```txt
Header.Payload.Signature
```

가운데에 있는 Payload에는 사용자 id, 권한, 만료 시간 같은 정보가 들어갈 수 있습니다.

```json
{
  "sub": "user-1",
  "nickname": "taewok",
  "role": "USER",
  "exp": 1712061029
}
```

여기서 꼭 기억해야 할 점이 있습니다.

JWT decode는 복호화가 아닙니다. Base64로 인코딩된 값을 읽기 쉬운 형태로 바꾸는 것에 가깝습니다.

그래서 비밀번호나 주민등록번호 같은 민감한 정보는 절대 JWT payload에 넣으면 안 됩니다.

---

## jwt-decode 설치하기

```bash
npm install jwt-decode
```

설치 후 import해서 사용할 수 있습니다.

```tsx
import { jwtDecode } from "jwt-decode";
```

---

## 기본 사용법

```tsx
import { jwtDecode } from "jwt-decode";

interface TokenPayload {
  sub: string;
  nickname: string;
  role: "USER" | "ADMIN";
  exp: number;
}

const token = localStorage.getItem("accessToken");

if (token) {
  const decoded = jwtDecode<TokenPayload>(token);

  console.log(decoded.nickname);
  console.log(decoded.role);
}
```

TypeScript를 사용한다면 payload 타입을 직접 정의해두는 것이 좋습니다.

이렇게 하면 `decoded.nickname`, `decoded.role`처럼 값을 사용할 때 타입 도움을 받을 수 있습니다.

---

## 로그인 응답에서 토큰 읽기

로그인 API 응답 헤더에서 `Authorization` 값을 받는 경우도 있습니다.

```tsx
const handleLogin = async (credentials: LoginRequest) => {
  const response = await api.post("/api/auth/login", credentials);

  const authHeader = response.headers.authorization;
  const token = authHeader?.replace("Bearer ", "");

  if (!token) return;

  const decoded = jwtDecode<TokenPayload>(token);

  localStorage.setItem("accessToken", token);
  setUser({
    id: decoded.sub,
    nickname: decoded.nickname,
    role: decoded.role,
  });
};
```

`Bearer ` 문자열을 제거한 뒤 실제 토큰만 decode합니다.

---

## 만료 시간 확인하기

JWT payload에는 `exp`가 들어 있는 경우가 많습니다.

`exp`는 초 단위 Unix timestamp입니다.

```tsx
const isExpired = (exp: number) => {
  return Date.now() >= exp * 1000;
};
```

현재 시간은 밀리초 단위이고, `exp`는 초 단위이기 때문에 `1000`을 곱해 비교합니다.

```tsx
if (isExpired(decoded.exp)) {
  console.log("토큰이 만료되었습니다.");
}
```

---

## try-catch로 감싸기

토큰이 비어 있거나 형식이 잘못된 경우 `jwtDecode`에서 에러가 발생할 수 있습니다.

그래서 실제 코드에서는 try-catch로 감싸는 편이 안전합니다.

```tsx
try {
  const decoded = jwtDecode<TokenPayload>(token);
  return decoded;
} catch {
  localStorage.removeItem("accessToken");
  return null;
}
```

잘못된 토큰이면 저장소에서 제거하고 로그인 상태를 초기화하는 식으로 처리할 수 있습니다.

---

## 언제 사용하면 좋을까요?

`jwt-decode`는 화면에 표시할 기본 사용자 정보를 빠르게 읽을 때 유용합니다.

예를 들어 닉네임, 역할, 토큰 만료 시간처럼 이미 토큰에 포함된 정보를 확인할 수 있습니다.

하지만 최신 사용자 정보가 반드시 필요하거나, 보안상 중요한 정보가 필요하다면 서버 API를 다시 호출하는 것이 더 적절합니다.

---

## 마무리

`jwt-decode`를 사용하면 React에서 JWT payload를 간단히 읽을 수 있습니다.

다만 decode는 검증이나 복호화가 아니기 때문에 보안 기능으로 오해하면 안 됩니다.

토큰에는 최소한의 정보만 넣고, 민감하거나 최신성이 중요한 데이터는 서버를 통해 다시 확인하는 흐름으로 설계하는 것이 좋습니다.
