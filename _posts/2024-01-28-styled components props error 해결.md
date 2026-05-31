---
title: "[Error] styled-components props 경고 해결하기"
date: 2024-01-28T18:00:00
categories: [error]
tags: [error, react, styled-components]
description: "styled-components에서 스타일용 props가 DOM으로 전달되어 발생하는 React 경고를 transient props로 해결하는 방법을 정리했습니다."
custom_style: true
---

## 발생한 문제

styled-components로 컴포넌트를 만들고 props를 전달하다 보면 React 콘솔에 경고가 나타날 수 있습니다.

예를 들어 스타일 계산에만 쓰려고 `backgroundColor` 같은 prop을 넘겼는데, 이 값이 실제 DOM 요소까지 전달되는 경우입니다.

React는 알 수 없는 DOM 속성을 발견하면 경고를 보여줍니다.

---

## 왜 이런 경고가 발생할까요?

styled-components에서 만든 컴포넌트도 결국 내부적으로는 실제 HTML 요소를 렌더링합니다.

```tsx
const Box = styled.div<{ backgroundColor: string }>`
  background-color: ${({ backgroundColor }) => backgroundColor};
`;

<Box backgroundColor="red" />;
```

위 코드에서 `backgroundColor`는 스타일 계산에는 필요하지만, 실제 `div` 태그의 표준 속성은 아닙니다.

그래서 React가 DOM에 알 수 없는 속성이 전달되었다고 경고할 수 있습니다.

---

## 해결 방법: transient props 사용하기

styled-components에서는 스타일 전용 props 앞에 `$`를 붙이는 방식을 권장합니다.

```tsx
const Box = styled.div<{ $backgroundColor: string }>`
  background-color: ${({ $backgroundColor }) => $backgroundColor};
`;

<Box $backgroundColor="red" />;
```

이렇게 `$`가 붙은 prop을 transient props라고 부릅니다.

transient props는 styled-components 내부에서 스타일 계산에 사용할 수 있지만, 실제 DOM 요소로는 전달되지 않습니다.

---

## 버튼 예시

```tsx
import styled from "styled-components";

interface ButtonProps {
  $variant: "primary" | "secondary";
}

const Button = styled.button<ButtonProps>`
  padding: 10px 16px;
  border: none;
  border-radius: 4px;
  color: white;
  background-color: ${({ $variant }) =>
    $variant === "primary" ? "#2563eb" : "#6b7280"};
`;

export default Button;
```

사용할 때는 이렇게 작성합니다.

```tsx
<Button $variant="primary">저장</Button>
```

`$variant`는 스타일에는 사용되지만 실제 button DOM에는 전달되지 않습니다.

---

## 언제 $를 붙이면 좋을까요?

아래처럼 스타일 계산에만 필요한 값이라면 `$`를 붙이는 것이 좋습니다.

- `$color`
- `$size`
- `$variant`
- `$isActive`
- `$backgroundColor`

반대로 실제 DOM 속성으로 전달되어야 하는 값은 `$`를 붙이지 않습니다.

예를 들어 `disabled`, `type`, `aria-label` 같은 속성은 실제 DOM에 필요합니다.

---

## 마무리

styled-components에서 스타일용 props를 그대로 넘기면 React DOM 경고가 발생할 수 있습니다.

스타일 계산에만 필요한 props라면 이름 앞에 `$`를 붙여 transient props로 관리하면 됩니다.

이 방식은 경고를 줄이고, 컴포넌트 props의 의도도 더 명확하게 만들어줍니다.
