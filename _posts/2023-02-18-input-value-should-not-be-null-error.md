---
title: "[React] input value should not be null 에러 해결하기"
date: 2023-02-18T14:31:00
categories: [react, error]
tags: [react, error, input, controlled-component]
description: "React input에서 value prop이 null일 때 발생하는 경고의 원인과 빈 문자열 fallback으로 해결하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 어떤 에러인가요?

React에서 `input`을 다루다 보면 콘솔에 다음과 같은 경고가 나타날 수 있습니다.

```txt
Warning: `value` prop on `input` should not be null.
```

프로젝트를 진행하던 중 백엔드에서 받아온 값을 `input`의 `value`에 넣었는데, 해당 값이 `null`로 들어오면서 이 경고를 만났습니다.

화면 동작 자체는 크게 문제 없어 보일 수 있지만, 콘솔 경고는 가능한 한 정리해두는 편이 좋습니다.

---

## 💡 왜 발생할까요?

React에서 `value`를 가진 `input`은 보통 controlled component로 관리됩니다.

```jsx
<input value={text} onChange={handleChange} />
```

이때 `value`에는 문자열이 들어가는 것이 자연스럽습니다.

그런데 `text` 값이 `null`이면 React는 다음과 같이 경고합니다.

```txt
input value로 null을 넣지 말고, 빈 문자열이나 undefined를 사용하세요.
```

즉, 문제는 `input` 값으로 `null`이 들어간다는 점입니다.

---

## 🛠️ 해결 방법

가장 간단한 해결 방법은 `value`에 fallback 값을 넣어주는 것입니다.

```jsx
<input value={text || ""} onChange={handleChange} />
```

이렇게 작성하면 `text`가 `null`이거나 `undefined`일 때 빈 문자열 `""`이 들어갑니다.

조금 더 명확하게 `null`과 `undefined`만 처리하고 싶다면 nullish coalescing 연산자 `??`를 사용할 수도 있습니다.

```jsx
<input value={text ?? ""} onChange={handleChange} />
```

`||`와 `??`는 비슷해 보이지만 차이가 있습니다.

```txt
text || ""
→ 빈 문자열, 0, false도 fallback 처리

text ?? ""
→ null 또는 undefined일 때만 fallback 처리
```

input 값에서는 보통 `text ?? ""` 방식이 더 의도가 명확합니다.

---

## ✅ 정리

`value prop on input should not be null` 경고는 `input`의 `value`에 `null`이 들어갔을 때 발생합니다.

- `input`의 `value`에는 문자열을 넣는 것이 안전합니다.
- 서버 데이터가 `null`일 수 있다면 fallback 값을 준비해야 합니다.
- `value={text ?? ""}`처럼 작성하면 `null`이나 `undefined`를 빈 문자열로 처리할 수 있습니다.

백엔드 데이터와 연결된 폼에서는 `null` 값이 들어올 수 있으니, input에 값을 넣기 전에 한 번 방어해두면 좋습니다.
