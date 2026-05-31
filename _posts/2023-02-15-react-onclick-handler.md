---
title: "[React] 클릭 이벤트 onClick 사용법 정리"
date: 2023-02-15T11:25:00
categories: [react]
tags: [react, onclick, event, frontend]
description: "React에서 요소를 클릭했을 때 이벤트를 처리하는 onClick의 기본 사용법과 핸들러 함수를 분리하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 React에서 클릭 이벤트는 어떻게 처리할까요?

React에서 사용자와 상호작용할 때 가장 자주 사용하는 이벤트 중 하나가 `onClick`입니다.

버튼을 클릭했을 때 콘솔을 찍거나, state를 변경하거나, API 요청을 보내는 등 다양한 작업을 할 수 있습니다.

이번 글에서는 `onClick`의 기본 사용법과 핸들러 함수를 분리하는 방법을 정리해보겠습니다.

---

## 🛠️ onClick 기본 사용법

React에서는 클릭 이벤트를 처리할 때 `onClick`을 사용합니다.

HTML의 `onclick`과 비슷해 보이지만, React에서는 카멜 케이스인 `onClick`으로 작성합니다.

```jsx
function App() {
  return (
    <div>
      <button onClick={() => console.log("click!")}>클릭</button>
    </div>
  );
}

export default App;
```

버튼을 클릭하면 콘솔에 `"click!"`이 출력됩니다.

---

## ⚠️ 함수 호출문을 바로 넣으면 안 돼요

주의할 점이 있습니다.

다음처럼 작성하면 클릭할 때 실행되는 것이 아니라, 컴포넌트가 렌더링될 때 바로 실행됩니다.

```jsx
<button onClick={console.log("click!")}>클릭</button>
```

`onClick`에는 실행 결과가 아니라 **실행할 함수 자체**를 넘겨야 합니다.

그래서 다음처럼 작성해야 합니다.

```jsx
<button onClick={() => console.log("click!")}>클릭</button>
```

또는 함수를 따로 분리해서 넘길 수도 있습니다.

---

## 🧩 핸들러 함수 분리하기

로직이 짧을 때는 인라인으로 작성해도 괜찮습니다.

하지만 코드가 길어지거나 여러 작업이 들어가면 핸들러 함수를 따로 분리하는 편이 읽기 좋습니다.

```jsx
function App() {
  const handleClick = () => {
    console.log("click");
  };

  return (
    <div>
      <button onClick={handleClick}>클릭</button>
    </div>
  );
}

export default App;
```

`onClick={handleClick}`처럼 함수 이름만 넘기면 클릭했을 때 React가 해당 함수를 실행합니다.

---

## 📦 이벤트 객체 받기

클릭한 요소의 정보가 필요하다면 이벤트 객체를 받을 수 있습니다.

```jsx
function App() {
  const handleClick = (event) => {
    console.log(event);
    console.log(event.target);
  };

  return (
    <div>
      <button onClick={handleClick}>클릭</button>
    </div>
  );
}

export default App;
```

React는 이벤트 핸들러의 첫 번째 인자로 이벤트 객체를 자동으로 넘겨줍니다.

따라서 굳이 다음처럼 한 번 더 감싸지 않아도 됩니다.

```jsx
<button onClick={(event) => handleClick(event)}>클릭</button>
```

물론 추가 인자를 함께 넘겨야 할 때는 화살표 함수로 감싸야 합니다.

```jsx
<button onClick={(event) => handleClick(event, 1)}>클릭</button>
```

---

## 🧠 state 변경에 활용하기

`onClick`은 state를 변경할 때도 자주 사용합니다.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  const increase = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={increase}>증가</button>
    </div>
  );
}
```

버튼을 누를 때마다 `count`가 1씩 증가합니다.

---

## ✅ 정리

React에서 클릭 이벤트를 처리할 때는 `onClick`을 사용합니다.

- React에서는 `onclick`이 아니라 `onClick`으로 작성합니다.
- `onClick`에는 함수 호출 결과가 아니라 함수 자체를 넘겨야 합니다.
- 로직이 길어지면 핸들러 함수를 분리하는 편이 좋습니다.
- 이벤트 객체는 핸들러의 첫 번째 인자로 받을 수 있습니다.
- 추가 인자를 넘길 때는 화살표 함수로 감싸면 됩니다.

클릭 이벤트는 React에서 가장 자주 쓰는 이벤트 중 하나라서 기본 패턴을 익혀두면 좋습니다.
