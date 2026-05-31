---
title: "[React] 프로젝트 파비콘과 사이트 제목 변경하기"
date: 2023-04-02T18:10:00
categories: [react]
tags: [react, frontend, html, seo]
description: "React 프로젝트에서 브라우저 탭에 보이는 파비콘과 사이트 제목을 변경하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

React 프로젝트를 처음 만들면 브라우저 탭에 기본 React 아이콘과 `React App`이라는 제목이 보입니다.

기능 구현에는 큰 영향을 주지 않지만, 실제 서비스처럼 보이게 만들려면 이 두 가지를 꼭 바꿔주는 것이 좋습니다.

이번 글에서는 React 프로젝트에서 파비콘과 사이트 제목을 변경하는 방법을 간단히 정리해볼게요.

---

## 파비콘이란?

파비콘은 브라우저 탭, 북마크, 방문 기록 등에 표시되는 작은 아이콘입니다.

예를 들어 브라우저 탭 왼쪽에 보이는 작은 이미지가 파비콘입니다.

서비스를 만들 때 파비콘을 설정해두면 사용자가 여러 탭을 열어둔 상황에서도 사이트를 더 쉽게 알아볼 수 있습니다.

---

## 파비콘 파일 준비하기

먼저 사용할 아이콘 파일을 준비합니다.

보통 다음과 같은 파일을 사용합니다.

- `favicon.ico`
- `logo.png`
- `icon.svg`

React 프로젝트에서는 이 파일을 `public` 폴더 안에 넣으면 됩니다.

```txt
public
├─ favicon.ico
├─ index.html
└─ logo.png
```

---

## index.html에서 파비콘 변경하기

`public/index.html` 파일을 열어보면 `<head>` 안에 이런 코드가 있습니다.

```html
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
```

기본 파비콘 대신 `logo.png`를 사용하고 싶다면 이렇게 바꾸면 됩니다.

```html
<link rel="icon" href="%PUBLIC_URL%/logo.png" />
```

`%PUBLIC_URL%`은 React에서 `public` 폴더 경로를 가리키는 값이라고 생각하면 됩니다.

---

## 사이트 제목 변경하기

브라우저 탭에 보이는 제목은 `<title>` 태그에서 변경할 수 있습니다.

기본값은 보통 이렇게 되어 있습니다.

```html
<title>React App</title>
```

원하는 서비스 이름으로 바꿔주면 됩니다.

```html
<title>Mbti World</title>
```

이제 브라우저 탭에는 `React App` 대신 `Mbti World`가 표시됩니다.

---

## 페이지마다 제목을 다르게 보여주고 싶다면?

단일 제목만 필요하다면 `index.html`의 `<title>`만 바꿔도 충분합니다.

하지만 페이지마다 다른 제목을 보여주고 싶을 때도 있습니다.

예를 들면 이런 식입니다.

- 홈: `Mbti World`
- 프로필: `프로필 | Mbti World`
- 로그인: `로그인 | Mbti World`

이럴 때는 `react-helmet` 같은 라이브러리를 사용할 수 있습니다.

```bash
npm install react-helmet
```

사용 예시는 다음과 같습니다.

```tsx
import { Helmet } from "react-helmet";

const ProfilePage = () => {
  return (
    <>
      <Helmet>
        <title>프로필 | Mbti World</title>
      </Helmet>

      <h1>프로필 관리</h1>
    </>
  );
};

export default ProfilePage;
```

이렇게 하면 사용자가 프로필 페이지에 들어왔을 때 브라우저 탭 제목이 `프로필 | Mbti World`로 바뀝니다.

---

## 변경이 바로 보이지 않을 때

파비콘은 브라우저 캐시 때문에 바로 바뀌지 않을 때가 있습니다.

그럴 때는 다음 방법을 시도해볼 수 있습니다.

- 새로고침을 여러 번 해보기
- 강력 새로고침하기
- 브라우저 캐시 삭제하기
- 파일 이름을 새롭게 바꾸기

예를 들어 `favicon.ico`를 계속 같은 이름으로 두면 예전 이미지가 남아 보일 수 있습니다.

이럴 때는 `favicon-v2.ico`처럼 파일명을 바꾸고 `index.html`의 경로도 같이 수정하면 더 확실하게 반영됩니다.

---

## 마무리

파비콘과 사이트 제목은 작은 설정처럼 보이지만, 사용자가 서비스를 인식하는 첫 화면에 가깝습니다.

React 프로젝트에서는 `public/index.html`에서 파비콘과 `<title>`을 변경하면 기본 설정을 쉽게 바꿀 수 있습니다.

페이지마다 제목을 다르게 관리해야 한다면 `react-helmet` 같은 도구를 함께 사용하면 더 유연하게 처리할 수 있습니다.
