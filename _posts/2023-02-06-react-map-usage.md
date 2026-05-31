---
title: "[React] map 함수로 리스트 렌더링하기와 key prop 이해하기"
date: 2023-02-06T22:10:00
categories: [react, javascript]
tags: [react, javascript, map, key, frontend]
description: "React에서 map 함수로 리스트를 렌더링하는 방법과 unique key 경고가 발생하는 이유, key를 올바르게 지정하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 React에서 반복되는 UI는 어떻게 만들까요?

React를 사용할 때 반복되는 UI를 하나하나 직접 작성하는 것은 비효율적입니다.

예를 들어 이름 목록을 `<li>`로 출력해야 한다면, 배열과 `map`을 함께 사용하는 편이 훨씬 좋습니다.

이번 글에서는 `map`으로 HTML 태그를 반복 렌더링하는 방법과, 이때 자주 만나는 `key` prop 경고를 함께 정리해보겠습니다.

---

## 🛠️ map으로 리스트 렌더링하기

배열에 담긴 데이터를 기반으로 `<li>` 태그를 반복해서 출력해보겠습니다.

```jsx
function App() {
  const array = ["손명오", "최혜정", "문동은", "박연진", "이사라", "하도영"];

  return (
    <div className="container">
      <ul>
        {array.map((name) => (
          <li>{name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

위 코드는 배열의 길이만큼 `<li>`를 만들어 화면에 보여줍니다.

```txt
손명오
최혜정
문동은
박연진
이사라
하도영
```

이렇게 `map`을 사용하면 데이터가 100개, 1000개로 늘어나도 UI 코드는 간결하게 유지됩니다.

---

## ⚠️ unique key 경고가 나타나는 이유

화면은 잘 보이지만 개발자 도구 콘솔을 열어보면 다음과 같은 경고를 만날 수 있습니다.

```txt
Warning: Each child in a list should have a unique "key" prop.
```

리스트의 각 자식 요소는 고유한 `key` prop을 가져야 한다는 뜻입니다.

React는 리스트를 다시 렌더링할 때 어떤 항목이 추가되었는지, 삭제되었는지, 위치가 바뀌었는지 알아야 합니다. 이때 `key`가 각 항목을 구분하는 이름표 역할을 합니다.

---

## ✅ key로 고유한 값 넣어주기

가장 간단한 해결 방법은 반복되는 태그에 고유한 값을 `key`로 넣어주는 것입니다.

```jsx
function App() {
  const array = ["손명오", "최혜정", "문동은", "박연진", "이사라", "하도영"];

  return (
    <div className="container">
      <ul>
        {array.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

여기서는 이름이 모두 다르기 때문에 `key={name}`을 사용할 수 있습니다.

React 입장에서는 다음처럼 각 항목을 구분할 수 있게 됩니다.

```txt
손명오 → key: "손명오"
최혜정 → key: "최혜정"
문동은 → key: "문동은"
```

---

## 🧩 데이터에 중복이 있다면?

만약 배열에 중복된 값이 있다면 `key={name}`은 안전하지 않습니다.

```js
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

이 상태에서 `key={name}`을 사용하면 `"하도영"`이라는 key가 두 번 생깁니다.

React의 key는 같은 리스트 안에서 반드시 고유해야 하기 때문에 경고가 발생할 수 있습니다.

---

## 🔢 index를 key로 사용해도 될까요?

중복된 값 때문에 고유한 key를 만들기 어렵다면 `map`의 두 번째 인자인 `index`를 사용할 수도 있습니다.

```jsx
{
  array.map((name, index) => (
    <li key={index}>{name}</li>
  ));
}
```

다만 `index`를 key로 사용하는 것은 항상 좋은 선택은 아닙니다.

리스트의 순서가 바뀌거나, 중간에 항목이 추가/삭제되는 경우에는 React가 항목을 정확히 추적하지 못해 예상치 못한 UI 문제가 생길 수 있습니다.

그래서 실무에서는 가능하면 데이터 자체에 있는 고유한 `id`를 key로 사용하는 것이 좋습니다.

```jsx
const users = [
  { id: 1, name: "손명오" },
  { id: 2, name: "최혜정" },
];

{
  users.map((user) => (
    <li key={user.id}>{user.name}</li>
  ));
}
```

`index` key는 리스트가 정적이고 순서가 바뀌지 않는 경우에만 제한적으로 사용하는 편이 안전합니다.

---

## ✅ 정리

React에서 반복되는 UI를 만들 때는 `map`을 자주 사용합니다.

- 배열 데이터를 JSX로 변환할 때 `map`을 사용합니다.
- 리스트 항목에는 고유한 `key` prop이 필요합니다.
- `key`는 React가 각 항목을 구분하는 이름표 역할을 합니다.
- 가능하면 `index`보다 데이터의 고유한 `id`를 key로 사용하는 것이 좋습니다.
- 정적인 리스트라면 `index`를 사용할 수도 있습니다.

리스트 렌더링은 React에서 정말 자주 쓰이는 패턴입니다. `map`과 `key`의 역할을 함께 이해해두면 훨씬 안정적인 UI를 만들 수 있습니다.
