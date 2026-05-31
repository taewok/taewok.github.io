---
title: "[TypeScript] react-router-dom 타입과 useParams 사용법"
date: 2023-03-01T16:01:00
categories: [typescript, react]
tags: [typescript, react, react-router-dom, frontend]
description: "TypeScript 프로젝트에서 react-router-dom을 사용할 때 라우팅을 설정하고 useParams 값을 타입 안전하게 다루는 방법을 정리했습니다."
custom_style: true
---

## 🧐 TypeScript에서 React Router를 사용할 때 무엇이 다를까요?

React에서 페이지 이동을 구현할 때 `react-router-dom`을 자주 사용합니다. JavaScript 프로젝트에서는 바로 사용해도 큰 문제가 없지만, TypeScript에서는 URL 파라미터 같은 값의 타입을 함께 신경 써야 합니다.

예를 들어 `/detail/:id` 경로에서 `id` 값을 가져올 때, 이 값이 문자열인지 undefined일 수 있는지 TypeScript가 확인합니다.

이번 글에서는 TypeScript 환경에서 `react-router-dom`을 설치하고 `useParams`를 타입 안전하게 사용하는 방법을 정리해보겠습니다.

---

## 📦 설치하기

먼저 `react-router-dom`을 설치합니다.

```bash
npm install react-router-dom
```

이전 버전에서는 `@types/react-router-dom`을 별도로 설치해야 하는 경우가 있었지만, 최근 버전의 `react-router-dom`은 타입이 함께 포함되어 있습니다.

만약 오래된 버전을 사용하고 있고 타입 에러가 난다면 다음 패키지가 필요할 수 있습니다.

```bash
npm install -D @types/react-router-dom
```

사용 중인 버전에 따라 필요한 설치가 달라질 수 있으니, 타입이 잡히지 않을 때만 `@types` 패키지를 확인하면 됩니다.

---

## 🧱 기본 라우팅 설정

기본 라우팅 구조는 JavaScript에서 사용할 때와 거의 같습니다.

```tsx
import { BrowserRouter, Route, Routes } from "react-router-dom";
import Detail from "./pages/Detail";
import Home from "./pages/Home";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/detail/:id" element={<Detail />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

여기서 `/detail/:id`는 동적 라우팅입니다. 예를 들어 `/detail/10`으로 접속하면 `id` 값은 `"10"`이 됩니다.

---

## 🔍 useParams로 URL 파라미터 가져오기

`Detail` 컴포넌트에서는 `useParams`로 URL 파라미터를 가져올 수 있습니다.

```tsx
import { useParams } from "react-router-dom";

export default function Detail() {
  const params = useParams();

  return <div>현재 아이디: {params.id}</div>;
}
```

하지만 TypeScript에서는 `params.id`가 항상 있다고 확신할 수 없습니다. 그래서 보통 `string | undefined`로 다루게 됩니다.

---

## 🧩 useParams 타입 지정하기

파라미터 이름을 타입으로 지정하면 더 명확하게 사용할 수 있습니다.

```tsx
import { useParams } from "react-router-dom";

type RouteParams = {
  id: string;
};

export default function Detail() {
  const { id } = useParams<RouteParams>();

  return <div>현재 아이디: {id}</div>;
}
```

다만 실제로는 URL 구조가 바뀌거나 잘못 접근하는 경우도 있을 수 있으므로, `id`가 없는 상황을 처리해두는 편이 안전합니다.

```tsx
export default function Detail() {
  const { id } = useParams<RouteParams>();

  if (!id) {
    return <div>잘못된 접근입니다.</div>;
  }

  return <div>현재 아이디: {id}</div>;
}
```

---

## ✅ 정리

TypeScript에서 `react-router-dom`을 사용할 때도 기본 라우팅 방식은 JavaScript와 거의 같습니다.

- 최신 `react-router-dom`은 보통 타입이 포함되어 있습니다.
- 오래된 버전에서는 `@types/react-router-dom`이 필요할 수 있습니다.
- `useParams`로 URL 파라미터를 가져올 수 있습니다.
- 파라미터 이름을 타입으로 지정하면 더 명확하게 사용할 수 있습니다.
- URL 파라미터는 없을 수도 있으므로 방어 코드도 함께 고려하는 것이 좋습니다.

라우팅 자체보다 URL에서 가져온 값을 타입 안전하게 다루는 부분이 TypeScript에서 특히 중요합니다.
