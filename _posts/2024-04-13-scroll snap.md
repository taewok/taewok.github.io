---
title: "[CSS] CSS만으로 Scroll Snap 구현하기"
date: 2024-04-13T00:35:00
categories: [css]
tags: [css, scroll-snap, layout]
description: "CSS scroll-snap-type과 scroll-snap-align을 사용해 스크롤 위치가 자연스럽게 맞춰지는 UI를 구현하는 방법을 정리했습니다."
custom_style: true
---

## Scroll Snap이란?

Scroll Snap은 사용자가 스크롤했을 때 특정 요소 위치에 스크롤이 자연스럽게 맞춰지도록 하는 CSS 기능입니다.

예를 들어 상품 카드나 섹션이 하나씩 딱 맞춰 보이게 만들 때 사용할 수 있습니다.

JavaScript 없이 CSS만으로 구현할 수 있어서 간단한 스냅 UI에는 매우 유용합니다.

---

## 기본 구조

Scroll Snap은 부모 요소와 자식 요소에 각각 속성을 지정합니다.

```html
<div class="scroll-container">
  <section class="snap-item">1</section>
  <section class="snap-item">2</section>
  <section class="snap-item">3</section>
</div>
```

```css
.scroll-container {
  height: 100vh;
  overflow-y: auto;
  scroll-snap-type: y mandatory;
}

.snap-item {
  height: 100vh;
  scroll-snap-align: start;
}
```

부모에는 `scroll-snap-type`, 자식에는 `scroll-snap-align`을 적용하는 것이 핵심입니다.

---

## scroll-snap-type

`scroll-snap-type`은 스냅 방향과 강도를 정합니다.

```css
scroll-snap-type: y mandatory;
```

앞의 `y`는 세로 방향 스크롤을 의미합니다.

가로 방향이라면 `x`를 사용합니다.

```css
scroll-snap-type: x mandatory;
```

`mandatory`는 스크롤이 끝났을 때 반드시 가까운 snap 지점에 맞춰지도록 합니다.

---

## scroll-snap-align

`scroll-snap-align`은 자식 요소가 snap 지점에 어떻게 맞춰질지 정합니다.

```css
.snap-item {
  scroll-snap-align: start;
}
```

자주 사용하는 값은 다음과 같습니다.

| 값 | 의미 |
| --- | --- |
| `start` | 요소의 시작 지점을 기준으로 맞춤 |
| `center` | 요소의 가운데를 기준으로 맞춤 |
| `end` | 요소의 끝 지점을 기준으로 맞춤 |

전체 화면 섹션은 보통 `start`를 많이 사용합니다.

카드 캐러셀처럼 가운데 정렬이 필요하면 `center`를 사용할 수 있습니다.

---

## 가로 스크롤 예시

```html
<div class="card-list">
  <article class="card">A</article>
  <article class="card">B</article>
  <article class="card">C</article>
</div>
```

```css
.card-list {
  display: flex;
  gap: 16px;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
}

.card {
  flex: 0 0 80%;
  scroll-snap-align: center;
}
```

이렇게 하면 가로로 스크롤할 때 카드가 가운데에 맞춰집니다.

---

## 사용할 때 주의할 점

Scroll Snap은 스크롤 위치를 강제로 맞추는 기능이기 때문에 콘텐츠 길이와 사용자 스크롤 경험을 함께 고려해야 합니다.

너무 작은 영역마다 스냅이 걸리면 오히려 답답하게 느껴질 수 있습니다.

또한 복잡한 제어가 필요하다면 CSS만으로는 부족할 수 있고, JavaScript를 함께 사용해야 할 수도 있습니다.

---

## 마무리

CSS Scroll Snap은 섹션이나 카드 단위로 스크롤 위치를 자연스럽게 맞출 때 유용합니다.

부모 요소에는 `scroll-snap-type`, 자식 요소에는 `scroll-snap-align`을 적용하면 기본 구조를 만들 수 있습니다.

단순한 스냅 UI라면 JavaScript 없이 CSS만으로도 충분히 구현할 수 있습니다.
