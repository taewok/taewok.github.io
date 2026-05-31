---
title: "[React] textarea 높이 자동 조절: Tailwind CSS와 useRef로 구현하기"
date: 2026-04-07T17:23:41
categories: [frontend, react]
tags: [textarea, useRef, scrollHeight, ui-ux]
description: "Tailwind CSS와 React useRef를 활용해 내용에 따라 자연스럽게 늘어나는 textarea를 구현하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: textarea 높이를 자연스럽게 맞추고 싶었어요

댓글창이나 채팅 입력창을 만들다 보면 `textarea` 높이를 내용에 맞게 늘리고 싶을 때가 많습니다.

기본 `textarea`는 정해진 높이 안에서 스크롤이 생깁니다. 기능적으로는 문제가 없지만, 사용자가 긴 문장을 입력할 때 내부 스크롤이 생기면 입력 흐름이 조금 답답하게 느껴질 수 있어요.

처음에는 `style={{ height: ... }}`를 상태값으로 관리하거나, 입력할 때마다 직접 높이를 계산하는 방식으로 접근했습니다. 하지만 Tailwind CSS를 사용하고 있는 프로젝트에서는 레이아웃과 스타일은 클래스에 맡기고, 정말 동적으로 계산해야 하는 높이만 JavaScript로 다루는 편이 더 깔끔했습니다.

이번 글에서는 `useRef`, `scrollHeight`, Tailwind CSS를 조합해서 자동으로 높이가 늘어나는 `textarea`를 구현해보겠습니다.

---

## 💡 핵심은 scrollHeight입니다

`textarea`의 내용이 길어져도 요소의 실제 `height`가 자동으로 늘어나지는 않습니다.

이때 사용할 수 있는 값이 `scrollHeight`입니다.

`scrollHeight`는 요소 안의 콘텐츠가 전부 보이기 위해 필요한 전체 높이를 의미합니다. 화면에 보이는 영역뿐 아니라, 스크롤 안쪽에 숨어 있는 내용까지 포함한 높이라고 보면 됩니다.

```ts
const height = textarea.scrollHeight;
```

자동 높이 조절의 흐름은 단순합니다.

1. 입력값이 바뀝니다.
2. `textarea` 높이를 잠시 `auto`로 되돌립니다.
3. 현재 내용 기준의 `scrollHeight`를 다시 측정합니다.
4. 측정한 값을 `textarea.style.height`에 넣습니다.

여기서 `height = "auto"`로 초기화하는 과정이 중요합니다. 이 과정이 없으면 글을 지웠을 때 높이가 줄어들지 않고, 이전에 커진 높이가 그대로 남을 수 있습니다.

---

## 🛠️ 구현 코드

레이아웃과 시각적인 스타일은 Tailwind CSS 클래스로 처리하고, 높이 계산만 `ref`로 접근해 제어했습니다.

```tsx
import { useEffect, useRef, useState } from "react";

export default function AutoResizableTextarea() {
  const [text, setText] = useState("");
  const textareaRef = useRef<HTMLTextAreaElement | null>(null);

  const resizeTextarea = () => {
    const textarea = textareaRef.current;

    if (!textarea) return;

    // 먼저 높이를 초기화해야 글을 지웠을 때도 높이가 다시 줄어듭니다.
    textarea.style.height = "auto";

    // 현재 내용 전체를 보여주기 위해 필요한 높이를 적용합니다.
    textarea.style.height = `${textarea.scrollHeight}px`;
  };

  useEffect(() => {
    resizeTextarea();
  }, [text]);

  return (
    <div className="mx-auto w-full max-w-md p-4">
      <label className="mb-2 block text-sm font-medium text-gray-700">
        자동 높이 조절 입력창
      </label>

      <textarea
        ref={textareaRef}
        value={text}
        onChange={(event) => setText(event.target.value)}
        rows={1}
        placeholder="내용을 입력하세요..."
        className="
          w-full resize-none overflow-hidden rounded-lg border border-gray-300
          bg-white p-3 text-gray-900 shadow-sm transition-all duration-75
          focus:border-transparent focus:ring-2 focus:ring-blue-500
        "
      />
    </div>
  );
}
```

`rows={1}`은 처음 높이를 한 줄 기준으로 잡기 위한 값입니다.

그리고 `resize-none`을 넣어 사용자가 직접 크기를 조절하지 않도록 했습니다. 자동 높이 조절 UI에서는 사용자 조절과 코드 조절이 동시에 섞이면 의도하지 않은 결과가 나올 수 있기 때문이에요.

---

## 📌 useRef를 사용하는 이유

React에서 DOM 요소의 실제 높이를 읽으려면 결국 DOM 노드에 접근해야 합니다.

이때 `document.querySelector`로 직접 찾을 수도 있지만, React 컴포넌트 안에서는 `useRef`를 사용하는 편이 더 자연스럽습니다.

```tsx
const textareaRef = useRef<HTMLTextAreaElement | null>(null);
```

`ref`를 연결하면 해당 DOM 노드를 컴포넌트 내부에서 안전하게 참조할 수 있습니다.

```tsx
<textarea ref={textareaRef} />
```

이 방식의 장점은 다음과 같습니다.

- 컴포넌트 안에서 필요한 DOM만 직접 참조할 수 있습니다.
- 전역 선택자에 의존하지 않아 다른 요소와 충돌할 가능성이 줄어듭니다.
- 입력값 변경 시점과 높이 조절 로직을 React 흐름 안에서 함께 관리할 수 있습니다.

---

## ⚠️ 높이를 auto로 되돌리는 이유

자동 높이 조절에서 가장 자주 놓치는 부분이 이 코드입니다.

```ts
textarea.style.height = "auto";
textarea.style.height = `${textarea.scrollHeight}px`;
```

높이를 바로 `scrollHeight`로만 지정하면, 글을 추가할 때는 잘 늘어납니다. 하지만 글을 지울 때는 기존 높이가 남아 있어서 `scrollHeight`가 기대만큼 작아지지 않을 수 있습니다.

그래서 먼저 `auto`로 되돌려 브라우저가 현재 내용 기준으로 높이를 다시 계산할 수 있게 만든 뒤, 새 `scrollHeight`를 적용해야 합니다.

---

## 🎨 Tailwind CSS와 함께 쓸 때 주의할 점

Tailwind CSS를 사용할 때는 고정 높이 클래스와 충돌하지 않도록 조심해야 합니다.

예를 들어 이런 클래스를 같이 넣으면 문제가 생길 수 있습니다.

```tsx
className="h-10 ..."
```

`h-10`은 높이를 고정합니다. 그런데 JavaScript에서도 `textarea.style.height`를 계속 바꾸고 있으니, 높이를 누가 결정하는지 애매해집니다.

자동 높이 조절이 필요한 `textarea`에서는 보통 다음 조합이 더 안전합니다.

- 초기 줄 수는 `rows={1}`로 정합니다.
- 사용자의 수동 크기 조절은 `resize-none`으로 막습니다.
- 내부 스크롤은 `overflow-hidden`으로 숨깁니다.
- 실제 높이는 `scrollHeight`로 계산해 적용합니다.

---

## ✅ 마무리

`textarea` 자동 높이 조절은 복잡한 기능처럼 보이지만, 원리는 단순합니다.

핵심은 현재 내용 전체를 보여주기 위해 필요한 높이인 `scrollHeight`를 읽고, 그 값을 실제 높이에 반영하는 것입니다.

React에서는 `useRef`로 DOM에 접근하고, Tailwind CSS는 레이아웃과 시각적인 스타일을 담당하도록 역할을 나누면 코드가 훨씬 읽기 좋아집니다.

댓글창, 채팅 입력창, 메모 입력창처럼 사용자가 긴 텍스트를 자연스럽게 입력해야 하는 UI라면 이 패턴을 적용해볼 만합니다.
