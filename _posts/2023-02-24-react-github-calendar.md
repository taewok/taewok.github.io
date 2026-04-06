---
title: "[React] 내 사이트에 깃허브 잔디 심기: react-github-calendar 사용법"
date: 2023-02-24T18:34:00
categories: [react]
tags: [react, react-github-calendar, github, portfolio, frontend]
description: "React 프로젝트에서 백엔드 도움 없이 프론트엔드 라이브러리(react-github-calendar)만으로 깃허브 커밋 기록(잔디)을 가져오는 방법과 다양한 커스텀 옵션을 알아봅니다."
custom_style: true
---

개발자 포트폴리오를 만들 때 빠질 수 없는 것이 바로 **'깃허브 잔디(Contribution Graph)'**입니다. 🌱

처음에는 백엔드 API를 통해 User 정보를 받아와야 하나 고민했지만, 찾아보니 프론트엔드에서 **`react-github-calendar`** 라이브러리만 사용하면 아주 간편하게 잔디를 심을 수 있다는 것을 알게 되었습니다.

이번 글에서는 설치부터 커스텀 옵션까지 빠르게 정리해 보겠습니다.

---

## 1. 라이브러리 설치

먼저 프로젝트 터미널에서 패키지를 설치합니다.

```bash
# npm 사용 시
npm install react-github-calendar

# yarn 사용 시
yarn add react-github-calendar
```

---

## 2. 기본 사용법 (Basic Usage)

사용법은 매우 간단합니다. 컴포넌트를 import 하고 `username` 속성에 본인의 **깃허브 아이디**만 넣어주면 끝입니다.

```jsx
import React from "react";
import GitHubCalendar from "react-github-calendar";

const App = () => {
  return (
    <div>
      {/* username에 본인의 깃허브 아이디를 입력하세요 */}
      <GitHubCalendar username="유저이름" />
    </div>
  );
};

export default App;
```

### 실행 결과

코드를 실행하면 아래와 같이 내 깃허브의 잔디가 예쁘게 불러와진 것을 확인할 수 있습니다.

![react-github-calendar 적용 예시](https://user-images.githubusercontent.com/88264006/221151045-f4bd2e6c-e759-4fd3-be4e-467d33b4db10.png)

---

## 3. 다양한 옵션 (Props)

`GitHubCalendar` 컴포넌트는 다양한 Props를 통해 스타일을 커스텀할 수 있습니다.

| 속성 (Prop)     | 설명                                                | 예시 코드                            |
| :-------------- | :-------------------------------------------------- | :----------------------------------- |
| **year**        | 특정 연도의 데이터를 가져옵니다. (기본값: 작년)     | `<GitHubCalendar year="2023" />`     |
| **color**       | 잔디의 색상(테마)을 설정합니다. (가장 진한 색 기준) | `<GitHubCalendar color="#ffffff" />` |
| **fontSize**    | 글자 크기를 조절합니다.                             | `<GitHubCalendar fontSize={16} />`   |
| **blockSize**   | 잔디 블록(네모) 하나의 크기를 조절합니다.           | `<GitHubCalendar blockSize={15} />`  |
| **blockMargin** | 잔디 블록 사이의 간격을 조절합니다.                 | `<GitHubCalendar blockMargin={5} />` |
| **blockRadius** | 잔디 블록의 둥근 정도(Border Radius)를 설정합니다.  | `<GitHubCalendar blockRadius={2} />` |

> **Tip:** 최근 버전에서는 `color` 대신 `theme` 속성을 사용하거나 색상 팔레트를 직접 주입해야 하는 경우도 있으니, 적용이 안 된다면 [공식 문서](https://www.npmjs.com/package/react-github-calendar)를 확인해 보세요.

---

## 마치며

백엔드 연동 없이 라이브러리 하나로 포트폴리오의 퀄리티를 높일 수 있어 매우 유용했습니다.  
여러분도 자신의 사이트에 멋진 잔디를 심어보세요! 🌿

혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.  
지적이나 피드백은 언제나 환영입니다. 👋
