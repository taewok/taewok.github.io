---
title: "[React] Framer Motion 제대로 이해하고 다루기"
date: 2026-06-03T10:30:00Z
categories: [frontend]
tags: [react, framer-motion, motion, animation, ui]
description: "Framer Motion의 핵심 개념부터 motion 컴포넌트, transition, variants, AnimatePresence, layout 애니메이션, MotionValue, 성능과 접근성까지 한 번에 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 애니메이션은 왜 어렵게 느껴질까요?

React에서 애니메이션을 만들다 보면 처음에는 간단해 보입니다.

버튼에 hover 효과를 주거나, 모달이 열릴 때 살짝 나타나는 정도는 CSS transition으로도 충분합니다.

하지만 조금만 복잡해지면 고민이 생깁니다.

- 컴포넌트가 사라지기 전에 exit 애니메이션을 보여주고 싶을 때
- 리스트 순서가 바뀔 때 자연스럽게 이동시키고 싶을 때
- 여러 요소의 애니메이션을 순차적으로 실행하고 싶을 때
- 스크롤 위치에 따라 값을 바꾸고 싶을 때
- React 상태 변화와 애니메이션을 연결하고 싶을 때

이런 상황에서는 CSS만으로 처리하기가 점점 까다로워집니다.

`Framer Motion`은 이런 React 애니메이션 문제를 훨씬 선언적으로 다룰 수 있게 해주는 라이브러리입니다.

이번 글에서는 단순 사용법을 넘어, Framer Motion을 이해하고 실제 프로젝트에서 안정적으로 다루기 위해 알아야 할 핵심 개념들을 정리해보겠습니다.

---

## 📌 Framer Motion과 Motion for React

예전에는 보통 패키지 이름과 import를 이렇게 사용했습니다.

```bash
npm install framer-motion
```

```tsx
import { motion } from "framer-motion";
```

최근 공식 문서에서는 `Motion for React`라는 이름으로 안내되며, 다음 import를 사용합니다.

```bash
npm install motion
```

```tsx
import { motion } from "motion/react";
```

프로젝트에 따라 둘 중 하나를 사용하게 될 수 있습니다.

기존 프로젝트에서 `framer-motion`을 사용하고 있다면 그 문법을 이해하는 것이 중요하고, 새 프로젝트라면 공식 문서 기준인 `motion/react`를 확인하는 것이 좋습니다.

이 글에서는 최신 문서에서 안내하는 `motion/react` import를 기준으로 설명하겠습니다.

---

## 💡 핵심 사고방식: 상태를 애니메이션 값으로 연결하기

Framer Motion을 이해할 때 가장 중요한 생각은 이것입니다.

```txt
React 상태가 바뀐다
→ motion 컴포넌트의 animate 값이 바뀐다
→ Framer Motion이 중간 프레임을 계산한다
→ 화면이 자연스럽게 움직인다
```

일반 React 컴포넌트는 값이 바뀌면 바로 새 스타일을 적용합니다.

```tsx
<div style={{ opacity: isOpen ? 1 : 0 }} />
```

반면 `motion` 컴포넌트는 값이 바뀌는 과정을 애니메이션으로 만들어줍니다.

```tsx
<motion.div animate={{ opacity: isOpen ? 1 : 0 }} />
```

즉, 개발자는 시작값과 끝값을 선언하고, Framer Motion은 그 사이의 움직임을 계산합니다.

---

## 🧱 motion 컴포넌트 기본 사용법

Framer Motion의 가장 기본 API는 `motion` 컴포넌트입니다.

```tsx
import { motion } from "motion/react";

const Box = () => {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.3 }}
    >
      부드럽게 나타나는 박스
    </motion.div>
  );
};
```

여기서 세 가지 prop이 핵심입니다.

| prop | 역할 |
| --- | --- |
| `initial` | 처음 렌더링될 때의 시작 상태 |
| `animate` | 애니메이션이 끝났을 때의 목표 상태 |
| `transition` | 애니메이션 속도, 곡선, 스프링 설정 |

위 코드는 처음에는 투명하고 아래에 있던 요소가, 점점 선명해지면서 원래 위치로 올라오는 애니메이션을 만듭니다.

```txt
opacity: 0 -> 1
y: 20 -> 0
```

---

## 🎚️ transition 이해하기

`transition`은 애니메이션이 어떻게 움직일지 정하는 옵션입니다.

가장 단순하게는 `duration`을 줄 수 있습니다.

```tsx
<motion.div
  animate={{ opacity: 1 }}
  transition={{ duration: 0.4 }}
/>
```

조금 더 자연스러운 움직임을 만들고 싶다면 `ease`를 사용할 수 있습니다.

```tsx
<motion.div
  animate={{ opacity: 1, y: 0 }}
  transition={{
    duration: 0.4,
    ease: "easeOut",
  }}
/>
```

크기나 위치가 움직이는 UI에는 `spring`도 자주 사용합니다.

```tsx
<motion.div
  animate={{ scale: 1 }}
  transition={{
    type: "spring",
    stiffness: 300,
    damping: 24,
  }}
/>
```

`spring`은 정해진 시간 안에 끝나는 애니메이션이라기보다, 물리적인 탄성처럼 움직입니다.

자주 쓰는 기준은 다음과 같습니다.

- `duration`: 일정한 시간 안에 끝나는 단순 애니메이션
- `ease`: 등장, 사라짐, fade 같은 부드러운 전환
- `spring`: 버튼, 카드, 드래그, 레이아웃 이동처럼 물리감이 필요한 움직임

---

## 🎭 variants로 애니메이션 상태 이름 붙이기

`initial`, `animate`에 객체를 직접 넣는 방식은 간단하지만, 애니메이션이 많아지면 코드가 길어집니다.

이때 `variants`를 사용하면 애니메이션 상태에 이름을 붙일 수 있습니다.

```tsx
import { motion } from "motion/react";

const cardVariants = {
  hidden: {
    opacity: 0,
    y: 20,
  },
  visible: {
    opacity: 1,
    y: 0,
  },
};

const Card = () => {
  return (
    <motion.div
      variants={cardVariants}
      initial="hidden"
      animate="visible"
      transition={{ duration: 0.3 }}
    >
      카드
    </motion.div>
  );
};
```

이렇게 하면 애니메이션의 의미가 더 잘 드러납니다.

```txt
hidden 상태에서 visible 상태로 이동한다
```

객체 값만 보는 것보다 읽기 쉽고, 여러 컴포넌트에서 재사용하기도 좋습니다.

---

## 👨‍👩‍👧 부모와 자식 애니메이션 연결하기

`variants`의 좋은 점은 부모와 자식 애니메이션을 연결할 수 있다는 것입니다.

예를 들어 리스트 아이템이 하나씩 순서대로 나타나게 만들 수 있습니다.

```tsx
const listVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.08,
    },
  },
};

const itemVariants = {
  hidden: {
    opacity: 0,
    y: 12,
  },
  visible: {
    opacity: 1,
    y: 0,
  },
};

const AnimatedList = ({ items }: { items: string[] }) => {
  return (
    <motion.ul
      variants={listVariants}
      initial="hidden"
      animate="visible"
    >
      {items.map((item) => (
        <motion.li key={item} variants={itemVariants}>
          {item}
        </motion.li>
      ))}
    </motion.ul>
  );
};
```

`staggerChildren`은 자식 애니메이션을 일정 간격으로 지연시킵니다.

```txt
첫 번째 아이템 등장
0.08초 뒤 두 번째 아이템 등장
0.08초 뒤 세 번째 아이템 등장
```

이런 식으로 리스트, 메뉴, 카드 그리드에 생동감을 줄 수 있습니다.

---

## 🖱️ hover, tap, drag 같은 제스처 다루기

Framer Motion은 사용자 인터랙션에 반응하는 애니메이션도 쉽게 만들 수 있습니다.

```tsx
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: "spring", stiffness: 400, damping: 18 }}
>
  클릭하기
</motion.button>
```

자주 사용하는 제스처 prop은 다음과 같습니다.

| prop | 동작 |
| --- | --- |
| `whileHover` | hover 중일 때 |
| `whileTap` | 누르고 있을 때 |
| `whileFocus` | focus 되었을 때 |
| `whileDrag` | drag 중일 때 |

드래그도 간단히 만들 수 있습니다.

```tsx
<motion.div
  drag
  dragConstraints={{ left: -100, right: 100, top: -50, bottom: 50 }}
  className="h-20 w-20 rounded-full bg-blue-500"
/>
```

다만 드래그는 UX와 접근성까지 함께 고려해야 합니다. 단순 장식이 아니라 실제 기능이라면 키보드나 터치 환경에서도 자연스럽게 동작하는지 확인하는 것이 좋습니다.

---

## 🚪 AnimatePresence로 exit 애니메이션 만들기

React에서 조건부 렌더링으로 컴포넌트를 제거하면, 보통 바로 DOM에서 사라집니다.

```tsx
{isOpen && <Modal />}
```

이 경우 사라지는 애니메이션을 보여줄 시간이 없습니다.

`AnimatePresence`를 사용하면 컴포넌트가 제거되기 전에 `exit` 애니메이션을 실행할 수 있습니다.

```tsx
import { AnimatePresence, motion } from "motion/react";

const ModalExample = ({ isOpen }: { isOpen: boolean }) => {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          key="modal"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.2 }}
        >
          모달 내용
        </motion.div>
      )}
    </AnimatePresence>
  );
};
```

여기서 중요한 점은 `exit`입니다.

```tsx
exit={{ opacity: 0 }}
```

컴포넌트가 사라질 때 바로 제거하지 않고, opacity가 0이 되는 애니메이션을 실행한 뒤 DOM에서 제거합니다.

---

## ⚠️ AnimatePresence에서 자주 하는 실수

`AnimatePresence`가 동작하지 않는 경우는 대부분 구조 문제입니다.

### 1. 조건부 렌더링이 AnimatePresence 안에 있어야 합니다

```tsx
<AnimatePresence>
  {isOpen && <motion.div exit={{ opacity: 0 }} />}
</AnimatePresence>
```

반대로 이렇게 작성하면 exit 애니메이션이 기대처럼 동작하지 않을 수 있습니다.

```tsx
{isOpen && (
  <AnimatePresence>
    <motion.div exit={{ opacity: 0 }} />
  </AnimatePresence>
)}
```

`AnimatePresence` 자체가 함께 사라지면, 자식의 exit를 관리할 수 없기 때문입니다.

### 2. 리스트에서는 key가 중요합니다

리스트에서 요소가 추가되거나 제거될 때는 `key`가 정확해야 합니다.

```tsx
<AnimatePresence>
  {items.map((item) => (
    <motion.li
      key={item.id}
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    >
      {item.label}
    </motion.li>
  ))}
</AnimatePresence>
```

React가 어떤 요소가 사라졌는지 알아야 Framer Motion도 exit 애니메이션을 적용할 수 있습니다.

---

## 📐 layout 애니메이션으로 위치 변화 부드럽게 만들기

리스트 순서가 바뀌거나, 카드 크기가 바뀌거나, 필터링으로 요소 위치가 달라지는 경우가 있습니다.

이때 `layout` prop을 사용하면 레이아웃 변화가 자연스럽게 애니메이션됩니다.

```tsx
{items.map((item) => (
  <motion.div
    key={item.id}
    layout
    className="rounded border p-4"
  >
    {item.title}
  </motion.div>
))}
```

`layout`은 요소의 이전 위치와 다음 위치를 비교해서, 그 사이를 transform 기반 애니메이션으로 연결합니다.

이 기능은 다음 UI에서 특히 유용합니다.

- 필터링되는 카드 목록
- 순서가 바뀌는 리스트
- 펼쳐졌다 접히는 아코디언
- 선택 상태에 따라 크기가 변하는 탭

---

## 🔁 layoutId로 shared layout animation 만들기

`layoutId`를 사용하면 서로 다른 컴포넌트 사이에서도 같은 요소처럼 이어지는 애니메이션을 만들 수 있습니다.

대표적인 예시는 탭 underline입니다.

```tsx
const tabs = ["home", "profile", "settings"];

const TabExample = ({ activeTab }: { activeTab: string }) => {
  return (
    <div className="flex gap-4">
      {tabs.map((tab) => (
        <button key={tab} className="relative px-3 py-2">
          {tab}

          {activeTab === tab && (
            <motion.div
              layoutId="active-tab-indicator"
              className="absolute inset-x-0 bottom-0 h-0.5 bg-blue-500"
            />
          )}
        </button>
      ))}
    </div>
  );
};
```

선택된 탭이 바뀌면 underline 요소는 DOM상으로는 다른 위치에 렌더링됩니다.

하지만 같은 `layoutId`를 가지고 있기 때문에 Framer Motion은 이전 위치에서 다음 위치로 이어지는 애니메이션을 만들어줍니다.

이런 패턴은 탭, 카드 상세 전환, 이미지 갤러리, 선택 표시 UI에서 자주 쓰입니다.

---

## 🌊 MotionValue 이해하기

`MotionValue`는 React state와 조금 다릅니다.

React state가 바뀌면 컴포넌트가 다시 렌더링됩니다.

```tsx
const [x, setX] = useState(0);
```

반면 MotionValue는 값이 바뀌어도 React 리렌더링을 직접 일으키지 않고, 애니메이션 값으로 효율적으로 업데이트됩니다.

```tsx
import { motion, useMotionValue } from "motion/react";

const MotionValueExample = () => {
  const x = useMotionValue(0);

  return (
    <motion.div
      drag="x"
      style={{ x }}
      className="h-20 w-20 rounded bg-blue-500"
    />
  );
};
```

`x` 값은 드래그에 따라 계속 바뀌지만, 매 프레임 React 컴포넌트가 다시 렌더링되는 방식은 아닙니다.

그래서 스크롤, 드래그, 포인터 이동처럼 값이 자주 바뀌는 애니메이션에 적합합니다.

---

## 🔄 useTransform으로 값 변환하기

`useTransform`은 하나의 MotionValue를 다른 값으로 변환할 때 사용합니다.

예를 들어 x 위치에 따라 opacity를 바꿔보겠습니다.

```tsx
import { motion, useMotionValue, useTransform } from "motion/react";

const TransformExample = () => {
  const x = useMotionValue(0);
  const opacity = useTransform(x, [-100, 0, 100], [0.2, 1, 0.2]);

  return (
    <motion.div
      drag="x"
      style={{ x, opacity }}
      className="h-20 w-20 rounded bg-blue-500"
    />
  );
};
```

이 코드는 다음 의미입니다.

```txt
x가 -100이면 opacity는 0.2
x가 0이면 opacity는 1
x가 100이면 opacity는 0.2
```

스크롤 진행률에 따라 opacity, scale, y 값을 바꾸는 UI도 같은 방식으로 만들 수 있습니다.

---

## 📜 useScroll로 스크롤 애니메이션 만들기

`useScroll`은 스크롤 위치를 MotionValue로 제공합니다.

```tsx
import { motion, useScroll } from "motion/react";

const ScrollProgress = () => {
  const { scrollYProgress } = useScroll();

  return (
    <motion.div
      style={{ scaleX: scrollYProgress }}
      className="fixed left-0 top-0 h-1 origin-left bg-blue-500"
    />
  );
};
```

`scrollYProgress`는 전체 페이지 스크롤 진행률을 `0`부터 `1` 사이의 값으로 표현합니다.

```txt
페이지 맨 위: 0
페이지 중간: 0.5
페이지 끝: 1
```

이 값을 `scaleX`에 연결하면 상단 진행 바를 만들 수 있습니다.

---

## 🎮 useAnimationControls로 명령형 애니메이션 실행하기

대부분의 경우에는 `animate` prop만으로 충분합니다.

하지만 특정 이벤트 이후 순서대로 애니메이션을 실행해야 할 때는 `useAnimationControls`를 사용할 수 있습니다.

```tsx
import { motion, useAnimationControls } from "motion/react";

const ControlsExample = () => {
  const controls = useAnimationControls();

  const handleClick = async () => {
    await controls.start({ x: 100 });
    await controls.start({ rotate: 180 });
    await controls.start({ x: 0, rotate: 0 });
  };

  return (
    <div>
      <button type="button" onClick={handleClick}>
        애니메이션 실행
      </button>

      <motion.div
        animate={controls}
        className="h-20 w-20 rounded bg-blue-500"
      />
    </div>
  );
};
```

이 방식은 애니메이션을 명령형으로 제어할 수 있다는 장점이 있습니다.

다만 너무 많이 사용하면 코드 흐름이 복잡해질 수 있습니다. 단순한 상태 전환은 `animate`, 순차 실행이나 외부 이벤트 기반 제어는 `useAnimationControls`처럼 나누어 생각하면 좋습니다.

---

## 🧑‍🦽 접근성: reduced motion 고려하기

모든 사용자가 애니메이션을 편하게 느끼는 것은 아닙니다.

어떤 사용자는 움직임이 많은 UI에서 어지러움이나 불편함을 느낄 수 있습니다. 그래서 `prefers-reduced-motion` 설정을 고려하는 것이 좋습니다.

Framer Motion에서는 `useReducedMotion`을 사용할 수 있습니다.

```tsx
import { motion, useReducedMotion } from "motion/react";

const AccessibleCard = () => {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      initial={{ opacity: 0, y: shouldReduceMotion ? 0 : 20 }}
      animate={{ opacity: 1, y: 0 }}
    >
      접근성을 고려한 카드
    </motion.div>
  );
};
```

또는 `MotionConfig`로 하위 컴포넌트 전체에 reduced motion 정책을 적용할 수도 있습니다.

```tsx
import { MotionConfig } from "motion/react";

const App = () => {
  return (
    <MotionConfig reducedMotion="user">
      <Page />
    </MotionConfig>
  );
};
```

애니메이션은 UI를 더 좋게 만들 수 있지만, 사용자가 불편함을 느끼지 않도록 조절할 수 있어야 합니다.

---

## ⚡ 성능을 위해 기억할 점

Framer Motion을 사용할 때도 CSS 애니메이션과 마찬가지로 성능을 고려해야 합니다.

가장 안전한 속성은 보통 `transform`과 `opacity`입니다.

```tsx
<motion.div animate={{ opacity: 1, y: 0, scale: 1 }} />
```

반대로 `width`, `height`, `top`, `left` 같은 속성은 레이아웃 계산을 유발할 수 있습니다.

```tsx
// 신중하게 사용해야 하는 예시
<motion.div animate={{ width: 300 }} />
```

가능하면 다음처럼 생각하는 것이 좋습니다.

```txt
위치 이동
→ x, y, translate 사용

크기 변화
→ scale 또는 layout 애니메이션 고려

등장/사라짐
→ opacity 사용
```

물론 `layout` prop은 레이아웃 변화를 애니메이션하기 위해 만들어진 기능이므로, 필요한 곳에서는 적극적으로 사용할 수 있습니다. 다만 아주 많은 리스트 아이템에 동시에 적용하면 비용이 커질 수 있으니 실제 화면 규모를 보면서 적용하는 것이 좋습니다.

---

## 🧩 Next.js에서 사용할 때

Next.js App Router에서 Framer Motion을 사용할 때는 클라이언트 컴포넌트에서 사용하는 것이 일반적입니다.

```tsx
"use client";

import { motion } from "motion/react";

const AnimatedSection = () => {
  return (
    <motion.section
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
    >
      애니메이션 섹션
    </motion.section>
  );
};
```

`motion` 컴포넌트는 브라우저에서 동작하는 애니메이션과 연결되기 때문에, 서버 컴포넌트 안에서 직접 사용하기보다는 별도의 클라이언트 컴포넌트로 분리하는 편이 안전합니다.

---

## 🚧 자주 만나는 문제 정리

### 1. exit 애니메이션이 실행되지 않아요

대부분 `AnimatePresence` 위치가 잘못되었거나 `key`가 없는 경우입니다.

조건부 렌더링은 `AnimatePresence` 안에 두고, 리스트라면 고유한 `key`를 꼭 넣어야 합니다.

### 2. 애니메이션이 너무 과해 보여요

duration을 줄이거나, 움직이는 거리를 줄여보는 것이 좋습니다.

```tsx
initial={{ opacity: 0, y: 8 }}
animate={{ opacity: 1, y: 0 }}
transition={{ duration: 0.2 }}
```

실무 UI에서는 큰 움직임보다 짧고 작은 움직임이 더 자연스럽게 느껴지는 경우가 많습니다.

### 3. 리렌더링이 많아질까 걱정돼요

`motion` 컴포넌트의 애니메이션 값은 매 프레임 React state를 바꾸는 방식과 다르게 동작합니다.

특히 `MotionValue`를 사용하면 자주 변하는 값을 React 리렌더링 없이 애니메이션 값으로 다룰 수 있습니다.

### 4. layout 애니메이션이 이상하게 튀어요

레이아웃이 바뀌는 요소에 `layout`을 붙였는지 확인해야 합니다.

또 이미지처럼 크기가 늦게 결정되는 요소가 있다면, 이미지 크기를 미리 지정하거나 레이아웃이 안정된 뒤 애니메이션되도록 구조를 잡는 것이 좋습니다.

---

## ✅ 정리

Framer Motion을 잘 다루려면 단순히 `motion.div`를 외우는 것보다, 어떤 문제를 어떤 API로 풀어야 하는지 이해하는 것이 중요합니다.

핵심을 정리하면 다음과 같습니다.

- `motion` 컴포넌트는 React 요소에 애니메이션 기능을 더합니다.
- `initial`, `animate`, `exit`는 시작, 목표, 종료 상태를 표현합니다.
- `transition`은 애니메이션의 속도와 움직임 방식을 정합니다.
- `variants`는 애니메이션 상태에 이름을 붙이고 재사용하기 좋습니다.
- `AnimatePresence`는 컴포넌트가 사라지기 전 exit 애니메이션을 가능하게 합니다.
- `layout`은 위치나 크기 변화가 생길 때 자연스럽게 이어줍니다.
- `layoutId`는 서로 다른 위치의 요소를 하나의 흐름처럼 연결합니다.
- `MotionValue`, `useTransform`, `useScroll`은 자주 변하는 애니메이션 값을 효율적으로 다룰 수 있게 해줍니다.
- `useReducedMotion`과 `MotionConfig`로 접근성도 함께 고려해야 합니다.

애니메이션은 UI를 화려하게 만드는 장식만은 아닙니다.

상태 변화가 어디서 일어났는지 알려주고, 사용자가 화면의 흐름을 이해하게 돕는 중요한 피드백이 될 수 있습니다.

Framer Motion을 사용할 때도 이 기준을 기억하면 좋습니다.

```txt
움직임이 기능을 설명하는가?
사용자의 흐름을 더 자연스럽게 만드는가?
불필요하게 시선을 빼앗지는 않는가?
```

이 질문에 답하면서 적용하면, Framer Motion은 단순한 애니메이션 라이브러리를 넘어 React UI의 완성도를 높여주는 좋은 도구가 됩니다.
