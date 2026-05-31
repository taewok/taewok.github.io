---
title: "[React] lazy와 Suspense로 코드 스플리팅하기"
date: 2023-07-29T18:39:00
categories: [react]
tags: [react, lazy, suspense, code-splitting]
description: "React lazy와 Suspense를 사용해 필요한 컴포넌트를 늦게 불러오고 초기 번들 크기를 줄이는 방법을 정리했습니다."
custom_style: true
---

## React lazy란?

`React.lazy`는 컴포넌트를 필요한 시점에 동적으로 불러오게 해주는 기능입니다.

앱이 커지면 처음 로딩할 때 다운로드해야 하는 JavaScript 번들 크기도 커집니다.

모든 페이지 코드를 처음부터 한 번에 불러오기보다, 사용자가 해당 페이지에 접근했을 때 필요한 코드만 불러오면 초기 로딩을 줄일 수 있습니다.

이 방식을 코드 스플리팅이라고 부릅니다.

---

## 기본 사용법

```tsx
import { lazy, Suspense } from "react";

const MyPage = lazy(() => import("./MyPage"));

const App = () => {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <MyPage />
    </Suspense>
  );
};

export default App;
```

`lazy`는 동적 import를 사용해 컴포넌트를 불러옵니다.

`Suspense`는 컴포넌트가 로딩되는 동안 보여줄 fallback UI를 담당합니다.

---

## 라우트 단위로 나누기

실무에서는 페이지 단위로 lazy를 적용하는 경우가 많습니다.

```tsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";

const HomePage = lazy(() => import("./pages/HomePage"));
const MyPage = lazy(() => import("./pages/MyPage"));

const App = () => {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/mypage" element={<MyPage />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
};

export default App;
```

이렇게 하면 사용자가 `/mypage`에 접근할 때 `MyPage` 관련 코드가 로드됩니다.

---

## fallback UI

`fallback`에는 로딩 중 보여줄 UI를 넣습니다.

```tsx
<Suspense fallback={<PageSpinner />}>
  <MyPage />
</Suspense>
```

단순한 텍스트보다 스피너나 skeleton UI를 넣으면 사용자 입장에서 더 자연스럽게 느껴질 수 있습니다.

---

## lazy를 사용할 때 주의할 점

`React.lazy`는 default export를 기대합니다.

```tsx
export default MyPage;
```

만약 named export만 있는 파일이라면 바로 lazy로 가져오기 어렵습니다.

그럴 때는 중간에서 default 형태로 맞춰줘야 합니다.

```tsx
const MyPage = lazy(() =>
  import("./MyPage").then((module) => ({
    default: module.MyPage,
  })),
);
```

---

## 언제 사용하면 좋을까요?

lazy는 모든 컴포넌트에 무조건 적용하는 기능은 아닙니다.

자주 쓰는 작은 컴포넌트까지 나누면 오히려 네트워크 요청이 많아지고 관리가 복잡해질 수 있습니다.

보통은 아래처럼 크고 독립적인 단위에 적용합니다.

- 페이지 컴포넌트
- 관리자 화면
- 모달 안의 무거운 기능
- 차트, 에디터처럼 용량이 큰 라이브러리를 사용하는 영역

---

## 마무리

`React.lazy`와 `Suspense`를 사용하면 필요한 컴포넌트를 필요한 시점에 불러올 수 있습니다.

초기 번들 크기를 줄이고 싶다면 페이지 단위 코드 스플리팅부터 적용해보는 것이 좋습니다.

다만 너무 작은 단위까지 나누기보다, 실제로 로딩 비용이 큰 화면이나 기능을 중심으로 적용하는 편이 효과적입니다.
