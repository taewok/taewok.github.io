---
title: "[TypeScript] 타입스크립트란? 자바스크립트와의 차이"
date: 2023-03-03T10:01:00+09:00
categories: [typescript]
tags: [typescript, javascript, es5, esnext]
description: "TypeScript가 무엇인지, JavaScript와 어떤 차이가 있는지, 타입을 사용하면 어떤 장점이 있는지 정리했습니다."
custom_style: true
---

## 🧐 TypeScript는 무엇일까요?

TypeScript는 JavaScript에 타입 시스템을 추가한 언어입니다. JavaScript 문법을 기반으로 하면서, 변수나 함수 인자에 어떤 타입이 들어와야 하는지 미리 표시할 수 있습니다.

예를 들어 문자열이 들어와야 하는 곳에 숫자를 넣으면 TypeScript가 코드 작성 단계에서 에러를 알려줍니다.

---

## 💡 JavaScript와 TypeScript의 관계

JavaScript 생태계를 이해할 때는 다음 세 가지를 구분해보면 좋습니다.

- `ES5`: 오래된 브라우저에서도 동작하는 표준 JavaScript입니다.
- `ESNext`: ES6 이후 계속 추가되고 있는 최신 JavaScript 문법을 의미합니다.
- `TypeScript`: JavaScript에 타입 기능을 추가한 언어입니다.

브라우저는 TypeScript를 직접 실행하지 못합니다. TypeScript 코드는 컴파일 과정을 거쳐 JavaScript 코드로 변환된 뒤 실행됩니다.

---

## 🧩 타입이 있으면 무엇이 좋을까요?

타입을 사용하는 가장 큰 이유는 실수를 더 빨리 발견하기 위해서입니다.

예를 들어 다음 함수는 이름과 나이를 받아 사용자 정보를 만든다고 가정해볼게요.

```js
function makeName(name, age) {
  return `${name}은 ${age}살입니다.`;
}

makeName(32, "오윤희");
```

JavaScript에서는 인자의 순서를 바꿔도 문법적으로는 에러가 나지 않습니다. 실제 실행 결과를 보고 나서야 문제를 발견할 수 있습니다.

TypeScript에서는 인자의 타입을 미리 지정할 수 있습니다.

```ts
function makeName(name: string, age: number) {
  return `${name}은 ${age}살입니다.`;
}

makeName(32, "오윤희");
```

이 코드는 TypeScript에서 에러가 납니다.

```txt
number 형식의 인수는 string 형식의 매개변수에 할당될 수 없습니다.
```

즉, 코드를 실행하기 전에 잘못된 사용을 미리 확인할 수 있습니다.

---

## 🔄 TypeScript는 어떻게 실행될까요?

브라우저는 TypeScript를 직접 이해하지 못합니다. 그래서 TypeScript 코드는 JavaScript로 변환되어야 합니다.

이 과정을 컴파일 또는 트랜스파일이라고 부릅니다.

```txt
TypeScript 코드
↓
TypeScript Compiler
↓
JavaScript 코드
↓
브라우저 실행
```

TypeScript Compiler, 줄여서 `tsc`가 이 변환을 담당합니다.

---

## ✅ 정리

TypeScript는 JavaScript에 타입을 더한 언어입니다.

- JavaScript 문법을 기반으로 합니다.
- 변수, 함수 인자, 반환값에 타입을 지정할 수 있습니다.
- 잘못된 타입 사용을 코드 작성 단계에서 발견할 수 있습니다.
- 브라우저에서 실행되기 전 JavaScript로 변환됩니다.

처음에는 타입을 적는 일이 번거롭게 느껴질 수 있지만, 프로젝트가 커질수록 실수를 줄이고 코드를 이해하기 쉽게 만드는 데 큰 도움이 됩니다.
