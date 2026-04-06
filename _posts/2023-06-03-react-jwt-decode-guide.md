---
title: "[React] JWT Payload 읽기: jwt-decode로 사용자 정보 추출하는 실무 가이드"
date: 2023-06-03T23:10:00
categories: [react]
tags: [react, jwt, jwt-decode, frontend]
description: "React 환경에서 API 응답으로 받은 JWT 토큰의 payload를 jwt-decode 라이브러리를 통해 안전하게 해독하고 사용자 정보를 활용하는 방법을 알아봅니다."
custom_style: true
---

---

<p>
    <strong>💡 도입: 왜 JWT를 디코딩해야 했을까?</strong><br/>
    최근 프로젝트를 진행하며 로그인 후 사용자의 닉네임과 권한(Role)을 즉시 UI에 반영해야 하는 상황을 겪었습니다. 백엔드에서 별도의 유저 정보 API를 제공하기도 하지만, 매번 네트워크 요청을 보내기엔 비효율적이라는 생각이 들었죠. 이때 <b>accessToken의 Payload</b>에 담긴 데이터를 클라이언트에서 직접 읽어 해결한 과정을 공유합니다.
</p>

## <b>JWT와 Payload의 이해</b>

로그인 성공 시 서버는 `accessToken`을 발급합니다. 이 토큰은 `Header.Payload.Signature` 세 부분으로 구성되는데, 우리가 필요한 정보(ID, 이름, 만료시간 등)는 중간의 **Payload**에 담겨 있습니다.

<div style="background-color: #fff3cd; border: 1px solid #ffeeba; padding: 15px; border-radius: 5px; margin: 15px 0; color:black;">
    <strong>⚠️ 주의사항</strong><br/>
    JWT 디코딩은 '복호화(Decryption)'가 아니라 <b>'Base64 디코딩'</b>입니다. 누구나 읽을 수 있는 데이터이므로 비밀번호나 민감한 개인정보를 토큰에 담아서는 절대 안 된다는 것을 다시 한번 깨달았습니다.
</div>

---

## <b>라이브러리 설치 및 기본 설정</b>

직접 Base64를 파싱할 수도 있지만, 안정성과 만료 시간(exp) 체크 등 편의성을 위해 `jwt-decode` 라이브러리를 사용했습니다.

```jsx
npm install jwt-decode
# 또는
yarn add jwt-decode
```

---

## <b>실전 적용 코드: 로그인 핸들러</b>

실제 Axios를 활용한 로그인 로직에서 토큰을 추출하고 유저 정보를 뽑아내는 흐름입니다.

```jsx
import { jwtDecode } from "jwt-decode";

const handleLogin = async (credentials) => {
  try {
    const res = await api.post(`/api/auth/login`, credentials);

    // 1. Authorization 헤더에서 Bearer 토큰 추출
    const authHeader = res.headers.authorization;
    const token = authHeader.split(" ")[1];

    if (token) {
      // 2. jwtDecode를 이용한 페이로드 추출
      const decoded = jwtDecode(token);

      console.log("해독된 유저 정보:", decoded);

      /* 출력 예시 (decoded 객체):
         {
           "sub": "1234567890",
           "name": "Gemini",
           "role": "ADMIN",
           "iat": 1516239022,
           "exp": 1712061029
         }
      */

      // 3. 전역 상태(Context/Redux)에 저장하여 UI에서 즉시 사용
      setUserInfo({
        name: decoded.name,
        role: decoded.role,
      });

      localStorage.setItem("accessToken", token);
    }
  } catch (error) {
    console.error("로그인 실패:", error);
  }
};
```

---

## <b>실무 핵심 포인트 (Experience Insight)</b>

개발하면서 겪은 시행착오와 팁을 정리했습니다.

- **언제 써야 하는가?**
  - API 추가 호출 없이 사용자 이름, 등급 등을 즉시 표시하고 싶을 때.
  - 토큰의 `exp`(만료 시간)를 체크하여 자동으로 로그아웃 처리하거나 토큰 갱신(Refresh) 로직을 실행할 때 유용합니다.
- **실수하기 쉬운 부분**
  - `jwt-decode`는 라이브러리 버전에 따라 `import jwt_decode from 'jwt-decode'` 또는 `import { jwtDecode } from 'jwt-decode'`로 방식이 다를 수 있습니다. (최신 버전은 후자 지향)
  - 토큰이 유효하지 않은 형식일 때 호출하면 에러가 발생하므로 `try-catch`로 감싸는 것이 안전합니다.

---

## <b>방식 비교: 디코딩 vs 유저 정보 API</b>

| 비교 항목     | jwt-decode 활용                | 유저 정보 API 별도 호출          |
| :------------ | :----------------------------- | :------------------------------- |
| 속도          | 로컬에서 즉시 실행 (매우 빠름) | 네트워크 지연 발생 (HTTP 요청)   |
| 데이터 신선도 | 토큰 갱신 전까지 고정          | 호출 시점의 최신 DB 상태 반영    |
| 서버 부하     | 없음                           | API 호출마다 서버 자원 사용      |
| 추천 상황     | UI 표시용 기본 정보 활용       | 민감한 정보 및 최신 상태 필수 시 |

---

## <b>마무리하며</b>

<p>
    프론트엔드에서 JWT를 디코딩하는 기술은 이제 선택이 아닌 필수인 것 같습니다. 특히 <b>사용자 경험(UX)</b> 측면에서 로그인 직후 "OOO님 환영합니다!"를 로딩 없이 보여줄 수 있다는 점이 큰 매력이었습니다. 다만, 보안을 위해 토큰 내에 과도한 정보를 담지 않도록 백엔드 개발자와 충분히 협의하는 과정이 중요하다는 것을 다시금 느꼈습니다.
</p>
