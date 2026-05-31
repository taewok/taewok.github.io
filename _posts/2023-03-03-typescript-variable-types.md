---
title: "[TypeScript] 타입스크립트 기본 타입 정리"
date: 2023-03-03T17:01:00
categories: [typescript]
tags: [typescript, type, interface, enum, frontend]
description: "TypeScript에서 사용하는 기본 타입 number, string, boolean, array, tuple, enum, void, any의 사용법을 예제와 함께 정리했습니다."
custom_style: true
---

## 🧐 TypeScript의 기본 타입을 알아볼까요?

TypeScript를 사용하는 가장 큰 이유는 값의 타입을 명확하게 정해서 실수를 줄이기 위해서입니다. 변수나 함수에 어떤 값이 들어올 수 있는지 미리 정해두면, 잘못된 사용을 코드 작성 단계에서 발견할 수 있습니다.

이번 글에서는 TypeScript에서 자주 사용하는 기본 타입들을 예제로 정리해보겠습니다.

---

## 🔢 원시 타입

### number

`number`는 숫자를 나타내는 타입입니다. 정수와 실수 모두 `number`에 포함됩니다.

```ts
const age: number = 22;
const weight: number = 65.5;
```

### string

`string`은 문자열 타입입니다.

```ts
const name: string = "나나";
```

### boolean

`boolean`은 `true` 또는 `false` 값을 나타냅니다.

```ts
const isStudent: boolean = true;
```

### null / undefined

값이 없거나 아직 정의되지 않은 상태를 나타낼 때 사용합니다.

```ts
const empty: null = null;
const nothing: undefined = undefined;
```

---

## 📦 배열 타입

배열 타입은 두 가지 방식으로 작성할 수 있습니다.

```ts
const names: string[] = ["오윤희", "천서진"];
const numbers: Array<number> = [1, 2, 3];
```

두 가지 이상의 타입이 섞인 배열은 union type으로 표현할 수 있습니다.

```ts
const mixData: (string | number)[] = [1, "Two", 3];
```

`array`라는 소문자 타입은 존재하지 않으므로 `string[]`, `number[]`, `Array<string>`처럼 내부 타입을 함께 작성해야 합니다.

---

## 🧩 Tuple

튜플은 배열과 비슷하지만, 각 위치에 들어갈 타입과 길이를 정해두는 타입입니다.

```ts
const user: [number, string, string] = [22, "오윤희", "password"];
```

위 예제에서는 첫 번째 값은 숫자, 두 번째와 세 번째 값은 문자열이어야 합니다.

---

## 🧱 Object

객체 타입은 속성 이름과 타입을 함께 작성합니다.

```ts
const person: { name: string; age: number } = {
  name: "나나",
  age: 22,
};
```

실무에서는 객체 구조가 길어질 수 있기 때문에 `interface`나 `type`으로 분리하는 경우가 많습니다.

```ts
interface Person {
  name: string;
  age: number;
}

const person: Person = {
  name: "나나",
  age: 22,
};
```

---

## 🏷️ Enum

`enum`은 여러 상수 값에 이름을 붙여 관리할 때 사용합니다.

```ts
enum Status {
  Ready,
  Start,
  Finish,
}

console.log(Status.Ready);
// 0
```

값을 직접 지정할 수도 있습니다.

```ts
enum Color {
  Red = 10,
  Green,
  Blue,
}

console.log(Color.Green);
// 11
```

---

## ⚠️ any

`any`는 어떤 타입이든 허용하는 타입입니다.

```ts
let value: any = "Hello";

value = 123;
value = false;
```

편해 보이지만 `any`를 많이 사용하면 TypeScript를 쓰는 장점이 줄어듭니다. 타입을 알 수 없는 외부 데이터나 임시 코드에서만 제한적으로 사용하는 편이 좋습니다.

---

## 🚫 void

`void`는 함수가 값을 반환하지 않을 때 사용합니다.

```ts
const printName = (): void => {
  console.log("눈누나난");
};
```

---

## ✅ 정리

TypeScript에는 다양한 기본 타입이 있습니다.

- `number`: 숫자
- `string`: 문자열
- `boolean`: 참 또는 거짓
- `null`, `undefined`: 값이 없거나 정의되지 않은 상태
- `string[]`, `Array<number>`: 배열
- `tuple`: 길이와 순서가 정해진 배열
- `object`: 객체 구조
- `enum`: 이름이 있는 상수 집합
- `any`: 모든 타입 허용
- `void`: 반환값 없음

처음에는 타입을 하나씩 적는 것이 낯설 수 있지만, 기본 타입만 익혀도 TypeScript 코드를 훨씬 안정적으로 작성할 수 있습니다.
