---
title: "[React] createPortal로 DOM 밖으로 UI 보내기"
date: 2026-06-19T10:30:00Z
categories: [frontend]
tags: [react, createportal, portal, modal, tooltip]
description: "React createPortal이 무엇인지, DOM 구조와 React 트리를 어떻게 분리하는지, 그리고 모달과 툴팁 같은 UI에 어떻게 활용하는지 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 왜 굳이 DOM 밖으로 보내야 할까요?

React로 UI를 만들다 보면 요소가 화면의 다른 위치에 있어야 할 때가 있습니다.

예를 들어 이런 경우입니다.

- 모달이 부모 요소의 `overflow: hidden`에 잘리지 않아야 할 때
- 툴팁이나 드롭다운이 다른 레이아웃 위에 떠야 할 때
- 사이드바 안에 있는 버튼이지만 팝오버는 화면 최상단에서 렌더링하고 싶을 때
- z-index 충돌을 피하고 싶을 때

이런 상황에서 자주 쓰는 기능이 `createPortal`입니다.

처음 보면 이름이 조금 낯설 수 있지만, 핵심은 단순합니다.

```txt
React 트리는 그대로 유지하면서
실제 DOM만 다른 곳에 렌더링한다
```

이번 글에서는 `createPortal`이 정확히 무엇을 하는지, 왜 필요한지, 그리고 실무에서 어떻게 활용하는지 차근차근 정리해보겠습니다.

---

## 💡 createPortal이란?

React 공식 문서에서는 `createPortal`이 일부 children을 **다른 DOM 위치에 렌더링**할 수 있게 해준다고 설명합니다.

기본 사용 형태는 다음과 같습니다.

```tsx
import {createPortal} from 'react-dom';

createPortal(children, domNode);
```

예를 들어 이런 JSX가 있다고 해보겠습니다.

```tsx
import {createPortal} from 'react-dom';

export default function Example() {
  return (
    <div>
      <p>이 문장은 부모 div 안에 렌더링됩니다.</p>

      {createPortal(
        <p>이 문장은 body 안에 렌더링됩니다.</p>,
        document.body
      )}
    </div>
  );
}
```

화면상으로는 두 문장이 서로 다른 위치에 보입니다.

하지만 React 입장에서 두 번째 `<p>`는 여전히 `Example` 컴포넌트의 자식입니다.

즉, `createPortal`은 JSX의 부모-자식 관계를 바꾸는 것이 아니라, **실제 DOM에 꽂히는 위치만 바꾸는 도구**입니다.

---

## 📌 React 트리와 DOM 트리는 다를 수 있어요

`createPortal`을 이해할 때 가장 중요한 포인트는 이 부분입니다.

```txt
React 트리
→ 컴포넌트 관계와 상태 흐름을 나타냄

DOM 트리
→ 실제 브라우저에 그려지는 위치를 나타냄
```

`createPortal`은 DOM 위치만 바꿉니다.

그래서 이런 일이 가능합니다.

- React 상태와 context는 부모 컴포넌트에서 그대로 받습니다.
- 이벤트는 React 트리 기준으로 버블링됩니다.
- 화면에 그려지는 위치는 `document.body` 같은 별도 DOM 노드로 이동합니다.

이 성질 덕분에 모달이나 툴팁처럼 화면의 다른 계층 위에 떠 있어야 하는 UI를 만들기 좋습니다.

---

## 🧩 왜 그냥 div 안에 두면 안 될까요?

모달을 예로 들어보겠습니다.

부모 요소 중 하나가 이런 스타일을 가지고 있을 수 있습니다.

```css
overflow: hidden;
transform: translateY(0);
position: relative;
```

이런 환경에서는 모달이나 드롭다운이 부모 영역에 잘리거나, z-index 때문에 의도한 계층보다 아래에 깔릴 수 있습니다.

`createPortal`을 사용하면 이 문제를 피할 수 있습니다.

```txt
부모 안에 있어야 하는 건 React 구조
화면 위로 떠야 하는 건 DOM 위치
```

그래서 실제 렌더링 위치를 `body` 아래로 보내고, 스타일은 독립적으로 잡는 방식이 잘 맞습니다.

---

## 🛠️ 기본 예제

가장 단순한 포털 예시입니다.

```tsx
import {createPortal} from 'react-dom';

export default function PortalExample() {
  return (
    <div className="rounded-lg border p-4">
      <p>이 문장은 현재 영역 안에 있습니다.</p>

      {createPortal(
        <p className="fixed bottom-4 right-4 rounded bg-black px-3 py-2 text-white">
          이 문장은 body에 렌더링됩니다.
        </p>,
        document.body
      )}
    </div>
  );
}
```

이 코드를 보면 DOM 위치는 `document.body`로 빠져 있지만, 컴포넌트 안에서 아주 자연스럽게 작성하고 있습니다.

이게 `createPortal`의 매력입니다.

원하는 DOM 위치를 직접 붙잡으면서도, 코드 구조는 React 방식 그대로 유지할 수 있습니다.

---

## 🧭 실무에서 가장 많이 쓰는 곳: 모달

`createPortal`의 대표적인 사용처는 모달입니다.

모달은 보통 현재 페이지 위에 떠야 하고, 부모 레이아웃의 영향을 거의 받지 않아야 합니다.

```tsx
import {createPortal} from 'react-dom';

type ModalProps = {
  open: boolean;
  onClose: () => void;
};

export function Modal({open, onClose}: ModalProps) {
  if (!open) return null;

  return createPortal(
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50">
      <div className="w-[min(90vw,32rem)] rounded-lg bg-white p-6 shadow-xl">
        <h2 className="text-lg font-semibold">모달 제목</h2>
        <p className="mt-2 text-sm text-gray-600">포털로 렌더링된 모달입니다.</p>

        <div className="mt-6 flex justify-end">
          <button
            type="button"
            onClick={onClose}
            className="rounded bg-black px-4 py-2 text-white"
          >
            닫기
          </button>
        </div>
      </div>
    </div>,
    document.body
  );
}
```

이 구조가 좋은 이유는 분명합니다.

- 모달이 부모의 `overflow`에 잘리지 않습니다.
- `z-index`를 페이지 레이아웃과 분리해서 관리하기 쉽습니다.
- 모달 컴포넌트를 어디서 호출하든 같은 방식으로 동작합니다.

---

## 🧷 툴팁과 드롭다운에도 잘 어울려요

모달만큼 자주 쓰이는 곳이 툴팁과 드롭다운입니다.

특정 요소를 기준으로 위치를 계산한 뒤, 화면 최상단 레이어에 렌더링하는 방식입니다.

```tsx
import {createPortal} from 'react-dom';

type TooltipProps = {
  open: boolean;
  x: number;
  y: number;
  text: string;
};

export function Tooltip({open, x, y, text}: TooltipProps) {
  if (!open) return null;

  return createPortal(
    <div
      className="absolute rounded bg-gray-900 px-2 py-1 text-xs text-white shadow-lg"
      style={{left: x, top: y}}
    >
      {text}
    </div>,
    document.body
  );
}
```

이렇게 하면 툴팁을 부모 컴포넌트의 레이아웃 안에 억지로 넣지 않아도 됩니다.

툴팁은 화면에 떠 있어야 하니까요.

---

## 📦 context와 이벤트는 그대로 이어져요

포털을 처음 접하면 이런 걱정이 들 수 있습니다.

```txt
DOM이 body로 빠지면
React context나 이벤트는 끊기지 않을까?
```

React는 포털의 DOM 위치만 바꿉니다.

그래서 포털 안의 컴포넌트는 부모 트리의 context를 그대로 사용할 수 있습니다.

```tsx
const theme = useContext(ThemeContext);
```

또 이벤트도 React 트리 기준으로 버블링됩니다.

즉, 포털 내부를 클릭했을 때도 React 관점에서는 부모 컴포넌트의 이벤트 흐름을 이어받습니다.

이 점은 실무에서 꽤 중요합니다.

모달 내부 클릭 이벤트를 다룰 때, DOM 구조만 보고 판단하면 오해하기 쉽기 때문입니다.

---

## ⚠️ 주의할 점 1: domNode는 미리 존재해야 해요

React 공식 문서에서도 `domNode`는 이미 존재하는 DOM 노드여야 한다고 안내합니다.

```tsx
createPortal(children, document.body);
```

가장 흔한 예시는 `document.body`입니다.

하지만 특정 `div`에 렌더링하고 싶다면 그 노드가 먼저 DOM에 있어야 합니다.

```tsx
const target = document.getElementById('portal-root');

if (!target) return null;

return createPortal(<div>...</div>, target);
```

클라이언트에서만 DOM을 읽을 수 있으므로, Next.js 같은 환경에서는 보통 클라이언트 컴포넌트에서 사용합니다.

---

## ⚠️ 주의할 점 2: 포커스 관리가 필요해요

모달처럼 사용자와 상호작용하는 포털 UI는 접근성도 신경 써야 합니다.

특히 모달은 다음이 중요합니다.

- 열렸을 때 포커스가 모달 안으로 이동해야 합니다.
- 닫혔을 때 원래 포커스로 돌아가는 편이 좋습니다.
- Tab 키로 바깥 요소로 빠져나가지 않도록 제어해야 합니다.

React 공식 문서도 포털을 쓸 때 모달 접근성을 따르라고 안내합니다.

포털은 UI를 화면 위로 띄워주는 도구일 뿐, 접근성을 자동으로 해결해주지는 않습니다.

그래서 실제 프로젝트에서는 포커스트랩, `aria-modal`, `role="dialog"` 같은 요소를 함께 챙겨야 합니다.

---

## 🧪 Next.js에서 사용할 때

Next.js App Router에서는 보통 포털을 클라이언트 컴포넌트에서 사용합니다.

```tsx
'use client';

import {createPortal} from 'react-dom';
import {useEffect, useState} from 'react';

export default function ClientPortal() {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) return null;

  return createPortal(
    <div className="fixed bottom-4 right-4 rounded bg-black px-3 py-2 text-white">
      클라이언트에서 렌더링되는 포털
    </div>,
    document.body
  );
}
```

여기서 `mounted` 체크를 넣는 이유는 서버 렌더링 단계에서 `document`를 바로 쓰면 안 되기 때문입니다.

`createPortal` 자체는 React DOM API지만, 실제 DOM 노드를 가리켜야 하므로 브라우저 환경에서 안전하게 실행하는 흐름을 잡아주는 편이 좋습니다.

---

## 🔁 포털과 일반 렌더링의 차이

한 번 더 정리해보면 차이는 꽤 분명합니다.

| 항목 | 일반 렌더링 | createPortal |
| --- | --- | --- |
| DOM 위치 | 부모 요소 안 | 지정한 DOM 노드 |
| React 트리 | 부모-자식 관계 유지 | 부모-자식 관계 유지 |
| overflow 영향 | 받을 수 있음 | 피하기 쉬움 |
| z-index 관리 | 레이아웃에 따라 복잡 | 상대적으로 단순 |
| 대표 용도 | 일반 UI | 모달, 툴팁, 드롭다운 |

즉, `createPortal`은 화면 위에 떠야 하는 UI를 위한 도구라고 생각하면 이해가 쉽습니다.

---

## 🚧 자주 헷갈리는 부분

### 1. 포털을 쓰면 상태가 따로 놀지 않나요?

아니요. React 상태는 그대로입니다.

포털은 어디에 그릴지만 바꿉니다.

### 2. 포털 내부 클릭이 부모 이벤트에 영향을 주나요?

React 이벤트는 DOM이 아니라 React 트리 기준으로 버블링됩니다.

그래서 포털 안에서 클릭해도 부모 컴포넌트의 React 이벤트 핸들러가 호출될 수 있습니다.

### 3. 포털을 쓰면 무조건 `document.body`여야 하나요?

그렇지는 않습니다.

`body`가 가장 흔할 뿐이고, 앱 안에 별도의 `portal-root`를 만들어 쓰는 방식도 좋습니다.

### 4. 포털을 쓰면 무조건 더 좋은가요?

그렇지도 않습니다.

단순히 같은 컨테이너 안에서 해결할 수 있는 UI라면 굳이 포털이 필요하지 않을 수 있습니다.

포털은 구조를 복잡하게 만들 수 있으니, 정말 DOM 계층을 벗어나야 할 때 쓰는 편이 좋습니다.

---

## ✅ 정리

`createPortal`은 React 트리는 유지하면서 실제 DOM 렌더링 위치만 바꾸는 기능입니다.

핵심만 다시 보면 이렇습니다.

- JSX의 부모-자식 관계는 유지됩니다.
- 실제 DOM 위치만 `domNode`로 이동합니다.
- context와 React 이벤트는 그대로 이어집니다.
- 모달, 툴팁, 드롭다운처럼 화면 위에 떠야 하는 UI에 적합합니다.
- `domNode`는 미리 존재해야 하며, Next.js에서는 클라이언트 환경을 고려해야 합니다.
- 접근성, 특히 포커스 관리는 별도로 챙겨야 합니다.

처음에는 “왜 이렇게까지 해야 하지?” 싶을 수 있지만, 포털을 이해하면 모달 계층, 툴팁 위치, 드롭다운 충돌 같은 문제를 훨씬 깔끔하게 다룰 수 있습니다.

한 줄로 요약하면 이렇습니다.

```txt
React 구조는 그대로 두고
화면에 보이는 위치만 바꾼다
```

이 감각을 잡으면 `createPortal`은 더 이상 낯선 API가 아니라, 화면 위 계층을 다루는 꽤 유용한 도구가 됩니다.
