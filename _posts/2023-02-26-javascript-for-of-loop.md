---
title: "[JavaScript] for...in과 for...of 차이점 정리"
date: 2023-02-26T15:30:00
categories: [javascript]
tags: [javascript, for-in, for-of, loop, es6]
description: "JavaScript의 반복문 for...in과 for...of의 문법과 차이점, 각각 언제 사용하는지 예제로 정리했습니다."
custom_style: true
---

## 🧐 for...in과 for...of는 뭐가 다를까요?

JavaScript에서 반복문을 사용하다 보면 `for...in`과 `for...of`를 만나게 됩니다.

이름이 비슷해서 헷갈리기 쉽지만, 두 반복문은 사용하는 목적이 다릅니다.

간단히 말하면 다음과 같습니다.

```txt
for...in
→ key를 순회합니다.

for...of
→ value를 순회합니다.
```

이번 글에서는 두 반복문의 차이를 예제로 정리해보겠습니다.

---

## 🔑 for...in

`for...in`은 객체의 key를 순회할 때 사용합니다.

배열에 사용하면 배열의 index가 나옵니다.

```js
const array = ["문동은", "손명오", "박재준", "이사라", "최혜정"];

for (const index in array) {
  console.log(index);
}

// "0"
// "1"
// "2"
// "3"
// "4"
```

배열의 값이 아니라 인덱스가 출력되는 것을 볼 수 있습니다.

객체에서는 다음처럼 key를 순회할 수 있습니다.

```js
const user = {
  name: "문동은",
  age: 23,
  job: "teacher",
};

for (const key in user) {
  console.log(key, user[key]);
}

// name 문동은
// age 23
// job teacher
```

그래서 `for...in`은 배열보다 객체를 순회할 때 더 잘 어울립니다.

---

## 📦 for...of

`for...of`는 반복 가능한 객체의 value를 순회할 때 사용합니다.

배열의 실제 값을 하나씩 꺼내고 싶다면 `for...of`가 더 적합합니다.

```js
const array = ["문동은", "손명오", "박재준", "이사라", "최혜정"];

for (const value of array) {
  console.log(value);
}

// "문동은"
// "손명오"
// "박재준"
// "이사라"
// "최혜정"
```

문자열도 반복 가능한 값이기 때문에 `for...of`로 순회할 수 있습니다.

```js
const text = "hello";

for (const char of text) {
  console.log(char);
}

// h
// e
// l
// l
// o
```

---

## 📊 비교 정리

| 구분 | for...in | for...of |
| :--- | :--- | :--- |
| 순회 대상 | key 또는 index | value |
| 배열에서 결과 | 인덱스 | 실제 값 |
| 객체 순회 | 적합 | 일반 객체에는 바로 사용 불가 |
| 배열 순회 | 권장하지 않음 | 적합 |

---

## ✅ 언제 무엇을 쓰면 좋을까요?

객체의 key를 순회해야 한다면 `for...in`을 사용할 수 있습니다.

```js
for (const key in object) {
  console.log(key, object[key]);
}
```

배열이나 문자열의 값을 순회해야 한다면 `for...of`를 사용하면 됩니다.

```js
for (const item of array) {
  console.log(item);
}
```

---

## ✅ 정리

`for...in`과 `for...of`는 비슷해 보이지만 목적이 다릅니다.

- `for...in`은 key를 순회합니다.
- `for...of`는 value를 순회합니다.
- 배열의 값을 꺼낼 때는 `for...of`가 더 자연스럽습니다.
- 객체의 key를 확인할 때는 `for...in`을 사용할 수 있습니다.

배열인지 객체인지, 그리고 key가 필요한지 value가 필요한지를 기준으로 선택하면 덜 헷갈립니다.
