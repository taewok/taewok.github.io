---
title: "[React] requestAnimationFrame으로 채팅창 스크롤 최적화하기"
date: 2026-04-16T18:03:00Z
categories: [frontend]
tags: [javascript, requestanimationframe, scroll-optimization, browser-rendering]
description: "채팅창에 새 메시지가 추가된 뒤 스크롤을 안정적으로 최하단으로 이동시키기 위해 requestAnimationFrame을 활용한 과정을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 새 메시지가 왔는데 스크롤이 애매하게 멈췄어요

채팅 UI를 만들 때 자주 필요한 기능이 있습니다.

새 메시지가 추가되면 채팅창 스크롤을 가장 아래로 내려서 사용자가 최신 메시지를 바로 볼 수 있게 하는 기능입니다.

처음에는 메시지를 추가한 직후 바로 스크롤을 내렸습니다.

```ts
element.scrollTop = element.scrollHeight;
```

그런데 실제 화면에서는 가끔 스크롤이 끝까지 내려가지 않거나, 연속으로 메시지가 들어올 때 살짝 덜컥거리는 느낌이 있었습니다.

이 문제는 메시지 배열은 업데이트되었지만, 브라우저가 아직 새 DOM 높이를 계산하기 전일 수 있기 때문에 생깁니다. 즉, JavaScript는 실행됐지만 화면의 레이아웃 계산은 아직 끝나지 않은 시점인 거예요.

이럴 때 사용할 수 있는 API가 `requestAnimationFrame`입니다.

---

## 💡 requestAnimationFrame은 어떤 역할을 할까요?

`requestAnimationFrame`은 브라우저에게 다음 화면을 그리기 직전에 콜백을 실행해달라고 요청하는 API입니다.

```ts
requestAnimationFrame(() => {
  // 다음 프레임 직전에 실행됩니다.
});
```

`setTimeout`은 단순히 일정 시간이 지난 뒤 콜백을 실행합니다. 반면 `requestAnimationFrame`은 브라우저의 렌더링 흐름에 맞춰 실행됩니다.

채팅 스크롤처럼 DOM 높이가 바뀐 뒤 그 값을 기준으로 작업해야 할 때는 이 차이가 꽤 중요합니다.

---

## 🛠️ 채팅창 스크롤에 적용하기

메시지가 바뀔 때마다 다음 프레임에 스크롤을 아래로 이동시키는 예제입니다.

```tsx
"use client";

import { useEffect, useRef, useState } from "react";

const ChatRoom = () => {
  const [messages, setMessages] = useState<string[]>([]);
  const scrollRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    const frameId = requestAnimationFrame(() => {
      const scrollElement = scrollRef.current;

      if (!scrollElement) return;

      scrollElement.scrollTo({
        top: scrollElement.scrollHeight,
        behavior: "smooth",
      });
    });

    return () => {
      cancelAnimationFrame(frameId);
    };
  }, [messages]);

  const handleSendMessage = () => {
    setMessages((prevMessages) => [
      ...prevMessages,
      "새 메시지가 도착했습니다!",
    ]);
  };

  return (
    <div className="flex h-[500px] flex-col overflow-hidden rounded-2xl border bg-gray-950">
      <div ref={scrollRef} className="flex-1 space-y-3 overflow-y-auto p-4">
        {messages.map((message, index) => (
          <div
            key={`${message}-${index}`}
            className="w-fit max-w-[80%] rounded-xl bg-gray-800 px-3 py-2 text-white"
          >
            {message}
          </div>
        ))}
      </div>

      <div className="border-t border-gray-800 p-4">
        <button
          type="button"
          onClick={handleSendMessage}
          className="w-full rounded-xl bg-blue-600 py-3 font-semibold text-white"
        >
          메시지 전송 테스트
        </button>
      </div>
    </div>
  );
};

export default ChatRoom;
```

여기서 중요한 부분은 `useEffect` 안에서 바로 `scrollTo`를 호출하지 않고, `requestAnimationFrame` 안에서 호출했다는 점입니다.

```tsx
const frameId = requestAnimationFrame(() => {
  const scrollElement = scrollRef.current;

  if (!scrollElement) return;

  scrollElement.scrollTo({
    top: scrollElement.scrollHeight,
    behavior: "smooth",
  });
});
```

메시지 상태가 바뀐 뒤 브라우저가 다음 프레임을 준비하는 시점에 스크롤 높이를 읽기 때문에, 새 메시지가 반영된 높이를 기준으로 이동할 수 있습니다.

---

## 🧹 cancelAnimationFrame으로 정리하기

`requestAnimationFrame`은 예약된 콜백을 나중에 실행합니다.

컴포넌트가 사라졌는데 예약된 콜백이 뒤늦게 실행되면, 이미 없는 DOM을 참조하려고 할 수 있습니다. 그래서 `useEffect`의 cleanup에서 예약을 취소해주는 편이 안전합니다.

```tsx
return () => {
  cancelAnimationFrame(frameId);
};
```

채팅처럼 메시지가 빠르게 여러 번 추가될 수 있는 UI에서는 이런 정리 코드가 특히 중요합니다.

---

## 📊 setTimeout과 비교해보기

비슷한 문제를 해결하려고 다음처럼 작성하는 경우도 있습니다.

```tsx
setTimeout(() => {
  scrollToBottom();
}, 0);
```

이 방식도 운 좋게 동작할 때가 많습니다. 하지만 `setTimeout`은 브라우저 렌더링 타이밍에 맞춰 실행되는 API가 아닙니다.

비교하면 이렇게 볼 수 있습니다.

| 항목 | setTimeout | requestAnimationFrame |
| --- | --- | --- |
| 실행 기준 | 지정한 시간 | 브라우저 렌더링 프레임 |
| DOM 측정 안정성 | 상황에 따라 흔들릴 수 있음 | 레이아웃 계산 이후에 맞추기 쉬움 |
| 애니메이션/스크롤 | 부드럽지 않을 수 있음 | 프레임 흐름에 맞추기 좋음 |
| 백그라운드 탭 | 계속 실행될 수 있음 | 브라우저가 실행 빈도를 조절함 |

채팅 스크롤처럼 화면 업데이트와 밀접한 작업은 `requestAnimationFrame`이 더 잘 맞습니다.

---

## ✅ 마무리

새 메시지가 추가된 뒤 스크롤을 아래로 내리는 기능은 단순해 보이지만, 실제로는 DOM 업데이트와 브라우저 렌더링 타이밍을 함께 고려해야 합니다.

`requestAnimationFrame`을 사용하면 브라우저가 다음 화면을 그리기 직전의 안정적인 시점에 스크롤 위치를 계산할 수 있습니다.

DOM 크기를 읽거나, 스크롤을 이동시키거나, 화면 변화 직후 위치를 계산해야 한다면 `setTimeout`보다 `requestAnimationFrame`을 먼저 떠올려보면 좋습니다.
