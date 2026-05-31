---
title: "[JavaScript] const, let, var 차이점 정리"
date: 2023-02-25T19:30:00
categories: [javascript]
tags: [javascript, es6, const, let, var]
description: "JavaScript에서 변수를 선언할 때 사용하는 const, let, var의 차이점과 어떤 상황에서 어떤 키워드를 쓰면 좋은지 정리했습니다."
custom_style: true
---

## 🧐 const, let, var는 어떻게 다를까요?

JavaScript ES6 이전에는 변수를 선언할 때 주로 `var`를 사용했습니다.

하지만 ES6 이후에는 `const`와 `let`이 도입되면서 변수 선언 방식이 더 명확해졌습니다.

처음에는 셋의 차이가 헷갈릴 수 있습니다.

```txt
const는 언제 쓰지?
let은 언제 쓰지?
var는 아직 써도 되나?
```

이번 글에서는 `const`, `let`, `var`의 차이와 권장 사용 방식을 정리해보겠습니다.

---

## 🔒 const

`const`는 재할당할 수 없는 변수를 선언할 때 사용합니다.

```js
const age = 21;

age = 22;
// TypeError: Assignment to constant variable.
```

또한 `const`는 선언과 동시에 값을 할당해야 합니다.

```js
const name;
// SyntaxError: Missing initializer in const declaration
```

```js
const name = "독고진";
```

기본적으로 값이 다시 바뀌지 않는다면 `const`를 사용하는 것이 좋습니다.

---

## ✏️ let

`let`은 값이 바뀔 수 있는 변수를 선언할 때 사용합니다.

```js
let name = "독고진";

name = "구애정";

console.log(name);
// "구애정"
```

하지만 같은 이름으로 다시 선언할 수는 없습니다.

```js
let name = "독고진";
let name = "구애정";
// SyntaxError
```

즉, `let`은 재할당은 가능하지만 재선언은 불가능합니다.

---

## ⚠️ var

`var`는 ES6 이전에 사용하던 변수 선언 키워드입니다.

현재는 가능하면 사용하지 않는 편이 좋습니다.

가장 큰 이유는 같은 이름으로 다시 선언해도 에러가 나지 않는다는 점입니다.

```js
var age = 28;
console.log(age);
// 28

var age = "old";
console.log(age);
// "old"
```

실수로 같은 변수명을 다시 선언해도 조용히 덮어쓰기 때문에 코드가 길어질수록 버그를 만들기 쉽습니다.

---

## 🧱 스코프 차이

`const`와 `let`은 블록 스코프를 가집니다.

```js
if (true) {
  const a = 1;
  let b = 2;
}

console.log(a);
console.log(b);
// ReferenceError
```

반면 `var`는 함수 스코프를 가집니다.

```js
if (true) {
  var a = 1;
}

console.log(a);
// 1
```

이런 차이 때문에 `var`는 예상보다 넓은 범위에서 접근될 수 있습니다.

---

## 📊 비교 정리

| 키워드 | 재할당 | 재선언 | 스코프 | 권장 여부 |
| :--- | :---: | :---: | :--- | :--- |
| `const` | 불가능 | 불가능 | 블록 스코프 | 기본으로 사용 |
| `let` | 가능 | 불가능 | 블록 스코프 | 값이 바뀔 때 사용 |
| `var` | 가능 | 가능 | 함수 스코프 | 사용 지양 |

---

## ✅ 정리

모던 JavaScript에서는 다음 기준으로 변수를 선언하면 좋습니다.

1. 기본적으로 `const`를 사용합니다.
2. 값이 바뀌어야 하는 경우에만 `let`을 사용합니다.
3. `var`는 특별한 이유가 없다면 사용하지 않습니다.

이 기준만 지켜도 변수 관련 실수를 많이 줄일 수 있습니다.
