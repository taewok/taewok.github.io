---
title: "[CSS] normalize.css란 무엇이고 왜 사용할까?"
date: 2025-10-10 21:40:00 +0900
categories: [css, style]
tags: [css, reset, normalize, frontend]
description: "normalize.css의 역할과 reset.css와의 차이, 설치 및 사용 방법을 정리했습니다."
custom_style: true
---

## normalize.css란?

브라우저마다 HTML 요소의 기본 스타일은 조금씩 다릅니다.

`normalize.css`는 이런 브라우저 간 스타일 차이를 줄여주는 CSS 라이브러리입니다.

모든 기본 스타일을 제거하는 것이 아니라, 유용한 기본 스타일은 유지하면서 브라우저별 차이를 보정하는 데 목적이 있습니다.

---

## reset.css와의 차이

`reset.css`는 기본 스타일을 거의 모두 제거하고 0에서 다시 시작하는 방식입니다.

반면 `normalize.css`는 기본 스타일을 어느 정도 유지하면서 브라우저 간 차이를 맞춥니다.

| 구분 | reset.css | normalize.css |
| --- | --- | --- |
| 목적 | 기본 스타일 제거 | 기본 스타일 유지와 차이 보정 |
| 느낌 | 강한 초기화 | 부드러운 표준화 |
| 사용처 | 디자인 시스템이 명확한 프로젝트 | 기본 HTML 스타일을 어느 정도 활용하는 프로젝트 |

---

## 언제 사용하면 좋을까요?

normalize.css는 아래 상황에 잘 어울립니다.

- 브라우저 기본 스타일을 완전히 지우고 싶지는 않을 때
- 버튼, input 같은 기본 요소의 접근성과 UX를 어느 정도 유지하고 싶을 때
- 프로젝트 초기에 빠르게 일관된 스타일 기반을 만들고 싶을 때
- reset.css처럼 모든 스타일을 다시 정의하는 방식이 부담스러울 때

반대로 모든 컴포넌트 스타일을 디자인 시스템으로 통제해야 한다면 reset.css가 더 잘 맞을 수 있습니다.

---

## 설치하기

```bash
npm install normalize.css
```

설치 후 전역 스타일 파일에서 import합니다.

```css
@import "normalize.css";
```

또는 JavaScript 진입 파일에서 import할 수도 있습니다.

```js
import "normalize.css";
```

---

## normalize.css를 써도 추가 스타일은 필요해요

normalize.css는 모든 스타일 문제를 해결해주는 도구는 아닙니다.

브라우저 차이를 줄여줄 뿐, 프로젝트 디자인에 필요한 spacing, typography, layout 규칙은 여전히 직접 작성해야 합니다.

그래서 보통 normalize.css를 적용한 뒤 global style이나 design token을 함께 정의합니다.

```css
body {
  font-family: system-ui, sans-serif;
  color: #111;
  background: #fff;
}

button {
  cursor: pointer;
}
```

---

## 마무리

normalize.css는 브라우저 기본 스타일을 완전히 제거하지 않고, 브라우저마다 다른 부분을 보정해주는 CSS 라이브러리입니다.

reset.css보다 부드러운 방식으로 스타일 기반을 맞추고 싶을 때 사용하기 좋습니다.

프로젝트에서 기본 스타일을 얼마나 통제하고 싶은지에 따라 reset.css와 normalize.css 중 적절한 방식을 선택하면 됩니다.
