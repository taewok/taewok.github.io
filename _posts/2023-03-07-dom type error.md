---
title: "[TypeScript] DOM 조작 시 타입 에러 해결하기"
date: 2023-03-07T17:01:00
categories: [typescript]
tags: [typescript, dom, queryselector, html-element]
description: "TypeScript에서 querySelectorAll로 DOM을 선택할 때 발생하는 Element 타입 에러의 원인과 HTMLElement 타입 지정 방법을 정리했습니다."
custom_style: true
---

## 🧐 JavaScript에서는 되는데 TypeScript에서는 왜 에러가 날까요?

JavaScript에서는 DOM 요소를 선택한 뒤 바로 스타일을 바꾸는 코드를 자주 작성합니다. 그런데 TypeScript에서는 같은 코드에 빨간 줄이 생길 때가 있습니다.

대표적인 예가 `querySelectorAll`로 요소를 선택한 뒤 `style`에 접근하는 경우입니다.

```tsx
const input = document.querySelectorAll(".input");

input.style.color = "red";
// Property 'style' does not exist on type 'NodeListOf<Element>'.
```

이번 글에서는 이 에러가 왜 발생하는지, 그리고 DOM 요소 타입을 어떻게 지정하면 되는지 정리해보겠습니다.

---

## ⚠️ querySelectorAll은 배열처럼 보이지만 NodeList입니다

`querySelectorAll`은 여러 요소를 가져오기 때문에 배열처럼 보이는 값을 반환합니다.

```tsx
const inputs = document.querySelectorAll(".input");
```

하지만 이 값은 정확히는 `NodeListOf<Element>`입니다. 그래서 바로 `inputs.style`처럼 접근할 수 없습니다. 여러 요소 목록 자체에는 `style` 속성이 없기 때문입니다.

먼저 개별 요소에 접근해야 합니다.

```tsx
const inputs = document.querySelectorAll(".input");

inputs[0].style.color = "red";
```

그런데 이 코드도 TypeScript에서는 에러가 날 수 있습니다.

```txt
Property 'style' does not exist on type 'Element'.
```

---

## 💡 Element에는 style이 없을 수 있어요

TypeScript 입장에서 `Element`는 아주 넓은 타입입니다.

HTML 요소일 수도 있고, SVG 요소일 수도 있습니다. 그래서 `Element` 타입에는 `style`, `value`, `checked` 같은 HTML 전용 속성이 안전하게 있다고 보장할 수 없습니다.

그래서 TypeScript에게 "이 요소는 HTML 요소야"라고 알려줘야 합니다.

---

## 🛠️ HTMLElement로 타입 지정하기

단순히 `style` 속성만 사용하고 싶다면 `HTMLElement`를 사용할 수 있습니다.

```tsx
const inputs = document.querySelectorAll<HTMLElement>(".input");

inputs[0].style.color = "red";
```

`querySelectorAll<HTMLElement>`처럼 제네릭을 사용하면 선택된 요소의 타입을 지정할 수 있습니다.

---

## 🧩 태그별 전용 타입 사용하기

input 태그처럼 특정 태그에만 있는 속성을 사용해야 한다면 더 구체적인 타입을 지정하는 것이 좋습니다.

```tsx
const inputs = document.querySelectorAll<HTMLInputElement>(".input");

inputs[0].value = "hello";
inputs[0].focus();
```

자주 사용하는 DOM 타입은 다음과 같습니다.

| 태그 | 타입 | 자주 쓰는 속성 |
| :--- | :--- | :--- |
| `input` | `HTMLInputElement` | `value`, `checked`, `focus()` |
| `a` | `HTMLAnchorElement` | `href`, `target` |
| `button` | `HTMLButtonElement` | `disabled`, `type` |
| `img` | `HTMLImageElement` | `src`, `alt` |
| `div` | `HTMLDivElement` | `style`, `dataset` |

---

## ✅ null도 함께 처리하기

`querySelector`는 요소를 찾지 못하면 `null`을 반환합니다.

```tsx
const link = document.querySelector<HTMLAnchorElement>(".link");

link.href = "https://google.com";
```

위 코드는 `link`가 `null`일 수 있어서 에러가 납니다. 조건문으로 확인한 뒤 사용하는 것이 안전합니다.

```tsx
const link = document.querySelector<HTMLAnchorElement>(".link");

if (link) {
  link.href = "https://google.com";
}
```

간단한 접근이라면 옵셔널 체이닝을 사용할 수도 있습니다.

```tsx
link?.focus();
```

---

## ✅ 정리

TypeScript에서 DOM을 다룰 때는 선택한 요소가 어떤 HTML 요소인지 타입으로 알려주는 것이 중요합니다.

- `querySelectorAll`은 기본적으로 `NodeListOf<Element>`를 반환합니다.
- `Element`는 넓은 타입이라 `style`, `value` 같은 속성을 바로 사용할 수 없습니다.
- `querySelectorAll<HTMLElement>()`처럼 타입을 지정할 수 있습니다.
- input, button, img처럼 태그별 전용 타입을 사용하면 더 안전합니다.
- `querySelector`는 `null`을 반환할 수 있으므로 조건문이나 옵셔널 체이닝을 함께 사용하면 좋습니다.

DOM 타입을 조금 더 구체적으로 지정해두면 TypeScript가 더 정확하게 도와줄 수 있습니다.
