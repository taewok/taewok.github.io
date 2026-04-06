---
title: "[React] 리스트 클릭 시 삭제하기: filter 함수 활용 (ToDo List 예제)"
date: 2023-02-02T18:14:00
categories: [javascript, react]
tags: [react, javascript, filter, state, list-manipulation]
description: "React에서 배열의 filter 메서드를 활용하여 리스트의 특정 요소를 삭제하는 방법을 단계별로 알아봅니다. useState와 함께 ToDo List 삭제 기능을 구현해봅시다."
custom_style: true
---

React로 투두 리스트(ToDo List)와 같은 목록형 UI를 만들 때 가장 필수적인 기능 중 하나가 바로 **삭제 기능**입니다.  
이번 글에서는 자바스크립트의 `filter` 함수를 활용하여, **버튼 클릭 시 원하는 요소만 쏙 골라 삭제하는 기능**을 구현해 보겠습니다. ✂️

---

## 1. 기본 UI 및 기능 구현 (App.js, App.css)

먼저 삭제 기능을 넣기 전, 기본적인 리스트 추가 기능과 스타일을 잡아보겠습니다.

### App.js (기본 구조)

`useState`를 사용하여 리스트 데이터(`list`)와 입력값(`text`)을 관리합니다.

```javascript
import { useState } from "react";
import "./App.css";

function App() {
  // 리스트 데이터 관리
  const [list, setList] = useState([1, 2, 3, 4, 5, 6, 7]);
  // Input 입력값 관리
  const [text, setText] = useState("");

  // Input 값이 바뀔 때마다 실행
  const textOnChange = (e) => {
    setText(e.target.value);
  };

  // 엔터를 누르거나 + 버튼 클릭 시 실행 (리스트 추가)
  const onSubmit = (e) => {
    e.preventDefault();
    setList((prevList) => [...prevList, text]);
    setText(""); // 입력창 초기화
  };

  return (
    <div className="container">
      <form className="textForm" onSubmit={onSubmit}>
        <input value={text} onChange={textOnChange} />
        <button type="submit">+</button>
      </form>
      <ul className="list">
        {list.map((data, index) => (
          // key는 고유한 값이어야 합니다. (실무에서는 index보다 고유 ID 권장)
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

### App.css (스타일링)

간단한 카드 형태의 UI를 잡기 위한 CSS 코드입니다.

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
  background-color: rgb(255, 255, 255);
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

.list > li > button {
  background-color: transparent;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
}

.list > li > button:hover {
  background-color: goldenrod;
  color: white;
  border-color: goldenrod;
}
```

### 실행 화면 확인

위 코드를 실행하면 아래와 같이 값을 입력하여 리스트를 추가할 수 있는 깔끔한 UI가 생성됩니다.

![리스트 추가 화면](https://user-images.githubusercontent.com/88264006/216276223-01fbcb1d-e37e-47fc-b224-066131fd694e.png)

하지만 아직 'X' 버튼을 눌러도 아무런 반응이 없습니다. 이제 삭제 기능을 달아봅시다.

---

## 2. 핵심 로직: filter로 삭제 기능 구현하기

삭제 기능의 핵심은 **"내가 클릭한 요소만 빼고 나머지로 새로운 배열을 만드는 것"**입니다. 이때 `filter` 함수가 아주 유용하게 쓰입니다.

**로직 설명:**

1. 삭제 버튼을 클릭할 때 해당 요소의 `index`를 함수에 전달합니다.
2. `filter`를 사용하여 전체 리스트의 인덱스와 내가 클릭한 인덱스가 **일치하지 않는(`!==`)** 요소만 남깁니다.
3. 필터링된 새로운 배열을 `setList`로 업데이트합니다.

![삭제 버튼 인터랙션](https://user-images.githubusercontent.com/88264006/216278057-52f32dd4-0146-4134-9b92-fee2936f7aec.png)

---

## 3. 최종 완성 코드 (App.js)

기존 코드에 `XBtnOnClick` 함수를 추가하고, 버튼의 `onClick` 이벤트에 연결했습니다.

```javascript
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
    // 빈 값 입력 방지 (선택 사항)
    if (text === "") return;
    setList((prevList) => [...prevList, text]);
    setText("");
  };

  // [추가된 코드] X 버튼 클릭 시 해당 요소를 삭제하는 함수
  const XBtnOnClick = (id) => {
    // 클릭한 요소의 인덱스(id)와 다른 요소들만 남김
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
            {/* 버튼 클릭 시 현재 인덱스를 넘겨줍니다 */}
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

## 마치며

오늘은 React의 상태 관리(`useState`)와 자바스크립트의 배열 메서드(`filter`)를 조합하여 **리스트 삭제 기능**을 구현해 보았습니다.

이 패턴은 React 개발에서 정말 자주 사용되므로 꼭 숙지해 두시는 것이 좋습니다. 코드에 대해 궁금한 점이나 피드백이 있다면 언제든 댓글로 남겨주세요! 👋
