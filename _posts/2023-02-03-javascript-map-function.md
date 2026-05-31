---
title: "[JavaScript] 자바스크립트 배열 map 함수 정리"
date: 2023-02-03T19:10:00
categories: [javascript]
tags: [javascript, map, array, es6]
description: "JavaScript 배열 메서드 map의 기본 문법, 콜백 매개변수, 객체 배열 활용 예제와 원본 배열을 변경하지 않는 특징을 정리했습니다."
custom_style: true
---

## 🧐 map은 언제 사용할까요?

JavaScript에서 배열을 다룰 때 `map`은 정말 자주 사용하는 메서드입니다.

`filter`가 조건에 맞는 요소만 골라내는 역할이라면, `map`은 배열의 각 요소를 원하는 형태로 **변환**할 때 사용합니다.

예를 들어 숫자 배열의 모든 값에 2를 곱하거나, 사용자 객체 배열에서 이름만 뽑아낼 때 사용할 수 있어요.

---

## 💡 map 기본 문법

`map`은 배열의 요소를 하나씩 순회하면서 콜백 함수를 실행합니다.

그리고 콜백 함수가 반환한 값들로 새로운 배열을 만듭니다.

```js
const newArray = array.map((value, index, array) => {
  return 반환할_값;
});
```

핵심은 `return`한 값이 새 배열의 요소가 된다는 점입니다.

```js
const numbers = [1, 2, 3];

const doubled = numbers.map((number) => number * 2);

console.log(doubled);
// [2, 4, 6]
```

---

## 🧩 map 콜백 함수의 세 가지 매개변수

`map`의 콜백 함수는 `filter`와 마찬가지로 세 가지 값을 받을 수 있습니다.

```js
array.map((value, index, array) => {
  return 반환할_값;
});
```

- `value`: 현재 처리 중인 요소의 값입니다.
- `index`: 현재 요소의 인덱스입니다.
- `array`: `map`을 호출한 원본 배열입니다.

예제로 확인해볼게요.

```js
const array = [1, 2, 3, 4, 5];
```

### 1. value

```js
array.map((value) => {
  console.log(value);
  return value;
});

// 1
// 2
// 3
// 4
// 5
```

### 2. index

```js
array.map((value, index) => {
  console.log(index);
  return value;
});

// 0
// 1
// 2
// 3
// 4
```

### 3. array

```js
array.map((value, index, array) => {
  console.log(array);
  return value;
});

// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// ...
```

대부분은 `value`만으로 충분하지만, 위치 정보가 필요할 때는 `index`도 함께 사용할 수 있습니다.

---

## 🛠️ map 실전 활용 예시

### 1. 모든 값 2배 만들기

```js
const array = [1, 2, 3, 4, 5];

const newArray = array.map((value) => value * 2);

console.log(newArray);
// [2, 4, 6, 8, 10]
```

각 요소에 2를 곱한 값이 새 배열에 담겼습니다.

### 2. 조건에 따라 값 바꾸기

`map` 안에서 조건문을 사용할 수도 있습니다.

```js
const array = [1, 2, 3, 4, 5];

const newArray = array.map((value) => {
  if (value >= 3) {
    return "*";
  }

  return value;
});

console.log(newArray);
// [1, 2, "*", "*", "*"]
```

`filter`는 조건에 맞지 않는 요소를 제거하지만, `map`은 배열의 길이를 유지하면서 값을 바꿉니다.

### 3. 객체 배열에서 특정 값만 뽑기

실무에서 가장 많이 쓰는 패턴 중 하나입니다.

```js
const users = [
  { name: "최혜정", age: 27 },
  { name: "박연진", age: 21 },
  { name: "손명오", age: 25 },
  { name: "문동은", age: 23 },
];

const names = users.map((user) => user.name);

console.log(names);
// ["최혜정", "박연진", "손명오", "문동은"]
```

객체 배열에서 필요한 값만 꺼내 새로운 배열을 만들 수 있습니다.

---

## 🔒 원본 배열은 변하지 않아요

`map`도 `filter`와 마찬가지로 원본 배열을 직접 변경하지 않습니다.

```js
const array = [1, 2, 3, 4, 5];
const newArray = array.map((value) => value * 2);

console.log("원본 배열:", array);
console.log("새로운 배열:", newArray);

// 원본 배열: [1, 2, 3, 4, 5]
// 새로운 배열: [2, 4, 6, 8, 10]
```

원본 데이터를 안전하게 유지하면서 변환된 새 배열을 만들 수 있기 때문에 React에서도 자주 사용됩니다.

---

## ✅ 정리

`map`은 배열의 각 요소를 원하는 형태로 변환할 때 사용하는 메서드입니다.

- 배열의 모든 요소를 순회합니다.
- 콜백 함수가 반환한 값으로 새 배열을 만듭니다.
- 원본 배열은 변경하지 않습니다.
- 객체 배열에서 특정 값만 뽑아낼 때도 유용합니다.
- React에서 리스트를 렌더링할 때도 자주 사용됩니다.

배열 데이터를 다른 형태로 바꿔야 한다면 `map`을 떠올려보면 좋겠습니다.
