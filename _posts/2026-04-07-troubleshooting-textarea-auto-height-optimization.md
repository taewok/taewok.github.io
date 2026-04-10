---
title: "[React] textarea 높이 자동 조절: Tailwind CSS와 useRef로 구현하는 깔끔한 UX"
date: 2026-04-07T17:23:41
categories: [frontend, react]
tags: [textarea, useRef, scrollHeight, ui-ux]
description: "인라인 스타일 없이 Tailwind CSS와 React useRef를 활용하여 내용에 따라 늘어나는 textarea를 구현하는 최적의 방법을 소개합니다."
custom_style: true
---

---

## 🚀 도입: 인라인 스타일 탈출, 그리고 자동 높이 조절

프로젝트를 진행하다 보면 유독 `textarea`가 말썽을 부릴 때가 많다. 기본적으로 제공되는 고정 높이는 사용자 경험(UX)을 해치기 일쑤다. 처음에는 자바스크립트로 높이를 조절하면서 `style={{ height: ... }}` 같은 인라인 스타일을 남발하곤 했다.

하지만 **Tailwind CSS**를 도입하면서 "어떻게 하면 인라인 스타일을 최소화하면서 유연한 UI를 만들 수 있을까?"를 고민하게 되었다. 오늘은 인라인 스타일의 지저분함을 걷어내고, **Tailwind CSS와 useRef**를 조합해 세련되게 늘어나는 `textarea`를 구현한 과정을 공유한다.

---

## 💡 핵심 개념: 왜 scrollHeight인가?

`textarea`는 내부 콘텐츠가 아무리 길어져도 DOM 노드 자체의 `height`는 변하지 않는다. 이때 우리가 참고해야 할 속성이 바로 `scrollHeight`다.

`scrollHeight`는 패딩을 포함하여 화면에 보이지 않는 콘텐츠까지 합친 전체 높이를 말한다. 이 값을 실시간으로 추적해 요소의 `style.height`에 주입하는 것이 핵심이다. (이 부분은 아쉽게도 아직 순수 CSS만으로는 완벽한 호환성을 보장하기 어렵기에 JS의 힘을 빌려야 한다.)

---

## 🛠️ 실전 코드: Tailwind CSS + useRef

인라인 스타일을 최소화하기 위해 레이아웃과 디자인은 Tailwind 클래스로 처리하고, 동적인 높이 값만 Ref를 통해 제어한다.

```jsx
import { useRef, useEffect, useState } from "react";

export default function AutoResizableTextarea() {
  const [text, setText] = useState("");
  const textareaRef = useRef(null);

  const handleResizeHeight = () => {
    const textarea = textareaRef.current;
    if (textarea) {
      // 1. 높이를 초기화하여 줄어들 때의 scrollHeight를 정확히 측정
      textarea.style.height = "auto";
      // 2. 측정된 scrollHeight를 실제 높이에 반영
      textarea.style.height = `${textarea.scrollHeight}px`;
    }
  };

  useEffect(() => {
    handleResizeHeight();
  }, [text]); // 텍스트 상태가 변할 때마다 실행

  return (
    <div className="w-full max-w-md p-4 mx-auto">
      <label className="block mb-2 text-sm font-medium text-gray-700">
        자동 높이 조절 입력창
      </label>
      <textarea
        ref={textareaRef}
        value={text}
        onChange={(e) => setText(e.target.value)}
        rows={1}
        placeholder="내용을 입력하세요..."
        className="w-full p-3 border border-gray-300 rounded-lg shadow-sm 
                   focus:ring-2 focus:ring-blue-500 focus:border-transparent 
                   resize-none overflow-hidden transition-all duration-75 
                   bg-white text-gray-900"
      />
    </div>
  );
}
```

---

## 📊 비교: 왜 useRef 방식이 더 나은가?

| 항목        | 순수 Javascript (Vanilla)              | React useRef 방식                               |
| :---------- | :------------------------------------- | :---------------------------------------------- |
| 접근 방식   | `document.querySelector`로 직접 탐색   | React 노드 참조를 통해 안전하게 접근            |
| 상태 동기화 | 이벤트 리스너와 상태가 따로 놀 수 있음 | `useEffect` 의존성 배열로 상태 변화에 즉각 대응 |
| 코드 응집도 | 스크립트가 분산될 우려가 큼            | 컴포넌트 내부에 로직이 캡슐화됨                 |
| 성능        | 불필요한 DOM 탐색 발생 가능            | 참조값 유지로 불필요한 오버헤드 감소            |

**결론:** 리액트 환경에서 직접적인 DOM API를 사용하는 `Vanilla JS` 방식은 렌더링 타이밍을 놓치거나 메모리 누수를 유발할 수 있다. `useRef`를 활용하면 리액트의 생명주기 안에서 안전하게 DOM을 조절할 수 있어 훨씬 견고한 코드가 된다.

---

## 📝 실전 포인트 & 인사이트

<div class="highlight-box">
    <strong>언제 써야 하는가?</strong><br/>
    댓글창이나 모바일 메신저 UI처럼 한 줄로 시작해서 글이 길어지는 인터페이스에 필수적이다. 특히 Tailwind의 <code>resize-none</code> 클래스와 함께 사용해 사용자가 임의로 크기를 조절하지 못하게 막으면서 시스템이 자동으로 맞춰주는 것이 가장 깔끔하다.
</div>

<div class="warning-box">
    <strong>실수하기 쉬운 부분</strong><br/>
    Tailwind를 쓸 때 <code>h-10</code> 같이 고정 높이 클래스를 넣어버리면 JS가 부여하는 <code>style.height</code>와 충돌이 발생한다. 초기 높이는 <code>rows={1}</code> 속성이나 패딩으로 조절하고, 높이 관련 Tailwind 클래스는 피하는 것이 좋다.
</div>

<div class="highlight-box">
    <strong>개발 중 겪은 삽질 (Troubleshooting)</strong><br/>
    텍스트를 한꺼번에 지울 때 높이가 즉시 줄어들지 않는 버그를 겪었다. 원인은 <code>textarea.style.height = 'auto'</code> 코드를 빠뜨렸기 때문이었다. 이 코드가 있어야 현재 높이의 제약 없이 브라우저가 콘텐츠의 '진짜 높이'를 다시 계산할 수 있다.
</div>

---

## ✅ 마무리하며

인라인 스타일을 지양하고 Tailwind CSS를 쓰는 이유는 코드의 가독성과 유지보수 때문이다. 하지만 높이 조절처럼 **"계산된 값"**이 필요한 순간에는 JS의 힘을 빌릴 수밖에 없다.

중요한 것은 그 JS 로직을 리액트답게(`useRef`, `useEffect`) 작성하고, 나머지 디자인은 Tailwind로 깔끔하게 분리하는 균형이다. 이제 여러분의 프로젝트에서도 스크롤바 없는 쾌적한 입력창을 구현해 보길 바란다!
