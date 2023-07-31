---
title: "[React] lazy란?"
date: 2023-07-29T18:39:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b>lazy란?</b>

<h3><blockquote>정의
</blockquote></h3>

React에서 lazy란 코드 스플리팅(Code Splitting)을 지원하기 위한 기능 중 하나이며, 코드 스플리팅은 애플리케이션의 번들 크기를 줄이고 초기 로딩 속도를 개선하는 데 사용되는 기술입니다.

<h3><blockquote>장점
</blockquote></h3>

- 초기 로딩 속도 개선: 사용자가 애플리케이션을 처음 방문할 때 필요한 컴포넌트만 로드되므로 초기 로딩 속도가 개선됩니다.

- 작은 번들 크기: 코드 스플리팅을 통해 애플리케이션 번들의 크기가 줄어듭니다. 작은 번들은 사용자가 애플리케이션을 다운로드하는 데 걸리는 시간을 단축시키며, 데이터 사용량을 줄여줍니다.

- 효율적인 자원 사용: lazy를 사용하면 사용자가 실제로 필요로 하는 컴포넌트만 로드되므로, 불필요한 자원 로딩을 방지할 수 있습니다. 이로 인해 메모리 사용과 네트워크 요청이 최적화됩니다.

- 유연한 로딩 전략: React.lazy와 React.Suspense를 활용하면 로딩 화면을 커스텀하게 처리할 수 있습니다. 로딩 중에 표시할 UI나 메시지를 자유롭게 설정하여 사용자 경험을 개선할 수 있습니다.

---

## <b>lazy 사용</b>

Suspense를 감싸는 절차가 꼭 필요하진 않지만 동적으로 불러온 컴포넌트의 로딩 상태를 처리할 수 있는 직접적인 방법이 없으므로 강력하게 추천된다.

```js
import { lazy, Suspense } from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";

// 1. 코드 스플리팅할 컴포넌트 생성
const MyComponent = lazy(() => import("./MyComponent"));

function MyAppComponent() {
  return (
    <BrowserRouter>
      {/* 2. Suspense 컴포넌트로 감싸기 */}
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          {/* 3. lazy 함수로 동적 로드된 컴포넌트 사용 */}
          <Route path={"/mypage"} element={<MyComponent />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

- <b>fallback</b>: <strong>lazy()</strong>로 인해 코드 스플리팅된 컴포넌트가 로드되기를 기다리는 동안 보여줄 컨텐츠를 지정하는데 사용됩니다.

## <b>마치며</b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>