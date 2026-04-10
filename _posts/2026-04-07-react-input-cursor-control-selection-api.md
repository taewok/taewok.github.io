---
title: "[React] Input 커서 위치에 텍스트 삽입하기: useRef와 Selection API 활용법"
date: 2026-04-07T03:15:20
categories: [frontend]
tags: [react, useRef, selection-api, input-control]
description: "Input의 특정 커서 위치를 찾아 텍스트를 삽입하고, 커서를 자유자재로 이동시키는 실무 테크닉을 정리합니다."
custom_style: true
---

---

## 🚀 도입: "커서가 왜 맨 뒤로만 갈까?"

채팅 서비스나 에디터를 만들다 보면 특정 버튼(예: 이모지, 태그)을 눌렀을 때 **현재 커서가 위치한 곳**에 텍스트를 쏙 집어넣어야 할 때가 있습니다.

처음에는 단순히 `value = value + "문자열"` 식으로 코드를 짰는데, 이렇게 하니 커서가 어디에 있든 무조건 맨 뒤에만 글자가 붙는 문제가 발생했습니다. 사용자는 중간에 오타를 수정하다가 버튼을 눌렀는데, 커서가 뜬금없이 맨 뒤로 날아가 버리니 UX가 엉망이 되었죠.

이를 해결하기 위해 DOM에 직접 접근하는 **useRef**와 브라우저의 **Selection API**를 조합하여 문제를 해결한 과정을 기록합니다.

---

## 💡 핵심 개념: Selection API

`input`이나 `textarea` 엘리먼트는 현재 선택된 영역의 시작과 끝을 알려주는 프로퍼티를 내장하고 있습니다.

- **selectionStart**: 선택된 텍스트의 시작 인덱스 (커서 위치)
- **selectionEnd**: 선택된 텍스트의 끝 인덱스
- **setSelectionRange(start, end)**: 프로그래밍 방식으로 커서 위치를 강제 지정

이 값들을 활용하면 전체 문자열을 `[커서 앞 부분] + [추가할 문자열] + [커서 뒷 부분]` 형태로 조립할 수 있습니다.

---

## 🛠️ 실전 코드 예시: 특정 위치 삽입 & 커서 이동

아래는 버튼을 눌렀을 때 커서 위치에 `[추가]`라는 텍스트를 넣고, 커서를 삽입된 글자 바로 뒤로 옮겨주는 로직입니다.

```jsx
import { useRef, useState } from "react";

export default function InputEditor() {
  const [text, setText] = useState("");
  const inputRef = useRef(null);

  const insertTextAtCursor = () => {
    const input = inputRef.current;
    if (!input) return;

    // 1. 현재 커서 위치(인덱스) 파악
    const start = input.selectionStart;
    const end = input.selectionEnd;
    const insertValue = "[추가]";

    // 2. 기존 문자열을 커서 기준으로 쪼개서 재조립
    const newText =
      text.substring(0, start) + insertValue + text.substring(end);

    setText(newText);

    // 3. UX 핵심: 커서 위치 재설정
    // React의 상태 업데이트가 DOM에 반영된 직후 실행하기 위해 큐에 넣음
    setTimeout(() => {
      input.focus();
      const newCursorPos = start + insertValue.length;
      input.setSelectionRange(newCursorPos, newCursorPos);
    }, 0);
  };

  return (
    <div class="p-4 border rounded-lg bg-white">
      <input
        ref={inputRef}
        value={text}
        onChange={(e) => setText(e.target.value)}
        className="border p-2 w-64 rounded"
        placeholder="텍스트를 입력하세요"
      />
      <button
        onClick={insertTextAtCursor}
        className="ml-2 px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        텍스트 삽입
      </button>
    </div>
  );
}
```

---

## 📊 기능 비교: 제어 방식의 차이

| 항목        | 단순 State 결합        | Selection API 활용        |
| :---------- | :--------------------- | :------------------------ |
| 삽입 위치   | 항상 문자열의 `마지막` | 현재 `커서가 있는 위치`   |
| UX 품질     | 낮음 (맥락 끊김)       | 높음 (입력 흐름 유지)     |
| 구현 난이도 | 매우 쉬움              | 보통 (Ref 제어 필요)      |
| 주요 용도   | 단순 전송, 초기화      | 이모지, 멘션, 템플릿 삽입 |

---

## 📝 실전 포인트 & 인사이트

<div class="highlight-box">
    <strong>✅ 언제 써야 하는가?</strong><br/>
    단순한 입력 폼이 아니라, <b>멘션(@) 기능을 구현하거나 이모지 피커, 템플릿 문구 삽입</b> 기능을 만들 때 필수적입니다. 사용자가 입력 중인 맥락을 깨지 않는 것이 핵심입니다.
</div>

<div class="warning-box">
    <strong>⚠️ 실수하기 쉬운 부분</strong><br/>
    React에서 **setText**로 상태를 바꾼 직후에 바로 **setSelectionRange**를 호출하면 작동하지 않습니다. React가 리렌더링을 완료하고 실제 DOM에 반영하기 전이기 때문입니다. <code class="point-text">setTimeout(..., 0)</code>을 사용하거나 <code class="point-text">useEffect</code>를 활용해 렌더링 이후 시점을 잡아야 합니다.
</div>

<div class="highlight-box">
    <strong>🚀 개발 중 겪은 인사이트 (Troubleshooting)</strong><br/>
    버튼을 클릭하는 순간 `input`의 포커스가 버튼으로 옮겨가면서 커서 위치 정보가 소실되는 경우가 있습니다. 이럴 때는 버튼의 `onMouseDown` 이벤트에서 `e.preventDefault()`를 호출하여 포커스 탈취를 막거나, 위 코드처럼 작업 직후 강제로 <code class="point-text">input.focus()</code>를 다시 호출해줘야 합니다.
</div>

---

## ✅ 마무리하며

단순히 값을 넣는 것은 쉽지만, 사용자의 커서 흐름을 유지하는 것은 **한 끗 차이의 디테일**입니다. **useRef**를 단순히 DOM을 잡는 용도로만 쓰지 말고, 브라우저가 제공하는 표준 API인 **Selection API**와 결합하여 더 수준 높은 에디터 환경을 구축해 보시기 바랍니다.

이제 버튼을 눌렀을 때 커서가 맨 뒤로 튕겨 나가서 사용자가 다시 마우스를 잡아야 하는 불편함은 안녕입니다!
