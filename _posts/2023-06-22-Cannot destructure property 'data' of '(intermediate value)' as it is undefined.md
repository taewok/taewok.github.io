---
title: "[Error] console.log(Error) 객체 안나오는 현상"
date: 2023-06-22T18:13:000
categories: [Error]
tags: [error] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray" class="h2">Cannot destructure property 'data' of '(intermediate value)' as it is undefined. 발생</b>

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/df319a46-ad81-4eaa-beb8-d2535b80b9d9"/>

원래 평상시에 위처럼 찍혀야 할 error 코드가 아래처럼 나오기 시작했다....<br/>
백에서 어떤 error인지 보내준 message 확인해야 하는데....

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/efaf403d-53fe-4976-8b7a-a80f458ca917"/>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
원인
</blockquote>

응답이 올 떄 통신을 가로채는 코드가 문제였던 것 이다...<br/>
이미 진짜 error 코드는 여기서 쓰였다...

```js
// 응답 인터셉터 추가하기
api.interceptors.response.use(
  function (response) {
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    const {
      response: { status },
    } = error;
    // 응답 오류가 있는 작업 수행
    if (status === 401) {
      removeCookie();
      window.location.replace("/login");
      window.location.reload();
    }
  }
);
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
해결
</blockquote>

Promise.reject(error)를 return 해주니 다시 정상적인 error 코드가 나온다.

```js
// 응답 인터셉터 추가하기
api.interceptors.response.use(
  function (response) {
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    const {
      response: { status },
    } = error;
    // 응답 오류가 있는 작업 수행
    if (status === 401) {
      removeCookie();
      window.location.replace("/login");
      window.location.reload();
    }
    return Promise.reject(error);
  }
);
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>