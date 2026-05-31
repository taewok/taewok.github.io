---
title: "[TypeScript] styled-components에서 타입 지정하기"
date: 2023-02-28T15:30:00
categories: [typescript, react]
tags: [typescript, react, styled-components, props, frontend]
description: "TypeScript 환경에서 styled-components를 사용할 때 props 타입을 지정하고 안전하게 스타일을 제어하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 styled-components에서도 타입이 필요할까요?

React에서 `styled-components`를 사용하면 컴포넌트처럼 스타일을 작성할 수 있습니다. 여기에 TypeScript를 함께 사용하면 styled component에 전달하는 props도 타입으로 안전하게 관리할 수 있어요.

예를 들어 버튼의 색상이나 크기를 props로 바꿔야 한다면, 해당 props가 어떤 값을 받을 수 있는지 타입으로 명확히 정해두는 것이 좋습니다.

이번 글에서는 TypeScript 환경에서 `styled-components` props 타입을 지정하는 방법을 정리해보겠습니다.

---

## 📦 기본 예시

먼저 간단한 버튼 컴포넌트를 만들어보겠습니다.

```tsx
import styled from "styled-components";

const Button = styled.button`
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  background-color: #333;
  color: white;
`;

export default function App() {
  return <Button>버튼</Button>;
}
```

이 상태에서는 고정된 스타일만 사용할 수 있습니다. 이제 props를 받아 스타일을 바꿔보겠습니다.

---

## 🧩 props 타입 지정하기

`styled-components`에서 props 타입을 지정하려면 제네릭 형태로 타입을 넘겨주면 됩니다.

```tsx
interface ButtonProps {
  $primary?: boolean;
}

const Button = styled.button<ButtonProps>`
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  background-color: ${({ $primary }) => ($primary ? "#2563eb" : "#333")};
  color: white;
`;
```

이제 `$primary` 값에 따라 배경색을 다르게 줄 수 있습니다.

```tsx
export default function App() {
  return (
    <div>
      <Button>기본 버튼</Button>
      <Button $primary>주요 버튼</Button>
    </div>
  );
}
```

`ButtonProps`에 `$primary?: boolean`이라고 작성했기 때문에 `$primary`는 boolean 값만 받을 수 있습니다.

---

## 💡 props 이름 앞에 $를 붙이는 이유

예제에서 `primary`가 아니라 `$primary`라고 작성했습니다.

```tsx
interface ButtonProps {
  $primary?: boolean;
}
```

이렇게 `$`를 붙이는 이유는 DOM에 불필요한 속성이 전달되는 것을 막기 위해서입니다.

```tsx
<button primary="true">버튼</button>
```

위처럼 실제 HTML 태그에 알 수 없는 속성이 내려가면 React 경고가 발생할 수 있습니다. `styled-components`에서는 `$primary`처럼 transient props를 사용하면 스타일 계산에는 쓰이지만 DOM에는 전달되지 않습니다.

---

## 🛠️ 여러 props로 스타일 제어하기

버튼 크기까지 props로 제어하고 싶다면 타입을 더 확장할 수 있습니다.

```tsx
type ButtonSize = "small" | "medium" | "large";

interface ButtonProps {
  $primary?: boolean;
  $size?: ButtonSize;
}

const Button = styled.button<ButtonProps>`
  border: none;
  border-radius: 6px;
  color: white;
  background-color: ${({ $primary }) => ($primary ? "#2563eb" : "#333")};

  padding: ${({ $size }) => {
    if ($size === "small") return "4px 8px";
    if ($size === "large") return "12px 16px";
    return "8px 12px";
  }};
`;
```

사용할 때는 정해진 값만 넘길 수 있습니다.

```tsx
<Button $primary $size="large">
  저장하기
</Button>
```

만약 `$size="big"`처럼 타입에 없는 값을 넣으면 TypeScript가 에러를 알려줍니다.

---

## ✅ 정리

TypeScript와 `styled-components`를 함께 쓰면 스타일 props도 안전하게 관리할 수 있습니다.

- `styled.button<Props>`처럼 제네릭으로 props 타입을 지정합니다.
- 스타일 전용 props는 `$primary`처럼 `$`를 붙이면 DOM 전달을 막을 수 있습니다.
- union type을 사용하면 허용할 값의 범위를 제한할 수 있습니다.
- 잘못된 props 값은 TypeScript가 미리 잡아줍니다.

스타일이 props에 따라 바뀌는 컴포넌트라면 타입을 함께 정의해두는 것이 유지보수에 훨씬 좋습니다.
