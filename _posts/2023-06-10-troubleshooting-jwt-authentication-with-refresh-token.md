---
title: "JWT 로그인 방식 완벽 이해: Access/Refresh Token 기반 보안 최적화"
date: 2023-06-10T23:22:00
categories: [backend]
tags: [jwt, authentication, security, nodejs, web-dev]
description: "단순한 JWT 개념을 넘어 실무에서 발생하는 보안 이슈와 Refresh Token을 활용한 로그인 유지 전략을 개발자 관점에서 정리합니다."
custom_style: true
---

---

## 🚀 도입: 왜 우리는 JWT를 쓰는가?

초기 프로젝트를 진행할 때 세션(Session) 방식을 주로 사용했다. 하지만 서버가 늘어나면서 `세션 동기화 문제`에 직면했고, 이를 해결하기 위해 상태를 유지하지 않는(Stateless) `JWT(JSON Web Token)`를 도입하게 되었다.

단순히 토큰을 주고받는 것은 쉬웠지만, 실제 서비스에 적용하면서 `"만료된 토큰을 어떻게 처리할 것인가?"`와 `"보안과 사용자 경험 사이의 접점을 어떻게 찾을 것인가?"`에 대한 고민이 깊어졌다. 이 글에서는 그 고민의 결과인 JWT 구조와 최적화된 인증 흐름을 정리해본다.

---

## 💡 JWT의 구조와 특징

JWT는 세 개의 점(`.`)으로 구분된 세 가지 부분으로 구성된다. 각 부분은 단순 인코딩된 문자열이기에 누구나 디코딩할 수 있다는 점을 항상 유의해야 했다.

### 1. Header

토큰의 타입과 사용 중인 해싱 알고리즘을 정의한다.

```jsx
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2. Payload

사용자의 ID, 권한 등 실제 담고 싶은 데이터를 포함한다. **비밀번호 같은 민감한 정보는 절대 넣지 말아야 한다.**

```jsx
{
  "sub": "user123",
  "name": "Gemini",
  "iat": 1516239022
}
```

### 3. Signature

서버만 알고 있는 **Secret Key**를 이용해 생성된다. 토큰이 중간에 변조되지 않았음을 증명하는 핵심 장치다.

---

## 🛠️ 실전 코드 예시: 토큰 생성 (Node.js)

실무에서는 주로 `jsonwebtoken` 라이브러리를 사용한다. 아래는 실제 프로젝트에서 사용했던 토큰 발급 로직이다.

```jsx
const jwt = require("jsonwebtoken");

const SECRET_KEY = "my_super_secret_key"; // 환경변수 처리가 필수다!

// 1. Access Token 발급 (짧은 수명)
const generateAccessToken = (user) => {
  return jwt.sign({ id: user.id, email: user.email }, SECRET_KEY, {
    expiresIn: "15m",
  });
};

// 2. Refresh Token 발급 (긴 수명)
const generateRefreshToken = (user) => {
  return jwt.sign({ id: user.id }, SECRET_KEY, { expiresIn: "7d" });
};

// 실행 결과 예시
// AccessToken: eyJhbGciOiJIUzI1NiIsInR5cCI... (15분 후 만료)
```

<div class="warning-box">
    <strong>⚠️ 실무 주의사항:</strong> SECRET_KEY가 노출되면 누구나 관리자 권한의 토큰을 위조할 수 있다. 반드시 <code>.env</code> 파일에 저장하고 관리해야 한다.
</div>

---

## 🛡️ Access Token + Refresh Token 전략

AccessToken만 쓰면 만료 시마다 사용자가 다시 로그인해야 한다. 이를 해결하기 위해 `Refresh Token`을 도입하며 겪었던 프로세스를 정리했다.

1. `로그인 성공`: 서버는 AT(Access)와 RT(Refresh)를 모두 발급한다.
2. `저장`: 클라이언트는 AT를 메모리에, RT를 HttpOnly Cookie에 저장하는 것이 보안상 유리하다.
3. `요청`: AT를 Header에 담아 API를 호출한다.
4. `만료`: AT가 만료되면 서버는 401 에러를 반환한다.
5. `갱신`: 클라이언트는 저장된 RT를 서버로 보내 새로운 AT를 요청하고, 서버는 DB의 RT와 대조 후 재발급한다.

---

## 📊 비교 정리: Session vs JWT

| 항목        | Session                                  | JWT                                    |
| :---------- | :--------------------------------------- | :------------------------------------- |
| 저장소      | 서버 메모리/DB                           | 클라이언트 (Stateless)                 |
| 확장성      | 서버 확장이 어려움 (Sticky Session 필요) | 매우 용이함 (어느 서버든 검증 가능)    |
| 보안        | 상대적으로 안전 (서버가 제어권 가짐)     | 토큰 탈취 시 만료 전까지 통제가 어려움 |
| 데이터 크기 | 작음 (ID만 전송)                         | 큼 (Payload 정보를 모두 포함)          |

---

## 📝 실전 포인트 & 인사이트

<div class="highlight-box">
    <strong>언제 써야 하는가?</strong><br/>
    마이크로서비스 아키텍처(MSA)나 모바일 앱과 연동되는 웹 서비스라면 JWT가 필수적이다. 서버의 상태를 유지하지 않아도 되기 때문에 수평적 확장에 매우 유리하다.
</div>

<div class="highlight-box">
    <strong>실수하기 쉬운 부분</strong><br/>
    Payload에 민감한 정보를 넣는 실수다. JWT는 암호화가 아니라 '인코딩'된 것이므로, Base64 디코딩만으로 내용을 다 볼 수 있다. 오직 식별을 위한 최소 정보만 담자.
</div>

<div class="highlight-box">
    <strong>개발 중 겪은 삽질(Troubleshooting)</strong><br/>
    RT를 DB에 저장하지 않고 검증만 했을 때, 사용자의 계정을 강제로 로그아웃시켜야 하는 상황(보안 사고 등)에서 토큰을 무효화할 방법이 없었다. 결국 블랙리스트 방식이나 DB 대조 방식을 병행해야 한다는 것을 깨달았다.
</div>

---

## ✅ 마무리하며

JWT는 현대 웹 개발에서 매우 강력한 도구지만, 그만큼 `보안 설정`이 중요하다. 무분별하게 길게 설정된 만료 시간은 보안 구멍이 될 수 있다. `Access Token은 짧게(15~30분), Refresh Token은 적절히(7~14일) 가져가되 반드시 RT는 서버 측에서 검증 로직을 거치도록 설계하자.`

궁금한 점이나 본인만의 토큰 관리 노하우가 있다면 댓글로 공유 부탁드린다!
