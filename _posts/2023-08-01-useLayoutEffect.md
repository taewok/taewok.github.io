---
title: "[React] useLayoutEffect는 언제 사용해야 할까?"
date: 2023-08-01T15:19:00+09:00
categories: [frontend]
tags: [react, hooks, uselayouteffect, useeffect]
description: "React의 useLayoutEffect가 useEffect와 어떻게 다르고, DOM 측정이나 레이아웃 보정이 필요한 상황에서 언제 사용하면 좋은지 정리했습니다."
custom_style: true
---

## 🧐 들어가며: useEffect와 비슷한데 왜 따로 있을까요?

React를 사용하다 보면 대부분의 사이드 이펙트는 `useEffect`로 처리합니다.

API 요청, 이벤트 리스너 등록, 타이머 설정, 외부 라이브러리 연동 같은 작업은 보통 `useEffect`만으로 충분합니다.

그런데 React에는 `useEffect`와 이름이 비슷한 `useLayoutEffect`라는 훅도 있습니다.

처음 보면 이런 생각이 들 수 있어요.

```txt
useEffect가 있는데 useLayoutEffect는 왜 필요하지?
둘 다 렌더링 이후에 실행되는 것 아닌가?
```

둘 다 렌더링 이후에 실행되는 훅이라는 점은 비슷합니다. 하지만 실행되는 타이밍이 다릅니다.

이 차이 때문에 `useLayoutEffect`는 DOM의 크기나 위치를 측정하고, 화면이 그려지기 전에 레이아웃을 보정해야 하는 상황에서 유용합니다.

---

## 💡 useLayoutEffect란?

`useLayoutEffect`는 React 컴포넌트가 DOM에 반영된 직후, 브라우저가 화면을 그리기 전에 실행되는 훅입니다.

조금 더 쉽게 말하면 이런 흐름입니다.

```txt
1. React가 컴포넌트를 렌더링합니다.
2. 변경된 결과가 DOM에 반영됩니다.
3. useLayoutEffect가 실행됩니다.
4. 브라우저가 화면을 그립니다.
```

반면 `useEffect`는 브라우저가 화면을 그린 뒤에 실행됩니다.

```txt
1. React가 컴포넌트를 렌더링합니다.
2. 변경된 결과가 DOM에 반영됩니다.
3. 브라우저가 화면을 그립니다.
4. useEffect가 실행됩니다.
```

즉, `useLayoutEffect`는 화면이 사용자에게 보이기 전에 실행되고, `useEffect`는 화면이 보인 뒤에 실행됩니다.

이 차이가 작아 보이지만, DOM 측정이나 위치 보정이 필요한 UI에서는 꽤 중요합니다.

---

## 🔍 useEffect와 useLayoutEffect의 차이

두 훅의 가장 큰 차이는 실행 타이밍입니다.

| 항목 | useEffect | useLayoutEffect |
| :--- | :--- | :--- |
| 실행 시점 | 브라우저가 화면을 그린 뒤 | DOM 반영 직후, 화면을 그리기 전 |
| 화면 깜빡임 | 레이아웃 보정이 늦으면 보일 수 있음 | 화면에 보이기 전에 보정 가능 |
| 적합한 작업 | API 요청, 이벤트 등록, 로그, 타이머 | DOM 크기 측정, 스크롤 위치 보정, 레이아웃 조정 |
| 성능 영향 | 상대적으로 적음 | 화면 그리기를 막을 수 있음 |

정리하면 `useEffect`는 대부분의 사이드 이펙트에 적합하고, `useLayoutEffect`는 **화면이 그려지기 전에 반드시 처리해야 하는 DOM 관련 작업**에 적합합니다.

---

## 🛠️ 기본 사용법

사용법은 `useEffect`와 거의 같습니다.

```tsx
import { useLayoutEffect } from "react";

useLayoutEffect(() => {
  // DOM이 반영된 직후 실행됩니다.
  // 브라우저가 화면을 그리기 전에 실행된다는 점이 useEffect와 다릅니다.

  return () => {
    // cleanup이 필요하다면 여기서 정리합니다.
  };
}, []);
```

의존성 배열을 사용하는 방식도 `useEffect`와 같습니다.

```tsx
useLayoutEffect(() => {
  // value가 바뀔 때마다 실행됩니다.
}, [value]);
```

---

## 📏 예시 1. DOM 크기 측정하기

`useLayoutEffect`가 가장 잘 어울리는 상황은 DOM 크기나 위치를 측정해야 할 때입니다.

예를 들어 어떤 박스의 높이를 측정해서 화면에 보여줘야 한다고 해볼게요.

```tsx
import { useLayoutEffect, useRef, useState } from "react";

export default function BoxMeasure() {
  const boxRef = useRef<HTMLDivElement | null>(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    if (!boxRef.current) return;

    // DOM이 실제로 배치된 뒤 높이를 읽습니다.
    const nextHeight = boxRef.current.getBoundingClientRect().height;

    setHeight(nextHeight);
  }, []);

  return (
    <div>
      <div ref={boxRef} className="p-4 text-white bg-zinc-800">
        높이를 측정할 박스입니다.
      </div>

      <p>박스 높이: {height}px</p>
    </div>
  );
}
```

여기서 `getBoundingClientRect()`는 실제 DOM의 크기와 위치를 읽는 메서드입니다.

이런 측정은 DOM이 실제로 화면에 배치된 뒤에야 정확합니다. 그래서 렌더링 이전이 아니라 렌더링 이후에 실행되는 effect가 필요합니다.

다만 측정 결과를 바로 레이아웃 보정에 사용해야 한다면 `useLayoutEffect`가 더 적합합니다.

---

## 🧭 예시 2. 화면이 보이기 전에 위치 보정하기

툴팁이나 드롭다운을 만들 때는 먼저 DOM을 렌더링한 뒤, 실제 크기를 측정해서 위치를 조정해야 하는 경우가 많습니다.

예를 들어 툴팁이 화면 오른쪽을 벗어나면 왼쪽으로 옮겨야 할 수 있습니다.

```tsx
import { useLayoutEffect, useRef, useState } from "react";

export default function Tooltip() {
  const tooltipRef = useRef<HTMLDivElement | null>(null);
  const [left, setLeft] = useState(0);

  useLayoutEffect(() => {
    if (!tooltipRef.current) return;

    const rect = tooltipRef.current.getBoundingClientRect();
    const padding = 16;

    // 툴팁이 화면 오른쪽을 벗어나면 안쪽으로 밀어 넣습니다.
    if (rect.right > window.innerWidth - padding) {
      setLeft(window.innerWidth - rect.width - padding);
      return;
    }

    setLeft(rect.left);
  }, []);

  return (
    <div
      ref={tooltipRef}
      style={{ left }}
      className="fixed top-10 rounded bg-zinc-900 px-3 py-2 text-white"
    >
      툴팁 내용입니다.
    </div>
  );
}
```

만약 이 보정을 `useEffect`에서 처리하면, 처음에는 잘못된 위치의 툴팁이 잠깐 보이고 이후에 위치가 바뀔 수 있습니다.

반면 `useLayoutEffect`를 사용하면 브라우저가 화면을 그리기 전에 위치를 보정할 수 있어서 깜빡임을 줄일 수 있습니다.

---

## 🔄 예시 3. 스크롤 위치를 즉시 보정하기

채팅창이나 리스트 UI에서도 `useLayoutEffect`가 유용할 때가 있습니다.

예를 들어 메시지가 추가된 뒤 스크롤을 바로 아래로 내려야 한다고 해볼게요.

```tsx
import { useLayoutEffect, useRef } from "react";

interface ChatListProps {
  messages: string[];
}

export default function ChatList({ messages }: ChatListProps) {
  const scrollRef = useRef<HTMLDivElement | null>(null);

  useLayoutEffect(() => {
    if (!scrollRef.current) return;

    // 메시지가 DOM에 반영된 직후 스크롤 위치를 보정합니다.
    scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
  }, [messages]);

  return (
    <div ref={scrollRef} className="h-80 overflow-y-auto">
      {messages.map((message, index) => (
        <div key={index}>{message}</div>
      ))}
    </div>
  );
}
```

이처럼 DOM이 업데이트된 직후 곧바로 스크롤 위치를 맞춰야 할 때 `useLayoutEffect`를 사용할 수 있습니다.

다만 스크롤 애니메이션처럼 브라우저 프레임과 맞추는 작업이라면 `requestAnimationFrame`을 함께 고려하는 것도 좋습니다.

---

## ⚠️ useLayoutEffect를 남용하면 안 되는 이유

`useLayoutEffect`는 브라우저가 화면을 그리기 전에 실행됩니다.

이 말은 반대로, `useLayoutEffect` 안의 작업이 오래 걸리면 화면이 그려지는 시점도 늦어진다는 뜻입니다.

```txt
useLayoutEffect 작업이 오래 걸림
→ 브라우저 페인트가 지연됨
→ 사용자는 화면이 늦게 뜨는 것처럼 느낌
```

그래서 `useLayoutEffect`는 꼭 필요한 경우에만 사용하는 것이 좋습니다.

대부분의 작업은 `useEffect`로 충분합니다.

예를 들어 다음 작업들은 보통 `useEffect`가 더 적합합니다.

- API 요청
- 이벤트 리스너 등록
- 타이머 설정
- 로그 전송
- localStorage 읽기/쓰기

반대로 다음 작업들은 `useLayoutEffect`를 고려할 수 있습니다.

- DOM 크기 측정
- DOM 위치 측정
- 화면 깜빡임을 막아야 하는 레이아웃 보정
- 스크롤 위치를 즉시 맞춰야 하는 경우
- 렌더링 직후 포커스를 정확히 제어해야 하는 경우

---

## 🧩 SSR 환경에서 주의할 점

Next.js처럼 서버 렌더링을 사용하는 환경에서는 `useLayoutEffect`를 조심해야 합니다.

`useLayoutEffect`는 DOM을 전제로 하는 훅입니다. 그런데 서버에는 브라우저 DOM이 없습니다.

그래서 서버 렌더링 환경에서는 다음과 같은 경고를 만날 수 있습니다.

```txt
useLayoutEffect does nothing on the server
```

이런 경우에는 해당 컴포넌트를 클라이언트 컴포넌트로 만들거나, 정말 서버에서 렌더링될 필요가 있는지 다시 확인해야 합니다.

Next.js App Router라면 DOM을 직접 다루는 컴포넌트 상단에 `"use client"`를 선언해야 합니다.

```tsx
"use client";

import { useLayoutEffect } from "react";
```

또는 서버와 클라이언트 양쪽에서 안전하게 동작하는 커스텀 훅을 만들어 사용하는 경우도 있습니다.

```tsx
import { useEffect, useLayoutEffect } from "react";

const useIsomorphicLayoutEffect =
  typeof window !== "undefined" ? useLayoutEffect : useEffect;
```

다만 이런 패턴은 모든 문제를 해결해주는 마법 같은 도구는 아닙니다. DOM 측정이 꼭 필요한 컴포넌트라면 클라이언트에서만 실행되도록 구조를 잡는 것이 더 명확할 때가 많습니다.

---

## ✅ 정리

`useLayoutEffect`는 `useEffect`와 비슷하지만 실행 타이밍이 다릅니다.

- `useEffect`는 브라우저가 화면을 그린 뒤 실행됩니다.
- `useLayoutEffect`는 DOM이 반영된 직후, 브라우저가 화면을 그리기 전에 실행됩니다.
- DOM 크기나 위치를 측정하고 바로 레이아웃을 보정해야 할 때 유용합니다.
- 화면 깜빡임을 줄이고 싶은 UI에서 사용할 수 있습니다.
- 하지만 화면 그리기를 지연시킬 수 있으므로 꼭 필요한 경우에만 사용하는 것이 좋습니다.
- SSR 환경에서는 DOM이 없기 때문에 주의가 필요합니다.

결국 기준은 단순합니다.

**화면이 그려진 뒤 실행되어도 괜찮다면 `useEffect`, 화면이 그려지기 전에 DOM을 측정하거나 보정해야 한다면 `useLayoutEffect`**를 사용하면 됩니다.
