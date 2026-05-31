---
title: "[JavaScript] this 동작 원리 정리"
date: 2023-02-18T19:10:00
categories: [javascript]
tags: [javascript, this, es6, arrow-function]
description: "JavaScript에서 this가 호출 방식에 따라 어떻게 달라지는지, 일반 함수와 화살표 함수의 차이를 예제로 정리했습니다."
custom_style: true
---

## 🧐 this는 왜 헷갈릴까요?

JavaScript를 공부하다 보면 `this`는 꽤 헷갈리는 개념 중 하나입니다.

어떤 코드에서는 `this`가 객체를 가리키고, 어떤 코드에서는 `undefined`가 나오기도 합니다.

핵심은 `this`가 대부분의 경우 **함수가 어떻게 호출되었는지**에 따라 달라진다는 점입니다.

이번 글에서는 일반 함수의 `this`와 화살표 함수의 `this`가 어떻게 다른지 예제로 정리해보겠습니다.

---

## 🧱 예제 코드 준비하기

먼저 객체 안에 메서드를 만들고, 그 안에서 다시 일반 함수를 호출해보겠습니다.

```js
const user = {
  name: "오윤희",
  getName: function () {
    console.log("1. 메서드 this:", this);

    const second = function () {
      console.log("2. 내부 함수 this:", this);
    };

    second();
  },
};

user.getName();
console.log("3. user 객체:", user);
```

이 코드를 실행하면 `this`가 두 번 출력됩니다.

---

## 🔍 첫 번째 this: 메서드로 호출한 경우

먼저 `user.getName()`을 살펴보겠습니다.

```js
user.getName();
```

이 경우 `getName`은 `user` 객체를 통해 호출되었습니다.

그래서 `getName` 내부의 `this`는 `user` 객체를 가리킵니다.

```txt
this → user 객체
```

일반적으로 객체의 메서드로 호출된 함수 안에서 `this`는 그 메서드를 호출한 객체를 가리킵니다.

---

## ⚠️ 두 번째 this: 일반 함수로 호출한 경우

이제 내부 함수 `second`를 보겠습니다.

```js
const second = function () {
  console.log(this);
};

second();
```

`second`는 `user.second()`처럼 객체를 통해 호출된 것이 아니라, 그냥 `second()` 형태로 호출되었습니다.

이런 경우 strict mode에서는 `this`가 `undefined`가 됩니다.

```txt
this → undefined
```

만약 strict mode가 아니라면 브라우저에서는 전역 객체인 `window`가 나올 수 있습니다.

`type="module"`로 JavaScript 파일을 불러오면 자동으로 strict mode가 적용되기 때문에 `undefined`가 나오는 것을 볼 수 있습니다.

```html
<script type="module" src="./index.js"></script>
```

---

## 💡 화살표 함수에서는 어떻게 다를까요?

내부 함수에서도 바깥의 `this`를 그대로 사용하고 싶다면 화살표 함수를 사용할 수 있습니다.

```js
const user = {
  name: "오윤희",
  getName: function () {
    console.log("1. 메서드 this:", this);

    const second = () => {
      console.log("2. 화살표 함수 this:", this);
    };

    second();
  },
};

user.getName();
```

이번에는 `second` 내부의 `this`도 `user` 객체를 가리킵니다.

화살표 함수는 자신만의 `this`를 만들지 않습니다. 대신 자신이 정의된 위치의 바깥 `this`를 그대로 사용합니다.

이것을 lexical this라고 부릅니다.

```txt
화살표 함수의 this
→ 호출 방식이 아니라 정의된 위치의 this를 사용
```

---

## 🧩 일반 함수와 화살표 함수 비교

두 함수의 차이를 정리하면 다음과 같습니다.

| 구분 | 일반 함수 | 화살표 함수 |
| :--- | :--- | :--- |
| this 결정 방식 | 호출 방식에 따라 결정 | 정의된 위치의 this를 사용 |
| 객체 메서드 | this가 호출한 객체를 가리킬 수 있음 | 메서드 자체에는 부적합할 수 있음 |
| 내부 콜백 | this가 바뀔 수 있음 | 바깥 this를 유지하기 좋음 |

그래서 객체의 메서드를 만들 때는 일반 함수를 사용하고, 메서드 내부의 콜백에서 바깥 `this`를 유지하고 싶을 때는 화살표 함수를 사용하는 경우가 많습니다.

---

## ✅ 정리

JavaScript의 `this`는 함수가 어떻게 호출되었는지에 따라 달라질 수 있습니다.

- 객체의 메서드로 호출하면 `this`는 그 객체를 가리킵니다.
- 일반 함수로 단독 호출하면 strict mode에서 `this`는 `undefined`가 됩니다.
- 화살표 함수는 자신만의 `this`를 만들지 않습니다.
- 화살표 함수는 바깥 스코프의 `this`를 그대로 사용합니다.

`this`가 헷갈릴 때는 먼저 "이 함수가 어떻게 호출되었지?"를 확인해보면 이해하기가 훨씬 쉬워집니다.
