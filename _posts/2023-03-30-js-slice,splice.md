---
title: "[JavaScript] 배열 자르기와 변경 비교: slice() vs splice()"
date: 2023-03-30T11:10:00
categories: [javascript]
tags: [javascript, array]
description: "JavaScript 배열 메서드 slice와 splice의 차이를 원본 배열 변경 여부를 중심으로 정리했습니다."
custom_style: true
---

## 들어가며

JavaScript에서 배열의 일부를 가져오거나, 배열 중간의 값을 삭제하고 싶을 때 `slice()`와 `splice()`를 자주 만나게 됩니다.

두 메서드는 이름도 비슷하고 배열을 다룬다는 점도 비슷해서 처음에는 헷갈리기 쉽습니다. 하지만 가장 중요한 차이가 하나 있어요.

`slice()`는 원본 배열을 바꾸지 않고, `splice()`는 원본 배열을 직접 바꿉니다.

이번 글에서는 이 차이를 기준으로 두 메서드를 비교해볼게요.

---

## slice(): 원본은 그대로 두고 일부만 복사하기

`slice()`는 배열에서 원하는 구간을 잘라 새 배열로 반환합니다.

```js
const fruits = ["apple", "banana", "cherry", "grape"];

const result = fruits.slice(1, 3);

console.log(result); // ["banana", "cherry"]
console.log(fruits); // ["apple", "banana", "cherry", "grape"]
```

`slice(1, 3)`은 인덱스 `1`부터 시작해서 인덱스 `3` 직전까지 가져옵니다. 즉, 끝 인덱스는 결과에 포함되지 않습니다.

여기서 중요한 점은 `fruits` 원본 배열이 그대로 남아 있다는 점입니다.

그래서 React에서 상태 배열을 다룰 때처럼 원본 데이터를 직접 수정하면 안 되는 상황에서는 `slice()`가 더 안전합니다.

---

## slice()의 기본 형태

```js
array.slice(start, end);
```

`start`는 자르기 시작할 인덱스입니다.

`end`는 자르기를 멈출 인덱스입니다. 이 인덱스의 값은 포함되지 않습니다.

```js
const numbers = [1, 2, 3, 4, 5];

console.log(numbers.slice(0, 2)); // [1, 2]
console.log(numbers.slice(2)); // [3, 4, 5]
```

두 번째 인자를 생략하면 시작 인덱스부터 배열 끝까지 가져옵니다.

---

## splice(): 원본 배열을 직접 수정하기

`splice()`는 배열의 특정 위치에서 값을 삭제하거나, 새로운 값을 추가하거나, 기존 값을 교체할 때 사용합니다.

```js
const fruits = ["apple", "banana", "cherry", "grape"];

const removed = fruits.splice(1, 2);

console.log(removed); // ["banana", "cherry"]
console.log(fruits); // ["apple", "grape"]
```

`splice(1, 2)`는 인덱스 `1`부터 시작해서 2개의 값을 삭제합니다.

그리고 삭제된 값들이 배열로 반환됩니다.

주의할 점은 원본 배열인 `fruits`가 실제로 바뀐다는 점입니다.

---

## splice()의 기본 형태

```js
array.splice(start, deleteCount, item1, item2, ...);
```

`start`는 변경을 시작할 인덱스입니다.

`deleteCount`는 삭제할 요소의 개수입니다.

그 뒤에 오는 값들은 해당 위치에 새로 추가할 요소입니다.

```js
const fruits = ["apple", "grape"];

fruits.splice(1, 0, "banana", "cherry");

console.log(fruits); // ["apple", "banana", "cherry", "grape"]
```

여기서는 `deleteCount`를 `0`으로 주었기 때문에 아무것도 삭제하지 않고, 인덱스 `1` 위치에 `"banana"`와 `"cherry"`를 추가합니다.

---

## 값을 교체할 수도 있어요

`splice()`는 삭제와 추가를 동시에 할 수 있기 때문에 값을 교체할 때도 사용할 수 있습니다.

```js
const fruits = ["apple", "banana", "cherry"];

fruits.splice(1, 1, "grape");

console.log(fruits); // ["apple", "grape", "cherry"]
```

인덱스 `1`의 `"banana"`를 삭제하고, 같은 자리에 `"grape"`를 넣은 예시입니다.

---

## slice와 splice 비교하기

| 구분 | slice() | splice() |
| --- | --- | --- |
| 원본 배열 | 변경하지 않음 | 변경함 |
| 반환값 | 잘라낸 새 배열 | 삭제된 요소 배열 |
| 주 사용 목적 | 배열 일부 복사 | 삭제, 추가, 교체 |
| 두 번째 인자 | 끝 인덱스 | 삭제할 개수 |

두 번째 인자의 의미가 서로 다르다는 점도 자주 헷갈리는 부분입니다.

`slice(1, 3)`은 인덱스 `1`부터 `3` 직전까지 가져온다는 뜻이고, `splice(1, 3)`은 인덱스 `1`부터 3개를 삭제한다는 뜻입니다.

---

## 언제 무엇을 쓰면 좋을까요?

원본 배열을 유지해야 한다면 `slice()`를 사용하면 됩니다.

```js
const copied = items.slice(0, 3);
```

배열 자체를 직접 수정해야 한다면 `splice()`를 사용할 수 있습니다.

```js
items.splice(2, 1);
```

다만 React 상태처럼 불변성을 지켜야 하는 데이터에서는 `splice()`를 바로 사용하는 것을 조심해야 합니다.

```js
// 권장하지 않는 방식
items.splice(2, 1);
setItems(items);
```

이런 경우에는 새 배열을 만들어 상태를 업데이트하는 편이 더 안전합니다.

---

## 마무리

`slice()`와 `splice()`를 구분할 때는 먼저 원본 배열이 바뀌는지부터 떠올리면 됩니다.

`slice()`는 원본을 그대로 둔 채 일부만 복사합니다.

`splice()`는 원본 배열을 직접 수정합니다.

이 차이만 확실히 기억해도 배열을 다루다가 예상치 못한 값 변경으로 고생하는 일을 많이 줄일 수 있습니다.
