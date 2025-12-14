---
title: "[React] 리액트 클릭 이벤트 onClick 완벽 가이드 (기본 문법, 함수 분리)"
date: 2023-02-15T11:25:00
categories: [React]
tags: [react, onclick, event, frontend]
description: "React에서 요소를 클릭했을 때 이벤트를 처리하는 onClick의 기본 사용법과 핸들러 함수를 분리하여 매개변수(Event 객체)를 다루는 방법까지 알아봅니다."
custom_style: true
---

React에서 사용자와 상호작용하기 위해 가장 많이 사용하는 이벤트 중 하나가 바로 **`onClick`**입니다.  
HTML의 `onclick` 속성과 비슷하지만, 리액트만의 문법(CamelCase, 함수 전달 방식)을 따르므로 정확한 사용법을 아는 것이 중요합니다. 🖱️

이번 글에서는 `onClick`의 기본 사용법부터 실무에서 사용하는 함수 분리 패턴까지 알아보겠습니다.

---

## 1. onClick 기본 사용법 (인라인 함수)

React에서 클릭 이벤트를 처리할 때는 카멜 케이스(CamelCase)인 `onClick`을 사용합니다.  
중괄호 `{}` 안에 실행할 함수를 넣어주면 되는데, 간단한 로직은 아래처럼 **화살표 함수(Arrow Function)**를 이용해 바로 작성할 수 있습니다.

**App.js**

```javascript
function App() {
  return (
    <div>
      {/* 화살표 함수로 콘솔 출력 로직을 직접 작성 */}
      <button onClick={() => console.log("click!")}>클릭!!!</button>
    </div>
  );
}

export default App;
```

### 실행 결과

버튼을 클릭하기 전에는 콘솔이 비어있지만, 클릭하는 순간 "click!"이 출력됩니다.

![onClick 실행 전](https://user-images.githubusercontent.com/88264006/218914559-c76a3e80-c6cc-4fd2-aa02-8467ecff54a7.png)

![onClick 실행 후](https://user-images.githubusercontent.com/88264006/218915181-13ed377a-b73e-49a7-9c85-b971f14d366d.png)

> **주의사항:** `onClick={console.log("click")}` 처럼 함수 호출문 자체를 넣으면 안 됩니다. 그렇게 하면 렌더링될 때 함수가 바로 실행되어 버립니다. 반드시 **함수 그 자체** 혹은 **화살표 함수** `() => ...` 형태로 전달해야 합니다.

---

## 2. onClick 함수 분리해서 관리하기 (Best Practice)

코드가 짧을 때는 인라인으로 작성해도 되지만, 로직이 복잡해지거나 실무 프로젝트에서는 **핸들러 함수를 컴포넌트 내부로 분리**하여 관리하는 것이 훨씬 깔끔하고 유지보수에 좋습니다.

### 1) 매개변수(Event 객체)가 필요할 때

클릭한 요소의 정보가 필요하다면 이벤트 객체 `e` (SynthenticEvent)를 받아올 수 있습니다.

```javascript
function App() {
  // 이벤트 객체(e)를 매개변수로 받는 함수 정의
  const handleClick = (e) => {
    console.log(e);
    console.log("이벤트 타겟:", e.target); // 클릭된 태그 확인 가능
  };

  return (
    <div>
      {/* 함수에 e를 전달 */}
      <button onClick={(e) => handleClick(e)}>클릭!!!</button>
    </div>
  );
}

export default App;
```

위 코드를 실행하고 버튼을 누르면 콘솔에 엄청난 양의 이벤트 정보가 출력되는 것을 볼 수 있습니다. 이를 통해 클릭한 좌표, 태그 정보 등을 활용할 수 있습니다.

![이벤트 객체 로그](https://user-images.githubusercontent.com/88264006/218948840-203b9607-a732-48a2-bcfa-2e6f9b8ccd3d.png)

### 2) 매개변수가 없을 때 (가장 깔끔한 방법)

특별히 전달할 인자가 없다면 굳이 `() =>` 화살표 함수를 쓰지 않고, **함수 이름만** 넣어주는 것이 가장 간결합니다.

```javascript
function App() {
  // 실행할 함수 정의
  const handleClick = () => {
    console.log("click");
  };

  return (
    <div>
      {/* 함수 이름만 전달 (가장 권장되는 방식) */}
      <button onClick={handleClick}>클릭!!!</button>
    </div>
  );
}

export default App;

// 실행 결과
// click
```

> **Tip:** `onClick={handleClick}`으로 작성해도 첫 번째 인자로 자동으로 이벤트 객체(`e`)가 넘어갑니다. 따라서 위 1번 예제도 `onClick={handleClick}`으로 줄여 쓸 수 있습니다.

---

## 마치며

오늘은 리액트 이벤트 처리의 기초인 `onClick`에 대해 알아보았습니다.
단순히 콘솔만 찍는 것이 아니라 state를 변경하거나 API를 호출하는 등 무궁무진하게 활용되니 꼭 익숙해지시길 바랍니다!

혹시 궁금한 점이나 피드백이 있다면 편하게 댓글 달아주세요. 👋
