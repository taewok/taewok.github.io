---
title: "[React] styled-components에서 props로 theme 값 접근하기"
date: 2024-01-26T16:40:00
categories: [react, javascript]
tags: [react, styled-components, theme, props]
description: "styled-components에서 props를 사용해 theme 객체의 색상과 폰트 값을 동적으로 선택하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

버튼 컴포넌트를 만들다 보면 색상, 폰트, 배경색이 조금씩 다른 버튼이 필요해집니다.

이때 매번 새 컴포넌트를 만들기보다 props로 값만 바꿀 수 있으면 재사용성이 좋아집니다.

styled-components에서는 props와 theme을 함께 사용해 스타일 값을 동적으로 선택할 수 있습니다.

---

## theme 예시

먼저 theme 객체가 있다고 가정해볼게요.

```ts
export const theme = {
  fontSizes: {
    small: "12px",
    medium: "14px",
    large: "18px",
  },
  colors: {
    black: "#111111",
    white: "#ffffff",
    orange: "#ff8a00",
    grey: "#999999",
  },
};
```

이 값을 `ThemeProvider`로 앱에 전달하면 styled-components 안에서 `theme`으로 접근할 수 있습니다.

---

## props로 theme key 전달하기

```tsx
import styled from "styled-components";

interface ButtonProps {
  $fontSize: "small" | "medium" | "large";
  $color: "black" | "white" | "orange" | "grey";
}

const Button = styled.button<ButtonProps>`
  padding: 10px 30px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  font-size: ${({ theme, $fontSize }) => theme.fontSizes[$fontSize]};
  color: ${({ theme, $color }) => theme.colors[$color]};
`;
```

`$fontSize`와 `$color`로 theme 객체의 key를 전달하고 있습니다.

대괄호 표기법을 사용하면 문자열 key로 객체 속성에 접근할 수 있습니다.

---

## 사용하는 쪽 코드

```tsx
const App = () => {
  return (
    <>
      <Button $fontSize="small" $color="grey">
        작은 버튼
      </Button>

      <Button $fontSize="large" $color="orange">
        강조 버튼
      </Button>
    </>
  );
};
```

props 값만 바꿔도 같은 버튼 컴포넌트에서 다른 스타일을 적용할 수 있습니다.

---

## transient props를 사용하는 이유

styled-components에서 스타일용 props는 `$`를 붙이는 것이 좋습니다.

```tsx
<Button $color="orange" />
```

이런 props를 transient props라고 부릅니다.

`$`를 붙이면 해당 prop이 실제 DOM 속성으로 전달되지 않습니다.

스타일 계산에만 필요한 값이라면 `$color`, `$size`처럼 작성하는 습관을 들이면 React 경고를 줄일 수 있습니다.

---

## 마무리

styled-components에서는 props와 theme을 조합해 스타일을 유동적으로 바꿀 수 있습니다.

theme key를 props로 전달하고, 스타일 내부에서 `theme.colors[$color]`처럼 접근하면 됩니다.

스타일에만 사용하는 props라면 `$`를 붙여 transient props로 관리하는 것이 안전합니다.
