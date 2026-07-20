---
title: "[React] 프론트엔드 애니메이션 제대로 이해하고 사용하기"
date: 2026-07-20T10:00:00Z
categories: [frontend]
tags: [react, animation, css, requestanimationframe, framer-motion, frontend]
description: "React에서 CSS transition, keyframes, requestAnimationFrame, Web Animations API, Motion 라이브러리까지 애니메이션 기술을 상황별로 이해하고 활용하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며: 애니메이션은 꾸미기가 아니라 상태 변화의 설명이다

프론트엔드에서 애니메이션은 단순히 화면을 화려하게 만드는 장식처럼 보일 때가 많아요.

하지만 실제 서비스에서 좋은 애니메이션은 사용자가 화면 변화를 이해하도록 도와줍니다.

- 버튼을 눌렀을 때 반응이 느껴진다
- 메뉴가 어디에서 나타났는지 알 수 있다
- 모달이 갑자기 튀어나오지 않고 자연스럽게 열린다
- 리스트 순서가 바뀔 때 무엇이 어디로 이동했는지 보인다
- 페이지 전환이 끊기지 않고 이어진다

즉, 애니메이션은 "움직이는 효과"가 아니라 **상태 변화에 방향과 맥락을 붙이는 기술**에 가까워요.

이번 글에서는 React에서 애니메이션을 다룰 때 알아야 할 기술을 큰 흐름으로 정리해보겠습니다.

---

## React 애니메이션을 이해하는 큰 지도

React에서 애니메이션을 구현하는 방법은 여러 가지가 있어요.

처음부터 라이브러리만 외우기보다, 어떤 층위의 기술인지 나눠보면 훨씬 이해하기 쉽습니다.

```txt
CSS transition
→ 상태가 바뀔 때 부드럽게 전환

CSS keyframes
→ 정해진 움직임을 시간 순서대로 재생

requestAnimationFrame
→ JavaScript로 매 프레임 직접 제어

Web Animations API
→ 브라우저 내장 animation API를 JS로 제어

Motion / Framer Motion
→ React 상태, mount/unmount, layout 변화까지 선언적으로 애니메이션 처리
```

각 기술은 서로 대체 관계라기보다 역할이 다릅니다.

작은 hover 효과에 Framer Motion을 꼭 쓸 필요는 없고, 복잡한 exit animation을 CSS만으로 억지로 처리할 필요도 없어요.

중요한 건 "이 상황에서 가장 단순하고 안정적인 도구가 무엇인가"를 판단하는 겁니다.

---

## 1. 가장 먼저 CSS transition부터 이해하기

가장 기본적인 애니메이션은 CSS `transition`입니다.

`transition`은 어떤 CSS 값이 A에서 B로 바뀔 때 중간 과정을 부드럽게 보간해줍니다.

```css
.button {
  background-color: #27272a;
  transform: translateY(0);
  transition:
    background-color 150ms ease,
    transform 150ms ease;
}

.button:hover {
  background-color: #3f3f46;
  transform: translateY(-2px);
}
```

이 코드는 hover 상태가 되었을 때 배경색과 위치가 부드럽게 바뀌게 해줍니다.

React에서는 상태에 따라 className을 바꾸는 식으로 자주 사용해요.

```tsx
"use client";

import { useState } from "react";

export default function TogglePanel() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <section>
      <button type="button" onClick={() => setIsOpen((prev) => !prev)}>
        Toggle
      </button>

      <div className={isOpen ? "panel panel-open" : "panel"}>
        내용이 들어갑니다.
      </div>
    </section>
  );
}
```

```css
.panel {
  opacity: 0;
  transform: translateY(8px);
  transition:
    opacity 180ms ease,
    transform 180ms ease;
}

.panel-open {
  opacity: 1;
  transform: translateY(0);
}
```

여기서 React의 역할은 `isOpen`이라는 상태를 바꾸는 것이고, 실제 움직임은 CSS가 맡습니다.

이 구조가 가장 가볍고 안정적인 경우가 많아요.

---

## transition은 언제 쓰면 좋을까?

`transition`은 상태가 두 개일 때 특히 좋습니다.

- hover
- focus
- active
- open / close
- selected / unselected
- enabled / disabled

예를 들면 버튼, 탭, 토글, 드롭다운, 작은 패널에 잘 맞아요.

```txt
상태 A
↓
상태 B
```

이처럼 시작 상태와 끝 상태가 명확하면 CSS transition만으로 충분한 경우가 많습니다.

---

## 2. CSS keyframes로 정해진 동작 만들기

`transition`은 상태 변화 사이를 부드럽게 이어주는 도구예요.

반면 `keyframes`는 움직임의 단계를 직접 정의합니다.

```css
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(8px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.toast {
  animation: fadeUp 180ms ease-out;
}
```

토스트가 처음 나타날 때 한 번 재생되는 애니메이션이라면 이런 식으로 작성할 수 있어요.

조금 더 단계가 있는 움직임도 만들 수 있습니다.

```css
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
    transform: translateX(-2px);
  }

  100% {
    transform: translateX(0);
  }
}

.input-error {
  animation: shake 220ms ease;
}
```

이런 애니메이션은 에러 입력, 로딩 상태, 강조 효과처럼 "특정 순간에 한 번 재생되는 움직임"에 잘 어울립니다.

---

## transition과 keyframes의 차이

둘의 차이를 이렇게 기억하면 좋아요.

```txt
transition
→ CSS 값이 바뀔 때 부드럽게 이어준다

keyframes
→ 정해진 시간표대로 움직임을 재생한다
```

예를 들어 버튼 hover는 transition이 자연스럽습니다.

```txt
기본 버튼 → hover 버튼
```

반면 로딩 스피너는 keyframes가 더 자연스러워요.

```txt
0도 → 360도 → 다시 반복
```

```css
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.spinner {
  animation: spin 800ms linear infinite;
}
```

---

## 3. 무엇을 움직여야 성능이 좋을까?

애니메이션에서 가장 중요한 성능 기준은 어떤 CSS 속성을 움직이느냐예요.

가능하면 아래 두 속성을 중심으로 움직이는 것이 좋습니다.

- `transform`
- `opacity`

이 두 속성은 브라우저가 비교적 효율적으로 처리할 수 있어요.

```css
.good {
  transform: translateY(12px);
  opacity: 0;
}
```

반대로 아래 속성을 계속 애니메이션하면 레이아웃 계산 비용이 커질 수 있습니다.

- `width`
- `height`
- `top`
- `left`
- `margin`
- `padding`

예를 들어 요소를 오른쪽으로 움직이고 싶을 때는 `left`보다 `transform`을 먼저 고려하는 편이 좋아요.

```css
/* 아쉬운 방식 */
.box {
  left: 100px;
}

/* 더 나은 방식 */
.box {
  transform: translateX(100px);
}
```

애니메이션이 버벅인다면 먼저 "내가 어떤 CSS 속성을 움직이고 있지?"부터 확인해보면 좋습니다.

---

## will-change는 언제 쓸까?

`will-change`는 브라우저에게 "이 요소는 곧 바뀔 예정이야"라고 알려주는 CSS 속성이에요.

```css
.card {
  will-change: transform;
}
```

하지만 모든 요소에 습관처럼 붙이면 오히려 메모리 사용량이 늘 수 있습니다.

그래서 정말 자주 움직이는 요소나, 애니메이션 직전에 잠깐 적용하는 방식이 좋아요.

```css
.card:hover {
  will-change: transform;
}
```

`will-change`는 성능 문제를 해결하는 마법 버튼이 아니라, 브라우저에게 주는 힌트라고 보는 편이 안전합니다.

---

## 4. React 상태와 애니메이션 연결하기

React에서 애니메이션은 보통 상태 변화와 연결됩니다.

```tsx
const [isOpen, setIsOpen] = useState(false);
```

이 상태가 className을 바꿉니다.

```tsx
<div className={isOpen ? "modal modal-open" : "modal"} />
```

CSS는 그 className 변화에 맞춰 움직입니다.

```css
.modal {
  opacity: 0;
  transform: scale(0.96);
  transition:
    opacity 160ms ease,
    transform 160ms ease;
}

.modal-open {
  opacity: 1;
  transform: scale(1);
}
```

이 흐름은 React와 CSS의 역할이 깔끔하게 나뉘어서 좋아요.

```txt
React
→ 지금 열렸는지 닫혔는지 결정

CSS
→ 어떻게 움직일지 결정
```

대부분의 작은 UI 애니메이션은 이 방식으로 충분합니다.

---

## 5. 사라지는 애니메이션이 어려운 이유

React에서 애니메이션을 만들 때 가장 많이 막히는 부분이 exit animation입니다.

예를 들어 이렇게 조건부 렌더링을 하면:

```tsx
{
  isOpen && <Modal />;
}
```

`isOpen`이 `false`가 되는 순간 `Modal`은 React 트리에서 바로 사라집니다.

```txt
isOpen: true
→ Modal 렌더링

isOpen: false
→ Modal 즉시 제거
```

이러면 CSS로 닫히는 애니메이션을 줄 시간이 없습니다.

그래서 exit animation이 필요한 경우에는 두 가지 방법을 생각할 수 있어요.

- CSS로 직접 mount 상태와 visible 상태를 분리한다
- Motion의 `AnimatePresence` 같은 라이브러리를 사용한다

간단한 경우에는 mount 상태를 따로 둘 수 있습니다.

```tsx
"use client";

import { useEffect, useState } from "react";

export default function FadeModal({ open }: { open: boolean }) {
  const [mounted, setMounted] = useState(open);

  useEffect(() => {
    if (open) {
      setMounted(true);
      return;
    }

    const timerId = window.setTimeout(() => {
      setMounted(false);
    }, 180);

    return () => {
      window.clearTimeout(timerId);
    };
  }, [open]);

  if (!mounted) {
    return null;
  }

  return <div className={open ? "modal modal-open" : "modal"}>모달 내용</div>;
}
```

이 방식은 원리를 이해하기 좋지만, 컴포넌트가 많아지면 관리가 번거로워질 수 있어요.

---

## 6. Motion으로 선언적인 애니메이션 만들기

React에서 복잡한 애니메이션을 다룰 때는 Motion, 예전 이름으로 Framer Motion을 많이 사용합니다.

Motion의 장점은 React 컴포넌트의 상태 변화와 애니메이션을 선언적으로 연결할 수 있다는 점이에요.

```tsx
import { motion } from "motion/react";

export default function MotionBox() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 12 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.2 }}
    >
      자연스럽게 나타나는 박스
    </motion.div>
  );
}
```

여기서 중요한 props는 세 가지입니다.

- `initial`: 처음 상태
- `animate`: 목표 상태
- `transition`: 시간, easing, delay 같은 전환 설정

CSS className을 직접 바꾸는 대신, 움직임을 컴포넌트 props로 표현하는 방식이에요.

---

## AnimatePresence로 exit animation 처리하기

Motion을 쓰면 React에서 어려웠던 사라지는 애니메이션도 훨씬 편해집니다.

```tsx
import { AnimatePresence, motion } from "motion/react";

export default function Modal({ isOpen }: { isOpen: boolean }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.16 }}
        >
          모달 내용
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

`AnimatePresence`는 자식이 React 트리에서 제거될 때 `exit` 애니메이션을 실행할 수 있게 도와줍니다.

그래서 모달, 토스트, 드롭다운, 리스트 아이템 제거에 특히 유용해요.

---

## variants로 여러 요소를 함께 움직이기

여러 요소가 순서대로 나타나는 애니메이션은 `variants`로 정리할 수 있습니다.

```tsx
import { motion } from "motion/react";

const listVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.06,
    },
  },
};

const itemVariants = {
  hidden: {
    opacity: 0,
    y: 8,
  },
  visible: {
    opacity: 1,
    y: 0,
  },
};

export default function AnimatedList() {
  return (
    <motion.ul variants={listVariants} initial="hidden" animate="visible">
      {["React", "CSS", "Motion"].map((item) => (
        <motion.li key={item} variants={itemVariants}>
          {item}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

`variants`를 쓰면 부모와 자식 애니메이션을 같은 이름으로 연결할 수 있어요.

작은 예제에서는 조금 길어 보일 수 있지만, 실제 UI가 커질수록 애니메이션 의도를 한곳에 모을 수 있어서 편합니다.

---

## 7. layout animation 이해하기

리스트 정렬, 카드 확장, 탭 underline 이동처럼 위치나 크기가 바뀌는 애니메이션은 직접 구현하기 까다롭습니다.

예를 들어 카드가 위에서 아래로 이동한다면 브라우저는 최종 위치만 알고, 중간 움직임은 우리가 계산해야 할 수 있어요.

Motion은 이런 layout 변화도 `layout` prop으로 처리할 수 있습니다.

```tsx
import { motion } from "motion/react";

export default function Card({ children }: { children: React.ReactNode }) {
  return <motion.div layout>{children}</motion.div>;
}
```

요소의 크기나 위치가 바뀌면 Motion이 이전 레이아웃과 새로운 레이아웃 사이를 부드럽게 이어줍니다.

탭 underline처럼 서로 다른 요소 사이를 이어주고 싶다면 `layoutId`를 사용할 수 있어요.

```tsx
{
  tabs.map((tab) => (
    <button key={tab.id} onClick={() => setActiveTab(tab.id)}>
      {tab.label}
      {activeTab === tab.id && <motion.span layoutId="tab-underline" />}
    </button>
  ));
}
```

이런 애니메이션은 CSS만으로 처리하기보다 Motion의 도움을 받는 편이 훨씬 자연스럽습니다.

---

## 8. requestAnimationFrame은 언제 필요할까?

`requestAnimationFrame`은 브라우저가 다음 화면을 그리기 직전에 함수를 실행하도록 예약하는 API입니다.

MDN 문서에서도 다음 repaint 전에 콜백을 호출하도록 요청하는 메서드로 설명합니다.

```ts
let frameId: number;

const animate = () => {
  // 다음 프레임에서 실행할 작업
  frameId = requestAnimationFrame(animate);
};

frameId = requestAnimationFrame(animate);

cancelAnimationFrame(frameId);
```

React에서 직접 `requestAnimationFrame`을 쓸 일은 생각보다 많지 않아요.

대부분은 CSS나 Motion 같은 라이브러리가 프레임 관리를 대신해줍니다.

하지만 이런 경우에는 직접 사용할 수 있습니다.

- 스크롤 위치를 기반으로 직접 계산해야 할 때
- canvas 애니메이션을 만들 때
- DOM 측정과 다음 프레임 처리가 필요할 때
- 외부 라이브러리 없이 아주 세밀한 제어가 필요할 때

주의할 점은 매 프레임마다 React state를 바꾸면 렌더링 비용이 커질 수 있다는 거예요.

매 프레임 바뀌는 값은 가능하면 `ref`에 저장하거나, DOM style을 직접 다루거나, MotionValue 같은 도구를 사용하는 편이 좋습니다.

---

## 9. Web Animations API도 있다

브라우저에는 Web Animations API도 있습니다.

```tsx
"use client";

import { useEffect, useRef } from "react";

export default function WebAnimationBox() {
  const boxRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    const element = boxRef.current;

    if (!element) {
      return;
    }

    const animation = element.animate(
      [
        { opacity: 0, transform: "translateY(8px)" },
        { opacity: 1, transform: "translateY(0)" },
      ],
      {
        duration: 180,
        easing: "ease-out",
        fill: "forwards",
      },
    );

    return () => {
      animation.cancel();
    };
  }, []);

  return <div ref={boxRef}>Web Animations API</div>;
}
```

이 방식은 CSS와 JavaScript 사이에 있는 느낌이에요.

브라우저 내장 API로 애니메이션을 실행하고, JS에서 재생, 정지, 취소 같은 제어를 할 수 있습니다.

React 프로젝트에서는 Motion 같은 라이브러리를 더 자주 쓰지만, 브라우저가 기본으로 제공하는 애니메이션 API를 알아두면 내부 원리를 이해하는 데 도움이 됩니다.

---

## 10. 스크롤 애니메이션은 신중하게 다루기

스크롤 기반 애니메이션은 매력적이지만, 가장 쉽게 과해지는 영역이기도 해요.

좋은 스크롤 애니메이션은 사용자가 현재 위치와 콘텐츠 흐름을 이해하게 도와줍니다.

반대로 너무 많은 요소가 동시에 움직이면 읽기 피로도가 높아질 수 있어요.

Motion에서는 `useScroll`, `useTransform` 같은 Hook으로 스크롤 값을 애니메이션 값에 연결할 수 있습니다.

```tsx
"use client";

import { motion, useScroll, useTransform } from "motion/react";

export default function ScrollProgress() {
  const { scrollYProgress } = useScroll();
  const scaleX = useTransform(scrollYProgress, [0, 1], [0, 1]);

  return (
    <motion.div
      style={{ scaleX }}
      className="fixed left-0 top-0 h-1 w-full origin-left bg-blue-500"
    />
  );
}
```

여기서 핵심은 React state를 계속 바꾸지 않는다는 점이에요.

MotionValue가 프레임 단위의 값을 관리하기 때문에, 매 스크롤마다 React 컴포넌트가 다시 렌더링되는 구조를 피할 수 있습니다.

---

## 11. 접근성: 움직임을 줄이고 싶은 사용자도 있다

애니메이션은 모두에게 좋은 경험은 아닐 수 있어요.

어떤 사용자는 큰 움직임 때문에 어지러움이나 불편함을 느낄 수 있습니다.

그래서 `prefers-reduced-motion`을 고려해야 합니다.

CSS에서는 이렇게 처리할 수 있어요.

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Motion에서는 `useReducedMotion`을 사용할 수 있습니다.

```tsx
import { motion, useReducedMotion } from "motion/react";

export default function AccessibleMotionBox() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={{ opacity: 0, y: shouldReduceMotion ? 0 : 12 }}
      animate={{ opacity: 1, y: 0 }}
    >
      접근성을 고려한 애니메이션
    </motion.div>
  );
}
```

움직임을 완전히 없애기보다, 큰 이동은 줄이고 opacity 정도만 남기는 방식도 좋습니다.

---

## 12. 애니메이션 도구 선택 기준

상황별로 어떤 도구를 선택하면 좋을지 정리해보면 이렇습니다.

| 상황                             | 추천 방식                                   |
| -------------------------------- | ------------------------------------------- |
| hover, focus, active 상태        | CSS transition                              |
| 로딩 스피너, 반복 효과           | CSS keyframes                               |
| 단순 open / close                | CSS transition                              |
| mount animation                  | CSS keyframes 또는 Motion                   |
| unmount / exit animation         | Motion AnimatePresence                      |
| 리스트 재정렬, layout 변화       | Motion layout                               |
| canvas, 게임, 직접 프레임 제어   | requestAnimationFrame                       |
| 브라우저 내장 JS 애니메이션 제어 | Web Animations API                          |
| 스크롤 진행률 연동               | Motion useScroll 또는 requestAnimationFrame |

처음부터 무거운 도구를 고를 필요는 없어요.

작은 UI는 CSS로 시작하고, React의 mount/unmount나 layout 변화처럼 복잡도가 올라갈 때 Motion을 쓰는 흐름이 가장 편합니다.

---

## 13. 좋은 애니메이션을 만드는 감각

기술만큼 중요한 건 애니메이션의 감각이에요.

실무에서는 다음 기준을 자주 생각합니다.

- 대부분의 UI 애니메이션은 150ms에서 300ms 사이가 자연스럽다
- hover 반응은 짧게, 페이지 전환은 조금 더 길게 잡는다
- 큰 이동보다 작은 이동이 안정적으로 느껴진다
- opacity만 쓰면 밋밋할 수 있고, transform을 살짝 섞으면 방향이 생긴다
- 모든 요소를 움직이기보다 중요한 변화만 움직인다
- 사용자가 기다려야 하는 애니메이션은 짧아야 한다

예를 들어 모달은 이렇게 시작해도 충분히 자연스럽습니다.

```tsx
<motion.div
  initial={{ opacity: 0, scale: 0.96 }}
  animate={{ opacity: 1, scale: 1 }}
  exit={{ opacity: 0, scale: 0.98 }}
  transition={{ duration: 0.16, ease: "easeOut" }}
/>
```

움직임이 크지 않아도, 등장과 사라짐에 맥락이 생깁니다.

---

## 14. 자주 하는 실수

### 1. 모든 것을 애니메이션하려고 하기

애니메이션이 많다고 좋은 UI가 되지는 않아요.

사용자의 시선이 필요한 곳에만 움직임을 주는 편이 더 좋습니다.

### 2. height auto 애니메이션을 쉽게 생각하기

`height: 0`에서 `height: auto`로 바로 transition을 걸기는 어렵습니다.

이 경우에는 `max-height`를 쓰거나, 실제 높이를 측정하거나, Motion의 layout animation을 쓰는 편이 더 편해요.

### 3. display none에 transition 걸기

`display: none`과 `display: block` 사이에는 transition이 자연스럽게 걸리지 않습니다.

숨김 처리는 `opacity`, `visibility`, `pointer-events`, mount 상태를 함께 조합하는 경우가 많아요.

### 4. 매 프레임 React state를 바꾸기

스크롤이나 마우스 좌표처럼 자주 바뀌는 값을 매번 state로 저장하면 렌더링이 많아질 수 있습니다.

이런 값은 `ref`, MotionValue, CSS variable 같은 방식도 함께 고려하는 게 좋아요.

### 5. prefers-reduced-motion을 무시하기

접근성을 고려하지 않은 큰 애니메이션은 누군가에게 실제 불편이 될 수 있습니다.

작게라도 reduced motion 대응을 넣어두면 UI 완성도가 올라갑니다.

---

## 정리

React에서 애니메이션을 잘 다루려면 라이브러리 하나를 외우는 것보다, 각 기술의 역할을 구분하는 게 먼저입니다.

```txt
CSS transition
→ 상태 A에서 B로 부드럽게 전환

CSS keyframes
→ 정해진 움직임을 재생

requestAnimationFrame
→ 매 프레임 직접 제어

Web Animations API
→ 브라우저 내장 애니메이션을 JS로 제어

Motion
→ React 상태, mount/unmount, layout 변화를 선언적으로 애니메이션
```

작은 인터랙션은 CSS로 충분합니다.

컴포넌트가 사라지는 애니메이션, 리스트 재정렬, layout 변화, 스크롤 연동처럼 React 상태와 깊게 연결되는 움직임은 Motion 같은 라이브러리가 훨씬 편해요.

그리고 어떤 방식을 쓰든 `transform`, `opacity`, duration, easing, reduced motion을 함께 생각하면 애니메이션 품질이 훨씬 좋아집니다.

좋은 애니메이션은 눈에 띄기보다 이해를 돕습니다. 사용자가 "움직였다"보다 "자연스럽다"고 느끼게 만드는 것이 가장 좋은 방향이에요.

## 참고

- [MDN: CSS transition](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/transition)
- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
- [MDN: will-change](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/will-change)
- [Motion for React](https://motion.dev/docs/react)
- [Motion layout animation](https://motion.dev/docs/react-layout-animations)
- [Motion AnimatePresence](https://motion.dev/docs/react-animate-presence)
