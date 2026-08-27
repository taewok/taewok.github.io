---
title: "[CSS] 초보 개발자를 위한 애니메이션 효과 만들기"
date: 2026-08-27T00:50:00Z
categories: [css, frontend]
tags: [css, animation, transition, keyframes, transform, frontend]
description: "CSS transition과 @keyframes를 이용해 버튼 hover, 클릭 효과, 카드 등장, 로딩 스피너 같은 기본 애니메이션을 만드는 방법을 초보자 관점에서 정리했습니다."
custom_style: true
---

## 들어가며: 애니메이션은 어려운 기술일까요?

웹 페이지를 만들다 보면 이런 효과를 자주 보게 됩니다.

- 버튼에 마우스를 올리면 살짝 떠오름
- 카드를 클릭하면 눌리는 느낌이 남
- 모달이 갑자기 나타나지 않고 부드럽게 등장함
- 로딩 중일 때 원이 빙글빙글 돎
- 리스트 아이템이 아래에서 위로 올라오며 나타남

처음에는 이런 애니메이션이 복잡한 JavaScript나 라이브러리로만 가능할 것처럼 느껴질 수 있습니다.

하지만 간단한 UI 애니메이션은 CSS만으로도 충분히 만들 수 있습니다.

CSS 애니메이션을 배울 때 가장 먼저 이해해야 하는 도구는 두 가지입니다.

```txt
transition
→ 상태가 바뀔 때 부드럽게 전환하기

@keyframes + animation
→ 정해진 움직임을 시간 순서대로 재생하기
```

이번 글에서는 초보 개발자가 바로 따라 할 수 있도록 `transition`, `transform`, `opacity`, `@keyframes`, `animation`을 차근차근 정리해보겠습니다.

---

## 먼저 transition과 animation의 차이

CSS로 움직임을 만들 때 가장 헷갈리는 부분이 `transition`과 `animation`의 차이입니다.

두 기능은 모두 요소를 움직이거나 변화시킬 수 있지만, 쓰는 상황이 다릅니다.

| 구분 | transition | animation |
| --- | --- | --- |
| 시작 조건 | hover, active, class 변경 같은 상태 변화가 필요 | 자동 실행 가능 |
| 움직임 단계 | 시작 상태와 끝 상태 중심 | 여러 단계 지정 가능 |
| 반복 | 보통 반복하지 않음 | 반복 가능 |
| 대표 예시 | 버튼 hover, 메뉴 열림, 카드 확대 | 로딩 스피너, 흔들림, 등장 효과 |

쉽게 말하면 이렇습니다.

```txt
사용자의 행동이나 상태 변화에 반응한다면 transition
시간 순서대로 정해진 동작을 재생한다면 animation
```

버튼에 마우스를 올렸을 때 색이 바뀌는 효과는 `transition`이 잘 맞습니다.

로딩 아이콘이 계속 회전하는 효과는 `animation`이 잘 맞습니다.

---

## transition 기본 문법

`transition`은 CSS 값이 바뀔 때 그 중간 과정을 부드럽게 만들어줍니다.

```css
.button {
  background-color: #27272a;
  transition: background-color 200ms ease;
}

.button:hover {
  background-color: #3f3f46;
}
```

이 코드는 버튼에 마우스를 올렸을 때 배경색이 즉시 바뀌지 않고 `200ms` 동안 부드럽게 바뀌게 합니다.

`transition`은 보통 다음 값을 함께 씁니다.

```css
transition: property duration timing-function delay;
```

하나씩 보면 이렇습니다.

| 값 | 의미 | 예시 |
| --- | --- | --- |
| `property` | 어떤 속성을 전환할지 | `background-color` |
| `duration` | 얼마나 오래 걸릴지 | `200ms`, `0.3s` |
| `timing-function` | 속도 곡선 | `ease`, `linear`, `ease-in-out` |
| `delay` | 언제 시작할지 | `100ms` |

가장 자주 쓰는 형태는 다음과 같습니다.

```css
transition: all 200ms ease;
```

하지만 실무에서는 `all`보다 바뀌는 속성을 직접 적는 편이 더 좋습니다.

```css
transition:
  background-color 200ms ease,
  transform 200ms ease;
```

이렇게 하면 어떤 속성에 애니메이션이 걸리는지 명확해집니다.

---

## 예제 1: 버튼 hover 효과 만들기

가장 기본적인 버튼 hover 효과부터 만들어보겠습니다.

```html
<button class="primary-button">시작하기</button>
```

```css
.primary-button {
  border: 0;
  border-radius: 8px;
  padding: 12px 18px;
  background-color: #2563eb;
  color: white;
  font-weight: 700;
  cursor: pointer;
  transition:
    background-color 180ms ease,
    transform 180ms ease,
    box-shadow 180ms ease;
}

.primary-button:hover {
  background-color: #1d4ed8;
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(37, 99, 235, 0.25);
}
```

여기서 핵심은 `transform: translateY(-2px)`입니다.

`translateY(-2px)`는 요소를 위로 `2px` 이동시킵니다.

```txt
기본 상태
→ 원래 위치

hover 상태
→ 위로 2px 이동
```

그리고 `transition`이 있기 때문에 이 이동이 부드럽게 보입니다.

---

## 예제 2: 클릭했을 때 눌리는 버튼 만들기

버튼은 hover뿐 아니라 클릭했을 때의 느낌도 중요합니다.

이때는 `:active`를 사용할 수 있습니다.

```css
.primary-button:active {
  transform: translateY(0);
  box-shadow: 0 4px 10px rgba(37, 99, 235, 0.2);
}
```

전체 흐름은 이렇게 됩니다.

```txt
기본 상태
→ 버튼이 평평하게 있음

hover
→ 살짝 위로 떠오름

active
→ 다시 아래로 눌림
```

이런 작은 차이가 버튼을 더 자연스럽게 느껴지게 만듭니다.

초보 단계에서는 거창한 애니메이션보다 이런 작은 피드백을 먼저 연습하는 것이 좋습니다.

---

## transform을 자주 쓰는 이유

애니메이션을 만들 때는 `top`, `left`, `width`, `height`보다 `transform`을 자주 사용합니다.

예를 들어 요소를 오른쪽으로 움직이고 싶을 때 이렇게 할 수도 있습니다.

```css
.box:hover {
  left: 20px;
}
```

하지만 보통은 이렇게 씁니다.

```css
.box:hover {
  transform: translateX(20px);
}
```

`transform`은 요소의 위치, 크기, 회전 등을 바꾸는 데 사용합니다.

자주 쓰는 값은 다음과 같습니다.

| 값 | 의미 |
| --- | --- |
| `translateX(20px)` | 가로 방향으로 이동 |
| `translateY(-10px)` | 세로 방향으로 이동 |
| `scale(1.05)` | 크기를 1.05배 확대 |
| `rotate(12deg)` | 12도 회전 |

애니메이션에서는 보통 `transform`과 `opacity` 조합을 많이 씁니다.

```css
.card {
  opacity: 0;
  transform: translateY(12px);
}

.card.show {
  opacity: 1;
  transform: translateY(0);
}
```

이 조합만 잘 써도 대부분의 등장 효과를 만들 수 있습니다.

---

## 예제 3: 카드가 부드럽게 떠오르는 효과

카드 UI에 마우스를 올렸을 때 살짝 떠오르는 효과를 만들어보겠습니다.

```html
<article class="card">
  <h2>CSS Animation</h2>
  <p>transition으로 자연스러운 hover 효과를 만들 수 있습니다.</p>
</article>
```

```css
.card {
  max-width: 320px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  background-color: white;
  box-shadow: 0 2px 8px rgba(15, 23, 42, 0.08);
  transition:
    transform 200ms ease,
    box-shadow 200ms ease;
}

.card:hover {
  transform: translateY(-6px);
  box-shadow: 0 16px 32px rgba(15, 23, 42, 0.14);
}
```

카드 hover 효과는 실무에서 정말 자주 사용됩니다.

다만 너무 많이 띄우면 화면이 가벼워 보일 수 있으므로 `4px`에서 `8px` 정도의 작은 이동으로 시작하는 것이 좋습니다.

---

## 예제 4: 메뉴 열림 효과 만들기

이번에는 class가 바뀔 때 transition이 동작하는 예시를 보겠습니다.

```html
<button class="menu-button">메뉴</button>

<div class="dropdown is-open">
  <a href="/">홈</a>
  <a href="/posts">글 목록</a>
  <a href="/about">소개</a>
</div>
```

```css
.dropdown {
  width: 180px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 8px;
  background-color: white;
  opacity: 0;
  transform: translateY(-8px);
  pointer-events: none;
  transition:
    opacity 160ms ease,
    transform 160ms ease;
}

.dropdown.is-open {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}
```

여기서 중요한 부분은 `opacity`, `transform`, `pointer-events`입니다.

```txt
opacity
→ 눈에 보이는 정도

transform
→ 나타나는 방향

pointer-events
→ 닫힌 상태에서 클릭되지 않게 처리
```

닫힌 상태에서는 투명하고 위로 살짝 올라가 있습니다.

열린 상태가 되면 불투명해지고 원래 위치로 내려옵니다.

이런 식으로 만들면 드롭다운이 갑자기 튀어나오지 않고 자연스럽게 보입니다.

---

## @keyframes 기본 문법

이번에는 `animation`을 보겠습니다.

`transition`은 상태 변화가 필요하지만, `animation`은 정해진 움직임을 자동으로 재생할 수 있습니다.

먼저 `@keyframes`로 움직임의 단계를 정의합니다.

```css
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

그다음 요소에 `animation`을 적용합니다.

```css
.fade-up {
  animation: fadeUp 400ms ease both;
}
```

여기서 `both`는 `animation-fill-mode` 값입니다.

애니메이션이 끝난 뒤 마지막 상태를 유지하게 해줍니다.

```txt
fadeUp
→ 사용할 keyframes 이름

400ms
→ 실행 시간

ease
→ 속도 곡선

both
→ 시작 전과 종료 후 스타일 유지 방식
```

---

## 예제 5: 화면에 등장하는 카드 만들기

페이지에 들어왔을 때 카드가 아래에서 올라오며 나타나는 효과를 만들어보겠습니다.

```html
<article class="post-card fade-up">
  <h2>새로운 글</h2>
  <p>CSS keyframes로 등장 효과를 만들 수 있습니다.</p>
</article>
```

```css
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-up {
  animation: fadeUp 400ms ease both;
}
```

이 효과는 카드, 모달, 토스트 메시지, 섹션 제목 등에 자주 사용됩니다.

다만 모든 요소가 동시에 움직이면 산만할 수 있습니다.

여러 요소를 순서대로 보여주고 싶다면 `animation-delay`를 사용할 수 있습니다.

```css
.post-card:nth-child(1) {
  animation-delay: 0ms;
}

.post-card:nth-child(2) {
  animation-delay: 80ms;
}

.post-card:nth-child(3) {
  animation-delay: 160ms;
}
```

이렇게 하면 카드가 약간의 시간차를 두고 나타납니다.

---

## 예제 6: 로딩 스피너 만들기

계속 반복되는 움직임은 `animation`이 잘 어울립니다.

대표적인 예시가 로딩 스피너입니다.

```html
<div class="spinner" aria-label="로딩 중"></div>
```

```css
.spinner {
  width: 32px;
  height: 32px;
  border: 4px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 800ms linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
```

여기서 `infinite`는 애니메이션을 계속 반복하겠다는 뜻입니다.

```txt
spin
→ keyframes 이름

800ms
→ 한 바퀴 도는 시간

linear
→ 일정한 속도

infinite
→ 무한 반복
```

스피너처럼 계속 같은 속도로 돌아야 하는 경우에는 `linear`가 자연스럽습니다.

반대로 버튼이나 카드 hover처럼 시작과 끝에 약간의 부드러움이 필요한 경우에는 `ease`나 `ease-out`을 자주 씁니다.

---

## 예제 7: 흔들리는 에러 메시지 만들기

폼 검증에 실패했을 때 입력창이나 메시지를 살짝 흔들어 줄 수 있습니다.

```html
<p class="error-message shake">비밀번호를 입력해주세요.</p>
```

```css
.error-message {
  color: #dc2626;
  font-size: 14px;
}

.shake {
  animation: shake 300ms ease;
}

@keyframes shake {
  0% {
    transform: translateX(0);
  }

  25% {
    transform: translateX(-4px);
  }

  50% {
    transform: translateX(4px);
  }

  75% {
    transform: translateX(-3px);
  }

  100% {
    transform: translateX(0);
  }
}
```

`@keyframes`는 `from`, `to`만 사용할 수도 있고, 이렇게 퍼센트 단위로 여러 단계를 만들 수도 있습니다.

```txt
0%
→ 시작

50%
→ 중간

100%
→ 끝
```

흔들림 효과는 너무 강하면 불편할 수 있으므로 짧고 작게 주는 편이 좋습니다.

---

## transition이 동작하지 않는 것처럼 보일 때

초보 단계에서 자주 만나는 문제가 있습니다.

```txt
transition을 넣었는데 왜 애니메이션이 안 되지?
```

대부분은 다음 이유 중 하나입니다.

### 1. 바뀌는 값이 없다

`transition`은 값이 바뀔 때만 동작합니다.

```css
.box {
  transition: transform 200ms ease;
}
```

이렇게만 쓰면 아무 일도 일어나지 않습니다.

변화하는 상태가 필요합니다.

```css
.box:hover {
  transform: translateY(-4px);
}
```

### 2. display는 부드럽게 바뀌지 않는다

다음 코드는 기대한 것처럼 동작하지 않습니다.

```css
.menu {
  display: none;
  transition: opacity 200ms ease;
}

.menu.is-open {
  display: block;
  opacity: 1;
}
```

`display: none`에서 `display: block`으로 바뀌는 것은 중간 상태를 만들 수 없습니다.

그래서 메뉴나 모달 등장 효과는 보통 `opacity`와 `transform`을 사용합니다.

```css
.menu {
  opacity: 0;
  transform: translateY(-8px);
  pointer-events: none;
  transition:
    opacity 200ms ease,
    transform 200ms ease;
}

.menu.is-open {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}
```

### 3. 시작 상태가 정의되어 있지 않다

끝 상태만 있고 시작 상태가 없으면 브라우저가 무엇에서 무엇으로 바뀌는지 알기 어렵습니다.

```css
.modal.is-open {
  opacity: 1;
  transform: scale(1);
}
```

닫힌 상태도 함께 정의해두는 것이 좋습니다.

```css
.modal {
  opacity: 0;
  transform: scale(0.96);
  transition:
    opacity 180ms ease,
    transform 180ms ease;
}

.modal.is-open {
  opacity: 1;
  transform: scale(1);
}
```

---

## duration은 어느 정도가 좋을까?

애니메이션 시간은 너무 짧으면 티가 나지 않고, 너무 길면 답답합니다.

초보 단계에서는 아래 기준으로 시작하면 좋습니다.

| 상황 | 추천 시간 |
| --- | --- |
| 버튼 hover | `120ms` ~ `200ms` |
| 클릭 피드백 | `80ms` ~ `150ms` |
| 드롭다운 열림 | `150ms` ~ `250ms` |
| 모달 등장 | `180ms` ~ `300ms` |
| 페이지 섹션 등장 | `300ms` ~ `500ms` |
| 로딩 스피너 1회전 | `700ms` ~ `1200ms` |

중요한 UI일수록 움직임이 너무 과하면 안 됩니다.

애니메이션은 사용자의 작업을 방해하지 않고, 변화가 일어났다는 사실을 설명하는 정도가 좋습니다.

---

## easing은 무엇을 고르면 좋을까?

`ease`, `linear`, `ease-in`, `ease-out`, `ease-in-out` 같은 값은 애니메이션의 속도 곡선을 정합니다.

처음에는 이렇게 기억해도 충분합니다.

| easing | 느낌 | 사용 예시 |
| --- | --- | --- |
| `linear` | 일정한 속도 | 스피너, 진행 바 |
| `ease` | 기본적인 부드러움 | 일반 hover |
| `ease-out` | 빠르게 시작해서 부드럽게 끝남 | 등장 효과 |
| `ease-in` | 천천히 시작해서 빠르게 사라짐 | 퇴장 효과 |
| `ease-in-out` | 시작과 끝이 모두 부드러움 | 패널 열고 닫기 |

초보 단계에서는 다음 조합을 추천합니다.

```css
/* 버튼, 카드 hover */
transition: transform 180ms ease;

/* 등장 효과 */
animation: fadeUp 360ms ease-out both;

/* 반복 회전 */
animation: spin 900ms linear infinite;
```

이 정도만으로도 대부분의 기본 UI 애니메이션을 만들 수 있습니다.

---

## 접근성: 움직임을 줄이고 싶은 사용자도 있어요

애니메이션은 사용성을 높일 수 있지만, 어떤 사용자에게는 불편할 수도 있습니다.

운영체제에서 움직임 줄이기 설정을 켜둔 사용자를 위해 CSS에서는 `prefers-reduced-motion`을 사용할 수 있습니다.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    scroll-behavior: auto !important;
    transition-duration: 0.01ms !important;
  }
}
```

이 코드는 사용자가 움직임을 줄이도록 설정한 경우 대부분의 애니메이션을 사실상 아주 짧게 만듭니다.

조금 더 섬세하게 처리하고 싶다면 특정 애니메이션만 제거할 수도 있습니다.

```css
@media (prefers-reduced-motion: reduce) {
  .fade-up,
  .shake,
  .spinner {
    animation: none;
  }

  .card,
  .primary-button {
    transition: none;
  }
}
```

애니메이션을 만들 때는 "멋있게 움직이는가"뿐 아니라 "움직이지 않아도 사용할 수 있는가"도 함께 생각하는 것이 좋습니다.

---

## 초보자가 먼저 연습하면 좋은 순서

CSS 애니메이션을 처음 배운다면 다음 순서로 연습해보면 좋습니다.

```txt
1. button hover 색상 변경
2. button hover translateY 적용
3. card hover shadow 적용
4. dropdown opacity + transform 전환
5. @keyframes fadeUp 등장 효과
6. @keyframes spin 로딩 스피너
7. prefers-reduced-motion 적용
```

이 순서대로 익히면 `transition`과 `animation`의 차이를 자연스럽게 이해할 수 있습니다.

처음부터 복잡한 3D 효과나 스크롤 애니메이션을 만들려고 하면 CSS가 어렵게 느껴질 수 있습니다.

작은 효과를 많이 만들어보는 편이 훨씬 좋습니다.

---

## 전체 예제 코드

마지막으로 버튼, 카드, 스피너를 한 번에 볼 수 있는 간단한 예제를 정리해보겠습니다.

```html
<button class="primary-button">시작하기</button>

<article class="card fade-up">
  <h2>CSS 애니메이션</h2>
  <p>transition과 keyframes로 기본 효과를 만들 수 있습니다.</p>
</article>

<div class="spinner" aria-label="로딩 중"></div>
```

```css
.primary-button {
  border: 0;
  border-radius: 8px;
  padding: 12px 18px;
  background-color: #2563eb;
  color: white;
  font-weight: 700;
  cursor: pointer;
  transition:
    background-color 180ms ease,
    transform 180ms ease,
    box-shadow 180ms ease;
}

.primary-button:hover {
  background-color: #1d4ed8;
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(37, 99, 235, 0.25);
}

.primary-button:active {
  transform: translateY(0);
  box-shadow: 0 4px 10px rgba(37, 99, 235, 0.2);
}

.card {
  max-width: 320px;
  margin-top: 24px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 20px;
  background-color: white;
  box-shadow: 0 2px 8px rgba(15, 23, 42, 0.08);
  transition:
    transform 200ms ease,
    box-shadow 200ms ease;
}

.card:hover {
  transform: translateY(-6px);
  box-shadow: 0 16px 32px rgba(15, 23, 42, 0.14);
}

.spinner {
  width: 32px;
  height: 32px;
  margin-top: 24px;
  border: 4px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 800ms linear infinite;
}

.fade-up {
  animation: fadeUp 400ms ease-out both;
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

이 예제만 이해해도 기본적인 UI 애니메이션은 꽤 많이 만들 수 있습니다.

---

## 정리

CSS 애니메이션은 처음부터 어렵게 접근할 필요가 없습니다.

먼저 두 가지를 구분하면 됩니다.

```txt
transition
→ 상태 변화가 있을 때 부드럽게 전환

animation
→ keyframes에 적어둔 움직임을 시간 순서대로 재생
```

그리고 애니메이션을 만들 때는 `transform`과 `opacity`를 먼저 떠올리면 좋습니다.

- 버튼 hover는 `transition`
- 카드 떠오름은 `transition`
- 드롭다운 열림은 `opacity + transform + transition`
- 카드 등장 효과는 `@keyframes + animation`
- 로딩 스피너는 `rotate + infinite animation`
- 움직임이 불편한 사용자를 위해 `prefers-reduced-motion`

좋은 애니메이션은 화려한 효과가 아니라 사용자가 변화를 자연스럽게 이해하도록 돕는 효과입니다.

그래서 처음에는 작고 짧게, 그리고 필요한 곳에만 넣는 연습부터 시작하면 좋습니다.

## 참고

- [MDN: CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Transitions)
- [MDN: transition](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/transition)
- [MDN: CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Animations)
- [MDN: @keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/%40keyframes)
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/%40media/prefers-reduced-motion)
