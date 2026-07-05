---
title: "[Next.js] Zustand로 undo, redo 기능 구현하기"
date: 2026-06-25T10:00:00Z
categories: [frontend]
tags: [nextjs, react, zustand, undo, redo, state-management]
description: "Next.js 프로젝트에서 Zustand를 활용해 past, present, future 구조로 undo와 redo 기능을 구현하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며: undo와 redo는 상태 기록 문제다

문서 편집기, 이미지 편집기, 폼 빌더, 캔버스 도구 같은 UI를 만들다 보면 사용자가 방금 한 작업을 되돌리고 싶어 하는 순간이 생겨요.

예를 들면 이런 기능입니다.

- 텍스트를 수정했다가 이전 내용으로 되돌리기
- 삭제한 블록 다시 복구하기
- 카드 순서를 바꿨다가 원래대로 돌리기
- 설정값을 변경한 뒤 이전 상태로 돌아가기

이때 필요한 기능이 `undo`와 `redo`예요.

겉으로 보면 단순히 "이전으로", "다시 실행" 버튼처럼 보이지만, 내부적으로는 상태를 어떻게 기록하고 되돌릴지의 문제입니다.

이번 글에서는 Next.js 프로젝트에서 Zustand를 사용해 undo, redo 기능을 직접 구현해보겠습니다.

---

## undo와 redo의 핵심 구조

undo와 redo는 보통 세 가지 상태로 생각하면 이해하기 쉬워요.

```ts
type HistoryState<T> = {
  past: T[];
  present: T;
  future: T[];
};
```

각 값의 역할은 이렇습니다.

- `past`: 이전 상태 목록
- `present`: 현재 상태
- `future`: 되돌린 뒤 다시 실행할 수 있는 상태 목록

예를 들어 현재 텍스트가 이렇게 변했다고 해볼게요.

```txt
""
↓
"안녕"
↓
"안녕하세요"
↓
"안녕하세요 Zustand"
```

현재 상태가 `"안녕하세요 Zustand"`라면 내부 구조는 대략 이렇게 볼 수 있어요.

```ts
{
  past: ["", "안녕", "안녕하세요"],
  present: "안녕하세요 Zustand",
  future: []
}
```

여기서 undo를 누르면 현재 상태가 `future`로 이동하고, `past`의 마지막 값이 `present`가 됩니다.

```ts
{
  past: ["", "안녕"],
  present: "안녕하세요",
  future: ["안녕하세요 Zustand"]
}
```

redo를 누르면 반대로 `future`의 첫 번째 값을 다시 `present`로 가져옵니다.

```ts
{
  past: ["", "안녕", "안녕하세요"],
  present: "안녕하세요 Zustand",
  future: []
}
```

이 구조만 이해하면 구현은 생각보다 단순해져요.

---

## 왜 Zustand와 잘 어울릴까?

undo, redo는 여러 컴포넌트에서 함께 사용할 가능성이 높아요.

예를 들어 편집기 화면을 생각해보면:

- 입력 영역은 현재 값을 바꾼다
- 툴바는 undo, redo 버튼을 보여준다
- 저장 버튼은 현재 값을 읽는다
- 미리보기는 현재 값을 구독한다

이 상태를 각 컴포넌트의 `useState`로 흩어놓으면 금방 복잡해질 수 있어요.

Zustand를 사용하면 편집 상태와 히스토리 조작 함수를 하나의 store에 모아둘 수 있습니다.

```txt
EditorInput
↓
updateText()
↓
Zustand Store
↓
Toolbar, Preview, SaveButton이 같은 상태를 구독
```

그래서 undo, redo처럼 여러 컴포넌트가 공유해야 하는 상태에는 Zustand가 꽤 잘 맞습니다.

---

## 설치하기

먼저 Zustand를 설치합니다.

```bash
npm install zustand
```

Next.js App Router에서 클라이언트 컴포넌트에서 사용할 store라면, store 파일 자체에는 굳이 `"use client"`를 붙이지 않아도 됩니다.

다만 store를 사용하는 컴포넌트는 브라우저에서 동작해야 하므로 `"use client"`가 필요해요.

---

## 가장 단순한 편집 상태부터 만들기

먼저 undo, redo 없이 현재 텍스트만 관리하는 store를 만들어볼게요.

```ts
// src/stores/useEditorStore.ts
import { create } from "zustand";

type EditorStore = {
  text: string;
  setText: (text: string) => void;
};

export const useEditorStore = create<EditorStore>((set) => ({
  text: "",
  setText: (text) => set({ text }),
}));
```

이 상태는 사용하기 쉽지만, 이전 값이 남지 않아요.

```txt
text: "안녕하세요"
↓
text: "안녕하세요 Zustand"
```

새 값으로 덮어쓰기 때문에 이전 상태를 되돌릴 방법이 없습니다.

그래서 이제 상태를 히스토리 구조로 바꿔보겠습니다.

---

## past, present, future 구조로 바꾸기

편집 상태를 다음처럼 나눕니다.

```ts
type EditorHistory = {
  past: string[];
  present: string;
  future: string[];
};
```

그리고 store에는 상태를 바꾸는 함수까지 함께 둡니다.

```ts
// src/stores/useEditorHistoryStore.ts
import { create } from "zustand";

type EditorHistoryStore = {
  past: string[];
  present: string;
  future: string[];
  updateText: (nextText: string) => void;
  undo: () => void;
  redo: () => void;
  reset: () => void;
};

export const useEditorHistoryStore = create<EditorHistoryStore>((set) => ({
  past: [],
  present: "",
  future: [],

  updateText: (nextText) =>
    set((state) => {
      if (state.present === nextText) {
        return state;
      }

      return {
        past: [...state.past, state.present],
        present: nextText,
        future: [],
      };
    }),

  undo: () =>
    set((state) => {
      if (state.past.length === 0) {
        return state;
      }

      const previous = state.past[state.past.length - 1];
      const newPast = state.past.slice(0, -1);

      return {
        past: newPast,
        present: previous,
        future: [state.present, ...state.future],
      };
    }),

  redo: () =>
    set((state) => {
      if (state.future.length === 0) {
        return state;
      }

      const next = state.future[0];
      const newFuture = state.future.slice(1);

      return {
        past: [...state.past, state.present],
        present: next,
        future: newFuture,
      };
    }),

  reset: () =>
    set({
      past: [],
      present: "",
      future: [],
    }),
}));
```

이 코드에서 가장 중요한 함수는 `updateText`, `undo`, `redo`예요.

---

## updateText 이해하기

`updateText`는 새로운 값을 현재 상태로 바꾸는 함수입니다.

```ts
updateText: (nextText) =>
  set((state) => {
    if (state.present === nextText) {
      return state;
    }

    return {
      past: [...state.past, state.present],
      present: nextText,
      future: [],
    };
  }),
```

여기서 중요한 흐름은 세 가지예요.

첫 번째, 현재 값과 새 값이 같으면 아무것도 하지 않습니다.

```ts
if (state.present === nextText) {
  return state;
}
```

같은 값을 계속 히스토리에 넣으면 undo를 눌러도 화면이 바뀌지 않는 이상한 경험이 생길 수 있어요.

두 번째, 현재 값을 `past`에 넣습니다.

```ts
past: [...state.past, state.present],
```

새로운 상태로 넘어가기 전에 지금 상태를 저장해두는 거예요.

세 번째, 새로운 값을 `present`로 바꾸고 `future`는 비웁니다.

```ts
present: nextText,
future: [],
```

왜 `future`를 비울까요?

undo를 한 뒤에 새로 입력을 시작하면, redo로 돌아갈 수 있던 미래 상태는 더 이상 유효하지 않기 때문이에요.

---

## undo 이해하기

undo는 `past`의 마지막 값을 현재 상태로 되돌리는 함수입니다.

```ts
undo: () =>
  set((state) => {
    if (state.past.length === 0) {
      return state;
    }

    const previous = state.past[state.past.length - 1];
    const newPast = state.past.slice(0, -1);

    return {
      past: newPast,
      present: previous,
      future: [state.present, ...state.future],
    };
  }),
```

흐름을 그림처럼 보면 이렇습니다.

```txt
past: ["A", "B"]
present: "C"
future: []

undo 실행

past: ["A"]
present: "B"
future: ["C"]
```

현재 상태 `"C"`는 완전히 사라지는 게 아니라 `future`로 이동해요.

그래야 redo를 눌렀을 때 다시 `"C"`로 돌아올 수 있습니다.

---

## redo 이해하기

redo는 undo와 반대로 움직입니다.

```ts
redo: () =>
  set((state) => {
    if (state.future.length === 0) {
      return state;
    }

    const next = state.future[0];
    const newFuture = state.future.slice(1);

    return {
      past: [...state.past, state.present],
      present: next,
      future: newFuture,
    };
  }),
```

흐름은 이렇게 볼 수 있어요.

```txt
past: ["A"]
present: "B"
future: ["C"]

redo 실행

past: ["A", "B"]
present: "C"
future: []
```

redo는 `future`에서 값을 하나 꺼내 현재 상태로 만들고, 기존 현재 상태는 다시 `past`에 넣습니다.

---

## Next.js 컴포넌트에서 사용하기

이제 store를 실제 화면에 연결해볼게요.

App Router 기준으로 클라이언트 컴포넌트에서 사용합니다.

```tsx
"use client";

import { useEditorHistoryStore } from "@/stores/useEditorHistoryStore";

export default function Editor() {
  const present = useEditorHistoryStore((state) => state.present);
  const updateText = useEditorHistoryStore((state) => state.updateText);

  return (
    <textarea
      value={present}
      onChange={(e) => updateText(e.target.value)}
      placeholder="내용을 입력해보세요."
    />
  );
}
```

이제 입력할 때마다 `present`가 바뀌고, 이전 값은 `past`에 쌓입니다.

---

## Toolbar에서 undo, redo 버튼 만들기

undo와 redo 버튼은 별도 컴포넌트로 분리할 수 있어요.

```tsx
"use client";

import { useEditorHistoryStore } from "@/stores/useEditorHistoryStore";

export default function EditorToolbar() {
  const undo = useEditorHistoryStore((state) => state.undo);
  const redo = useEditorHistoryStore((state) => state.redo);
  const canUndo = useEditorHistoryStore((state) => state.past.length > 0);
  const canRedo = useEditorHistoryStore((state) => state.future.length > 0);

  return (
    <div>
      <button type="button" onClick={undo} disabled={!canUndo}>
        Undo
      </button>
      <button type="button" onClick={redo} disabled={!canRedo}>
        Redo
      </button>
    </div>
  );
}
```

버튼 활성화 여부는 `past`와 `future` 길이로 판단합니다.

- `past.length > 0`: 되돌릴 상태가 있다
- `future.length > 0`: 다시 실행할 상태가 있다

이렇게 하면 더 이상 되돌릴 수 없을 때 버튼을 비활성화할 수 있어요.

---

## 입력할 때마다 히스토리가 너무 많이 쌓이는 문제

여기까지 구현하면 기능은 동작합니다.

하지만 textarea에 글자를 하나씩 입력할 때마다 히스토리가 쌓이는 문제가 생겨요.

```txt
"ㅎ"
"하"
"한"
"한ㄱ"
"한그"
"한글"
```

이러면 undo를 한 번 눌렀을 때 단어 단위가 아니라 글자 하나만 되돌아가서 사용성이 애매할 수 있습니다.

이 문제를 해결하는 방법은 여러 가지예요.

- 저장 버튼을 눌렀을 때만 히스토리에 기록하기
- debounce를 적용해서 입력이 멈췄을 때 기록하기
- 특정 액션 단위로만 기록하기
- 히스토리 최대 개수를 제한하기

이 글에서는 가장 이해하기 쉬운 방식으로 히스토리 최대 개수를 제한해보겠습니다.

---

## 히스토리 최대 개수 제한하기

히스토리를 무한히 쌓으면 메모리 사용량이 계속 늘어날 수 있어요.

그래서 최대 개수를 정해두는 게 좋습니다.

```ts
const MAX_HISTORY = 50;
```

`updateText`에서 `past`를 저장할 때 마지막 50개만 유지하도록 바꿔볼게요.

```ts
updateText: (nextText) =>
  set((state) => {
    if (state.present === nextText) {
      return state;
    }

    const nextPast = [...state.past, state.present].slice(-MAX_HISTORY);

    return {
      past: nextPast,
      present: nextText,
      future: [],
    };
  }),
```

이렇게 하면 오래된 히스토리는 자연스럽게 버리고, 최근 작업만 되돌릴 수 있습니다.

실제 편집기에서는 사용자가 기대하는 범위 안에서 적당한 값을 정하면 돼요.

---

## 최종 store 코드

위 내용을 합치면 store는 이렇게 정리할 수 있습니다.

```ts
// src/stores/useEditorHistoryStore.ts
import { create } from "zustand";

const MAX_HISTORY = 50;

type EditorHistoryStore = {
  past: string[];
  present: string;
  future: string[];
  updateText: (nextText: string) => void;
  undo: () => void;
  redo: () => void;
  reset: () => void;
};

export const useEditorHistoryStore = create<EditorHistoryStore>((set) => ({
  past: [],
  present: "",
  future: [],

  updateText: (nextText) =>
    set((state) => {
      if (state.present === nextText) {
        return state;
      }

      const nextPast = [...state.past, state.present].slice(-MAX_HISTORY);

      return {
        past: nextPast,
        present: nextText,
        future: [],
      };
    }),

  undo: () =>
    set((state) => {
      if (state.past.length === 0) {
        return state;
      }

      const previous = state.past[state.past.length - 1];
      const newPast = state.past.slice(0, -1);

      return {
        past: newPast,
        present: previous,
        future: [state.present, ...state.future],
      };
    }),

  redo: () =>
    set((state) => {
      if (state.future.length === 0) {
        return state;
      }

      const next = state.future[0];
      const newFuture = state.future.slice(1);

      return {
        past: [...state.past, state.present].slice(-MAX_HISTORY),
        present: next,
        future: newFuture,
      };
    }),

  reset: () =>
    set({
      past: [],
      present: "",
      future: [],
    }),
}));
```

이제 이 store만 있으면 여러 컴포넌트에서 같은 편집 상태와 undo, redo 기능을 함께 사용할 수 있습니다.

---

## 전체 화면 예시

간단한 화면으로 합치면 이렇게 사용할 수 있어요.

```tsx
"use client";

import { useEditorHistoryStore } from "@/stores/useEditorHistoryStore";

export default function EditorPage() {
  const present = useEditorHistoryStore((state) => state.present);
  const updateText = useEditorHistoryStore((state) => state.updateText);
  const undo = useEditorHistoryStore((state) => state.undo);
  const redo = useEditorHistoryStore((state) => state.redo);
  const reset = useEditorHistoryStore((state) => state.reset);
  const canUndo = useEditorHistoryStore((state) => state.past.length > 0);
  const canRedo = useEditorHistoryStore((state) => state.future.length > 0);

  return (
    <section>
      <div>
        <button type="button" onClick={undo} disabled={!canUndo}>
          Undo
        </button>
        <button type="button" onClick={redo} disabled={!canRedo}>
          Redo
        </button>
        <button type="button" onClick={reset}>
          Reset
        </button>
      </div>

      <textarea
        value={present}
        onChange={(e) => updateText(e.target.value)}
        placeholder="내용을 입력해보세요."
      />

      <pre>{present}</pre>
    </section>
  );
}
```

실제 UI에서는 버튼에 아이콘을 붙이거나, 단축키를 연결하면 더 자연스럽게 사용할 수 있어요.

---

## Ctrl+Z, Ctrl+Shift+Z 단축키 연결하기

편집기라면 버튼뿐 아니라 키보드 단축키도 기대하게 됩니다.

```tsx
"use client";

import { useEffect } from "react";
import { useEditorHistoryStore } from "@/stores/useEditorHistoryStore";

export function EditorShortcuts() {
  const undo = useEditorHistoryStore((state) => state.undo);
  const redo = useEditorHistoryStore((state) => state.redo);

  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      const isUndo = e.ctrlKey && e.key.toLowerCase() === "z" && !e.shiftKey;
      const isRedo = e.ctrlKey && e.shiftKey && e.key.toLowerCase() === "z";

      if (isUndo) {
        e.preventDefault();
        undo();
      }

      if (isRedo) {
        e.preventDefault();
        redo();
      }
    };

    window.addEventListener("keydown", handleKeyDown);

    return () => {
      window.removeEventListener("keydown", handleKeyDown);
    };
  }, [undo, redo]);

  return null;
}
```

macOS까지 고려한다면 `metaKey`도 함께 확인하면 됩니다.

```ts
const isModifierPressed = e.ctrlKey || e.metaKey;
```

단축키는 사용자가 이미 알고 있는 조작 방식이라, 편집 기능에서는 꽤 체감이 좋아요.

---

## 객체 상태에도 적용할 수 있을까?

지금은 문자열 하나만 다뤘지만, 실제 프로젝트에서는 객체 상태를 다루는 경우가 더 많아요.

예를 들어 폼 빌더라면 이런 상태가 될 수 있습니다.

```ts
type EditorState = {
  title: string;
  blocks: {
    id: string;
    type: "text" | "image";
    value: string;
  }[];
};
```

이때도 구조는 똑같습니다.

```ts
type EditorHistoryStore = {
  past: EditorState[];
  present: EditorState;
  future: EditorState[];
};
```

다만 객체나 배열을 다룰 때는 상태를 직접 수정하지 않고 새 객체를 만들어야 해요.

```ts
updateTitle: (title) =>
  set((state) => ({
    past: [...state.past, state.present],
    present: {
      ...state.present,
      title,
    },
    future: [],
  })),
```

상태가 깊어지면 `immer` 미들웨어를 함께 쓰는 방법도 있습니다.

---

## persist와 함께 써도 될까?

Zustand의 `persist` 미들웨어를 사용하면 상태를 localStorage에 저장할 수 있어요.

하지만 undo, redo 히스토리를 전부 저장할지는 신중하게 정하는 게 좋습니다.

예를 들어 사용자가 새로고침했을 때 현재 작성 중인 내용은 유지하고 싶지만, undo 히스토리까지 전부 유지할 필요는 없을 수 있어요.

그럴 때는 `present`만 저장하는 방향이 더 자연스럽습니다.

```txt
저장하면 좋은 값
present

굳이 저장하지 않아도 되는 값
past, future
```

히스토리는 현재 세션 안에서만 의미가 있는 경우가 많기 때문이에요.

---

## 구현할 때 주의할 점

undo, redo 기능을 만들 때는 다음 부분을 함께 생각하면 좋아요.

- 같은 상태를 중복으로 저장하지 않기
- 새 입력이 발생하면 `future` 비우기
- 히스토리 최대 개수 제한하기
- 너무 잦은 입력은 debounce나 액션 단위 저장 고려하기
- 객체 상태는 불변성을 지키며 업데이트하기
- 저장이 필요한 상태와 히스토리 상태를 구분하기

특히 `future`를 비우는 흐름은 중요합니다.

undo를 한 뒤 새 작업을 하면 이전 redo 경로는 더 이상 현재 작업 흐름과 맞지 않아요.

```txt
A → B → C
undo 해서 B로 돌아감
B에서 D를 새로 입력

이제 C로 redo하면 흐름이 어색해짐
```

그래서 새 작업이 들어오면 `future`를 비우는 것이 일반적인 동작입니다.

---

## 정리

Zustand로 undo, redo를 구현할 때 핵심은 어렵지 않습니다.

상태를 하나만 들고 있는 대신, 이전 상태와 미래 상태를 함께 관리하면 돼요.

```ts
{
  past: [],
  present: currentState,
  future: []
}
```

이 구조를 기준으로:

1. 상태가 바뀔 때 현재 값을 `past`에 저장한다
2. undo를 누르면 `past`의 마지막 값을 `present`로 가져온다
3. 기존 `present`는 `future`로 보낸다
4. redo를 누르면 `future`의 값을 다시 `present`로 가져온다
5. 새 작업이 시작되면 `future`를 비운다

이 흐름만 익히면 텍스트 편집기뿐 아니라 카드 정렬, 폼 빌더, 캔버스 도구 같은 기능에도 응용할 수 있습니다.

Zustand는 상태와 액션을 한곳에 모으기 쉬워서 undo, redo처럼 여러 컴포넌트가 함께 사용하는 기능과 잘 어울려요. 작게는 메모 입력부터 시작해서, 점점 더 복잡한 편집 경험으로 확장해보면 좋습니다.

## 참고

- [Zustand 공식 소개 문서](https://zustand.docs.pmnd.rs/getting-started/introduction)
- [Zustand Next.js 가이드](https://zustand.docs.pmnd.rs/guides/nextjs)
