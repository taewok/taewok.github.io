---
title: "[Error] TypeScript에서 is not a function 해결하기"
date: 2023-06-24T18:13:00
categories: [error]
tags: [error, typescript, react, props]
description: "React TypeScript에서 props를 잘못 전달해 setState가 함수로 인식되지 않는 문제를 정리했습니다."
custom_style: true
---

## 발생한 에러

부모 컴포넌트에서 만든 state와 setState 함수를 자식 컴포넌트로 넘기는 과정에서 다음과 같은 에러가 발생했습니다.

```txt
setCount is not a function
```

값은 잘 넘어오는 것처럼 보이는데, `setCount()`를 호출하는 순간 함수가 아니라는 에러가 난 상황입니다.

---

## 문제가 된 코드

처음에는 자식 컴포넌트를 이렇게 작성했습니다.

```tsx
const StarClick = (count: any, setCount: any) => {
  const starOnChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const num = Number(event.target.value);
    setCount(num);
  };

  return (
    <div>
      <input value={count} onChange={starOnChange} />
    </div>
  );
};

export default StarClick;
```

겉으로 보면 `count`와 `setCount`를 인자로 받는 것처럼 보입니다.

하지만 React 컴포넌트는 첫 번째 인자로 props 객체 하나를 받습니다.

즉, 위 코드에서 `count`는 실제 count 값이 아니라 props 객체 전체가 됩니다.

그리고 두 번째 인자인 `setCount`는 React가 넘겨주는 값이 아니기 때문에 `undefined`가 됩니다.

그래서 `setCount()`를 호출하면 `is not a function` 에러가 발생합니다.

---

## 해결 방법

props 객체에서 필요한 값을 구조 분해해서 꺼내야 합니다.

```tsx
interface StarClickProps {
  count: number;
  setCount: React.Dispatch<React.SetStateAction<number>>;
}

const StarClick = ({ count, setCount }: StarClickProps) => {
  const starOnChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const num = Number(event.target.value);
    setCount(num);
  };

  return (
    <div>
      <input value={count} onChange={starOnChange} />
    </div>
  );
};

export default StarClick;
```

이제 `count`와 `setCount`가 props 객체에서 정상적으로 꺼내집니다.

---

## 부모 컴포넌트 예시

부모에서는 이렇게 값을 전달할 수 있습니다.

```tsx
import { useState } from "react";
import StarClick from "./StarClick";

const Parent = () => {
  const [count, setCount] = useState(0);

  return <StarClick count={count} setCount={setCount} />;
};

export default Parent;
```

자식 컴포넌트는 props 객체 하나를 받고, 그 안에서 `count`와 `setCount`를 꺼내 사용합니다.

---

## setState 타입 이해하기

`setCount`의 타입은 아래처럼 작성할 수 있습니다.

```tsx
React.Dispatch<React.SetStateAction<number>>
```

처음 보면 길어 보이지만, 의미는 간단합니다.

`number` 상태를 변경할 수 있는 React setState 함수라는 뜻입니다.

상태가 문자열이라면 이렇게 바뀝니다.

```tsx
React.Dispatch<React.SetStateAction<string>>
```

---

## 마무리

React 컴포넌트는 여러 인자를 따로 받는 함수처럼 동작하지 않습니다.

항상 첫 번째 인자로 props 객체 하나를 받습니다.

그래서 자식 컴포넌트에서 값을 사용할 때는 `({ count, setCount })`처럼 구조 분해하거나, `props.count`, `props.setCount`처럼 접근해야 합니다.

TypeScript에서는 props 타입을 명확히 정의해두면 이런 실수를 더 빨리 찾을 수 있습니다.
