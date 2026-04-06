---
title: "[TypeScript] 타입스크립트 기본 타입 정리 (Primitive, Array, Enum, Any)"
date: 2023-03-03T17:01:00
categories: [typescript]
tags: [typescript, type, interface, enum, frontend]
description: "TypeScript에서 사용하는 핵심 타입(number, string, array, tuple, enum, void, any)의 정의와 올바른 선언 방법을 예제 코드와 함께 정리합니다."
custom_style: true
---

TypeScript를 사용하는 가장 큰 이유는 변수나 함수에 명확한 **타입(Type)**을 지정하여 에러를 사전에 방지하기 위함입니다.

오늘은 타입스크립트에서 가장 자주 사용되는 기본 타입들과 선언 방법을 정리해 보겠습니다. 📘

---

## 1. 기본 타입 (Primitive Types)

자바스크립트의 기본 자료형과 매핑되는 타입들입니다.

### number (숫자)

정수, 실수, Infinity, NaN 등 모든 숫자 값을 포함합니다.

```typescript
const age: number = 22;
const weight: number = 65.5;
```

### string (문자열)

작은따옴표(''), 큰따옴표(""), 백틱(``)으로 감싸진 텍스트입니다.

```typescript
const name: string = "나나";
```

### boolean (논리)

참(`true`)과 거짓(`false`)을 나타냅니다.

```typescript
const isStudent: boolean = true;
```

### null / undefined

값이 없거나 정의되지 않은 상태를 나타냅니다.

```typescript
const empty: null = null;
const nothing: undefined = undefined;
```

---

## 2. 참조 및 집합 타입 (Reference & Collection)

### Array (배열)

배열을 선언하는 방법은 두 가지가 있습니다.

1.  **`타입[]`**: 가장 많이 사용하는 방식
2.  **`Array<타입>`**: 제네릭 방식

```typescript
// 1. 문자열만 담을 수 있는 배열
const names: string[] = ["오윤희", "천서진"];

// 2. 숫자만 담을 수 있는 배열
const numbers: Array<number> = [1, 2, 3];

// 3. (중요) 두 가지 이상의 타입이 섞인 배열 (Union Type)
const mixData: (string | number)[] = [1, "Two", 3];
```

**주의:** `const data: array = []` 처럼 소문자 `array`라는 타입은 존재하지 않습니다. 반드시 `any[]` 혹은 `Array<string>` 처럼 내부 타입을 명시해야 합니다.

### Tuple (튜플)

배열과 비슷하지만, **요소의 개수(길이)와 순서에 맞는 타입**이 정확히 지정되어야 합니다.

```typescript
// 첫 번째는 숫자, 두 번째와 세 번째는 문자열이어야 함
const user: [number, string, string] = [22, "오윤희", "password"];
```

### Object (객체)

단순히 `object`라고 명시할 수도 있지만, 보통은 인터페이스(Interface)나 타입 별칭(Type Alias)을 사용하여 구체적인 속성을 정의합니다.

```typescript
const person: { name: string; age: number } = {
  name: "나나",
  age: 22,
};
```

---

## 3. 특수 타입 (Special Types)

### Enum (열거형)

특정 값(상수)들의 집합에 이름을 붙여 관리할 때 사용합니다. 인덱스 번호가 자동으로 매겨집니다.

```typescript
// 값을 지정하지 않으면 0부터 순차 증가
enum Status {
  Ready, // 0
  Start, // 1
  Finish, // 2
}

// 초기값을 주면 그 다음부터 1씩 증가
enum Color {
  Red = 10,
  Green, // 11
  Blue, // 12
}
```

### Any (모든 타입)

어떤 타입이든 들어올 수 있음을 의미합니다.
**주의:** `any`를 남발하면 타입스크립트를 쓰는 의미가 사라지므로, 타입 추론이 불가능하거나 외부 라이브러리를 사용할 때 등 **불가피한 경우에만 제한적으로 사용**해야 합니다.

```typescript
let value: any = "Hello";
value = 123; // 에러 없음
value = false; // 에러 없음
```

### Void (반환값 없음)

함수에서 값을 반환하지 않을 때 사용합니다.

```typescript
const printName = (): void => {
  console.log("눈누나난");
  // return 문이 없거나 return; 만 존재해야 함
};
```

### Symbol (심볼)

ES6에서 도입된 타입으로, 유일하고 변경 불가능한 값을 생성합니다. 주로 객체의 고유한 프로퍼티 키로 사용됩니다.

```typescript
const sym1 = Symbol("key");
const sym2 = Symbol("key");

console.log(sym1 === sym2); // false (설명이 같아도 완전히 다른 값 취급)
```

---

## 마치며

타입스크립트의 타입 시스템은 이보다 훨씬 깊고 방대하지만, 오늘 정리한 내용만 확실히 알아도 기본적인 개발에는 큰 무리가 없습니다.

특히 **배열(`[]`)**과 **튜플(`[type, type]`)**의 차이, 그리고 **`any`** 사용을 지양해야 한다는 점을 꼭 기억해 주세요!

궁금한 점이나 피드백이 있다면 언제든 댓글 남겨주세요. 👋
