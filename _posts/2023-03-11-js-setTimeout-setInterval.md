---
title: "[JavaScript] setTimeout과 setInterval 차이점 정리"
date: 2023-03-11T21:01:00
categories: [javascript]
tags: [javascript, timer, settimeout, setinterval]
description: "JavaScript 타이머 함수 setTimeout과 setInterval의 차이점, 사용법, clearTimeout과 clearInterval로 정리하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 JavaScript에서 시간을 다루는 방법

웹 페이지에서 특정 시간이 지난 뒤 코드를 실행하거나, 일정 간격으로 같은 작업을 반복해야 할 때가 있습니다.

JavaScript에서는 이런 작업을 위해 `setTimeout`과 `setInterval`을 사용할 수 있습니다.

두 함수는 비슷해 보이지만 역할이 다릅니다.

```txt
setTimeout
→ 일정 시간 뒤 한 번 실행

setInterval
→ 일정 시간마다 반복 실행
```

---

## ⏱️ setTimeout: 한 번만 실행하기

`setTimeout`은 지정한 시간이 지난 뒤 콜백 함수를 한 번 실행합니다.

```js
setTimeout(() => {
  console.log("1초 뒤에 실행됩니다.");
}, 1000);
```

두 번째 인자로 전달한 `1000`은 1000ms, 즉 1초를 의미합니다.

---

## 🛑 clearTimeout: 예약 취소하기

`setTimeout`은 타이머 id를 반환합니다. 이 id를 `clearTimeout`에 넘기면 예약된 작업을 취소할 수 있습니다.

```js
const timerId = setTimeout(() => {
  console.log("이 메시지는 보이지 않습니다.");
}, 3000);

clearTimeout(timerId);
```

위 코드에서는 3초가 지나기 전에 타이머를 취소했기 때문에 콘솔이 출력되지 않습니다.

---

## 🔁 setInterval: 반복 실행하기

`setInterval`은 지정한 시간마다 콜백 함수를 반복해서 실행합니다.

```js
let count = 0;

const intervalId = setInterval(() => {
  count += 1;
  console.log(`${count}초 경과`);
}, 1000);
```

이 코드는 1초마다 `count`를 1씩 증가시키고 콘솔에 출력합니다.

---

## 🧹 clearInterval: 반복 멈추기

`setInterval`은 직접 멈추기 전까지 계속 실행됩니다. 그래서 필요가 없어지면 반드시 `clearInterval`로 정리해야 합니다.

```js
const intervalId = setInterval(() => {
  console.log("반복 실행 중");
}, 1000);

clearInterval(intervalId);
```

React 같은 프레임워크에서는 컴포넌트가 사라질 때 interval을 정리하지 않으면 보이지 않는 곳에서 계속 실행될 수 있습니다.

---

## 📊 setTimeout과 setInterval 비교

| 함수 | 실행 방식 | 정리 함수 |
| :--- | :--- | :--- |
| `setTimeout` | 일정 시간 뒤 한 번 실행 | `clearTimeout` |
| `setInterval` | 일정 시간마다 반복 실행 | `clearInterval` |

---

## ✅ 정리

`setTimeout`과 `setInterval`은 JavaScript의 대표적인 타이머 함수입니다.

- `setTimeout`은 일정 시간 뒤 한 번만 실행합니다.
- `setInterval`은 일정 시간마다 반복 실행합니다.
- 예약된 timeout은 `clearTimeout`으로 취소할 수 있습니다.
- 반복 중인 interval은 `clearInterval`로 멈출 수 있습니다.
- 더 이상 필요 없는 타이머는 반드시 정리하는 것이 좋습니다.

시간을 기준으로 동작하는 UI를 만들 때 두 함수의 차이를 정확히 알고 사용하면 좋습니다.
