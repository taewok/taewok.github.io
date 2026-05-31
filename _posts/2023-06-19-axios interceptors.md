---
title: "[JavaScript] axios interceptors 사용하기"
date: 2023-06-19T19:49:00
categories: [javascript]
tags: [javascript, axios, interceptor]
description: "axios interceptors를 사용해 요청 전 헤더를 추가하고 응답 에러를 공통 처리하는 방법을 정리했습니다."
custom_style: true
---

## interceptors란?

axios interceptors는 요청이 서버로 나가기 전이나, 응답이 then/catch로 전달되기 전에 중간에서 가로채 처리할 수 있는 기능입니다.

쉽게 말하면 API 요청과 응답의 공통 처리 지점을 만드는 기능입니다.

예를 들어 모든 요청에 Authorization 헤더를 붙이거나, 401 에러가 발생했을 때 로그인 페이지로 이동시키는 작업을 한곳에서 처리할 수 있습니다.

---

## axios 인스턴스 만들기

먼저 공통으로 사용할 axios 인스턴스를 만들면 관리하기 편합니다.

```js
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

이제 프로젝트에서는 기본 `axios` 대신 `api`를 사용하면 됩니다.

---

## 요청 인터셉터

요청 인터셉터는 서버로 요청을 보내기 전에 실행됩니다.

```js
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("accessToken");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  },
  (error) => {
    return Promise.reject(error);
  },
);
```

이렇게 작성하면 매 API 요청마다 직접 Authorization 헤더를 넣지 않아도 됩니다.

토큰이 있으면 자동으로 헤더에 추가됩니다.

---

## 응답 인터셉터

응답 인터셉터는 서버 응답을 받은 뒤 실행됩니다.

```js
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("accessToken");
      window.location.href = "/login";
    }

    return Promise.reject(error);
  },
);
```

응답이 성공하면 그대로 반환하고, 에러가 발생하면 상태 코드를 확인해 공통 처리를 할 수 있습니다.

여기서 `return Promise.reject(error)`를 빼면 호출한 쪽의 `catch`나 React Query의 `onError`로 에러가 전달되지 않을 수 있으니 꼭 반환해주는 것이 좋습니다.

---

## 인터셉터 제거하기

인터셉터는 등록할 때 id를 반환합니다.

이 id를 사용하면 나중에 제거할 수 있습니다.

```js
const interceptorId = api.interceptors.response.use(
  (response) => response,
  (error) => Promise.reject(error),
);

api.interceptors.response.eject(interceptorId);
```

일반적인 앱 초기화 코드에서는 제거할 일이 많지 않지만, 테스트 환경이나 특정 컴포넌트 생명주기에 맞춰 등록하는 경우에는 유용합니다.

---

## Refresh Token 처리에도 자주 사용해요

Access Token이 만료되었을 때 새 토큰을 받아 다시 요청하는 로직도 응답 인터셉터에서 자주 처리합니다.

흐름은 대략 다음과 같습니다.

1. API 요청이 401로 실패합니다.
2. Refresh Token으로 새 Access Token을 요청합니다.
3. 새 토큰을 저장합니다.
4. 실패했던 요청을 다시 실행합니다.

이 흐름은 편리하지만, 동시에 여러 요청이 401로 실패하는 상황에서는 중복 갱신 문제가 생길 수 있습니다.

그래서 실제 서비스에서는 refresh 요청 중인지 관리하거나, 요청 큐를 두는 방식까지 함께 고려해야 합니다.

---

## 마무리

axios interceptors는 API 통신에서 반복되는 코드를 줄여주는 유용한 기능입니다.

요청 인터셉터는 토큰 헤더 추가처럼 서버로 보내기 전 작업에 사용하고, 응답 인터셉터는 에러 처리나 토큰 만료 처리처럼 응답 이후 작업에 사용합니다.

특히 에러 인터셉터에서는 처리 후 `Promise.reject(error)`를 반환해야 호출한 곳에서도 에러를 이어서 다룰 수 있습니다.
