---
title: "[React] react-icons 설치부터 사용법까지 정리"
date: 2023-02-07T17:04:00
categories: [react]
tags: [react, react-icons, frontend, css, npm]
description: "React 프로젝트에서 react-icons 라이브러리를 설치하고, 원하는 아이콘을 찾아 import한 뒤 스타일링하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 React에서 아이콘은 어떻게 사용할까요?

React 프로젝트를 만들다 보면 버튼, 메뉴, 입력창 등에 아이콘이 필요한 경우가 많습니다.

아이콘 이미지를 직접 다운로드해서 관리할 수도 있지만, 매번 파일을 찾고 크기를 맞추는 일이 번거로울 수 있습니다.

이럴 때 편하게 사용할 수 있는 라이브러리가 `react-icons`입니다.

`react-icons`를 사용하면 Font Awesome, Bootstrap Icons, Ant Design Icons 같은 여러 아이콘 세트를 React 컴포넌트처럼 사용할 수 있습니다.

---

## 📦 react-icons 설치하기

먼저 React 프로젝트에서 터미널을 열고 아래 명령어를 실행합니다.

```bash
npm install react-icons
```

`yarn`을 사용한다면 다음 명령어를 사용하면 됩니다.

```bash
yarn add react-icons
```

설치가 끝나면 `package.json`의 dependencies에 `react-icons`가 추가됩니다.

---

## 🔍 원하는 아이콘 찾기

설치가 끝났다면 [React Icons 공식 사이트](https://react-icons.github.io/react-icons/)에 접속합니다.

검색창에 원하는 키워드를 영어로 입력하면 관련 아이콘을 찾을 수 있습니다.

예를 들어 자물쇠 아이콘을 찾고 싶다면 `lock`을 검색하면 됩니다.

검색 결과에서 `AiFillLock` 같은 아이콘 이름을 확인할 수 있습니다.

여기서 아이콘 이름의 앞부분이 어떤 아이콘 그룹에서 가져와야 하는지 알려줍니다.

```txt
AiFillLock → ai
FaBeer → fa
BsFillAlarmFill → bs
```

---

## 🧩 import 규칙 이해하기

`react-icons`를 사용할 때 가장 중요한 부분은 import 경로입니다.

```js
import { 아이콘이름 } from "react-icons/아이콘그룹";
```

예를 들어 `AiFillLock`은 Ant Design Icons에 속한 아이콘이고, 경로는 `react-icons/ai`입니다.

```js
import { AiFillLock } from "react-icons/ai";
```

다른 예시도 보면 규칙이 조금 더 잘 보입니다.

```js
import { FaBeer } from "react-icons/fa";
import { BsFillAlarmFill } from "react-icons/bs";
```

아이콘 이름의 앞 두 글자를 보고 import 경로를 정한다고 생각하면 이해하기 쉽습니다.

---

## 🛠️ 화면에 아이콘 렌더링하기

가져온 아이콘은 React 컴포넌트처럼 사용할 수 있습니다.

```jsx
import { AiFillLock } from "react-icons/ai";
import "./App.css";

function App() {
  return (
    <div className="container">
      <AiFillLock />
    </div>
  );
}

export default App;
```

`AiFillLock`을 JSX 안에 작성하면 SVG 아이콘이 화면에 렌더링됩니다.

---

## 🎨 CSS로 크기와 색상 바꾸기

`react-icons`는 SVG 기반이기 때문에 CSS로 크기와 색상을 쉽게 바꿀 수 있습니다.

```css
body {
  height: 100vh;
  margin: 0;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container svg {
  font-size: 100px;
  color: #333;
}
```

또는 아이콘 컴포넌트에 직접 `size`, `color` props를 넘길 수도 있습니다.

```jsx
<AiFillLock size={100} color="#333" />
```

간단한 경우에는 props로 지정해도 좋고, 여러 아이콘을 같은 스타일로 관리해야 한다면 CSS 클래스를 사용하는 편이 좋습니다.

---

## 🧱 버튼 안에서 사용하기

아이콘은 버튼과 함께 사용할 때도 많습니다.

```jsx
import { AiFillLock } from "react-icons/ai";

function LoginButton() {
  return (
    <button className="login-button">
      <AiFillLock size={18} />
      로그인
    </button>
  );
}
```

```css
.login-button {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  background: white;
  cursor: pointer;
}
```

`inline-flex`와 `gap`을 함께 사용하면 아이콘과 텍스트 간격을 쉽게 맞출 수 있습니다.

---

## ✅ 정리

`react-icons`는 React 프로젝트에서 아이콘을 간편하게 사용할 수 있게 해주는 라이브러리입니다.

- `npm install react-icons`로 설치합니다.
- 공식 사이트에서 원하는 아이콘을 검색합니다.
- 아이콘 이름에 맞는 그룹 경로에서 import합니다.
- 아이콘은 React 컴포넌트처럼 사용할 수 있습니다.
- CSS 또는 `size`, `color` props로 스타일링할 수 있습니다.

이미지 파일을 직접 관리하지 않고도 다양한 아이콘을 빠르게 사용할 수 있어서, 작은 프로젝트부터 실무 프로젝트까지 두루 활용하기 좋습니다.
