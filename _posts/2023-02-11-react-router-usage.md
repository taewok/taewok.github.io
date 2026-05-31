---
title: "[React] react-router-dom 설치부터 기본 사용법까지"
date: 2023-02-11T16:33:00
categories: [react]
tags: [react, react-router, react-router-dom, spa, frontend]
description: "React에서 페이지 이동을 구현하는 react-router-dom의 설치 방법과 BrowserRouter, Routes, Route, Link의 기본 사용법을 정리했습니다."
custom_style: true
---

## 🧐 React에서 페이지 이동은 어떻게 할까요?

일반 HTML에서는 `<a>` 태그를 사용해 다른 페이지로 이동합니다.

하지만 React 앱에서 `<a>` 태그로 페이지를 이동하면 브라우저가 새로고침되면서 앱의 상태가 초기화될 수 있습니다.

React Router를 사용하면 페이지를 새로고침하지 않고 URL에 따라 필요한 컴포넌트만 바꿔 보여줄 수 있습니다. 이런 방식을 SPA, 즉 Single Page Application 방식이라고 부릅니다.

이번 글에서는 `react-router-dom`을 설치하고 기본 라우팅을 설정하는 방법을 정리해보겠습니다.

---

## 📦 react-router-dom 설치하기

React 웹 프로젝트에서는 `react-router-dom` 패키지를 설치합니다.

```bash
npm install react-router-dom
```

`yarn`을 사용한다면 다음 명령어를 사용하면 됩니다.

```bash
yarn add react-router-dom
```

---

## 🧱 기본 라우팅 구조 만들기

먼저 필요한 컴포넌트를 import합니다.

```js
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";
import About from "./About";
import Contents from "./Contents";
import Home from "./Home";
import "./App.css";
```

전체 구조는 다음과 같습니다.

```jsx
function App() {
  return (
    <BrowserRouter>
      <header>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contents">Contents</Link>
      </header>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contents" element={<Contents />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

위 코드에서 `Link`는 이동 버튼 역할을 하고, `Route`는 주소와 컴포넌트를 연결하는 역할을 합니다.

---

## 🧩 핵심 컴포넌트 이해하기

### BrowserRouter

`BrowserRouter`는 라우팅 기능을 사용할 영역을 감싸는 컴포넌트입니다.

보통 앱의 최상단에서 한 번 감싸줍니다.

```jsx
<BrowserRouter>
  <App />
</BrowserRouter>
```

### Routes

`Routes`는 여러 개의 `Route`를 감싸는 컨테이너입니다.

현재 URL과 일치하는 `Route`를 찾아 해당 컴포넌트를 렌더링합니다.

### Route

`Route`는 특정 경로와 컴포넌트를 연결합니다.

```jsx
<Route path="/about" element={<About />} />
```

- `path`: 주소 경로입니다.
- `element`: 해당 주소에서 보여줄 컴포넌트입니다.

### Link

`Link`는 페이지 이동에 사용합니다.

```jsx
<Link to="/about">About</Link>
```

HTML의 `<a>` 태그와 비슷하지만, 브라우저 새로고침 없이 주소만 바꿔줍니다.

---

## 📄 페이지 컴포넌트 만들기

라우팅 테스트를 위해 간단한 페이지 컴포넌트를 만들어보겠습니다.

```jsx
// Home.js
export default function Home() {
  return <div>Home</div>;
}
```

```jsx
// About.js
export default function About() {
  return <div>About</div>;
}
```

```jsx
// Contents.js
export default function Contents() {
  return <div>Contents</div>;
}
```

---

## 🎨 간단한 스타일 추가하기

상단 네비게이션을 보기 좋게 만들기 위해 CSS를 추가해볼 수 있습니다.

```css
header {
  display: flex;
  padding: 20px;
  background-color: #f0f0f0;
}

a {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px;
  width: 100px;
  height: 50px;
  background-color: beige;
  border-radius: 10px;
  text-decoration: none;
  color: black;
  font-weight: bold;
}

a:hover {
  background-color: bisque;
  cursor: pointer;
}
```

---

## ✅ 동작 방식 정리

예를 들어 사용자가 `About` 링크를 클릭하면 다음 일이 일어납니다.

1. 주소가 `/about`으로 바뀝니다.
2. 브라우저는 새로고침하지 않습니다.
3. `Routes`가 현재 주소와 일치하는 `Route`를 찾습니다.
4. `About` 컴포넌트가 화면에 렌더링됩니다.

이것이 React Router를 사용하는 가장 기본적인 흐름입니다.

---

## ✅ 정리

`react-router-dom`은 React 앱에서 페이지 이동을 구현할 때 사용하는 대표적인 라이브러리입니다.

- `BrowserRouter`로 라우팅 영역을 감쌉니다.
- `Routes` 안에 여러 `Route`를 작성합니다.
- `Route`는 `path`와 `element`를 연결합니다.
- `Link`를 사용하면 새로고침 없이 페이지를 이동할 수 있습니다.

React에서 여러 페이지처럼 보이는 화면을 만들고 싶다면 React Router의 기본 구조부터 익혀두면 좋습니다.
