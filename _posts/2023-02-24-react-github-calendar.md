---
title: "[React] react-github-calendar로 깃허브 잔디 보여주기"
date: 2023-02-24T18:34:00
categories: [react]
tags: [react, react-github-calendar, github, portfolio, frontend]
description: "React 프로젝트에서 react-github-calendar를 사용해 깃허브 커밋 기록을 화면에 보여주는 방법을 정리했습니다."
custom_style: true
---

## 🧐 포트폴리오에 깃허브 잔디를 보여주고 싶다면?

개발자 포트폴리오를 만들다 보면 깃허브 잔디, 즉 Contribution Graph를 보여주고 싶을 때가 있습니다.

직접 GitHub API를 호출해서 데이터를 가져올 수도 있지만, 간단히 보여주는 목적이라면 `react-github-calendar` 라이브러리를 사용하면 편합니다.

이번 글에서는 `react-github-calendar`를 설치하고 화면에 적용하는 방법을 정리해보겠습니다.

---

## 📦 라이브러리 설치하기

먼저 프로젝트 터미널에서 패키지를 설치합니다.

```bash
npm install react-github-calendar
```

`yarn`을 사용한다면 다음 명령어를 사용하면 됩니다.

```bash
yarn add react-github-calendar
```

---

## 🛠️ 기본 사용법

사용법은 간단합니다.

`GitHubCalendar` 컴포넌트를 import하고, `username`에 깃허브 아이디를 넣어주면 됩니다.

```jsx
import GitHubCalendar from "react-github-calendar";

function App() {
  return (
    <div>
      <GitHubCalendar username="github-username" />
    </div>
  );
}

export default App;
```

`github-username` 부분에는 본인의 GitHub 아이디를 넣으면 됩니다.

---

## 🎨 옵션으로 스타일 조정하기

`GitHubCalendar` 컴포넌트는 여러 props를 제공합니다.

| prop | 설명 | 예시 |
| :--- | :--- | :--- |
| `year` | 특정 연도의 데이터를 보여줍니다. | `<GitHubCalendar year={2023} />` |
| `fontSize` | 글자 크기를 조절합니다. | `<GitHubCalendar fontSize={16} />` |
| `blockSize` | 잔디 블록 하나의 크기를 조절합니다. | `<GitHubCalendar blockSize={15} />` |
| `blockMargin` | 잔디 블록 사이의 간격을 조절합니다. | `<GitHubCalendar blockMargin={5} />` |
| `blockRadius` | 잔디 블록의 둥근 정도를 조절합니다. | `<GitHubCalendar blockRadius={2} />` |

예를 들어 블록 크기와 간격을 조정하고 싶다면 다음처럼 작성할 수 있습니다.

```jsx
<GitHubCalendar
  username="github-username"
  blockSize={14}
  blockMargin={4}
  blockRadius={3}
/>
```

---

## 🌈 테마 커스텀하기

버전에 따라 색상 설정 방식이 다를 수 있습니다.

최근 버전에서는 `theme`을 사용해 단계별 색상을 직접 지정할 수 있습니다.

```jsx
const theme = {
  light: ["#ebedf0", "#9be9a8", "#40c463", "#30a14e", "#216e39"],
  dark: ["#161b22", "#0e4429", "#006d32", "#26a641", "#39d353"],
};

<GitHubCalendar username="github-username" theme={theme} />;
```

적용이 잘 되지 않는다면 사용 중인 `react-github-calendar` 버전의 공식 문서를 함께 확인하는 것이 좋습니다.

---

## ✅ 정리

`react-github-calendar`를 사용하면 React 프로젝트에서 깃허브 잔디를 간단히 보여줄 수 있습니다.

- `npm install react-github-calendar`로 설치합니다.
- `GitHubCalendar` 컴포넌트를 import합니다.
- `username`에 GitHub 아이디를 넣습니다.
- `blockSize`, `blockMargin`, `blockRadius` 같은 props로 모양을 조정할 수 있습니다.
- 색상은 버전에 따라 `theme` 설정을 확인해야 합니다.

포트폴리오에 GitHub 활동 기록을 보여주고 싶다면 가볍게 적용해보기 좋은 라이브러리입니다.
