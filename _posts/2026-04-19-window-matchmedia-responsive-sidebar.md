---
title: "[React] window.matchMedia로 반응형 사이드바 구현하기"
date: 2026-04-17T13:48:00Z
categories: [frontend]
tags: [javascript, matchmedia, responsive-design, sidebar-optimization]
description: "resize 이벤트 대신 window.matchMedia를 사용해 특정 브레이크포인트에서만 반응하는 사이드바를 구현한 과정을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: resize 이벤트로 사이드바를 제어하면 아쉬운 점

반응형 사이드바를 만들 때 흔히 이런 요구사항을 만납니다.

화면이 `1024px`보다 좁아지면 사이드바를 접고, 다시 넓어지면 펼치는 기능입니다.

처음에는 자연스럽게 `resize` 이벤트를 떠올릴 수 있습니다.

```tsx
window.addEventListener("resize", handleResize);
```

하지만 이 방식은 몇 가지 불편함이 있습니다.

- 창 크기가 1px만 바뀌어도 이벤트가 계속 실행됩니다.
- `debounce`나 `throttle` 같은 추가 최적화가 필요해질 수 있습니다.
- CSS에서는 미디어 쿼리를 쓰고, JavaScript에서는 `window.innerWidth`를 비교하다 보면 기준이 흩어집니다.

이럴 때 더 잘 어울리는 API가 `window.matchMedia`입니다.

---

## 💡 window.matchMedia란?

`window.matchMedia`는 CSS 미디어 쿼리를 JavaScript에서 사용할 수 있게 해주는 브라우저 API입니다.

```tsx
const mediaQuery = window.matchMedia("(max-width: 1023px)");

console.log(mediaQuery.matches);
```

`matches` 값은 현재 화면이 해당 미디어 쿼리 조건을 만족하는지 알려줍니다.

그리고 더 좋은 점은 조건이 바뀌는 순간에만 `change` 이벤트가 발생한다는 점입니다. 창을 계속 드래그하더라도 `1023px` 기준을 넘나드는 순간에만 로직을 실행할 수 있어요.

---

## 🛠️ 반응형 사이드바에 적용하기

아래 코드는 화면 너비가 `1024px` 미만이면 사이드바를 접고, 그 이상이면 다시 펼치는 예시입니다.

```tsx
import { useEffect, useState } from "react";

const SidebarLayout = () => {
  const [isFolded, setIsFolded] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia("(max-width: 1023px)");

    const updateSidebarState = (
      event: MediaQueryListEvent | MediaQueryList,
    ) => {
      // 조건을 만족하면 모바일/태블릿 영역으로 보고 사이드바를 접습니다.
      setIsFolded(event.matches);
    };

    // 첫 렌더링 시점에도 현재 화면 크기를 반영합니다.
    updateSidebarState(mediaQuery);

    mediaQuery.addEventListener("change", updateSidebarState);

    return () => {
      mediaQuery.removeEventListener("change", updateSidebarState);
    };
  }, []);

  return (
    <aside data-folded={isFolded}>
      {isFolded ? "접힌 사이드바" : "펼쳐진 사이드바"}
    </aside>
  );
};
```

핵심은 `window.innerWidth`를 직접 비교하지 않는 것입니다.

```tsx
const mediaQuery = window.matchMedia("(max-width: 1023px)");
```

CSS에서 쓰던 미디어 쿼리 기준을 JavaScript에서도 그대로 사용하니, 반응형 기준을 맞추기가 훨씬 쉬워집니다.

---

## 📌 resize 이벤트와 무엇이 다를까요?

`resize` 이벤트는 창 크기가 바뀌는 동안 계속 발생합니다.

반면 `matchMedia`의 `change` 이벤트는 미디어 쿼리의 참/거짓 상태가 바뀔 때만 실행됩니다.

예를 들어 기준이 `1024px`이라면 다음과 같이 동작합니다.

```txt
1200px -> 1100px: 실행 안 됨
1100px -> 1020px: 실행됨
1020px -> 900px: 실행 안 됨
900px -> 1040px: 실행됨
```

즉, 사이드바처럼 특정 브레이크포인트를 기준으로 UI 모드를 바꾸는 기능에는 `resize`보다 `matchMedia`가 더 적합합니다.

---

## 📊 비교 정리

| 항목 | resize + innerWidth | window.matchMedia |
| --- | --- | --- |
| 실행 빈도 | 창 크기가 바뀌는 동안 계속 실행 | 조건이 바뀔 때만 실행 |
| 기준 관리 | JS에서 숫자를 직접 비교 | CSS 미디어 쿼리와 같은 문법 사용 |
| 최적화 | debounce/throttle이 필요할 수 있음 | 상대적으로 단순함 |
| 어울리는 상황 | 실시간 크기 계산 | 반응형 모드 전환 |

실시간으로 차트 크기를 다시 계산해야 한다면 `resize`나 `ResizeObserver`가 더 알맞을 수 있습니다.

하지만 사이드바, 내비게이션, 모바일 레이아웃 전환처럼 특정 기준을 넘는지만 알면 되는 경우에는 `matchMedia`가 깔끔합니다.

---

## ⚠️ 주의할 점

`window` 객체를 사용하기 때문에 Next.js 같은 SSR 환경에서는 클라이언트에서만 실행되도록 해야 합니다.

보통 `useEffect` 안에서 사용하면 서버 렌더링 중에는 실행되지 않기 때문에 안전합니다.

또 아주 오래된 브라우저를 고려해야 한다면 `addEventListener("change", ...)` 대신 `addListener`를 확인해야 할 수도 있습니다. 최신 브라우저 기준이라면 `addEventListener`를 사용하면 됩니다.

---

## ✅ 마무리

반응형 UI를 만들 때 모든 상황을 `resize` 이벤트로 처리할 필요는 없습니다.

특정 브레이크포인트를 기준으로 UI 상태를 바꾸는 정도라면 `window.matchMedia`가 더 단순하고 명확합니다.

CSS 미디어 쿼리와 같은 기준을 JavaScript에서도 사용할 수 있기 때문에, 사이드바처럼 화면 크기에 따라 모드가 바뀌는 컴포넌트를 구현할 때 특히 유용합니다.
