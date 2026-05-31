---
title: "[Error] Cannot destructure property 'data' of undefined 해결하기"
date: 2023-06-22T18:13:00
categories: [error]
tags: [error, axios, interceptor, javascript]
description: "axios 응답 인터셉터에서 Promise.reject를 반환하지 않아 호출부에서 undefined를 구조 분해하게 되는 문제를 정리했습니다."
custom_style: true
---

## 발생한 에러

API 요청을 처리하던 중 다음과 같은 에러를 만났습니다.

```txt
Cannot destructure property 'data' of '(intermediate value)' as it is undefined.
```

보통 이런 코드를 작성했을 때 발생할 수 있습니다.

```js
const { data } = await api.get("/users/me");
```

`api.get()`의 결과가 정상적인 response 객체가 아니라 `undefined`가 되어버리면, `data`를 구조 분해할 수 없어 에러가 발생합니다.

---

## 원인

문제는 axios response interceptor의 에러 처리 부분에 있었습니다.

```js
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    const status = error.response.status;

    if (status === 401) {
      removeCookie();
      window.location.replace("/login");
    }
  },
);
```

에러가 발생했을 때 아무것도 반환하지 않고 있습니다.

JavaScript 함수에서 return이 없으면 기본적으로 `undefined`가 반환됩니다.

그 결과 API 호출부에서는 에러가 던져지는 대신 `undefined`를 받게 되고, `const { data } = undefined` 상황이 되어 에러가 발생합니다.

---

## 해결 방법

에러 처리 후에는 반드시 `Promise.reject(error)`를 반환해야 합니다.

```js
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    const status = error.response?.status;

    if (status === 401) {
      removeCookie();
      window.location.replace("/login");
    }

    return Promise.reject(error);
  },
);
```

이렇게 하면 인터셉터에서 공통 처리를 한 뒤에도, 호출한 쪽의 `catch`나 React Query의 `onError`로 에러가 정상적으로 전달됩니다.

---

## optional chaining도 함께 사용하기

네트워크 에러처럼 서버 응답 자체가 없는 경우에는 `error.response`가 없을 수 있습니다.

그래서 아래 코드처럼 바로 구조 분해하면 또 다른 에러가 날 수 있습니다.

```js
const {
  response: { status },
} = error;
```

조금 더 안전하게 쓰려면 optional chaining을 사용합니다.

```js
const status = error.response?.status;
```

이렇게 하면 `response`가 없는 에러도 안전하게 처리할 수 있습니다.

---

## 마무리

이 에러의 핵심 원인은 API 응답이 정말 `undefined`라기보다, interceptor에서 에러를 제대로 다시 던지지 않았다는 점이었습니다.

axios 응답 인터셉터에서 에러를 처리했다면 마지막에 `return Promise.reject(error)`를 작성해야 합니다.

또한 `error.response?.status`처럼 안전하게 접근하면 네트워크 에러 상황에서도 코드가 더 안정적으로 동작합니다.
