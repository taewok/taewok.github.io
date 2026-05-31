---
title: "[React] useLayoutEffect는 언제 사용해야 할까?"
date: 2023-08-01T15:19:00+09:00
categories: [frontend]
tags: [react, hooks, uselayouteffect, useeffect]
description: "React의 useLayoutEffect가 useEffect와 어떻게 다르고, DOM 측정이나 레이아웃 보정이 필요한 상황에서 언제 사용하면 좋은지 정리했습니다."
custom_style: true
---

## 들어가며

React에서 사이드 이펙트를 처리할 때는 대부분 `useEffect`를 사용합니다.

API 요청, 이벤트 리스너 등록, 타이머 설정, localStorage 접근 같은 작업은 보통 `useEffect`만으로 충분합니다.

그런데 React에는 이름이 비슷한 `useLayoutEffect`도 있습니다.

처음 보면 이런 생각이 들 수 있습니다.

```txt
useEffect가 있는데 useLayoutEffect는 왜 필요할까?
```

핵심 차이는 실행 시점입니다.

`useLayoutEffect`는 DOM이 변경된 직후, 브라우저가 화면을 그리기 전에 실행됩니다.

그래서 DOM 크기나 위치를 측정하고, 화면에 보이기 전에 레이아웃을 보정해야 하는 상황에서 유용합니다.

---

## useEffect와 useLayoutEffect의 차이

`useEffect`는 브라우저가 화면을 그린 뒤 실행됩니다.

```txt
React 렌더링
DOM 반영
브라우저 화면 그리기
useEffect 실행
```

반면 `useLayoutEffect`는 DOM 반영 직후, 화면이 그려지기 전에 실행됩니다.

```txt
React 렌더링
DOM 반영
useLayoutEffect 실행
브라우저 화면 그리기
```

이 차이 때문에 `useLayoutEffect` 안에서 레이아웃을 보정하면 사용자는 보정 전 화면을 거의 보지 않게 됩니다.

---

## 기본 사용법

사용법 자체는 `useEffect`와 거의 같습니다.

```tsx
import { useLayoutEffect } from "react";

useLayoutEffect(() => {
  // DOM 반영 직후, 브라우저가 화면을 그리기 전에 실행됩니다.

  return () => {
    // cleanup이 필요하면 여기서 처리합니다.
  };
}, []);
```

의존성 배열을 사용하는 방식도 같습니다.

```tsx
useLayoutEffect(() => {
  // value가 바뀔 때마다 실행됩니다.
}, [value]);
```

---

## DOM 크기 측정하기

`useLayoutEffect`가 잘 어울리는 대표적인 상황은 DOM 크기 측정입니다.

```tsx
import { useLayoutEffect, useRef, useState } from "react";

const BoxMeasure = () => {
  const boxRef = useRef<HTMLDivElement | null>(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    if (!boxRef.current) return;

    const nextHeight = boxRef.current.getBoundingClientRect().height;
    setHeight(nextHeight);
  }, []);

  return (
    <div>
      <div ref={boxRef}>높이를 측정할 박스입니다.</div>
      <p>박스 높이: {height}px</p>
    </div>
  );
};

export default BoxMeasure;
```

`getBoundingClientRect()`는 실제 DOM의 크기와 위치를 읽는 메서드입니다.

이런 측정값을 바탕으로 바로 UI를 보정해야 한다면 `useLayoutEffect`가 적합합니다.

---

## 화면 깜빡임 줄이기

툴팁이나 드롭다운처럼 위치를 계산해야 하는 UI에서는 처음 위치가 잘못 보였다가 바로 이동하는 깜빡임이 생길 수 있습니다.

```tsx
import { useLayoutEffect, useRef, useState } from "react";

const Tooltip = () => {
  const tooltipRef = useRef<HTMLDivElement | null>(null);
  const [left, setLeft] = useState(0);

  useLayoutEffect(() => {
    if (!tooltipRef.current) return;

    const rect = tooltipRef.current.getBoundingClientRect();
    const padding = 16;

    if (rect.right > window.innerWidth - padding) {
      setLeft(window.innerWidth - rect.width - padding);
      return;
    }

    setLeft(rect.left);
  }, []);

  return (
    <div ref={tooltipRef} style={{ left }} className="tooltip">
      툴팁 내용입니다.
    </div>
  );
};

export default Tooltip;
```

`useEffect`에서 위치를 보정하면 사용자가 잘못된 위치를 잠깐 볼 수 있습니다.

`useLayoutEffect`를 사용하면 브라우저가 화면을 그리기 전에 보정할 수 있어 깜빡임을 줄일 수 있습니다.

---

## 스크롤 위치 보정하기

채팅 목록처럼 새 메시지가 추가되면 바로 아래로 스크롤해야 하는 UI에도 사용할 수 있습니다.

```tsx
import { useLayoutEffect, useRef } from "react";

interface ChatListProps {
  messages: string[];
}

const ChatList = ({ messages }: ChatListProps) => {
  const scrollRef = useRef<HTMLDivElement | null>(null);

  useLayoutEffect(() => {
    if (!scrollRef.current) return;

    scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
  }, [messages]);

  return (
    <div ref={scrollRef} className="chat-list">
      {messages.map((message, index) => (
        <div key={index}>{message}</div>
      ))}
    </div>
  );
};

export default ChatList;
```

DOM에 새 메시지가 반영된 직후 스크롤 위치를 맞추기 때문에 더 즉각적으로 동작합니다.

---

## 남용하면 안 되는 이유

`useLayoutEffect`는 브라우저가 화면을 그리기 전에 실행됩니다.

그 말은 `useLayoutEffect` 안의 작업이 오래 걸리면 화면이 늦게 그려질 수 있다는 뜻입니다.

그래서 모든 effect를 `useLayoutEffect`로 바꾸는 것은 좋지 않습니다.

대부분의 작업은 `useEffect`로 충분합니다.

`useLayoutEffect`는 DOM 측정, 위치 보정, 스크롤 보정처럼 화면이 보이기 전에 처리해야 하는 작업에만 사용하는 것이 좋습니다.

---

## SSR 환경에서 주의하기

Next.js처럼 서버 렌더링을 사용하는 환경에서는 `useLayoutEffect`를 조심해야 합니다.

`useLayoutEffect`는 DOM이 있는 브라우저 환경을 전제로 합니다.

서버에는 DOM이 없기 때문에 경고를 만날 수 있습니다.

```txt
useLayoutEffect does nothing on the server
```

Next.js App Router라면 DOM을 직접 다루는 컴포넌트 상단에 `"use client"`를 선언해야 합니다.

```tsx
"use client";

import { useLayoutEffect } from "react";
```

---

## 마무리

`useEffect`와 `useLayoutEffect`는 사용법은 비슷하지만 실행 시점이 다릅니다.

`useEffect`는 화면이 그려진 뒤 실행되고, `useLayoutEffect`는 DOM 반영 직후 화면이 그려지기 전에 실행됩니다.

API 요청이나 이벤트 등록은 `useEffect`를 사용하고, DOM 크기 측정이나 레이아웃 보정처럼 화면에 보이기 전에 처리해야 하는 작업은 `useLayoutEffect`를 고려하면 됩니다.
