---
title: "[React] DOM 위치를 측정해 UI 동작 만들기"
date: 2026-06-01T11:30:00Z
categories: [frontend]
tags: [react, javascript, dom, getboundingclientrect, ui]
description: "getBoundingClientRect와 useRef를 사용해 DOM 요소의 위치와 크기를 측정하고, 스크롤 UI와 팝오버 같은 인터랙션에 활용하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 화면 위에서 요소가 어디에 있는지 알아야 할 때

프론트엔드 UI를 만들다 보면 단순히 데이터를 렌더링하는 것만으로는 부족할 때가 있습니다.

예를 들어 이런 상황입니다.

- 버튼 바로 아래에 팝오버를 띄우고 싶을 때
- 특정 요소가 화면에 들어왔는지 확인하고 싶을 때
- 스크롤 위치에 따라 헤더 스타일을 바꾸고 싶을 때
- 툴팁을 요소 옆에 정확히 붙이고 싶을 때
- 무한 스크롤 트리거가 화면 근처에 왔는지 판단하고 싶을 때

이런 기능들은 결국 공통적으로 한 가지 질문에서 시작합니다.

```txt
이 DOM 요소가 지금 화면의 어디에 있지?
```

이때 사용할 수 있는 대표적인 브라우저 API가 `getBoundingClientRect()`입니다.

이번 글에서는 `getBoundingClientRect()`로 DOM 요소의 위치와 크기를 측정하고, React에서 `useRef`와 함께 UI 동작을 만드는 방법을 정리해보겠습니다.

---

## 💡 getBoundingClientRect란?

`getBoundingClientRect()`는 DOM 요소의 크기와 위치 정보를 반환하는 메서드입니다.

```ts
const rect = element.getBoundingClientRect();
```

반환되는 값에는 다음과 같은 정보가 들어 있습니다.

```ts
const rect = element.getBoundingClientRect();

console.log(rect.top);
console.log(rect.left);
console.log(rect.width);
console.log(rect.height);
console.log(rect.bottom);
console.log(rect.right);
```

각 값은 대략 이렇게 이해하면 됩니다.

| 값 | 의미 |
| --- | --- |
| `top` | viewport 위쪽에서 요소의 위쪽까지의 거리 |
| `left` | viewport 왼쪽에서 요소의 왼쪽까지의 거리 |
| `width` | 요소의 너비 |
| `height` | 요소의 높이 |
| `bottom` | viewport 위쪽에서 요소의 아래쪽까지의 거리 |
| `right` | viewport 왼쪽에서 요소의 오른쪽까지의 거리 |

여기서 중요한 기준은 **viewport**입니다.

viewport는 현재 브라우저에서 실제로 보이는 화면 영역을 의미합니다. 즉, `top`은 문서 전체 기준 위치가 아니라 현재 화면 위쪽 기준 위치입니다.

---

## 📌 top 값은 스크롤에 따라 바뀝니다

`getBoundingClientRect().top`은 viewport 기준 값이기 때문에 스크롤하면 계속 바뀝니다.

예를 들어 어떤 요소가 문서에서 아래쪽에 있다고 해볼게요.

```ts
const rect = element.getBoundingClientRect();

console.log(rect.top);
```

처음에는 `top`이 `800`일 수 있습니다.

```txt
rect.top = 800
```

사용자가 아래로 스크롤하면 요소가 화면 위쪽에 가까워지기 때문에 `top` 값은 줄어듭니다.

```txt
rect.top = 300
rect.top = 100
rect.top = 0
rect.top = -50
```

요소가 화면 위로 지나가면 `top`은 음수가 될 수도 있습니다.

이 특성을 이용하면 요소가 화면 안에 들어왔는지 판단할 수 있습니다.

```ts
const isVisible = rect.top < window.innerHeight && rect.bottom > 0;
```

이 조건은 요소의 일부라도 viewport 안에 들어와 있는지 확인하는 기본적인 방식입니다.

---

## 🧭 문서 전체 기준 위치가 필요할 때

가끔은 viewport 기준 위치가 아니라 문서 전체 기준 위치가 필요할 때도 있습니다.

이때는 `rect.top`에 현재 스크롤 위치인 `window.scrollY`를 더하면 됩니다.

```ts
const rect = element.getBoundingClientRect();
const documentTop = rect.top + window.scrollY;
```

이 값은 문서 맨 위에서부터 해당 요소까지의 거리입니다.

```txt
문서 기준 top = viewport 기준 top + 현재 스크롤 위치
```

정리하면 이렇게 나눌 수 있습니다.

```txt
화면 기준 위치가 필요하다
→ getBoundingClientRect().top

문서 전체 기준 위치가 필요하다
→ getBoundingClientRect().top + window.scrollY
```

---

## 🛠️ React에서 useRef로 요소 측정하기

React에서는 DOM 요소에 직접 접근할 때 `useRef`를 사용합니다.

```tsx
import { useRef } from "react";

const Example = () => {
  const boxRef = useRef<HTMLDivElement | null>(null);

  const handleMeasure = () => {
    const box = boxRef.current;

    if (!box) return;

    const rect = box.getBoundingClientRect();

    console.log(rect.top, rect.left, rect.width, rect.height);
  };

  return (
    <div>
      <button type="button" onClick={handleMeasure}>
        위치 측정하기
      </button>

      <div ref={boxRef}>측정할 요소</div>
    </div>
  );
};
```

핵심은 `ref`를 DOM 요소에 연결하고, 필요한 시점에 `ref.current`에서 DOM을 꺼내는 것입니다.

```tsx
const box = boxRef.current;

if (!box) return;

const rect = box.getBoundingClientRect();
```

이렇게 하면 React 컴포넌트 안에서도 브라우저가 계산한 실제 위치와 크기를 읽을 수 있습니다.

---

## 👀 예시 1: 요소가 화면에 들어왔는지 확인하기

가장 기본적인 활용은 특정 요소가 화면에 보이는지 확인하는 것입니다.

```tsx
import { useEffect, useRef, useState } from "react";

const VisibilityBox = () => {
  const targetRef = useRef<HTMLDivElement | null>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const checkVisibility = () => {
      const target = targetRef.current;

      if (!target) return;

      const rect = target.getBoundingClientRect();

      const nextIsVisible =
        rect.top < window.innerHeight && rect.bottom > 0;

      setIsVisible(nextIsVisible);
    };

    checkVisibility();

    window.addEventListener("scroll", checkVisibility);
    window.addEventListener("resize", checkVisibility);

    return () => {
      window.removeEventListener("scroll", checkVisibility);
      window.removeEventListener("resize", checkVisibility);
    };
  }, []);

  return (
    <div>
      <div className="h-[800px]" />

      <div ref={targetRef}>
        {isVisible ? "화면에 보이고 있어요" : "아직 보이지 않아요"}
      </div>
    </div>
  );
};
```

이 예시는 스크롤이나 화면 크기 변화가 있을 때마다 요소의 위치를 다시 측정합니다.

다만 요소가 보이는지 판단하는 용도라면 실제 서비스에서는 `IntersectionObserver`가 더 적합할 때가 많습니다. `getBoundingClientRect()`는 직접 위치를 계산해야 하는 UI에서 더 빛을 발합니다.

---

## 🧩 예시 2: 버튼 아래에 팝오버 띄우기

`getBoundingClientRect()`는 팝오버나 툴팁 위치를 계산할 때도 자주 사용합니다.

버튼의 위치를 측정한 뒤, 그 아래에 팝오버를 배치해보겠습니다.

```tsx
import { useRef, useState } from "react";

interface PopoverPosition {
  top: number;
  left: number;
}

const PopoverExample = () => {
  const buttonRef = useRef<HTMLButtonElement | null>(null);
  const [position, setPosition] = useState<PopoverPosition | null>(null);

  const openPopover = () => {
    const button = buttonRef.current;

    if (!button) return;

    const rect = button.getBoundingClientRect();

    setPosition({
      top: rect.bottom + window.scrollY + 8,
      left: rect.left + window.scrollX,
    });
  };

  return (
    <div className="p-10">
      <button
        ref={buttonRef}
        type="button"
        onClick={openPopover}
        className="rounded bg-blue-600 px-4 py-2 text-white"
      >
        메뉴 열기
      </button>

      {position && (
        <div
          className="absolute rounded border bg-white p-3 shadow"
          style={{
            top: position.top,
            left: position.left,
          }}
        >
          버튼 아래에 표시되는 팝오버입니다.
        </div>
      )}
    </div>
  );
};
```

여기서는 `rect.bottom`을 사용했습니다.

```ts
top: rect.bottom + window.scrollY + 8;
```

`rect.bottom`은 viewport 기준으로 버튼의 아래쪽 위치입니다. 그런데 팝오버는 `absolute`로 문서 위에 배치하고 있으므로 `window.scrollY`를 더해 문서 기준 위치로 바꿔줍니다.

`8`은 버튼과 팝오버 사이의 간격입니다.

---

## 📐 useEffect와 useLayoutEffect 중 무엇을 써야 할까요?

DOM 측정은 렌더링 타이밍과 관련이 있습니다.

대부분의 경우에는 `useEffect`로도 충분합니다.

```tsx
useEffect(() => {
  const rect = element.getBoundingClientRect();
}, []);
```

하지만 측정한 값을 바로 레이아웃에 반영해야 하고, 사용자가 중간 상태를 보면 안 되는 경우라면 `useLayoutEffect`가 더 적합할 수 있습니다.

```tsx
useLayoutEffect(() => {
  const rect = element.getBoundingClientRect();
  setPosition(rect);
}, []);
```

차이를 간단히 정리하면 다음과 같습니다.

| Hook | 실행 시점 | 어울리는 상황 |
| --- | --- | --- |
| `useEffect` | 브라우저가 화면을 그린 뒤 | 일반적인 이벤트 등록, 가벼운 측정 |
| `useLayoutEffect` | 화면이 그려지기 전 | 측정 결과가 레이아웃에 바로 영향을 줄 때 |

단, `useLayoutEffect`는 화면 그리기를 지연시킬 수 있기 때문에 꼭 필요한 경우에만 사용하는 것이 좋습니다.

---

## ⚠️ 사용할 때 주의할 점

### 1. 너무 자주 측정하면 성능에 부담이 됩니다

`getBoundingClientRect()`는 브라우저가 계산한 레이아웃 정보를 읽습니다.

스크롤 이벤트에서 매번 많은 요소를 측정하면 성능에 부담이 될 수 있습니다. 필요한 요소만 측정하고, 너무 자주 실행된다면 `throttle`, `requestAnimationFrame`, `IntersectionObserver` 같은 방식도 함께 고려하는 것이 좋습니다.

### 2. 측정 전에 DOM이 존재하는지 확인해야 합니다

React에서는 처음 렌더링 시점에 `ref.current`가 아직 `null`일 수 있습니다.

```tsx
const element = targetRef.current;

if (!element) return;
```

이 확인을 빼먹으면 런타임 에러가 발생할 수 있습니다.

### 3. viewport 기준인지 문서 기준인지 구분해야 합니다

`getBoundingClientRect()`의 값은 viewport 기준입니다.

팝오버처럼 문서 위에 `absolute`로 배치해야 하는 UI라면 `window.scrollY`, `window.scrollX`를 더해야 할 수 있습니다.

```ts
const top = rect.bottom + window.scrollY;
const left = rect.left + window.scrollX;
```

이 기준을 헷갈리면 스크롤된 상태에서 팝오버나 툴팁 위치가 어긋나기 쉽습니다.

---

## ✅ 정리

`getBoundingClientRect()`는 DOM 요소의 현재 위치와 크기를 읽을 수 있는 브라우저 API입니다.

React에서는 `useRef`로 DOM 요소를 참조한 뒤, 필요한 시점에 `getBoundingClientRect()`를 호출해 UI 동작을 만들 수 있습니다.

핵심은 다음과 같습니다.

- `top`, `left`, `bottom`, `right`는 viewport 기준 값입니다.
- 문서 전체 기준 위치가 필요하면 `window.scrollY`, `window.scrollX`를 더합니다.
- React에서는 `useRef`로 DOM 요소에 접근합니다.
- 측정 결과가 레이아웃에 바로 영향을 주면 `useLayoutEffect`를 고려할 수 있습니다.
- 스크롤 중 너무 자주 측정하면 성능 문제가 생길 수 있습니다.

단순히 요소의 top 거리를 구하는 것에서 끝나지 않고, DOM 위치 측정은 팝오버, 툴팁, 스크롤 UI처럼 화면과 밀접한 인터랙션을 만드는 데 자주 쓰입니다.

브라우저가 이미 계산해둔 위치 정보를 잘 활용하면, UI를 훨씬 더 자연스럽고 정확하게 제어할 수 있습니다.
