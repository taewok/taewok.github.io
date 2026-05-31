---
title: "[JavaScript] Optional Chaining과 Nullish Coalescing"
date: 2023-07-14T17:50:00
categories: [javascript]
tags: [javascript, optional-chaining, nullish-coalescing]
description: "Optional Chaining과 Nullish Coalescing을 사용해 null, undefined 값을 안전하게 다루는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

프론트엔드에서 API 데이터를 다루다 보면 아직 값이 없거나, 특정 필드가 내려오지 않는 상황을 자주 만납니다.

이때 안전하게 값을 읽기 위해 사용하는 문법이 Optional Chaining `?.`입니다.

그리고 기본값을 넣을 때 자주 사용하는 문법이 Nullish Coalescing `??`입니다.

두 문법을 함께 사용하면 방어 코드를 훨씬 간결하게 만들 수 있습니다.

---

## Optional Chaining이란?

Optional Chaining은 객체의 중첩된 값을 읽을 때, 중간 값이 `null` 또는 `undefined`이면 에러를 내지 않고 `undefined`를 반환합니다.

```js
const user = null;

console.log(user.name); // TypeError
console.log(user?.name); // undefined
```

`user?.name`은 `user`가 있으면 `name`을 읽고, 없으면 바로 `undefined`를 반환합니다.

---

## 중첩 객체에서 사용하기

API 응답은 종종 깊은 구조를 가집니다.

```js
const response = {
  user: {
    profile: {
      nickname: "taewok",
    },
  },
};

const nickname = response?.user?.profile?.nickname;

console.log(nickname); // "taewok"
```

중간에 `user`나 `profile`이 없어도 에러가 발생하지 않습니다.

---

## 배열과 함수에도 사용할 수 있어요

배열 요소에 접근할 때도 사용할 수 있습니다.

```js
const users = null;

console.log(users?.[0]?.name); // undefined
```

함수가 있을 때만 호출하고 싶다면 이렇게 쓸 수 있습니다.

```js
props.onClose?.();
```

`onClose`가 있으면 호출하고, 없으면 아무 일도 하지 않습니다.

---

## Nullish Coalescing이란?

Nullish Coalescing `??`는 왼쪽 값이 `null` 또는 `undefined`일 때만 오른쪽 값을 사용합니다.

```js
const nickname = null;

console.log(nickname ?? "익명"); // "익명"
```

값이 실제로 없을 때만 기본값을 넣고 싶을 때 사용합니다.

---

## ||와 ??의 차이

`||`는 falsy 값을 모두 기본값으로 바꿉니다.

```js
const count = 0;

console.log(count || 10); // 10
console.log(count ?? 10); // 0
```

`0`, `""`, `false`도 유효한 값으로 봐야 한다면 `??`를 사용하는 것이 안전합니다.

```js
const title = "";

console.log(title || "제목 없음"); // "제목 없음"
console.log(title ?? "제목 없음"); // ""
```

빈 문자열도 의도한 값이라면 `??`가 더 적절합니다.

---

## 함께 사용하기

Optional Chaining과 Nullish Coalescing은 함께 사용할 때 특히 편합니다.

```js
const nickname = response?.user?.profile?.nickname ?? "익명";
```

`nickname`이 존재하면 해당 값을 사용하고, 중간 값이 없거나 최종 값이 `null` 또는 `undefined`이면 `"익명"`을 사용합니다.

React에서도 자주 사용합니다.

```tsx
const UserName = ({ user }: Props) => {
  return <span>{user?.profile?.nickname ?? "익명 사용자"}</span>;
};
```

---

## 마무리

`?.`는 중간 값이 없을 때 에러를 막아주는 문법입니다.

`??`는 값이 `null` 또는 `undefined`일 때만 기본값을 적용하는 문법입니다.

API 응답처럼 값이 항상 보장되지 않는 데이터를 다룰 때 두 문법을 함께 사용하면 코드가 훨씬 안전하고 읽기 쉬워집니다.
