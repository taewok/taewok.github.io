---
title: "[React] 접근성 있는 모달 만들기: focus trap과 ESC 닫기"
date: 2026-08-27T13:40:00Z
categories: [frontend]
tags: [react, modal, accessibility, focus-trap, keyboard, createportal]
description: "React에서 접근성 있는 모달을 만들기 위해 role, aria-modal, 초기 포커스 이동, focus trap, ESC 닫기, 포커스 복귀를 구현하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며: 모달은 화면에 띄우는 것만으로 끝나지 않아요

모달을 만들 때 처음에는 보통 이런 부분에 집중하게 됩니다.

- 배경을 어둡게 깔기
- 가운데에 박스 띄우기
- 닫기 버튼 만들기
- 바깥 영역을 클릭하면 닫기
- `z-index`를 높게 주기

눈으로 보기에는 이 정도만 해도 모달처럼 보입니다.

하지만 키보드로 조작해보면 이야기가 달라집니다.

```txt
모달을 열었는데 포커스는 여전히 뒤쪽 페이지에 있음
Tab을 누르니 모달 밖 버튼으로 이동함
ESC를 눌러도 닫히지 않음
모달을 닫았더니 원래 누르던 버튼 위치를 잃어버림
```

이런 상태라면 화면상으로는 모달이지만, 실제 사용자 경험은 불안정합니다.

특히 키보드만 사용하는 사용자나 스크린 리더 사용자에게 모달은 **현재 작업 흐름을 잠시 가로막는 중요한 UI**입니다. 그래서 모달이 열렸다면 브라우저의 포커스 흐름도 함께 모달 안으로 들어와야 합니다.

이번 글에서는 React에서 접근성 있는 모달을 만들기 위해 다음 기능들을 구현해보겠습니다.

```txt
1. role="dialog"와 aria-modal="true" 지정하기
2. 모달이 열릴 때 포커스를 모달 안으로 이동하기
3. Tab과 Shift + Tab이 모달 안에서만 순환되게 하기
4. ESC를 누르면 모달 닫기
5. 모달이 닫히면 원래 포커스 위치로 되돌리기
```

---

## 접근성 있는 모달의 기본 조건

WAI-ARIA Authoring Practices Guide에서는 모달 다이얼로그를 기존 화면 위에 올라오는 창으로 설명합니다. 이때 모달 뒤쪽 영역은 비활성 상태가 되어야 합니다.

즉, 사용자는 모달이 열려 있는 동안 뒤쪽 페이지를 조작하면 안 됩니다.

마우스 사용자에게는 배경 overlay가 그 역할을 어느 정도 해줍니다.

하지만 키보드 사용자에게는 overlay만으로 부족합니다.

키보드는 화면을 클릭하지 않고 `Tab` 키로 이동하기 때문에, 포커스가 모달 밖으로 빠져나가지 않게 따로 관리해야 합니다.

접근성 있는 모달에서 챙겨야 할 핵심은 다음과 같습니다.

| 항목 | 이유 |
| --- | --- |
| `role="dialog"` | 이 영역이 대화상자임을 보조 기술에 알려줍니다. |
| `aria-modal="true"` | 모달이 열려 있는 동안 뒤쪽 페이지가 비활성이라는 의미를 전달합니다. |
| `aria-labelledby` | 모달 제목을 연결해 모달의 이름을 제공합니다. |
| 초기 포커스 이동 | 모달이 열렸다는 사실을 키보드 사용자도 바로 알 수 있습니다. |
| focus trap | `Tab`으로 모달 밖으로 나가지 않게 막습니다. |
| ESC 닫기 | 키보드로 빠르게 모달을 닫을 수 있습니다. |
| 포커스 복귀 | 닫은 뒤 원래 작업하던 위치로 돌아갈 수 있습니다. |

이 중에서 이번 글은 `focus trap`과 `ESC` 닫기를 중심으로 봅니다.

---

## 먼저 완성된 사용 예시 보기

최종적으로는 이런 식으로 사용할 수 있는 모달 컴포넌트를 만들겠습니다.

```tsx
"use client";

import { useState } from "react";
import { AccessibleModal } from "./AccessibleModal";

export default function Example() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button type="button" onClick={() => setIsOpen(true)}>
        프로필 수정 열기
      </button>

      <AccessibleModal
        open={isOpen}
        title="프로필 수정"
        onClose={() => setIsOpen(false)}
      >
        <label>
          닉네임
          <input type="text" defaultValue="taewok" />
        </label>

        <div className="mt-6 flex justify-end gap-2">
          <button type="button" onClick={() => setIsOpen(false)}>
            취소
          </button>
          <button type="button">저장</button>
        </div>
      </AccessibleModal>
    </div>
  );
}
```

사용하는 쪽에서는 `open`, `title`, `onClose`, `children`만 넘기면 됩니다.

접근성 처리는 `AccessibleModal` 안에 모아두겠습니다.

---

## createPortal로 모달 위치 분리하기

모달은 보통 현재 컴포넌트의 DOM 안쪽보다 `document.body` 아래에 렌더링하는 편이 좋습니다.

부모 요소의 `overflow: hidden`, `transform`, `z-index` 같은 스타일 영향을 덜 받기 때문입니다.

React에서는 `createPortal`을 사용합니다.

```tsx
import { createPortal } from "react-dom";

createPortal(children, document.body);
```

`createPortal`은 실제 DOM 위치만 바꿉니다.

React 컴포넌트 관계, context, 이벤트 흐름은 기존 React 트리 기준으로 유지됩니다.

그래서 모달처럼 화면 가장 위에 떠야 하는 UI를 만들 때 잘 어울립니다.

---

## focus trap의 핵심 아이디어

focus trap은 이름만 보면 조금 복잡해 보이지만, 핵심은 단순합니다.

```txt
모달 안에서 Tab을 누르면 다음 포커스 요소로 이동한다.
마지막 요소에서 Tab을 누르면 첫 번째 요소로 돌아간다.
첫 번째 요소에서 Shift + Tab을 누르면 마지막 요소로 돌아간다.
```

예를 들어 모달 안에 이런 요소들이 있다고 해보겠습니다.

```txt
[닫기 버튼] → [닉네임 input] → [취소 버튼] → [저장 버튼]
```

일반적인 `Tab` 흐름은 다음과 같아야 합니다.

```txt
닫기 버튼
→ 닉네임 input
→ 취소 버튼
→ 저장 버튼
→ 닫기 버튼
```

반대로 `Shift + Tab`은 거꾸로 순환해야 합니다.

```txt
닫기 버튼
→ 저장 버튼
→ 취소 버튼
→ 닉네임 input
→ 닫기 버튼
```

이렇게 하면 모달이 열려 있는 동안 키보드 포커스가 모달 밖으로 빠져나가지 않습니다.

---

## 포커스 가능한 요소 찾기

먼저 모달 안에서 포커스 가능한 요소들을 찾아야 합니다.

대표적으로 이런 요소들이 있습니다.

- `button`
- `input`
- `select`
- `textarea`
- `a[href]`
- `tabindex`가 있는 요소

이를 selector로 정리하면 다음과 같습니다.

```tsx
const FOCUSABLE_SELECTOR = [
  "a[href]",
  "button:not([disabled])",
  "textarea:not([disabled])",
  "input:not([disabled])",
  "select:not([disabled])",
  "[tabindex]:not([tabindex='-1'])",
].join(",");
```

그리고 특정 컨테이너 안에서 포커스 가능한 요소를 배열로 가져오는 함수를 만듭니다.

```tsx
const getFocusableElements = (container: HTMLElement) => {
  return Array.from(
    container.querySelectorAll<HTMLElement>(FOCUSABLE_SELECTOR),
  ).filter((element) => {
    const isHidden =
      element.hidden ||
      element.getAttribute("aria-hidden") === "true";

    return !isHidden;
  });
};
```

`querySelectorAll`만 사용하면 숨겨진 요소까지 포함될 수 있습니다.

그래서 `hidden`, `aria-hidden` 정도는 제외해두는 것이 좋습니다.

프로젝트 상황에 따라 `display: none`, `visibility: hidden`까지 더 엄격하게 검사할 수도 있습니다.

---

## AccessibleModal 컴포넌트 만들기

이제 전체 모달 컴포넌트를 만들어보겠습니다.

```tsx
"use client";

import {
  type ReactNode,
  useEffect,
  useId,
  useRef,
} from "react";
import { createPortal } from "react-dom";

const FOCUSABLE_SELECTOR = [
  "a[href]",
  "button:not([disabled])",
  "textarea:not([disabled])",
  "input:not([disabled])",
  "select:not([disabled])",
  "[tabindex]:not([tabindex='-1'])",
].join(",");

const getFocusableElements = (container: HTMLElement) => {
  return Array.from(
    container.querySelectorAll<HTMLElement>(FOCUSABLE_SELECTOR),
  ).filter((element) => {
    const isHidden =
      element.hidden ||
      element.getAttribute("aria-hidden") === "true";

    return !isHidden;
  });
};

type AccessibleModalProps = {
  open: boolean;
  title: string;
  children: ReactNode;
  onClose: () => void;
};

export function AccessibleModal({
  open,
  title,
  children,
  onClose,
}: AccessibleModalProps) {
  const titleId = useId();
  const dialogRef = useRef<HTMLDivElement | null>(null);
  const previousActiveElementRef = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (!open) {
      return;
    }

    previousActiveElementRef.current =
      document.activeElement instanceof HTMLElement
        ? document.activeElement
        : null;

    const dialog = dialogRef.current;

    requestAnimationFrame(() => {
      if (!dialog) {
        return;
      }

      const focusableElements = getFocusableElements(dialog);
      const firstFocusableElement = focusableElements[0];

      if (firstFocusableElement) {
        firstFocusableElement.focus();
        return;
      }

      dialog.focus();
    });

    return () => {
      previousActiveElementRef.current?.focus();
    };
  }, [open]);

  useEffect(() => {
    if (!open) {
      return;
    }

    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.key === "Escape") {
        event.stopPropagation();
        onClose();
        return;
      }

      if (event.key !== "Tab") {
        return;
      }

      const dialog = dialogRef.current;

      if (!dialog) {
        return;
      }

      const focusableElements = getFocusableElements(dialog);

      if (focusableElements.length === 0) {
        event.preventDefault();
        dialog.focus();
        return;
      }

      const firstFocusableElement = focusableElements[0];
      const lastFocusableElement =
        focusableElements[focusableElements.length - 1];
      const activeElement = document.activeElement;

      if (event.shiftKey) {
        if (
          activeElement === firstFocusableElement ||
          !dialog.contains(activeElement)
        ) {
          event.preventDefault();
          lastFocusableElement.focus();
        }

        return;
      }

      if (activeElement === lastFocusableElement) {
        event.preventDefault();
        firstFocusableElement.focus();
      }
    };

    document.addEventListener("keydown", handleKeyDown);

    return () => {
      document.removeEventListener("keydown", handleKeyDown);
    };
  }, [open, onClose]);

  useEffect(() => {
    if (!open) {
      return;
    }

    const originalOverflow = document.body.style.overflow;
    document.body.style.overflow = "hidden";

    return () => {
      document.body.style.overflow = originalOverflow;
    };
  }, [open]);

  if (!open) {
    return null;
  }

  return createPortal(
    <div
      className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 px-4"
      onMouseDown={onClose}
    >
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby={titleId}
        tabIndex={-1}
        className="w-full max-w-md rounded-lg bg-white p-6 shadow-xl"
        onMouseDown={(event) => event.stopPropagation()}
      >
        <div className="flex items-start justify-between gap-4">
          <h2 id={titleId} className="text-lg font-semibold">
            {title}
          </h2>

          <button
            type="button"
            aria-label="모달 닫기"
            onClick={onClose}
            className="rounded px-2 py-1 text-sm"
          >
            닫기
          </button>
        </div>

        <div className="mt-4">{children}</div>
      </div>
    </div>,
    document.body,
  );
}
```

코드가 조금 길어 보이지만, 역할은 크게 네 가지입니다.

```txt
1. 모달이 열리면 이전 포커스 위치를 저장한다.
2. 모달 안의 첫 번째 포커스 가능 요소로 포커스를 이동한다.
3. Tab, Shift + Tab, Escape 키를 처리한다.
4. 모달이 닫히면 이전 포커스 위치로 되돌린다.
```

이제 중요한 부분만 하나씩 나눠서 보겠습니다.

---

## 모달이 열릴 때 포커스 이동하기

모달이 열렸을 때 포커스가 뒤쪽 페이지에 남아 있으면 키보드 사용자는 현재 위치를 알기 어렵습니다.

그래서 모달이 열리면 포커스를 모달 안으로 이동시켜야 합니다.

```tsx
previousActiveElementRef.current =
  document.activeElement instanceof HTMLElement
    ? document.activeElement
    : null;
```

먼저 모달을 열기 전에 포커스되어 있던 요소를 저장합니다.

보통은 모달을 연 버튼이 저장됩니다.

```tsx
requestAnimationFrame(() => {
  const focusableElements = getFocusableElements(dialog);
  const firstFocusableElement = focusableElements[0];

  if (firstFocusableElement) {
    firstFocusableElement.focus();
    return;
  }

  dialog.focus();
});
```

그다음 모달 안에서 포커스 가능한 첫 번째 요소로 포커스를 이동합니다.

만약 포커스 가능한 요소가 하나도 없다면 `dialog` 자체에 포커스를 줍니다. 그래서 dialog 컨테이너에 `tabIndex={-1}`을 지정했습니다.

```tsx
<div
  ref={dialogRef}
  role="dialog"
  aria-modal="true"
  aria-labelledby={titleId}
  tabIndex={-1}
>
```

`tabIndex={-1}`은 Tab 순서에 포함시키지는 않지만, 코드로는 `.focus()`를 호출할 수 있게 해줍니다.

---

## 모달이 닫히면 원래 위치로 포커스 되돌리기

모달을 닫은 뒤 포커스가 `body`나 엉뚱한 곳으로 가면 사용자는 다시 위치를 찾아야 합니다.

그래서 모달이 닫힐 때는 이전에 포커스되어 있던 요소로 되돌리는 것이 좋습니다.

```tsx
return () => {
  previousActiveElementRef.current?.focus();
};
```

예를 들어 사용자가 `프로필 수정 열기` 버튼을 눌러 모달을 열었다면, 모달을 닫은 뒤 다시 그 버튼으로 포커스가 돌아갑니다.

이 작은 처리가 키보드 사용자에게는 꽤 큰 차이를 만듭니다.

```txt
버튼에서 모달 열기
→ 모달 안으로 포커스 이동
→ 모달 닫기
→ 다시 버튼으로 포커스 복귀
```

사용자의 작업 흐름이 끊기지 않습니다.

---

## ESC로 모달 닫기

모달은 키보드로도 쉽게 닫을 수 있어야 합니다.

가장 일반적인 방식이 `Escape` 키입니다.

```tsx
if (event.key === "Escape") {
  event.stopPropagation();
  onClose();
  return;
}
```

`event.stopPropagation()`을 호출한 이유는 모달 안쪽에서 처리한 ESC 이벤트가 바깥의 다른 단축키 로직으로 퍼지는 것을 줄이기 위해서입니다.

예를 들어 페이지 전체에서 ESC를 다른 용도로 쓰고 있다면, 모달이 열려 있는 동안에는 모달 닫기가 우선이어야 합니다.

---

## Tab 키를 모달 안에 가두기

focus trap의 핵심은 `Tab` 키 처리입니다.

```tsx
if (event.key !== "Tab") {
  return;
}
```

`Tab`이 아닌 키는 무시합니다.

이제 모달 안의 포커스 가능한 요소들을 가져옵니다.

```tsx
const focusableElements = getFocusableElements(dialog);
```

포커스 가능한 요소가 없다면 dialog 자체에 포커스를 유지합니다.

```tsx
if (focusableElements.length === 0) {
  event.preventDefault();
  dialog.focus();
  return;
}
```

그다음 첫 번째 요소와 마지막 요소를 구합니다.

```tsx
const firstFocusableElement = focusableElements[0];
const lastFocusableElement =
  focusableElements[focusableElements.length - 1];
const activeElement = document.activeElement;
```

이제 두 가지 경우만 막으면 됩니다.

첫 번째 요소에서 `Shift + Tab`을 누르는 경우입니다.

```tsx
if (event.shiftKey) {
  if (
    activeElement === firstFocusableElement ||
    !dialog.contains(activeElement)
  ) {
    event.preventDefault();
    lastFocusableElement.focus();
  }

  return;
}
```

브라우저 기본 동작대로라면 첫 번째 요소 이전, 즉 모달 밖으로 이동할 수 있습니다.

그래서 기본 동작을 막고 마지막 요소로 포커스를 보냅니다.

두 번째는 마지막 요소에서 `Tab`을 누르는 경우입니다.

```tsx
if (activeElement === lastFocusableElement) {
  event.preventDefault();
  firstFocusableElement.focus();
}
```

이 경우도 브라우저 기본 동작대로라면 모달 밖으로 나갈 수 있으므로 첫 번째 요소로 되돌립니다.

---

## aria-labelledby로 모달 제목 연결하기

모달에는 사용자가 이해할 수 있는 이름이 있어야 합니다.

보통 모달 제목을 `aria-labelledby`로 연결합니다.

```tsx
const titleId = useId();
```

React의 `useId`를 사용하면 컴포넌트마다 안정적인 id를 만들 수 있습니다.

```tsx
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby={titleId}
>
  <h2 id={titleId}>프로필 수정</h2>
</div>
```

이렇게 하면 보조 기술이 이 dialog의 이름을 `프로필 수정`으로 인식할 수 있습니다.

만약 제목이 화면에 보이지 않는 모달이라면 `aria-label`을 직접 줄 수도 있습니다.

```tsx
<div role="dialog" aria-modal="true" aria-label="알림">
  ...
</div>
```

하지만 가능하다면 화면에 보이는 제목을 두고 `aria-labelledby`로 연결하는 편이 더 명확합니다.

---

## 바깥 클릭으로 닫기

모달 배경을 클릭하면 닫히는 UX도 자주 사용합니다.

```tsx
<div
  className="fixed inset-0 ..."
  onMouseDown={onClose}
>
  <div
    role="dialog"
    onMouseDown={(event) => event.stopPropagation()}
  >
    ...
  </div>
</div>
```

바깥 overlay에는 `onMouseDown={onClose}`를 달고, 모달 본문에서는 이벤트 전파를 막습니다.

그러면 본문을 클릭할 때는 닫히지 않고, 배경을 클릭할 때만 닫힙니다.

다만 바깥 클릭 닫기는 보조 기능에 가깝습니다.

접근성 관점에서는 닫기 버튼과 ESC 닫기가 더 중요합니다.

마우스를 쓰지 않는 사용자도 모달을 닫을 수 있어야 하기 때문입니다.

---

## body 스크롤 잠그기

모달이 열려 있을 때 뒤쪽 페이지가 스크롤되면 사용자가 현재 맥락을 잃기 쉽습니다.

간단하게는 `body`의 `overflow`를 잠글 수 있습니다.

```tsx
useEffect(() => {
  if (!open) {
    return;
  }

  const originalOverflow = document.body.style.overflow;
  document.body.style.overflow = "hidden";

  return () => {
    document.body.style.overflow = originalOverflow;
  };
}, [open]);
```

기존 값을 저장했다가 cleanup에서 되돌리는 점이 중요합니다.

무조건 빈 문자열로 되돌리면, 원래 페이지에서 설정해둔 `overflow` 값을 덮어버릴 수 있습니다.

---

## 여러 모달이 겹치는 경우는 어떻게 할까?

실무에서는 모달 위에 또 다른 모달이 열리는 경우도 있습니다.

예를 들어 로그인 모달 안에서 비밀번호 찾기 모달을 여는 상황입니다.

이때는 모든 모달이 각각 `keydown` 이벤트를 처리하면 문제가 생길 수 있습니다.

```txt
ESC 한 번 눌렀는데 여러 모달이 동시에 닫힘
아래쪽 모달의 focus trap까지 같이 동작함
```

그래서 중첩 모달을 지원한다면 **가장 위에 있는 모달만 키보드 이벤트를 처리**하도록 만들어야 합니다.

예를 들어 모달 stack을 관리하고 있다면 `isTopMost` 같은 값을 넘길 수 있습니다.

```tsx
<AccessibleModal
  open={isOpen}
  title="비밀번호 찾기"
  isTopMost={true}
  onClose={onClose}
>
  ...
</AccessibleModal>
```

그리고 keydown effect에서 위쪽 모달일 때만 동작하게 합니다.

```tsx
if (!open || !isTopMost) {
  return;
}
```

단일 모달만 사용하는 프로젝트라면 여기까지는 필요 없습니다.

하지만 전역 모달 시스템을 만든다면 꼭 고려해야 하는 부분입니다.

---

## 흔히 빠뜨리는 부분

### 1. 모달을 열고도 포커스를 이동하지 않기

모달이 화면에 보이더라도 포커스가 뒤쪽 페이지에 남아 있으면 키보드 사용자는 모달에 들어오지 못할 수 있습니다.

모달이 열리면 반드시 모달 안쪽으로 포커스를 옮겨야 합니다.

### 2. Tab이 모달 밖으로 빠져나가기

`role="dialog"`와 `aria-modal="true"`만 넣는다고 focus trap이 자동으로 생기지는 않습니다.

키보드 이벤트를 직접 처리하거나, 검증된 focus trap 라이브러리를 사용해야 합니다.

### 3. 닫은 뒤 포커스를 복귀하지 않기

닫기 버튼을 누른 뒤 포커스가 사라지면 다음 조작이 불편해집니다.

모달을 열기 전의 요소를 저장했다가 닫을 때 복귀시키는 흐름을 넣는 것이 좋습니다.

### 4. 닫기 버튼 없이 바깥 클릭에만 의존하기

바깥 클릭은 마우스 사용자에게는 편하지만 키보드 사용자에게는 의미가 없습니다.

명시적인 닫기 버튼과 ESC 닫기를 함께 제공해야 합니다.

### 5. `tabindex`에 양수를 사용하기

포커스 순서를 억지로 바꾸기 위해 `tabIndex={1}`, `tabIndex={2}`처럼 양수를 쓰는 것은 피하는 편이 좋습니다.

포커스 순서는 가능하면 DOM 순서와 자연스럽게 맞추고, 필요한 경우 `tabIndex={-1}` 정도만 사용하는 것이 안전합니다.

---

## 라이브러리를 써도 될까?

물론 가능합니다.

실무에서는 직접 구현보다 검증된 라이브러리를 쓰는 편이 더 안전할 때가 많습니다.

예를 들어 Headless UI, Radix UI, React Aria 같은 라이브러리는 dialog 접근성 처리를 상당 부분 대신 해줍니다.

하지만 직접 한 번 구현해보면 라이브러리를 사용할 때도 이해가 쉬워집니다.

```txt
왜 Dialog.Root가 필요한지
왜 Dialog.Title을 연결해야 하는지
왜 FocusScope 같은 컴포넌트가 있는지
왜 닫은 뒤 trigger로 포커스가 돌아오는지
```

이런 동작들이 전부 접근성 있는 모달을 만들기 위한 장치입니다.

---

## 정리

접근성 있는 모달은 단순히 화면 위에 떠 있는 박스가 아닙니다.

모달이 열리는 순간 사용자 흐름도 함께 모달 안으로 이동해야 합니다.

핵심만 다시 정리하면 다음과 같습니다.

- `role="dialog"`로 모달 영역의 역할을 알려줍니다.
- `aria-modal="true"`로 뒤쪽 페이지가 비활성이라는 의미를 전달합니다.
- `aria-labelledby`로 모달 제목을 연결합니다.
- 모달이 열릴 때 포커스를 모달 안으로 이동합니다.
- `Tab`, `Shift + Tab`이 모달 안에서만 순환되도록 만듭니다.
- `Escape` 키로 모달을 닫을 수 있게 합니다.
- 모달이 닫히면 원래 포커스 위치로 되돌립니다.
- 명시적인 닫기 버튼을 제공합니다.

한 줄로 요약하면 이렇습니다.

```txt
모달을 열었다면
시각적인 레이어뿐 아니라 포커스의 세계도 함께 모달 안으로 옮겨야 한다
```

처음에는 코드가 조금 길게 느껴질 수 있지만, 이 패턴을 한 번 만들어두면 로그인 모달, 확인 모달, 이미지 크롭 모달, 설정 모달 같은 UI에 계속 재사용할 수 있습니다.

## 참고

- [WAI-ARIA Authoring Practices Guide: Dialog Modal Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [React 공식 문서: createPortal](https://react.dev/reference/react-dom/createPortal)
- [MDN: Keyboard accessible](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Keyboard)
- [MDN: Keyboard-navigable JavaScript widgets](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets)
