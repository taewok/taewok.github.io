---
title: "[React] ref로 요소 안과 밖 클릭 구분하기"
date: 2026-06-27T10:00:00Z
categories: [frontend]
tags: [react, ref, useref, outside-click, dom-event]
description: "React에서 useRef, addEventListener, event.target, contains를 활용해 요소 내부 클릭과 외부 클릭을 구분하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며: 바깥을 클릭하면 닫히는 UI

드롭다운, 모달, 툴팁, 팝오버 같은 UI를 만들다 보면 이런 동작이 자주 필요해요.

```txt
메뉴 버튼 클릭
↓
드롭다운 열림
↓
드롭다운 바깥 영역 클릭
↓
드롭다운 닫힘
```

사용자 입장에서는 너무 자연스러운 동작이지만, 막상 구현하려고 하면 한 가지 질문이 생깁니다.

```txt
"지금 클릭한 곳이 이 요소 안쪽인지 바깥쪽인지 어떻게 알 수 있을까?"
```

이때 React의 `ref`와 브라우저 DOM API를 함께 사용합니다.

이번 글에서는 `useRef`만 살짝 쓰고 넘어가지 않고, 함께 등장하는 내장 기능들까지 하나씩 이해해보려고 합니다.

---

## 핵심 흐름 먼저 보기

요소 안과 밖 클릭을 구분하는 흐름은 이렇습니다.

```txt
1. useRef로 기준이 되는 DOM 요소를 잡는다
2. document에 click 이벤트를 등록한다
3. 사용자가 클릭하면 event.target을 확인한다
4. ref.current.contains(event.target)로 내부 클릭인지 검사한다
5. 내부 클릭이면 그대로 두고, 외부 클릭이면 닫는다
6. 컴포넌트가 사라질 때 이벤트를 정리한다
```

코드로 보면 가장 기본 형태는 이렇게 생겼어요.

```tsx
"use client";

import { useEffect, useRef, useState } from "react";

export default function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      const target = event.target;

      if (!(target instanceof Node)) {
        return;
      }

      if (
        dropdownRef.current &&
        !dropdownRef.current.contains(target)
      ) {
        setIsOpen(false);
      }
    };

    document.addEventListener("mousedown", handleClickOutside);

    return () => {
      document.removeEventListener("mousedown", handleClickOutside);
    };
  }, []);

  return (
    <div ref={dropdownRef}>
      <button type="button" onClick={() => setIsOpen((prev) => !prev)}>
        메뉴
      </button>

      {isOpen && (
        <div>
          <button type="button">프로필</button>
          <button type="button">설정</button>
          <button type="button">로그아웃</button>
        </div>
      )}
    </div>
  );
}
```

처음 보면 몇 가지가 낯설 수 있어요.

- `useRef`
- `ref.current`
- `document.addEventListener`
- `event.target`
- `contains`
- `removeEventListener`
- `target instanceof Node`

이제 하나씩 나눠서 보겠습니다.

---

## useRef는 무엇을 해줄까?

React에서 `useRef`는 렌더링 사이에 값을 기억할 수 있게 해주는 Hook이에요.

그중 DOM 요소를 다룰 때는 특정 HTML 요소를 가리키는 용도로 자주 사용합니다.

```tsx
const dropdownRef = useRef<HTMLDivElement | null>(null);
```

여기서 `dropdownRef`는 이런 모양의 객체입니다.

```ts
{
  current: null
}
```

처음에는 아직 DOM이 만들어지기 전이라 `current`가 `null`이에요.

그리고 JSX에 `ref`를 연결합니다.

```tsx
<div ref={dropdownRef}>
  ...
</div>
```

React가 이 `div`를 실제 DOM으로 만든 뒤에는 `dropdownRef.current`에 해당 DOM 요소를 넣어줍니다.

```ts
dropdownRef.current; // HTMLDivElement
```

즉, `useRef`는 "이 div를 나중에 코드에서 직접 확인하고 싶다"는 표시라고 볼 수 있어요.

React 공식 문서에서도 DOM 노드에 접근해야 할 때 ref를 사용할 수 있다고 설명합니다.

---

## ref.current는 왜 null일 수 있을까?

`useRef<HTMLDivElement | null>(null)`처럼 타입에 `null`이 들어가는 이유가 있어요.

컴포넌트가 처음 렌더링되기 전에는 아직 실제 DOM 요소가 없습니다.

```txt
컴포넌트 함수 실행
↓
JSX 반환
↓
React가 DOM 생성
↓
ref.current에 DOM 연결
```

그래서 클릭 판별을 할 때는 항상 `dropdownRef.current`가 있는지 먼저 확인해야 합니다.

```ts
if (!dropdownRef.current) {
  return;
}
```

혹은 조건문 안에서 이렇게 바로 사용할 수도 있어요.

```ts
if (
  dropdownRef.current &&
  !dropdownRef.current.contains(target)
) {
  setIsOpen(false);
}
```

이 확인을 빼먹으면 `null.contains(...)`처럼 실행되어 에러가 날 수 있습니다.

---

## document.addEventListener는 왜 사용할까?

바깥 클릭을 감지하려면 특정 버튼이나 드롭다운 안쪽만 보면 부족합니다.

왜냐하면 사용자는 화면 어디든 클릭할 수 있기 때문이에요.

그래서 클릭 이벤트를 `document`에 등록합니다.

```ts
document.addEventListener("mousedown", handleClickOutside);
```

`document`는 현재 페이지 전체 문서를 의미해요.

여기에 이벤트를 등록하면 페이지 어딘가에서 마우스가 눌렸을 때 `handleClickOutside`가 실행됩니다.

MDN 문서에서 `addEventListener`는 특정 이벤트가 전달될 때 실행할 함수를 등록하는 메서드라고 설명합니다.

즉, 이 코드는 이런 뜻이에요.

```txt
문서 전체에서 mousedown 이벤트가 발생하면
handleClickOutside 함수를 실행해줘
```

---

## click 대신 mousedown을 쓰는 이유

외부 클릭 감지 예제에서는 `click`도 많이 쓰고, `mousedown`도 많이 씁니다.

```ts
document.addEventListener("click", handleClickOutside);
```

```ts
document.addEventListener("mousedown", handleClickOutside);
```

둘 다 가능하지만, 동작 타이밍이 조금 달라요.

- `mousedown`: 마우스 버튼을 누르는 순간 발생
- `click`: 누르고 뗀 뒤 발생

드롭다운이나 팝오버처럼 빠르게 닫혀야 하는 UI에서는 `mousedown`을 쓰는 경우가 많습니다.

하지만 버튼 클릭 이벤트와의 순서가 중요하거나, 클릭 완료 후 처리하고 싶다면 `click`이 더 자연스러울 수 있어요.

핵심은 어떤 이벤트를 쓰든 구조는 같다는 점입니다.

```txt
document에서 이벤트를 듣고
event.target이 기준 요소 안에 있는지 검사한다
```

---

## event.target은 무엇일까?

이벤트 핸들러 안에서 클릭된 요소를 알고 싶을 때 `event.target`을 사용합니다.

```ts
const target = event.target;
```

MDN 문서에서는 `event.target`을 이벤트가 실제로 발생한 객체라고 설명합니다.

예를 들어 이런 구조가 있다고 해볼게요.

```tsx
<div>
  <button>
    <span>메뉴</span>
  </button>
</div>
```

사용자가 `span` 글자를 클릭했다면 `event.target`은 `span`일 수 있어요.

```txt
클릭한 실제 지점: span
event.target: span
```

반대로 이벤트 리스너를 등록한 곳은 `document`입니다.

이 차이를 이해하는 게 중요해요.

```txt
event.target
→ 실제로 클릭된 요소

document
→ 이벤트를 듣고 있는 대상
```

우리는 "사용자가 실제로 어디를 클릭했는지"가 필요하므로 `event.target`을 확인합니다.

---

## contains는 무엇을 검사할까?

이제 가장 중요한 `contains`입니다.

```ts
dropdownRef.current.contains(target)
```

`contains`는 어떤 DOM 노드가 다른 노드를 포함하고 있는지 확인하는 메서드예요.

예를 들어 이런 구조가 있다고 해볼게요.

```html
<div id="dropdown">
  <button>메뉴</button>
  <ul>
    <li>프로필</li>
  </ul>
</div>
```

`dropdown` 요소는 내부의 `button`, `ul`, `li`를 모두 포함합니다.

```ts
dropdown.contains(button); // true
dropdown.contains(li); // true
```

하지만 드롭다운 바깥에 있는 요소는 포함하지 않아요.

```ts
dropdown.contains(outsideButton); // false
```

그래서 외부 클릭 판별은 이렇게 할 수 있습니다.

```ts
const isInside = dropdownRef.current.contains(target);

if (!isInside) {
  setIsOpen(false);
}
```

읽어보면 그대로예요.

```txt
클릭된 target이 dropdown 안에 포함되어 있지 않다면
드롭다운을 닫는다
```

---

## target instanceof Node는 왜 필요할까?

TypeScript를 사용하면 이 부분에서 자주 막힙니다.

```ts
dropdownRef.current.contains(event.target);
```

이 코드가 타입 에러를 낼 수 있어요.

이유는 `event.target`의 타입이 `EventTarget | null`이기 때문입니다.

그런데 `contains`는 `Node` 타입을 받습니다.

```ts
contains(otherNode: Node | null): boolean
```

즉, TypeScript 입장에서는 이렇게 생각합니다.

```txt
event.target이 정말 DOM Node인지 확실하지 않은데?
```

그래서 먼저 확인해줍니다.

```ts
const target = event.target;

if (!(target instanceof Node)) {
  return;
}
```

이 조건문을 통과한 뒤에는 TypeScript가 `target`을 `Node`로 이해할 수 있어요.

그다음 `contains`에 안전하게 넘길 수 있습니다.

```ts
dropdownRef.current.contains(target);
```

이 코드는 단순한 타입 회피가 아니라, 브라우저 이벤트의 타입 구조를 안전하게 다루는 방식입니다.

---

## removeEventListener는 왜 꼭 필요할까?

이벤트를 등록했다면 정리도 해야 합니다.

```ts
document.addEventListener("mousedown", handleClickOutside);

return () => {
  document.removeEventListener("mousedown", handleClickOutside);
};
```

React 컴포넌트는 화면에서 사라질 수 있어요.

그런데 컴포넌트가 사라진 뒤에도 `document`에 이벤트 리스너가 남아 있으면, 필요 없는 함수가 계속 실행될 수 있습니다.

이런 문제가 생길 수 있어요.

- 이미 사라진 컴포넌트의 상태를 바꾸려고 함
- 이벤트 리스너가 중복 등록됨
- 디버깅하기 어려운 동작이 생김

그래서 `useEffect` 안에서 이벤트를 등록했다면, return 함수에서 제거하는 습관이 중요합니다.

MDN 문서에서도 `removeEventListener`는 등록된 이벤트 리스너를 제거하는 메서드라고 설명합니다.

---

## useEffect 안에서 등록하는 이유

이벤트 등록은 렌더링 중에 바로 하면 안 됩니다.

```tsx
document.addEventListener("mousedown", handleClickOutside);
```

이 코드를 컴포넌트 함수 본문에 직접 쓰면 렌더링될 때마다 이벤트가 계속 등록될 수 있어요.

그래서 `useEffect` 안에서 한 번 등록하고, 정리 함수에서 제거합니다.

```tsx
useEffect(() => {
  const handleClickOutside = (event: MouseEvent) => {
    // ...
  };

  document.addEventListener("mousedown", handleClickOutside);

  return () => {
    document.removeEventListener("mousedown", handleClickOutside);
  };
}, []);
```

의존성 배열을 빈 배열로 두면 컴포넌트가 마운트될 때 한 번 등록되고, 언마운트될 때 정리됩니다.

Next.js App Router에서는 이 코드가 브라우저에서만 실행되어야 하므로, 사용하는 컴포넌트 상단에 `"use client"`를 붙여야 합니다.

---

## 드롭다운 예제 다시 보기

이제 전체 코드를 다시 보면 훨씬 잘 읽힐 거예요.

```tsx
"use client";

import { useEffect, useRef, useState } from "react";

export default function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      const target = event.target;

      if (!(target instanceof Node)) {
        return;
      }

      const isOutside =
        dropdownRef.current &&
        !dropdownRef.current.contains(target);

      if (isOutside) {
        setIsOpen(false);
      }
    };

    document.addEventListener("mousedown", handleClickOutside);

    return () => {
      document.removeEventListener("mousedown", handleClickOutside);
    };
  }, []);

  return (
    <div ref={dropdownRef}>
      <button type="button" onClick={() => setIsOpen((prev) => !prev)}>
        메뉴
      </button>

      {isOpen && (
        <div>
          <button type="button">프로필</button>
          <button type="button">설정</button>
          <button type="button">로그아웃</button>
        </div>
      )}
    </div>
  );
}
```

이제 이 코드는 이렇게 읽을 수 있습니다.

```txt
dropdownRef가 감싸는 영역을 기준으로 삼고
문서 전체에서 mousedown을 듣다가
클릭된 target이 dropdownRef 안에 없으면
드롭다운을 닫는다
```

---

## 커스텀 훅으로 분리하기

외부 클릭 감지는 여러 곳에서 반복해서 쓰기 쉬워요.

그래서 커스텀 훅으로 분리하면 더 편합니다.

```tsx
import { RefObject, useEffect } from "react";

type UseOutsideClickOptions<T extends HTMLElement> = {
  ref: RefObject<T | null>;
  enabled?: boolean;
  onOutsideClick: () => void;
};

export const useOutsideClick = <T extends HTMLElement>({
  ref,
  enabled = true,
  onOutsideClick,
}: UseOutsideClickOptions<T>) => {
  useEffect(() => {
    if (!enabled) {
      return;
    }

    const handleMouseDown = (event: MouseEvent) => {
      const target = event.target;

      if (!(target instanceof Node)) {
        return;
      }

      if (ref.current && !ref.current.contains(target)) {
        onOutsideClick();
      }
    };

    document.addEventListener("mousedown", handleMouseDown);

    return () => {
      document.removeEventListener("mousedown", handleMouseDown);
    };
  }, [enabled, onOutsideClick, ref]);
};
```

사용하는 쪽은 훨씬 단순해집니다.

```tsx
"use client";

import { useRef, useState } from "react";
import { useOutsideClick } from "@/hooks/useOutsideClick";

export default function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement | null>(null);

  useOutsideClick({
    ref: dropdownRef,
    enabled: isOpen,
    onOutsideClick: () => setIsOpen(false),
  });

  return (
    <div ref={dropdownRef}>
      <button type="button" onClick={() => setIsOpen((prev) => !prev)}>
        메뉴
      </button>

      {isOpen && <div>드롭다운 내용</div>}
    </div>
  );
}
```

여기서 `enabled: isOpen`을 넣은 이유는 드롭다운이 닫혀 있을 때는 굳이 외부 클릭을 감지할 필요가 없기 때문이에요.

---

## onOutsideClick이 자주 바뀌는 문제

위 훅은 이해하기 쉽지만, `onOutsideClick` 함수가 렌더링마다 새로 만들어지면 effect도 다시 실행될 수 있어요.

작은 UI에서는 큰 문제가 되지 않지만, 더 안정적으로 만들고 싶다면 사용하는 쪽에서 `useCallback`을 사용할 수 있습니다.

```tsx
const handleOutsideClick = useCallback(() => {
  setIsOpen(false);
}, []);

useOutsideClick({
  ref: dropdownRef,
  enabled: isOpen,
  onOutsideClick: handleOutsideClick,
});
```

이렇게 하면 `onOutsideClick` 함수 참조가 불필요하게 바뀌는 일을 줄일 수 있습니다.

다만 처음 학습할 때는 `useCallback`보다 외부 클릭 판별 흐름을 먼저 이해하는 게 더 중요해요.

---

## 여러 ref를 기준으로 삼고 싶다면?

가끔은 하나의 요소가 아니라 여러 요소를 내부 영역으로 보고 싶을 때가 있어요.

예를 들어 버튼과 메뉴가 DOM상 떨어져 있을 수 있습니다.

```txt
button 영역
menu 영역
둘 다 클릭해도 닫히면 안 됨
```

이럴 때는 ref 배열을 받을 수 있게 만들 수 있어요.

```tsx
import { RefObject, useEffect } from "react";

type UseOutsideClickOptions<T extends HTMLElement> = {
  refs: RefObject<T | null>[];
  enabled?: boolean;
  onOutsideClick: () => void;
};

export const useOutsideClick = <T extends HTMLElement>({
  refs,
  enabled = true,
  onOutsideClick,
}: UseOutsideClickOptions<T>) => {
  useEffect(() => {
    if (!enabled) {
      return;
    }

    const handleMouseDown = (event: MouseEvent) => {
      const target = event.target;

      if (!(target instanceof Node)) {
        return;
      }

      const isInside = refs.some((ref) =>
        ref.current?.contains(target),
      );

      if (!isInside) {
        onOutsideClick();
      }
    };

    document.addEventListener("mousedown", handleMouseDown);

    return () => {
      document.removeEventListener("mousedown", handleMouseDown);
    };
  }, [enabled, onOutsideClick, refs]);
};
```

핵심은 같습니다.

```txt
여러 ref 중 하나라도 target을 포함하면 내부 클릭
그 어느 ref에도 포함되지 않으면 외부 클릭
```

---

## 모달에서 사용할 때 주의할 점

모달에서도 외부 클릭으로 닫는 패턴을 자주 사용합니다.

하지만 모달은 드롭다운보다 접근성 요구사항이 더 큽니다.

외부 클릭 닫기만 넣고 끝내기보다 다음도 함께 고려해야 해요.

- `Escape` 키로 닫기
- focus가 모달 안에 머물도록 관리하기
- 모달이 열렸을 때 배경 스크롤 막기
- `role="dialog"`와 `aria-modal="true"` 지정하기

외부 클릭 감지는 모달 UX의 일부일 뿐이에요.

그래서 모달처럼 중요한 UI에서는 `ref` 외부 클릭 로직과 접근성 처리를 함께 설계하는 게 좋습니다.

---

## 자주 하는 실수

### 1. ref를 기준 요소에 연결하지 않기

```tsx
const dropdownRef = useRef<HTMLDivElement | null>(null);

return <div>...</div>;
```

이렇게 ref를 만들기만 하고 JSX에 연결하지 않으면 `ref.current`는 계속 `null`입니다.

반드시 기준 요소에 연결해야 해요.

```tsx
return <div ref={dropdownRef}>...</div>;
```

### 2. removeEventListener를 빼먹기

이벤트를 등록하고 제거하지 않으면 컴포넌트가 사라진 뒤에도 이벤트가 남을 수 있어요.

항상 cleanup을 함께 둡니다.

```ts
return () => {
  document.removeEventListener("mousedown", handleMouseDown);
};
```

### 3. event.target을 바로 contains에 넣기

TypeScript에서는 `event.target`이 `Node`라고 보장되지 않아요.

그래서 먼저 확인해주는 게 안전합니다.

```ts
if (!(event.target instanceof Node)) {
  return;
}
```

### 4. 내부 버튼 클릭까지 닫혀버리는 구조

기준 ref가 너무 좁게 잡혀 있으면 내부라고 생각한 영역이 실제로는 바깥으로 판별될 수 있어요.

드롭다운 전체를 감싸는 부모에 ref를 붙였는지 확인해야 합니다.

---

## 정리

React에서 ref로 안쪽 클릭과 바깥 클릭을 구분하는 핵심은 단순합니다.

```txt
기준 DOM을 ref로 잡고
document에서 클릭을 듣고
event.target이 기준 DOM 안에 포함되는지 contains로 검사한다
```

이때 함께 이해해야 하는 기능은 다음과 같아요.

- `useRef`: DOM 요소를 기억하기 위해 사용
- `ref.current`: 실제 DOM 노드가 들어가는 위치
- `document.addEventListener`: 문서 전체 이벤트 감지
- `event.target`: 실제 클릭된 요소
- `Node`: DOM 트리의 기본 단위
- `contains`: 특정 노드가 다른 노드를 포함하는지 검사
- `removeEventListener`: 등록한 이벤트 정리
- `useEffect`: 이벤트 등록과 정리 타이밍 관리

외부 클릭 감지는 드롭다운, 모달, 툴팁, 팝오버에서 정말 자주 쓰이는 패턴이에요.

한 번 구조를 이해해두면 UI를 만들 때 훨씬 편해집니다. 특히 `contains(event.target)`의 의미를 제대로 잡아두면, "왜 안쪽 클릭인데 닫히지?" 같은 문제를 훨씬 빠르게 디버깅할 수 있어요.

## 참고

- [React 공식 문서: Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs)
- [MDN: EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [MDN: Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)
- [MDN: EventTarget.removeEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)
