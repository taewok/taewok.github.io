---
title: "[TypeScript] styled-components props 타입 지정하기"
date: 2023-03-16T17:01:00
categories: [typescript, react]
tags: [typescript, react, styled-components, props]
description: "TypeScript와 styled-components를 함께 사용할 때 props 타입을 지정하고 조건부 스타일을 안전하게 작성하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 styled-components props에도 타입이 필요해요

`styled-components`에서는 props를 이용해 스타일을 동적으로 바꿀 수 있습니다. 예를 들어 버튼의 색상, 크기, 애니메이션 지연 시간 등을 props로 전달할 수 있어요.

TypeScript를 함께 사용한다면 이 props에도 타입을 지정해두는 것이 좋습니다. 그래야 어떤 값이 들어와야 하는지 컴포넌트를 사용하는 시점에 바로 확인할 수 있습니다.

---

## 📦 설치하기

TypeScript 프로젝트에서 `styled-components`를 사용하려면 라이브러리를 설치합니다.

```bash
npm install styled-components
```

사용 중인 버전에 따라 타입 패키지가 필요할 수 있습니다.

```bash
npm install -D @types/styled-components
```

---

## 🛠️ props 전달하기

예를 들어 여러 input에 서로 다른 애니메이션 지연 시간을 주고 싶다고 해볼게요.

```tsx
const Answer = () => {
  return (
    <>
      <Input $delay={100} />
      <Input $delay={300} />
      <Input $delay={500} />
    </>
  );
};
```

여기서 `$delay`는 스타일 계산에만 사용할 props입니다.

---

## 🧩 단일 props 타입 지정하기

전달할 props가 하나라면 제네릭으로 바로 타입을 지정할 수 있습니다.

```tsx
const Input = styled.input<{ $delay: number }>`
  animation-delay: ${({ $delay }) => $delay}ms;
`;
```

이제 `$delay`에는 숫자만 전달할 수 있습니다.

```tsx
<Input $delay={300} />
```

문자열을 넣으면 TypeScript가 에러를 알려줍니다.

```tsx
<Input $delay="300" />
```

---

## 🧱 여러 props는 interface로 분리하기

전달할 props가 많아지면 `interface`로 분리하는 편이 읽기 좋습니다.

```tsx
import styled, { css } from "styled-components";

interface InputProps {
  $delay: number;
  $checked: boolean;
}

const Input = styled.input<InputProps>`
  width: 100px;
  height: 40px;

  ${({ $checked, $delay }) =>
    $checked &&
    css`
      animation-delay: ${$delay}ms;
    `}
`;
```

`$checked`가 `true`일 때만 `animation-delay` 스타일이 적용됩니다.

---

## 💡 props 이름 앞에 $를 붙이는 이유

예제에서는 `delay` 대신 `$delay`를 사용했습니다.

`$`가 붙은 props는 styled-components에서 transient props로 취급됩니다. 스타일 계산에는 사용할 수 있지만 실제 DOM 속성으로는 전달되지 않습니다.

이렇게 하면 다음처럼 알 수 없는 속성이 HTML에 내려가는 일을 막을 수 있습니다.

```html
<input delay="300" />
```

스타일 전용 props라면 `$delay`, `$checked`처럼 `$`를 붙이는 습관을 들이면 좋습니다.

---

## ✅ 정리

TypeScript와 `styled-components`를 함께 사용할 때는 스타일 props에도 타입을 지정하는 것이 좋습니다.

- 제네릭으로 props 타입을 넘길 수 있습니다.
- props가 하나라면 inline 타입으로도 충분합니다.
- props가 많다면 `interface`로 분리하면 읽기 좋습니다.
- 스타일 전용 props에는 `$`를 붙여 DOM 전달을 막을 수 있습니다.
- 조건부 스타일에는 `css` helper를 함께 사용할 수 있습니다.

동적인 스타일이 많아질수록 props 타입을 명확히 해두는 것이 유지보수에 도움이 됩니다.
