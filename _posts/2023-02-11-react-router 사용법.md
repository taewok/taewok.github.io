---
title: "[React] react-router 사용하기"
date: 2023-02-11T16:33:000
categories: [react,react-router]
tags: [react,react-router] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">react-router란?</b>
<p>a태그를 사용해 페이지 이동을 한다면 새로고침과 같은 현상이 일어난다. 하지만 <strong style="color:#ff526f">react-router</strong>는 신규 페이지를 렌더링 하지 않고 url 주소에 따라 선택된 데이터를 하나의 페이지에서 렌더링 해주는 라이브러리 라고 볼 수 있다.</p>

***

## <b style="border-bottom:2px solid gray">react-router 사용준비</b> 
### <b>1. 라이브러리 설치</b>
```
npm install react-router-dom //npm 사용시
yarn install react-router-dom //yarn 사용시
```
### <b>2. 기본 셋팅</b>
#### App.js
```js
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";
import About from "./About";
import Contents from "./Contents";
import Home from "./Home";
import "./App.css";

function App() {
  return (
    <BrowserRouter>
      <header>
        <Link to="/">Home</Link>
        <Link to="/About">About</Link>
        <Link to="/Contents">Contents</Link>
      </header>
      <Routes>
        <Route path="/" element={<Home></Home>}></Route>
        <Route path="/About" element={<About></About>}></Route>
        <Route path="/Contents" element={<Contents></Contents>}></Route>
      </Routes>
    </BrowserRouter>
  );
}
export default App;
```
+ <strong>BrowserRouter:</strong> 모든 <strong style="color:#ff526f">Routes,Route</strong>의 최상위에서 감싸줘야 한다. SPA의 장점인 브라우저가 깜빡이지 않고 다른 페이지로 이동할 수 있게 만들어준다. 
+ <strong>Routes:</strong> 여러 <strong style="color:#ff526f">Route</strong>를 감싸서 그 중 규칙이 일치하는 <strong style="color:#ff526f">Route</strong> 단 하나만을 렌더링 시켜주는 역할을 한다. 
+ <strong>Route</strong> 
    - <b>path:</b> path 속성에는 원하는 경로를 입력 
        - ex) "/" => "http://localhost:3000/"
        - ex) "/About" => "http://localhost:3000/About"
    - <b>element:</b> element속성에는 컴포넌트를 넣어 준다 현재 주소창과 path에 경로가 일치하면 불러온다.
+ <strong>Link:</strong> 주소의 경로만 바꾸는 기능 웹에서는 a태그로 나온다.
    - <b>to:</b> 바꿀 주소 경로 입력 

*** 

## <b style="border-bottom:2px solid gray">react-router로 페이지 이동하기</b>

#### Home.js
```javascript
import React from 'react';
const Home = () => {
    return (
        <div>
            Home
        </div>
    );
};
export default Home;
```
#### About.js
```javascript
import React from 'react';
const About = () => {
    return (
        <div>
            About
        </div>
    );
};
export default About;
```
#### Contents.js
```javascript
import React from 'react';
const Contents = () => {
    return (
        <div>
            Contents
        </div>
    );
};
export default Contents;
```
#### App.css
```css
header{
    display: flex;
}
a{
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 10px;
    width: 100px;
    height:50px;
    background-color: beige;
    border-radius: 10px;
    text-decoration: none;
}
a:hover{
    background-color: bisque;
}
```
<p>위 파일에 코드들까지 추가해주면 아래와 같은 화면이 나온다.</p>
<img src="https://user-images.githubusercontent.com/88264006/218254906-29bf16c0-db1d-46d1-862a-b225959aa3cb.png" alt="react-router 사용 첫 화면"/>
<p>상단에 About버튼을 눌러보자</p>
<img src="https://user-images.githubusercontent.com/88264006/218255065-130c0c02-808d-43f5-96e2-99f95a39f95d.png"/>
<p>상단에 About버튼을 누르자 주소창에 주소가 바뀌었지만 새로고침은 일어나지 않았다.</p> 
<p>그리고 주소창에 경로와 일치하는 About 컴포넌트가 아래 불러와졌다.</p>

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>