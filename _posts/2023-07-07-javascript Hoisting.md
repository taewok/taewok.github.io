---
title: "[JavaScript] 호이스팅 이해하기"
date: 2023-07-07T18:35:00
categories: [javascript]
tags: [javascript, hoisting, var, let, const]
description: "JavaScript 호이스팅의 의미와 var, let, const, 함수 선언식에서 어떻게 다르게 동작하는지 정리했습니다."
custom_style: true
---

## 호이스팅이란?

호이스팅은 JavaScript가 코드를 실행하기 전에 선언을 먼저 인식하는 동작을 말합니다.

말 그대로 코드가 실제로 위로 이동하는 것은 아니지만, 실행 컨텍스트가 만들어지는 과정에서 선언 정보가 먼저 준비됩니다.

그래서 어떤 코드는 선언보다 앞에서 사용해도 동작하고, 어떤 코드는 에러가 발생합니다.

---

## var의 호이스팅

`var`로 선언한 변수는 선언이 호이스팅되고, 값은 `undefined`로 초기화됩니다.

```js
console.log(name); // undefined

var name = "taewok";
```

JavaScript는 위 코드를 대략 이렇게 이해한다고 볼 수 있습니다.

```js
var name;

console.log(name); // undefined

name = "taewok";
```

선언은 먼저 준비되지만, 실제 값 할당은 원래 코드 위치에서 일어납니다.

그래서 에러는 나지 않지만 `undefined`가 출력됩니다.

---

## let과 const의 호이스팅

`let`과 `const`도 선언 자체는 호이스팅됩니다.

하지만 선언 전에 접근할 수 없습니다.

```js
console.log(name); // ReferenceError

let name = "taewok";
```

이 구간을 TDZ, 즉 Temporal Dead Zone이라고 부릅니다.

변수가 선언되기 전까지는 해당 변수에 접근할 수 없는 구간입니다.

`const`도 마찬가지입니다.

```js
console.log(count); // ReferenceError

const count = 1;
```

---

## 함수 선언식의 호이스팅

함수 선언식은 함수 전체가 호이스팅됩니다.

```js
hello(); // "안녕하세요"

function hello() {
  console.log("안녕하세요");
}
```

그래서 함수 선언식은 선언보다 먼저 호출해도 동작합니다.

---

## 함수 표현식은 다르게 동작해요

함수를 변수에 담는 함수 표현식은 변수 선언 방식에 따라 다르게 동작합니다.

```js
hello(); // ReferenceError

const hello = function () {
  console.log("안녕하세요");
};
```

`hello`는 `const`로 선언되었기 때문에 선언 전에 접근할 수 없습니다.

`var`를 사용하면 에러 형태가 조금 달라집니다.

```js
hello(); // TypeError: hello is not a function

var hello = function () {
  console.log("안녕하세요");
};
```

이 경우 `hello`는 `undefined`로 초기화되어 있기 때문에 함수처럼 호출할 수 없어 TypeError가 발생합니다.

---

## var, let, const 비교

| 선언 방식 | 선언 전 접근 | 결과 |
| --- | --- | --- |
| `var` | 가능 | `undefined` |
| `let` | 불가능 | `ReferenceError` |
| `const` | 불가능 | `ReferenceError` |
| 함수 선언식 | 가능 | 함수 호출 가능 |

실무에서는 예측하기 쉬운 코드를 위해 `var`보다 `let`과 `const`를 사용하는 것이 좋습니다.

---

## 마무리

호이스팅은 JavaScript가 선언을 먼저 인식하는 동작입니다.

`var`는 선언 전에 접근하면 `undefined`가 나오고, `let`과 `const`는 ReferenceError가 발생합니다.

함수 선언식은 선언 전에도 호출할 수 있지만, 코드 흐름을 명확히 하기 위해 가능하면 선언 후 사용하는 습관을 들이는 것이 좋습니다.
