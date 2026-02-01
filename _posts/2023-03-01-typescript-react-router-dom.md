---
title: "[TypeScript] 리액트 라우터(react-router-dom) 설치 및 타입 활용 가이드"
date: 2023-03-01T16:01:00
categories: [TypeScript, React]
tags: [typescript, react, react-router-dom, frontend]
description: "TypeScript 프로젝트에서 react-router-dom을 설치하는 방법(@types)과 useParams 등의 Hook을 타입 안전하게 사용하는 방법을 정리합니다."
custom_style: true
---

TypeScript 환경에서 React 프로젝트를 진행할 때, 가장 먼저 마주치는 난관 중 하나가 외부 라이브러리의 **타입 정의(Type Definition)** 문제입니다.

대표적인 라이브러리인 `react-router-dom` 또한 자바스크립트 기반으로 만들어졌기 때문에, 타입스크립트가 이해할 수 있도록 별도의 타입 패키지를 설치해줘야 합니다.

이번 글에서는 설치 방법과 TypeScript에서의 기본적인 활용법을 알아보겠습니다.

---

## 1. 라이브러리 설치

상황에 따라 필요한 명령어가 다릅니다.

### 1) 프로젝트에 react-router-dom이 없는 경우 (처음 설치)

라이브러리 본체와 타입 정의 패키지(`@types`)를 한 번에 설치합니다.

```bash
npm install react-router-dom @types/react-router-dom
# 또는 yarn add react-router-dom @types/react-router-dom
```

### 2) 이미 react-router-dom이 설치된 경우

타입스크립트로 마이그레이션 중이거나, 설치를 깜빡했다면 타입 패키지만 추가로 설치합니다.
(`@types` 패키지는 개발 단계에서만 필요하므로 `-D` 옵션을 권장합니다.)

```bash
npm install -D @types/react-router-dom
# 또는 yarn add -D @types/react-router-dom
```

**왜 @types가 필요한가요?**
`react-router-dom` 자체에는 타입 정보가 포함되어 있지 않습니다. `@types/react-router-dom`을 설치해야 VS Code 같은 에디터에서 자동 완성 기능을 지원받고, 컴파일 에러를 방지할 수 있습니다.

---

## 2. TypeScript에서 활용하기

설치가 끝났다면 기본적인 라우팅 설정은 자바스크립트와 동일합니다. 하지만 **Hooks**를 사용할 때 제네릭(Generic)을 활용하면 더욱 강력한 타입 안정성을 누릴 수 있습니다.

### 1) 기본 라우팅 설정 (App.tsx)

```tsx
import React from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import Home from "./pages/Home";
import Detail from "./pages/Detail";

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        {/* URL 파라미터를 받는 동적 라우팅 */}
        <Route path="/detail/:id" element={<Detail />} />
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

### 2) useParams 타입 지정하기 (핵심 ⭐)

URL 파라미터(예: `/detail/:id`)를 받아오는 `useParams`를 사용할 때, TypeScript에서는 어떤 파라미터가 들어올지 미리 알려주는 것이 좋습니다.

```tsx
import React from "react";
import { useParams } from "react-router-dom";

// 1. 파라미터의 타입을 인터페이스로 정의
interface RouteParams {
  id: string; // URL 파라미터는 기본적으로 모두 string입니다.
}

const Detail = () => {
  // 2. 제네릭을 사용하여 타입 주입
  // 이제 id는 string 혹은 undefined가 아닌, 명확한 string 타입으로 인식됩니다.
  const { id } = useParams<keyof RouteParams>() as RouteParams;

  return (
    <div>
      <h2>상세 페이지</h2>
      <p>현재 아이디: {id}</p>
    </div>
  );
};

export default Detail;
```

---

## 마치며

TypeScript를 사용하면 처음에는 설치할 것도 많고 설정할 것도 많아 번거롭게 느껴질 수 있습니다.
하지만 `useParams`나 `useLocation` 등을 사용할 때 자동 완성이 지원되고, 잘못된 코드를 미리 잡아주는 장점이 훨씬 큽니다.

혹시 궁금한 점이나 피드백이 있다면 편하게 댓글 달아주세요. 👋
