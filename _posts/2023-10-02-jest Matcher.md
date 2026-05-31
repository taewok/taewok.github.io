---
title: "[React] Jest Matcher 기본 정리"
date: 2023-10-02T21:05:00
categories: [react]
tags: [react, jest, test, matcher]
description: "Jest 테스트에서 자주 사용하는 matcher의 의미와 예시를 정리했습니다."
custom_style: true
---

## Matcher란?

Jest에서 matcher는 테스트 결과가 기대한 값과 맞는지 검증할 때 사용하는 함수입니다.

보통 `expect()`와 함께 사용합니다.

```js
expect(result).toBe(3);
```

여기서 `toBe`가 matcher입니다.

---

## toBe

`toBe`는 원시값을 정확히 비교할 때 사용합니다.

```js
test("1 + 2는 3입니다", () => {
  expect(1 + 2).toBe(3);
});
```

숫자, 문자열, boolean처럼 단순한 값을 비교할 때 잘 어울립니다.

---

## toEqual

객체나 배열을 비교할 때는 `toEqual`을 사용합니다.

```js
test("객체 값 비교", () => {
  const user = {
    name: "taewok",
    age: 20,
  };

  expect(user).toEqual({
    name: "taewok",
    age: 20,
  });
});
```

`toBe`는 같은 참조인지 비교하기 때문에 객체 비교에는 적합하지 않습니다.

객체의 내용이 같은지 보고 싶다면 `toEqual`을 사용하면 됩니다.

---

## toContain

배열이나 문자열에 특정 값이 포함되어 있는지 확인할 때 사용합니다.

```js
test("배열에 값이 포함되어 있습니다", () => {
  const fruits = ["apple", "banana", "grape"];

  expect(fruits).toContain("banana");
});
```

문자열에도 사용할 수 있습니다.

```js
expect("hello world").toContain("world");
```

---

## toMatch

`toMatch`는 문자열이 특정 정규식과 일치하는지 확인합니다.

```js
test("이메일 형식 확인", () => {
  const email = "test@example.com";

  expect(email).toMatch(/@/);
});
```

문자열 패턴을 검증할 때 유용합니다.

---

## null과 undefined 확인하기

값이 `null`인지 확인할 때는 `toBeNull`을 사용할 수 있습니다.

```js
expect(value).toBeNull();
```

값이 `undefined`가 아닌지 확인하려면 `toBeDefined`를 사용할 수 있습니다.

```js
expect(value).toBeDefined();
```

반대로 `undefined`인지 확인하려면 `toBeUndefined`를 사용합니다.

```js
expect(value).toBeUndefined();
```

---

## truthy와 falsy 확인하기

JavaScript의 truthy, falsy 값을 확인할 때는 아래 matcher를 사용할 수 있습니다.

```js
expect("hello").toBeTruthy();
expect("").toBeFalsy();
```

정확히 `true`인지 확인하고 싶다면 `toBe(true)`를 사용하고, truthy인지 정도만 확인하면 `toBeTruthy()`를 사용하면 됩니다.

---

## 숫자 비교하기

숫자의 크기를 비교할 때는 아래 matcher를 사용할 수 있습니다.

```js
expect(10).toBeGreaterThan(5);
expect(10).toBeLessThan(20);
expect(10).toBeGreaterThanOrEqual(10);
expect(10).toBeLessThanOrEqual(10);
```

범위를 검증할 때 자주 사용합니다.

---

## 길이 확인하기

배열이나 문자열의 길이를 확인할 때는 `toHaveLength`를 사용할 수 있습니다.

```js
expect(["a", "b", "c"]).toHaveLength(3);
expect("hello").toHaveLength(5);
```

---

## 마무리

Jest matcher는 테스트에서 기대값을 표현하는 도구입니다.

원시값은 `toBe`, 객체나 배열은 `toEqual`, 포함 여부는 `toContain`, 문자열 패턴은 `toMatch`를 자주 사용합니다.

matcher를 상황에 맞게 고르면 테스트 의도가 훨씬 명확해집니다.
