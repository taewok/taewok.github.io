---
title: "[React] map 함수 활용과 key prop의 중요성"
date: 2023-02-06T22:10:00
categories: [React, JavaScript]
tags: [react, javascript, map, key, frontend]
description: "React에서 map 함수를 사용하여 리스트를 렌더링하는 방법과 이때 반드시 발생하는 'unique key' 에러의 원인 및 해결 방법을 상세히 알아봅니다."
custom_style: true
---

React를 사용할 때 반복되는 UI 요소를 하나하나 하드코딩하는 것은 비효율적입니다.  
이때 **`map` 함수**를 활용하면 코드를 획기적으로 줄이고 유지보수하기 쉽게 만들 수 있습니다.

(혹시 아직 `map` 함수에 대해 잘 모른다면 [이전 포스팅](https://www.naver.com/)을 먼저 참고해주세요!)

이번 글에서는 `map`을 이용한 HTML 태그 반복 렌더링 방법과, 이때 필연적으로 마주치는 **Key Prop** 이슈에 대해 정리해 보겠습니다.

---

## 1. map 함수로 HTML 태그 반복 렌더링하기

배열에 담긴 데이터를 기반으로 `<li>` 태그를 반복해서 출력해보겠습니다.

```javascript
function App() {
  const array = ["손명오", "최혜정", "문동은", "박연진", "이사라", "하도영"];

  return (
    <div className="container">
      <ul>
        {/* 배열을 순회하며 li 태그를 생성 */}
        {array.map((name) => (
          <li>{name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

위 코드를 작성하고 실행하면, 배열의 길이만큼 리스트가 생성되어 아래와 같이 정상적으로 화면에 출력됩니다.

![리스트 렌더링 결과](https://user-images.githubusercontent.com/88264006/216997198-f77d4820-5ca7-41ca-927a-a792147e2a01.png)

이렇게 `map`을 사용하면 데이터가 100개, 1000개로 늘어나도 코드는 여전히 간결하게 유지됩니다. 👍

---

## 2. 콘솔에 나타난 빨간색 에러 (Key Warning)

하지만 화면은 잘 나오더라도 **F12(개발자 도구)**를 열어 콘솔(Console) 탭을 확인해보면 빨간색 경고 문구가 떠 있는 것을 볼 수 있습니다.

> **Warning: Each child in a list should have a unique "key" prop.**

![Key Prop 경고 화면](https://user-images.githubusercontent.com/88264006/216999657-1821a704-8e81-48c7-90d0-a6791a920df4.png)

"리스트의 각 자식 요소는 고유한 `key` prop을 가져야 한다"는 뜻입니다. 이 경고는 왜 발생하는 걸까요?

---

## 3. Error 해결 방법과 원인

### 해결 방법

가장 간단한 해결책은 반복되는 태그(`<li>`)에 고유한 값을 `key` 속성으로 넣어주는 것입니다.

```javascript
function App() {
  const array = ["손명오", "최혜정", "문동은", "박연진", "이사라", "하도영"];

  return (
    <div className="container">
      <ul>
        {/* key 속성에 고유한 값인 name을 부여 */}
        {array.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

### 왜 key가 필요할까? (공식 문서)

React 공식 문서에 따르면 `key`는 다음과 같은 중요한 역할을 합니다.

1.  **식별자 역할:** React가 어떤 항목이 변경, 추가, 또는 삭제되었는지 식별하는 것을 돕습니다.
2.  **성능 최적화:** 엘리먼트에 안정적인 고유성을 부여함으로써, React가 불필요한 리렌더링을 방지하고 효율적으로 DOM을 업데이트할 수 있게 합니다.

즉, **React가 일을 더 똑똑하고 빠르게 하기 위해 필요한 이름표**라고 생각하면 됩니다.

---

## 4. 만약 데이터에 중복이 있다면? (Duplicate Keys)

위 예제에서는 이름이 모두 달라서 문제가 없었지만, 만약 배열에 **중복된 데이터**가 있다면 어떻게 될까요?

```javascript
// "하도영"이 두 번 들어간 경우
const array = [
  "손명오",
  "최혜정",
  "문동은",
  "박연진",
  "이사라",
  "하도영",
  "하도영",
];
```

이 상태에서 `key={name}`을 사용하면 **"Encountered two children with the same key..."** 라는 또 다른 에러가 발생합니다.

![중복 키 에러](https://user-images.githubusercontent.com/88264006/217005721-e40c37bb-17ef-4985-bf6f-2c77d99a52be.png)

`key` 값은 형제 사이에서 **반드시 고유(Unique)**해야 하기 때문입니다. 이름을 개명할 수도 없고 난감하죠? 😅

### 해결책: Index 활용하기

이럴 때는 `map` 함수의 두 번째 인자인 **index(순서 번호)**를 활용하여 해결할 수 있습니다.

```javascript
{
  array.map((name, index) => (
    // index는 0, 1, 2... 순서대로 증가하므로 중복될 일이 없음
    <li key={index}>{name}</li>
  ));
}
```

> **💡 참고:** 리스트의 순서가 바뀌거나 항목이 추가/삭제될 가능성이 있는 경우에는 `index`를 key로 사용하는 것이 권장되지 않습니다. (성능 저하 및 버그 유발 가능) 하지만 위 예제처럼 **변하지 않는 고정된 리스트**라면 `index`를 사용해도 안전합니다.

---

## 마치며

오늘은 React에서 `map`을 사용하여 HTML을 반복 출력하는 방법과 `key` prop 설정의 중요성에 대해 알아보았습니다.

이 패턴은 React 개발의 90% 이상을 차지한다고 해도 과언이 아니니 꼭 숙지해 두시길 바랍니다! 궁금한 점이 있다면 언제든 댓글 달아주세요. 👋
