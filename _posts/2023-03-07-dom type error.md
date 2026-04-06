---
title: "[TypeScript] DOM 조작 시 타입 에러 해결하기: NodeListOf와 Element 타입 지정"
date: 2023-03-07T17:01:000
categories: [typescript]
tags: [typescript] #소문자만 가능
---

자바스크립트에서는 아무 문제 없던 HTML 조작 코드가 타입스크립트(TypeScript)에서는 빨간 줄을 띄우며 우리를 당황하게 하곤 합니다. 특히 `querySelectorAll`을 사용할 때 발생하는 타입 에러의 원인과 해결 방법을 정리해 보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">문제 발생: Property 'style' does not exist</b>

특정 클래스를 가진 요소들의 스타일을 변경하려 할 때 다음과 같은 에러를 마주했습니다.

```tsx
const input = document.querySelectorAll(".input");
input.style.color = "red";
// Error: Property 'style' does not exist on type 'NodeListOf<Element>'.
```

처음에는 `querySelectorAll`이 반환한 값이 **배열(NodeList)** 형태이기 때문에 인덱스로 접근하면 해결될 줄 알았습니다. 하지만 인덱스로 접근해도 에러는 사라지지 않습니다.

```tsx
const input = document.querySelectorAll(".input");
input[0].style.color = "red";
// Error: Property 'style' does not exist on type 'Element'.
```

### 왜 에러가 날까?

타입스크립트 입장에서 `document.querySelectorAll`은 기본적으로 `NodeListOf<Element>`를 반환합니다. 일반적인 `Element` 타입은 추상적인 개념이라 `style`, `href`, `src` 같은 구체적인 태그 속성을 포함하고 있지 않기 때문입니다.

---

## <b style="border-bottom:2px solid gray" class="h2">해결 방법: 구체적인 타입 지정</b>

이 문제를 해결하려면 해당 요소가 정확히 어떤 HTML 요소인지 타입스크립트에게 알려줘야 합니다.

### 1. HTMLElement로 지정하기

단순히 `style` 속성만 사용하고 싶다면 `HTMLElement`를 사용합니다.

```tsx
const input: NodeListOf<HTMLElement> = document.querySelectorAll(".input");
input[0].style.color = "red"; // 해결!
```

### 2. 태그별 전용 타입 사용하기

하지만 `input` 태그의 `value`나 `focus()` 함수처럼 특정 태그에만 있는 기능을 쓰려면 더 구체적인 타입을 지정해야 합니다.

---

## <b style="border-bottom:2px solid gray" class="h2">주요 HTML 태그별 타입 리스트</b>

각 태그마다 고유한 인터페이스가 존재합니다. 상황에 맞는 타입을 사용해 보세요.

| 태그    | 타입 (Interface)                          | 비고                             |
| :------ | :---------------------------------------- | :------------------------------- |
| input   | `HTMLInputElement`                        | `value`, `focus()`, `checked` 등 |
| a       | `HTMLAnchorElement`                       | `href`, `target` 등              |
| button  | `HTMLButtonElement`                       | `disabled`, `type` 등            |
| img     | `HTMLImageElement`                        | `src`, `alt` 등                  |
| h1 ~ h6 | `HTMLHeadingElement`                      | 제목 태그 공통                   |
| div / p | `HTMLDivElement` / `HTMLParagraphElement` | 각 태그 전용 타입                |

### 예시 코드

```tsx
const input: NodeListOf<HTMLInputElement> = document.querySelectorAll(".input");
input[0].focus(); // HTMLInputElement 타입이기에 가능합니다.

const link: HTMLAnchorElement | null = document.querySelector(".link");
if (link) link.href = "https://google.com";
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>타입스크립트에서 DOM을 다루는 것은 조금 까다롭지만, 정확한 타입을 지정해두면 런타임에 발생할 수 있는 예상치 못한 에러를 미리 방지할 수 있습니다. 요소가 <code>null</code>일 가능성까지 고려하여 옵셔널 체이닝(<code>?.</code>)이나 조건문을 함께 사용하면 더욱 안전한 코드가 됩니다.</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
