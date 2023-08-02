---
title: "[JavaScript] axios interceptors 사용하기"
date: 2023-06-19T19:49:000
categories: [JavaScript]
tags: [javascript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray" class="h2">interceptors란?</b>

axios interceptors는 request와 response가 then과 catch에 의해 처리되기 전에<br/> 가로채서 먼저
처리해주는 axios library이다.<br/> 유용한 사용방법은 Header추가, 인증관리, error처리 등의 작업이 있다.
<br/>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
사용하는 이유
</blockquote>

### <b>1. api 통신 처리시 반복될 부분 처리</b>

- header에 필수적으로 들어가야하는 부분이 있을 경우 매번 요청때마다 header에 값을 넣어서
  처리하는 것이 아닌 interceptors를 사용해서 api 요청시 자동으로 해당 값이 포함되게 한다.

### <b>2. 특정 error를 catch해서 작업 진행</b>

- jwt + refreshToken 로그인 방식에 유용
- accessToken 만료시 로그인 페이지로 자동이동

<br/>

---

## <b style="border-bottom:2px solid gray" class="h2">interceptors 사용법</b>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
request가 server에 전달되지 건에 가로채기
</blockquote>

```js
// 요청 인터셉터 추가하기
axios.interceptors.request.use(
  function (config) {
    // 요청이 전달되기 전에 작업 수행
    return config;
  },
  function (error) {
    // 요청 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);
```

<br/>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
server로부터 response를 받은 후 가로채기
</blockquote>

```js
// 응답 인터셉터 추가하기
axios.interceptors.response.use(
  function (response) {
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    // 응답 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
interceptors 제거
</blockquote>

```js
// 응답 인터셉터 추가하기
const myInterceptors = axios.interceptors.response.use(
  function (response) {
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    // 응답 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);

axios.interceptors.request.eject(myInterceptor);
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
원하는 주소 통신 발생에 interceptors 추가
</blockquote>

```js
export const api = axios.create({
  baseURL: "http://localhost:3000",
});

// 응답 인터셉터 추가하기
api.interceptors.response.use(
  function (response) {
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    // 응답 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);
```

---

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
