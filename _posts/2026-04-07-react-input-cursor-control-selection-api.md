---
title: "[React] input 커서 위치에 텍스트 삽입하기"
date: 2026-04-07T03:15:20
categories: [frontend]
tags: [react, useRef, selection-api, input-control]
description: "React에서 useRef와 Selection API를 사용해 input의 현재 커서 위치에 텍스트를 삽입하고 커서 위치를 유지하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

채팅 입력창이나 에디터를 만들다 보면 버튼을 눌렀을 때 현재 커서 위치에 특정 문구를 삽입해야 할 때가 있습니다.

예를 들어 이모지, 멘션, 템플릿 문구를 입력 중인 위치에 넣는 기능입니다.

단순히 `value + "문구"`처럼 처리하면 항상 문자열 끝에만 추가됩니다.

사용자가 문장 중간을 수정하던 중이라면 커서 흐름이 깨져서 불편한 UX가 됩니다.

이럴 때는 `useRef`와 브라우저의 Selection API를 함께 사용할 수 있습니다.

---

## Selection API에서 사용할 값

`input`과 `textarea`는 현재 선택 영역과 커서 위치를 알려주는 속성을 가지고 있습니다.

| 속성 | 의미 |
| --- | --- |
| `selectionStart` | 선택 영역의 시작 인덱스 |
| `selectionEnd` | 선택 영역의 끝 인덱스 |
| `setSelectionRange(start, end)` | 커서 또는 선택 영역 위치를 지정 |

선택된 영역이 없다면 `selectionStart`와 `selectionEnd`는 같은 값입니다.

---

## 현재 커서 위치에 텍스트 삽입하기

```tsx
import { useRef, useState } from "react";

const InputEditor = () => {
  const [text, setText] = useState("");
  const inputRef = useRef<HTMLInputElement | null>(null);

  const insertTextAtCursor = () => {
    const input = inputRef.current;
    if (!input) return;

    const start = input.selectionStart ?? text.length;
    const end = input.selectionEnd ?? text.length;
    const insertValue = "[추가]";

    const nextText =
      text.slice(0, start) + insertValue + text.slice(end);

    setText(nextText);

    requestAnimationFrame(() => {
      const nextCursorPosition = start + insertValue.length;

      input.focus();
      input.setSelectionRange(nextCursorPosition, nextCursorPosition);
    });
  };

  return (
    <div>
      <input
        ref={inputRef}
        value={text}
        onChange={(event) => setText(event.target.value)}
        placeholder="텍스트를 입력해주세요"
      />

      <button type="button" onClick={insertTextAtCursor}>
        텍스트 삽입
      </button>
    </div>
  );
};

export default InputEditor;
```

핵심은 현재 커서 위치를 기준으로 문자열을 앞부분과 뒷부분으로 나눈 뒤, 그 사이에 새 텍스트를 넣는 것입니다.

---

## 커서 위치를 다시 맞추는 이유

React에서 `setText`로 상태를 바꾸면 input의 value가 다시 렌더링됩니다.

이때 커서 위치가 예상과 다르게 이동할 수 있습니다.

그래서 텍스트를 삽입한 뒤 `setSelectionRange`로 커서를 삽입한 텍스트 뒤쪽으로 다시 맞춰줍니다.

`requestAnimationFrame`을 사용한 이유는 React 상태 변경이 DOM에 반영된 뒤 커서 위치를 조정하기 위해서입니다.

---

## 선택 영역을 대체할 수도 있어요

사용자가 텍스트 일부를 드래그해서 선택한 상태라면, 선택된 영역을 삽입 텍스트로 대체할 수 있습니다.

```tsx
const nextText =
  text.slice(0, start) + insertValue + text.slice(end);
```

`start`와 `end`가 다르면 그 사이의 문자열은 제거되고 `insertValue`가 들어갑니다.

이 방식은 에디터에서 선택 영역을 굵게 감싸거나 템플릿으로 바꿀 때도 활용할 수 있습니다.

---

## 버튼 클릭 시 focus가 빠지는 문제

버튼을 클릭하는 순간 input에서 focus가 빠져 커서 위치를 잃는 경우가 있습니다.

이럴 때는 버튼의 `onMouseDown`에서 기본 동작을 막을 수 있습니다.

```tsx
<button
  type="button"
  onMouseDown={(event) => event.preventDefault()}
  onClick={insertTextAtCursor}
>
  텍스트 삽입
</button>
```

또는 삽입 후 `input.focus()`를 다시 호출하는 방식도 함께 사용할 수 있습니다.

---

## 마무리

input의 현재 커서 위치에 텍스트를 삽입하려면 `selectionStart`, `selectionEnd`, `setSelectionRange`를 사용하면 됩니다.

React에서는 `useRef`로 input DOM에 접근하고, 상태 업데이트 후 커서 위치를 다시 맞춰주는 흐름이 중요합니다.

이 방식은 이모지 삽입, 멘션, 자동완성, 템플릿 문구 삽입 같은 입력 UX를 만들 때 유용하게 사용할 수 있습니다.
