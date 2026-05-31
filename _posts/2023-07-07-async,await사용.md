---
title: "[JavaScript] async와 await 기본 사용법"
date: 2023-07-07T16:18:00
categories: [javascript]
tags: [javascript, async, await, promise]
description: "JavaScript에서 async와 await를 사용해 비동기 코드를 읽기 쉽게 작성하는 방법을 정리했습니다."
custom_style: true
---

## async와 await가 필요한 이유

JavaScript에서는 API 요청, 타이머, 파일 처리처럼 시간이 걸리는 작업을 비동기로 처리합니다.

예전에는 Promise의 `then`을 이어 붙여서 처리하는 경우가 많았습니다.

```js
fetchUser()
  .then((user) => fetchPosts(user.id))
  .then((posts) => console.log(posts))
  .catch((error) => console.error(error));
```

이 방식도 사용할 수 있지만, 로직이 길어질수록 코드 흐름을 따라가기 어려워질 수 있습니다.

`async`와 `await`를 사용하면 비동기 코드를 동기 코드처럼 위에서 아래로 읽을 수 있습니다.

---

## async란?

`async`는 함수를 비동기 함수로 만들어주는 키워드입니다.

```js
const getNumber = async () => {
  return 1;
};

const result = getNumber();

console.log(result); // Promise { 1 }
```

`async` 함수는 항상 Promise를 반환합니다.

함수 안에서 단순히 `1`을 반환해도 실제 반환값은 `Promise`로 감싸집니다.

---

## await란?

`await`는 Promise가 처리될 때까지 기다린 뒤 결과값을 꺼내는 키워드입니다.

```js
const getNumber = async () => {
  return 1;
};

const run = async () => {
  const result = await getNumber();
  console.log(result); // 1
};

run();
```

`await`는 `async` 함수 안에서 사용할 수 있습니다.

Promise가 fulfilled 상태가 되면 결과값을 반환하고, rejected 상태가 되면 에러를 던집니다.

---

## API 요청 예시

```js
const getUser = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
  const data = await response.json();

  return data;
};
```

첫 번째 `await`는 서버 응답을 기다립니다.

두 번째 `await`는 응답 body를 JSON으로 변환하는 작업을 기다립니다.

---

## try-catch로 에러 처리하기

`await` 중 에러가 발생할 수 있으므로 실제 코드에서는 `try-catch`를 함께 사용하는 것이 좋습니다.

```js
const getUser = async () => {
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/users/1");
    const data = await response.json();

    return data;
  } catch (error) {
    console.error("사용자 정보를 가져오지 못했습니다.", error);
    return null;
  }
};
```

API 요청 실패, 네트워크 오류, JSON 변환 실패 같은 상황을 한곳에서 처리할 수 있습니다.

---

## Promise.all과 함께 사용하기

여러 비동기 작업을 동시에 실행해야 한다면 `Promise.all`을 함께 사용할 수 있습니다.

```js
const getPageData = async () => {
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts(),
  ]);

  return { user, posts };
};
```

서로 의존하지 않는 요청이라면 순서대로 기다리는 것보다 동시에 실행하는 편이 더 빠릅니다.

---

## 마무리

`async`는 함수를 Promise를 반환하는 비동기 함수로 만들어줍니다.

`await`는 Promise가 끝날 때까지 기다린 뒤 결과를 꺼내줍니다.

둘을 함께 사용하면 비동기 코드를 훨씬 읽기 쉽게 만들 수 있습니다. 실제 API 요청에서는 `try-catch`까지 함께 작성해두면 에러 상황도 안정적으로 처리할 수 있습니다.
