---
title: "[JavaScript] 자바스크립트 배열 filter 함수 정리"
date: 2023-02-01
categories: [javascript]
tags: [javascript, filter, array, es6]
description: "자바스크립트 배열 메서드 filter의 기본 문법, 콜백 매개변수, 원본 배열을 변경하지 않는 특징까지 예제로 정리했습니다."
custom_style: true
---

## 🧐 filter는 언제 사용할까요?

JavaScript에서 배열을 다룰 때 자주 사용하는 메서드 중 하나가 `filter`입니다.

이름 그대로 배열에서 특정 조건을 만족하는 요소만 **걸러내고 싶을 때** 사용합니다.

예를 들어 숫자 배열에서 3 이상인 값만 남기거나, 게시글 목록에서 특정 카테고리에 해당하는 글만 보여주고 싶을 때 사용할 수 있어요.

이번 글에서는 `filter`의 기본 문법부터 콜백 함수의 매개변수, 그리고 원본 배열이 변하지 않는 특징까지 차근차근 정리해보겠습니다.

---

## 💡 filter 기본 문법

`filter`는 배열의 각 요소를 하나씩 순회하면서 콜백 함수를 실행합니다.

그리고 콜백 함수의 결과가 `true`인 요소만 모아서 새로운 배열을 반환합니다.

```js
const newArray = array.filter((value, index, array) => {
  return 조건식;
});
```

조금 더 단순하게 보면 이렇게 이해할 수 있습니다.

```txt
조건을 만족한다 → 새 배열에 포함
조건을 만족하지 않는다 → 새 배열에서 제외
```

예를 들어 짝수만 남기고 싶다면 다음처럼 작성할 수 있습니다.

```js
const numbers = [1, 2, 3, 4, 5, 6];

const evenNumbers = numbers.filter((number) => number % 2 === 0);

console.log(evenNumbers);
// [2, 4, 6]
```

---

## 🧩 filter 콜백 함수의 세 가지 매개변수

`filter`의 콜백 함수는 세 가지 값을 받을 수 있습니다.

```js
array.filter((value, index, array) => {
  return 조건식;
});
```

- `value`: 현재 처리 중인 요소의 값입니다.
- `index`: 현재 요소의 인덱스입니다.
- `array`: `filter`를 호출한 원본 배열입니다.

하나씩 예제로 확인해볼게요.

```js
const array = [1, 2, 3, 4, 5];
```

### 1. value

`value`는 현재 순회 중인 요소의 값입니다.

```js
array.filter((value) => {
  console.log(value);
  return true;
});

// 1
// 2
// 3
// 4
// 5
```

### 2. index

`index`는 현재 요소가 배열에서 몇 번째 위치에 있는지 알려줍니다. 인덱스는 `0`부터 시작합니다.

```js
array.filter((value, index) => {
  console.log(index);
  return true;
});

// 0
// 1
// 2
// 3
// 4
```

### 3. array

세 번째 매개변수인 `array`는 현재 `filter`를 실행하고 있는 원본 배열입니다.

```js
array.filter((value, index, array) => {
  console.log(array);
  return true;
});

// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
```

대부분의 경우에는 `value`만 사용해도 충분하지만, 특정 위치의 요소를 제외하거나 원본 배열과 비교해야 할 때 `index`, `array`도 유용하게 사용할 수 있습니다.

---

## 🛠️ 실전 예제: 3 이상인 숫자만 추출하기

배열에서 3 이상인 숫자만 뽑아보겠습니다.

```js
const array = [1, 2, 3, 4, 5];

const newArray = array.filter((value) => value >= 3);

console.log(newArray);
// [3, 4, 5]
```

`value >= 3` 조건이 `true`인 값만 새 배열에 담겼습니다.

반대로 3보다 작은 값만 남기고 싶다면 조건만 바꾸면 됩니다.

```js
const smallNumbers = array.filter((value) => value < 3);

console.log(smallNumbers);
// [1, 2]
```

---

## 🔒 원본 배열은 변하지 않아요

`filter`를 사용할 때 꼭 기억해야 할 점이 있습니다.

`filter`는 원본 배열을 직접 수정하지 않고, 조건을 통과한 요소들로 **새로운 배열**을 만듭니다.

```js
const array = [1, 2, 3, 4, 5];
const newArray = array.filter((value) => value >= 3);

console.log("원본 배열:", array);
console.log("새로운 배열:", newArray);

// 원본 배열: [1, 2, 3, 4, 5]
// 새로운 배열: [3, 4, 5]
```

원본 배열이 그대로 유지되기 때문에 React에서 상태를 업데이트할 때도 자주 사용됩니다.

예를 들어 리스트에서 특정 항목을 삭제할 때도 `filter`를 사용할 수 있습니다.

```js
const items = ["apple", "banana", "orange"];

const nextItems = items.filter((item) => item !== "banana");

console.log(nextItems);
// ["apple", "orange"]
```

---

## ✅ 정리

`filter`는 배열에서 조건에 맞는 요소만 남기고 싶을 때 사용하는 메서드입니다.

- 콜백 함수가 `true`를 반환한 요소만 새 배열에 포함됩니다.
- 콜백 함수는 `value`, `index`, `array`를 받을 수 있습니다.
- 원본 배열은 변경되지 않습니다.
- React에서 리스트 삭제나 조건부 데이터 필터링에 자주 사용됩니다.

배열에서 필요한 데이터만 깔끔하게 골라내야 한다면 `filter`를 먼저 떠올려보면 좋겠습니다.
