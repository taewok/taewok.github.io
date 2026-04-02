---
title: "[React] 프로젝트의 얼굴 만들기: 파비콘(Favicon)과 사이트 타이틀 변경하기"
date: 2023-04-02T18:10:000
categories: [React]
tags: [react, frontend, html, seo]
---

리액트 프로젝트를 처음 생성하면 브라우저 탭에 기본 리액트 로고와 "React App"이라는 문구가 뜹니다. 프로젝트의 정체성을 살리기 위해 가장 먼저 해야 할 작업이 바로 **파비콘(Favicon)**과 **사이트 타이틀(Title)** 변경입니다.

오늘은 `public` 폴더 내의 파일을 수정하여 이 두 가지를 변경하는 방법을 알아보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. 파비콘(Favicon)이란?</b>

**파비콘(Favorite Icon)**은 브라우저 탭, 즐겨찾기 목록 등에서 웹사이트를 대표하는 작은 아이콘입니다. 보통 16x16 또는 32x32 픽셀의 크기를 가지며, 사용자가 여러 탭을 띄워놓았을 때 사이트를 빠르게 식별하게 도와줍니다.

### 파비콘 변경 방법

1.  사용할 이미지 파일(예: `logo.png` 또는 `favicon.ico`)을 `public` 폴더 안에 넣어줍니다.
2.  `public/index.html` 파일을 열고 `<head>` 태그 내의 `link` 태그를 수정합니다.

#### 수정 전 (기본값)

```html
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
```

#### 수정 후

```html
<link rel="icon" href="%PUBLIC_URL%/new-logo.png" />
```

---

## <b style="border-bottom:2px solid gray" class="h2">2. 웹 사이트 타이틀(Title) 변경</b>

웹 타이틀은 페이지의 제목을 나타내며, **SEO(검색엔진 최적화)**에서 가장 중요한 요소 중 하나입니다. 검색 결과 화면에 제목으로 노출되기 때문입니다.

### 타이틀 변경 방법

`public/index.html` 파일에서 `<title>` 태그를 찾아 원하는 문구로 수정합니다.

#### 수정 전

```html
<title>React App</title>
```

#### 수정 후

```html
<title>나만의 MBTI 테스트 | Mbti World</title>
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. 실전 팁: 페이지별로 타이틀을 바꾸고 싶다면?</b>

만약 `index.html`에서 수정한 고정 타이틀이 아니라, 사용자가 이동하는 페이지(Home, Profile 등)마다 타이틀을 다르게 보여주고 싶다면 **React Helmet** 라이브러리를 추천합니다.

```bash
npm i react-helmet
```

```tsx
import { Helmet } from "react-helmet";

const ProfilePage = () => {
  return (
    <div>
      <Helmet>
        <title>프로필 수정 | Mbti World</title>
      </Helmet>
      <h1>내 프로필 관리</h1>
    </div>
  );
};
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>작은 아이콘과 타이틀 하나가 웹사이트의 완성도를 결정합니다. 특히 타이틀은 검색 엔진이 내 사이트를 이해하는 첫 번째 단서라는 점</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
