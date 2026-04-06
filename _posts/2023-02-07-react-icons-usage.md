---
title: "[React] 리액트 아이콘 사용법(react-icons 설치부터 적용까지)"
date: 2023-02-07T17:04:00
categories: [react]
tags: [react, react-icons, frontend, css, npm]
description: "React 프로젝트에서 react-icons 라이브러리를 설치하고 사용하는 방법을 상세히 알아봅니다. 아이콘 검색부터 import 규칙, 스타일링까지 한 번에 정리해 드립니다."
custom_style: true
---

리액트 프로젝트를 진행하다 보면 메뉴바, 버튼 등에 들어갈 깔끔한 아이콘이 필요할 때가 많습니다.  
이미지 파일을 하나하나 구하는 대신, **[React Icons](https://react-icons.github.io/react-icons/)** 라이브러리를 사용하면 수천 개의 고퀄리티 벡터 아이콘을 간편하게 사용할 수 있습니다. 🎨

이번 글에서는 `react-icons`의 설치 방법부터 실제 코드 적용법까지 단계별로 알아보겠습니다.

---

## 1. 라이브러리 설치 (Installation)

먼저 진행 중인 React 프로젝트의 터미널(Terminal)을 열고 아래 명령어를 입력하여 라이브러리를 설치합니다.

```bash
# npm을 사용하는 경우
npm install react-icons --save

# yarn을 사용하는 경우
yarn add react-icons
```

설치가 완료되면 `package.json` 파일에 `react-icons`가 추가된 것을 확인할 수 있습니다.

![설치 완료 화면](https://user-images.githubusercontent.com/88264006/217190949-69a02f1c-1b6f-45cf-8429-8d0fbf7d8ccb.png)

---

## 2. 원하는 아이콘 찾기

설치가 끝났다면 [React Icons 공식 사이트](https://react-icons.github.io/react-icons/)에 접속합니다.  
왼쪽 상단 검색창에 원하는 키워드(영어)를 입력하여 아이콘을 찾을 수 있습니다.

저는 자물쇠 아이콘을 찾기 위해 **"lock"**을 검색해 보겠습니다.  
여러 아이콘 중 `AiFillLock`이라는 아이콘을 사용해 보겠습니다.

![아이콘 검색 화면](https://user-images.githubusercontent.com/88264006/217192731-0d9b5885-8736-4da3-851d-5a6b5a412ce2.png)

---

## 3. 프로젝트에 적용하기 (App.js & App.css)

이제 찾은 아이콘을 코드로 불러와 화면에 띄워보겠습니다.

### 핵심: Import 규칙 🌟

`react-icons`를 사용할 때 가장 중요한 것은 **Import 경로**입니다.

```javascript
import { 아이콘이름 } from "react-icons/아이콘그룹";
```

1.  **아이콘 이름:** 사용할 아이콘의 이름 (예: `AiFillLock`)
2.  **아이콘 그룹:** 아이콘 이름의 **앞 두 글자**를 소문자로 경로 뒤에 붙여줍니다.
    - `AiFillLock` (Ant Design Icons) → `react-icons/ai`
    - `FaBeer` (Font Awesome) → `react-icons/fa`
    - `BsFillAlarmFill` (Bootstrap Icons) → `react-icons/bs`

### 전체 코드 작성

**App.js**

```jsx
import { AiFillLock } from "react-icons/ai"; // Ant Design 아이콘 불러오기
import "./App.css";

function App() {
  return (
    <div className="container">
      {/* 컴포넌트처럼 사용하면 됩니다 */}
      <AiFillLock />
    </div>
  );
}

export default App;
```

**App.css**

아이콘을 화면 중앙에 배치하고 크기를 키우기 위해 스타일을 추가합니다. `react-icons`는 SVG 기반이므로 CSS로 크기나 색상을 자유롭게 조절할 수 있습니다.

```css
body {
  height: 100vh;
  margin: 0;
}

/* 중앙 정렬을 위한 컨테이너 설정 */
.container {
  display: flex;
  justify-content: center;
  align-items: center; /* 수직 중앙 정렬 추가 */
  height: 100vh; /* 전체 화면 높이 사용 */
}

/* 아이콘 스타일링 (svg 태그로 렌더링됩니다) */
svg {
  font-size: 100px; /* 크기 조절 (scale 대신 font-size 권장) */
  color: #333; /* 색상 변경 */
  transform: scale(6); /* 기존 예제 코드 유지 */
}
```

---

## 4. 결과 확인

코드를 저장하고 브라우저를 확인하면, 아래와 같이 자물쇠 아이콘이 화면에 크게 출력되는 것을 볼 수 있습니다!

![아이콘 출력 결과](https://user-images.githubusercontent.com/88264006/217201288-116989e4-84c7-4055-ae65-65bf5e922f44.png)

이제 여러분의 프로젝트에 멋진 아이콘들을 자유롭게 추가해 보세요. 😎

---

## 마치며

오늘은 리액트에서 가장 대중적인 아이콘 라이브러리인 `react-icons` 사용법을 알아보았습니다.

혹시 적용 중에 에러가 나거나 궁금한 점이 있다면 편하게 댓글 달아주세요.  
지적이나 피드백은 언제나 환영입니다! 👋
