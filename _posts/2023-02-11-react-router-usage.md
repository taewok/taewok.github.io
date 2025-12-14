---
title: "[React] 리액트 라우터(react-router-dom) 설치부터 기본 사용법까지"
date: 2023-02-11T16:33:00
categories: [React]
tags: [react, react-router, react-router-dom, spa, frontend]
description: "React에서 페이지 이동을 구현하는 react-router-dom의 설치 방법과 BrowserRouter, Routes, Route, Link 등 핵심 컴포넌트의 사용법을 예제와 함께 정리합니다."
custom_style: true
---

HTML의 `<a>` 태그를 사용하여 페이지를 이동하면 브라우저가 새로고침되면서 상태가 초기화되는 현상이 발생합니다.

하지만 **React Router**를 사용하면 페이지를 새로고침하지 않고(No Refresh), 주소(URL)에 따라 필요한 데이터만 갈아끼우며 렌더링하는 **SPA(Single Page Application)**를 쉽게 구현할 수 있습니다. 🚀

이번 글에서는 `react-router-dom`의 설치부터 기본 사용법까지 알아보겠습니다.

---

## 1. 사용 준비 (라이브러리 설치)

리액트 라우터를 사용하기 위해 패키지를 설치해야 합니다. 웹 개발 환경에서는 `react-router-dom`을 설치합니다.

```bash
# npm을 사용하는 경우
npm install react-router-dom

# yarn을 사용하는 경우
yarn add react-router-dom
```

---

## 2. 기본 세팅 및 핵심 컴포넌트 (App.js)

설치가 완료되었다면 `App.js`에서 라우팅 설정을 진행합니다.

### App.js 작성

```javascript
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";
import About from "./About";
import Contents from "./Contents";
import Home from "./Home";
import "./App.css";

function App() {
  return (
    <BrowserRouter>
      <header>
        {/* 페이지 이동을 위한 네비게이션 */}
        <Link to="/">Home</Link>
        <Link to="/About">About</Link>
        <Link to="/Contents">Contents</Link>
      </header>

      {/* URL에 따라 렌더링될 컴포넌트 설정 */}
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/About" element={<About />} />
        <Route path="/Contents" element={<Contents />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### 핵심 컴포넌트 설명

1.  **BrowserRouter**: 라우팅을 적용할 모든 컴포넌트(`Routes`, `Link` 등)의 최상위를 감싸주는 래퍼(Wrapper) 컴포넌트입니다. 브라우저의 History API를 사용하여 새로고침 없는 주소 변경을 가능하게 합니다.
2.  **Routes**: 여러 개의 `Route`를 감싸는 컨테이너입니다. URL이 변경되면 하위의 `Route`들을 탐색하여 **규칙이 일치하는 단 하나의 컴포넌트**만 렌더링합니다.
3.  **Route**: 실제 경로와 컴포넌트를 매핑하는 역할을 합니다.
    - **path**: 연결하고 싶은 주소 경로 (예: `"/"`, `"/About"`)
    - **element**: 해당 주소와 일치할 때 보여줄 컴포넌트
4.  **Link**: HTML의 `<a>` 태그와 비슷하지만, 새로고침 없이 주소만 변경해주는 컴포넌트입니다.
    - **to**: 이동할 경로

---

## 3. 페이지 컴포넌트 및 스타일 만들기

라우팅 테스트를 위해 간단한 페이지 컴포넌트 3개와 CSS를 작성해 봅시다.

### 컴포넌트 (Home, About, Contents)

**Home.js**

```javascript
import React from "react";

const Home = () => {
  return <div>Home</div>;
};

export default Home;
```

**About.js**

```javascript
import React from "react";

const About = () => {
  return <div>About</div>;
};

export default About;
```

**Contents.js**

```javascript
import React from "react";

const Contents = () => {
  return <div>Contents</div>;
};

export default Contents;
```

### 스타일링 (App.css)

네비게이션 버튼을 보기 좋게 꾸며줍니다.

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

## 4. 결과 확인 (SPA 동작 방식)

코드를 모두 작성하고 실행하면 아래와 같은 화면이 나타납니다.

![react-router 초기 실행 화면](https://user-images.githubusercontent.com/88264006/218254906-29bf16c0-db1d-46d1-862a-b225959aa3cb.png)

이제 상단의 **About** 버튼을 눌러보겠습니다.

![페이지 이동 후 화면](https://user-images.githubusercontent.com/88264006/218255065-130c0c02-808d-43f5-96e2-99f95a39f95d.png)

1.  주소창의 URL이 `/About`으로 변경되었습니다.
2.  하지만 브라우저 **새로고침(깜빡임)은 발생하지 않았습니다.**
3.  `Routes` 설정에 따라 하단 영역에 `About` 컴포넌트가 정상적으로 렌더링 되었습니다.

이것이 바로 React Router를 사용하는 이유이자 SPA의 핵심 기능입니다! 🎉

---

## 마치며

오늘은 리액트 프로젝트의 필수 라이브러리인 `react-router-dom`의 기초적인 사용법을 알아보았습니다.

SPA 구조를 이해하는 첫걸음이니 꼭 직접 실습해 보시길 바랍니다.
혹시 궁금한 점이나 잘못된 내용이 있다면 편하게 댓글 남겨주세요! 👋
