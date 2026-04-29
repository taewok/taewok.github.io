---
title: "[React] requestAnimationFrame으로 채팅창 스크롤 최적화하기: 끊김 없는 UI 경험의 비밀"
date: 2026-04-16T18:03:00Z
categories: [frontend]
tags:
  [javascript, requestanimationframe, scroll-optimization, browser-rendering]
description: "채팅창의 새로운 메시지가 올 때 스크롤이 자연스럽게 내려가지 않나요? requestAnimationFrame을 활용해 브라우저 렌더링 주기에 맞춘 매끄러운 스크롤 최적화 기법을 공유합니다."
custom_style: true
---

## 🧐 왜 스크롤 이동이 내 마음대로 안 될까?

채팅 서비스를 구현하다 보면 가장 신경 쓰이는 부분이 바로 **'새로운 메시지가 왔을 때 스크롤을 최하단으로 내리는 로직'**이다. 처음에는 단순히 새로운 데이터가 배열에 추가된 직후 `element.scrollTop = element.scrollHeight` 코드를 실행했다.

하지만 문제가 발생했다. 데이터는 분명히 추가되었는데, **스크롤이 미처 다 내려가지 못하고 이전 메시지까지만 머무는 현상**이 간혹 발생하거나, 연속으로 메시지가 올 때 UI가 미세하게 떨리는 느낌을 받았다.

이 문제는 브라우저가 DOM을 변경하고 실제로 화면에 그리는(Repaint) 타이밍과 내 자바스크립트 코드가 실행되는 타이밍이 어긋나서 발생하는 문제였다. 이를 해결하기 위해 나는 `requestAnimationFrame`에 주목했다.

---

## 💡 requestAnimationFrame()이란?

`requestAnimationFrame`은 브라우저가 다음 리페인트(Repaint)를 수행하기 직전에 지정된 함수를 실행하도록 요청하는 API다.

### 1. 왜 사용하는가?

일반적인 `setTimeout`이나 `setInterval`은 브라우저의 렌더링 프레임(보통 60fps)을 고려하지 않고 실행된다. 반면 `requestAnimationFrame`은 브라우저의 디스플레이 주기에 맞춰 실행되므로 **불필요한 리렌더링을 방지하고 애니메이션을 부드럽게** 만든다.

### 2. 언제 써야 할까?

- 스크롤 애니메이션 및 위치 제어
- Canvas 그래픽 작업
- JS 기반 레이아웃 변경 (Drag & Drop 등)
- **DOM 변경 직후 정확한 크기/위치 값을 계산해야 할 때**

---

## 🛠️ 실전 코드: 채팅창 스크롤 최적화

채팅 리스트가 업데이트된 후, DOM이 실제로 화면에 반영되어 `scrollHeight`가 최신화된 시점에 스크롤을 이동시키는 로직이다.

`requestAnimationFrame`을 사용하기 이전에는 새로운 messages가 추가되어 늘어난 높이를 측정하기도 전에 이미 스크롤 이벤트를 실행하는게 문제였다

```jsx
"use client";

import { useRef, useEffect, useState } from "react";

const ChatRoom = () => {
  const [messages, setMessages] = useState<string[]>([]);
  // 1. DOM에 직접 접근하기 위한 Ref
  const scrollRef = useRef<HTMLDivElement>(null);

  // 2. 메시지가 추가될 때마다 실행되는 Effect
  useEffect(() => {
    if (scrollRef.current) {
      // 브라우저가 다음 페인팅을 하기 직전에 스크롤 위치를 계산
      requestAnimationFrame(() => {
        if (scrollRef.current) {
          const { scrollHeight, clientHeight } = scrollRef.current;

          scrollRef.current.scrollTo({
            top: scrollHeight - clientHeight,
            behavior: "smooth",
          });
        }
      });
    }
  }, [messages]); // 메시지 배열이 바뀔 때마다 트리거

  const handleSendMessage = (text: string) => {
    setMessages((prev) => [...prev, text]);
  };

  return (
    <div className="flex flex-col h-[500px] bg-[#0B0E14] border border-border-main rounded-2xl overflow-hidden">
      {/* 3. 스크롤 컨테이너에 ref 연결 */}
      <div
        ref={scrollRef}
        className="flex-1 overflow-y-auto p-4 space-y-3 custom-scrollbar"
      >
        {messages.map((msg, i) => (
          <div key={i} className="p-3 bg-gray-800/50 text-white rounded-xl w-fit max-w-[80%] border border-white/5">
            {msg}
          </div>
        ))}
      </div>

      <div className="p-4 bg-gray-900/50">
        <button
          onClick={() => handleSendMessage("새 메시지가 도착했습니다! 💬")}
          className="w-full py-3 bg-brand hover:bg-brand-dark text-white rounded-xl font-semibold transition-all"
        >
          메시지 전송 테스트
        </button>
      </div>
    </div>
  );
};
```

### 💻 실행 결과

- 메세지가 추가됨과 동시에 브라우저 렌더링 최적 타이밍에 스크롤 함수가 예약된다.
- 사용자는 스크롤이 씹히거나 끊기는 느낌 없이 **항상 최신 대화 내용**을 즉각적으로 확인할 수 있다.

---

## 🚀 실무 포인트 (Insight)

### 1. 언제 써야 하는가?

단순히 값을 바꾸는 게 아니라, **"바뀐 값이 실제 DOM에 반영되어 크기(Height, Width)가 변한 직후"** 무언가를 해야 할 때 필수적이다. 채팅창 스크롤은 메시지가 들어가서 높이가 변해야 하므로 이 상황에 딱 맞다.

### 2. 실수하기 쉬운 부분

`requestAnimationFrame`은 비동기적으로 동작한다. 따라서 루프 내부에서 잘못 사용하면 의도치 않은 중첩 호출이 발생할 수 있다. 다행히 스크롤 이동은 여러 번 호출되어도 마지막에 설정된 위치로 이동하므로 비교적 안전하지만, `cancelAnimationFrame`을 통해 이전 요청을 취소하는 습관을 들이는 것이 성능상 이롭다.

### 3. 실제 개발 경험 (Troubleshooting)

처음에는 `setTimeout(() => {}, 0)`을 사용했었다. 하지만 이는 **Task Queue**로 들어가기 때문에 렌더링 주기와 상관없이 실행되어 간혹 1프레임 정도 밀리는 현상이 있었다. **`requestAnimationFrame`**으로 교체한 후에는 리페인트 직전에 정확히 실행되어 레이아웃 수치 계산이 훨씬 정확해졌다.

---

## 📊 비교 정리

| 항목        | setTimeout (0ms)                     | requestAnimationFrame                      |
| :---------- | :----------------------------------- | :----------------------------------------- |
| 실행 타이밍 | 이벤트 루프의 Task Queue 순서에 따름 | **브라우저의 다음 Repaint 직전**           |
| 부드러움    | 프레임 드랍이 발생할 수 있음         | 디스플레이 주기에 맞춰 **매우 부드러움**   |
| 배터리 소모 | 백그라운드 탭에서도 실행됨           | 탭이 비활성화되면 **자동 중지 (효율적)**   |
| 용도        | 단순 지연 실행                       | **애니메이션, UI 업데이트, 레이아웃 계산** |

---

## 🎁 마무리하며

채팅 스크롤 하나에도 브라우저의 렌더링 원리가 숨어 있다는 점이 흥미롭다. 단순히 기능을 구현하는 것을 넘어, **사용자가 느끼는 매끄러운 경험(UX)**을 위해서는 브라우저와 대화하는 법을 익혀야 한다.

`requestAnimationFrame`은 생각보다 많은 UI 이슈를 해결해 주는 마법 같은 도구다. 지금 내가 만든 채팅창 스크롤이 가끔 덜컥거린다면, 이 API를 즉시 적용해 보자.

<div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-500 my-5">
  <strong>💡 핵심 요약:</strong> DOM 변화에 따른 수치 계산이나 스크롤 이동이 필요할 때는 <code>setTimeout</code> 대신 <code>requestAnimationFrame</code>을 사용해 브라우저의 호흡과 맞추자!
</div>
