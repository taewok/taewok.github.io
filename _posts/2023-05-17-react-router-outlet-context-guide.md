---
title: "[React] Outlet으로 자식 라우트에 데이터 전달하기"
date: 2023-05-17T20:27:00
categories: [react]
tags: [react, react-router-dom, outlet, useoutletcontext, props-drilling]
description: "React Router의 Outlet context와 useOutletContext를 사용해 중첩 라우트 사이에서 데이터를 전달하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

React Router로 중첩 라우트를 만들다 보면 부모 라우트에서 가진 값을 자식 라우트 컴포넌트가 사용해야 할 때가 있습니다.

일반 컴포넌트라면 props로 넘기면 되지만, 라우트에서는 자식 컴포넌트가 `<Outlet />`을 통해 렌더링됩니다.

그래서 이런 고민이 생깁니다.

```tsx
<Outlet />
```

여기에 props를 어떻게 전달해야 할까요?

이럴 때 React Router의 `Outlet context`와 `useOutletContext`를 사용할 수 있습니다.

---

## Outlet context란?

`Outlet`은 중첩 라우트의 자식 컴포넌트가 렌더링될 위치를 표시하는 컴포넌트입니다.

그리고 `Outlet`에는 `context`라는 prop을 전달할 수 있습니다.

```tsx
<Outlet context={{ userId, setUserId }} />
```

자식 라우트에서는 `useOutletContext`로 이 값을 꺼내 사용할 수 있습니다.

---

## 부모 라우트에서 값 전달하기

예를 들어 회원가입 단계에서 부모 라우트가 공통 상태를 가지고 있다고 해볼게요.

```tsx
import { useState } from "react";
import { Outlet } from "react-router-dom";

const RegisterLayout = () => {
  const [email, setEmail] = useState("");

  return (
    <div>
      <h1>회원가입</h1>
      <Outlet context={{ email, setEmail }} />
    </div>
  );
};

export default RegisterLayout;
```

`Outlet`의 `context`에 객체 형태로 값을 넘겼습니다.

이 값은 `RegisterLayout` 아래에 렌더링되는 자식 라우트에서 사용할 수 있습니다.

---

## 자식 라우트에서 값 꺼내기

자식 컴포넌트에서는 `useOutletContext`를 사용합니다.

```tsx
import { useOutletContext } from "react-router-dom";

const EmailStep = () => {
  const { email, setEmail } = useOutletContext<{
    email: string;
    setEmail: React.Dispatch<React.SetStateAction<string>>;
  }>();

  return (
    <input
      value={email}
      onChange={(event) => setEmail(event.target.value)}
      placeholder="이메일을 입력해주세요"
    />
  );
};

export default EmailStep;
```

이제 자식 라우트에서도 부모가 가진 `email` 상태를 읽고 수정할 수 있습니다.

---

## 타입을 분리하면 더 깔끔해요

`useOutletContext` 안에 타입을 직접 작성해도 되지만, 재사용하려면 타입을 분리하는 편이 좋습니다.

```tsx
import type { Dispatch, SetStateAction } from "react";

export interface RegisterOutletContext {
  email: string;
  setEmail: Dispatch<SetStateAction<string>>;
}
```

그리고 자식 라우트에서 이렇게 사용할 수 있습니다.

```tsx
const { email, setEmail } = useOutletContext<RegisterOutletContext>();
```

타입을 분리해두면 여러 단계의 자식 라우트에서 같은 context 타입을 안정적으로 공유할 수 있습니다.

---

## 언제 사용하면 좋을까요?

`useOutletContext`는 전역 상태까지는 필요 없지만, 중첩 라우트 안에서만 공유하면 되는 값에 잘 맞습니다.

예를 들면 이런 상황입니다.

- 회원가입 단계별 입력값
- 마이페이지 하위 탭에서 공유하는 사용자 정보
- 대시보드 레이아웃에서 공유하는 필터 값
- 부모 라우트에서 가져온 공통 API 데이터

앱 전체에서 필요한 값이라면 Context API, Zustand, Redux 같은 전역 상태가 더 적합할 수 있습니다.

---

## 주의할 점

`useOutletContext`는 반드시 해당 `Outlet` 아래에서 렌더링되는 자식 라우트에서 사용해야 합니다.

라우트 구조 밖의 일반 컴포넌트에서 호출하면 기대한 값을 받을 수 없습니다.

또한 context 값이 커지고 역할이 많아지면 라우트 구조에 너무 강하게 묶일 수 있으니, 공유 범위가 넓어지는 순간에는 상태 관리 방식을 다시 고민하는 것이 좋습니다.

---

## 마무리

`Outlet context`는 중첩 라우트에서 부모와 자식 사이의 데이터를 간단하게 연결해주는 기능입니다.

props drilling 없이 라우트 단위의 로컬 상태를 공유할 수 있고, TypeScript와 함께 쓰면 타입 안정성도 챙길 수 있습니다.

전역 상태로 올리기에는 작고, props로 넘기기에는 라우트 구조상 애매한 값이라면 `useOutletContext`를 먼저 떠올려보면 좋습니다.
