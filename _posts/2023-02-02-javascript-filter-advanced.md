---
title: "[React] 리스트 클릭 시 삭제하기: filter 함수 활용"
date: 2023-02-02T18:14:00
categories: [javascript, react]
tags: [react, javascript, filter, state, list-manipulation]
description: "React에서 배열의 filter 메서드를 활용해 리스트의 특정 요소를 삭제하는 방법을 단계별로 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 리스트에서 특정 항목을 삭제하려면?

React로 투두 리스트 같은 목록형 UI를 만들다 보면 삭제 기능은 거의 반드시 필요합니다.

예를 들어 사용자가 `X` 버튼을 누르면 해당 항목만 리스트에서 사라져야 합니다.

이때 자주 사용하는 메서드가 JavaScript 배열의 `filter`입니다.

이번 글에서는 `useState`로 리스트를 관리하고, `filter`를 사용해 클릭한 항목만 삭제하는 방법을 정리해보겠습니다.

---

## 🧱 기본 UI 만들기

먼저 삭제 기능을 넣기 전에 간단한 리스트 추가 UI를 만들어볼게요.

```js
import { useState } from "react";
import "./App.css";

function App() {
  const [list, setList] = useState([1, 2, 3, 4, 5, 6, 7]);
  const [text, setText] = useState("");

  const textOnChange = (e) => {
    setText(e.target.value);
  };

  const onSubmit = (e) => {
    e.preventDefault();
    setList((prevList) => [...prevList, text]);
    setText("");
  };

  return (
    <div className="container">
      <form className="textForm" onSubmit={onSubmit}>
        <input value={text} onChange={textOnChange} />
        <button type="submit">+</button>
      </form>

      <ul className="list">
        {list.map((data, index) => (
          <li key={index}>
            {data}
            <button>X</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

아직 `X` 버튼을 눌러도 아무 동작을 하지 않습니다. 이제 여기에 삭제 기능을 붙여보겠습니다.

스타일은 간단히 다음처럼 작성할 수 있습니다.

```css
body {
  height: 100vh;
  background-color: rgb(37, 37, 37);
  margin: 0;
}

#root {
  display: flex;
  justify-content: center;
  height: 100%;
}

.container {
  margin-top: 100px;
  width: 300px;
  height: 300px;
  background-color: white;
  border-radius: 10px;
  padding: 20px;
  box-sizing: border-box;
}

.textForm > input {
  width: 75%;
  outline: none;
  padding: 5px;
}

.textForm > button {
  width: 20%;
  margin-left: 5px;
  outline: none;
  cursor: pointer;
}

.list {
  width: 100%;
  padding-left: 0;
  list-style: none;
  margin-top: 20px;
  max-height: 200px;
  overflow-y: auto;
}

.list > li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
  padding: 5px;
  border-bottom: 1px solid #eee;
}
```

---

## 💡 삭제 기능의 핵심 아이디어

삭제 기능의 핵심은 간단합니다.

**클릭한 항목만 제외하고 나머지 항목으로 새로운 배열을 만드는 것**입니다.

예를 들어 다음 배열이 있다고 해볼게요.

```js
const list = ["A", "B", "C"];
```

여기서 `"B"`를 삭제하고 싶다면 결과는 다음처럼 되어야 합니다.

```js
["A", "C"];
```

이때 `filter`를 사용하면 쉽게 만들 수 있습니다.

```js
const newList = list.filter((item) => item !== "B");
```

React에서는 이 새 배열을 `setList`에 넣어주면 화면도 다시 렌더링됩니다.

---

## 🛠️ 클릭한 항목의 index로 삭제하기

이번 예제에서는 삭제 버튼을 클릭할 때 현재 항목의 `index`를 넘겨주겠습니다.

```js
const XBtnOnClick = (id) => {
  const newList = list.filter((_, index) => index !== id);
  setList(newList);
};
```

여기서 중요한 부분은 이 조건입니다.

```js
index !== id;
```

현재 순회 중인 요소의 인덱스가 클릭한 인덱스와 다르면 남기고, 같으면 제외합니다.

```txt
클릭한 index와 다르다 → 남김
클릭한 index와 같다 → 삭제
```

---

## ✅ 최종 코드

삭제 기능을 포함한 전체 코드는 다음과 같습니다.

```js
import { useState } from "react";
import "./App.css";

function App() {
  const [list, setList] = useState([1, 2, 3, 4, 5, 6, 7]);
  const [text, setText] = useState("");

  const textOnChange = (e) => {
    setText(e.target.value);
  };

  const onSubmit = (e) => {
    e.preventDefault();

    if (text === "") return;

    setList((prevList) => [...prevList, text]);
    setText("");
  };

  const XBtnOnClick = (id) => {
    // 클릭한 index와 다른 항목만 남깁니다.
    const newList = list.filter((_, index) => index !== id);
    setList(newList);
  };

  return (
    <div className="container">
      <form className="textForm" onSubmit={onSubmit}>
        <input value={text} onChange={textOnChange} />
        <button type="submit">+</button>
      </form>

      <ul className="list">
        {list.map((data, index) => (
          <li key={index}>
            {data}
            <button onClick={() => XBtnOnClick(index)}>X</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

---

## ⚠️ index를 key로 써도 될까요?

예제에서는 이해를 쉽게 하기 위해 `key={index}`를 사용했습니다.

하지만 실무에서는 리스트 항목에 고유한 `id`가 있다면 그 값을 `key`로 사용하는 편이 더 좋습니다.

```js
{todos.map((todo) => (
  <li key={todo.id}>{todo.text}</li>
))}
```

항목의 순서가 바뀌거나 중간에 추가/삭제되는 경우에는 `index` key가 예상치 못한 UI 문제를 만들 수 있기 때문입니다.

---

## ✅ 정리

React에서 리스트 삭제 기능을 만들 때는 `filter`를 자주 사용합니다.

- 삭제할 항목을 식별합니다.
- `filter`로 해당 항목만 제외한 새 배열을 만듭니다.
- 새 배열을 `setState`로 업데이트합니다.
- 원본 배열을 직접 수정하지 않기 때문에 React 상태 관리와 잘 맞습니다.

리스트 추가와 삭제는 React에서 정말 자주 쓰이는 패턴입니다. `filter`를 익혀두면 상태 배열을 다룰 때 훨씬 편해집니다.
