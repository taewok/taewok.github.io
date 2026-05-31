---
title: "[JavaScript] classList로 클래스 추가, 삭제, 토글하기"
date: 2023-03-10T17:01:00
categories: [javascript]
tags: [javascript, dom, classlist, classname]
description: "JavaScript에서 classList를 사용해 DOM 요소의 클래스를 추가, 삭제, 토글하고 존재 여부를 확인하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 클래스는 어떻게 제어할까요?

프론트엔드 개발을 하다 보면 특정 요소에 클래스를 추가하거나 제거해야 하는 경우가 많습니다. 예를 들어 메뉴 열림 상태, 다크 모드, 애니메이션 트리거 같은 기능이 모두 클래스 제어와 관련이 있습니다.

예전에는 `className` 문자열을 직접 수정하는 방식도 많이 사용했지만, 지금은 `classList` API를 사용하면 더 안전하고 편하게 클래스를 제어할 수 있습니다.

---

## 💡 classList란?

`classList`는 DOM 요소의 클래스 목록을 다룰 수 있는 프로퍼티입니다.

```js
const box = document.querySelector("#box");

console.log(box.classList);
```

`className`은 클래스 전체를 문자열로 다루지만, `classList`는 `add`, `remove`, `toggle`, `contains` 같은 메서드를 제공합니다.

---

## ➕ add: 클래스 추가하기

`add`는 요소에 클래스를 추가합니다.

```js
const box = document.querySelector("#box");

box.classList.add("active");
```

여러 클래스를 한 번에 추가할 수도 있습니다.

```js
box.classList.add("fade-in", "highlight");
```

같은 클래스를 여러 번 추가해도 중복으로 들어가지 않습니다.

---

## ➖ remove: 클래스 삭제하기

`remove`는 지정한 클래스를 제거합니다.

```js
const box = document.querySelector("#box");

box.classList.remove("fade-in");
```

존재하지 않는 클래스를 제거하려고 해도 에러가 발생하지 않습니다.

---

## 🔄 toggle: 클래스 토글하기

`toggle`은 클래스가 없으면 추가하고, 있으면 제거합니다.

```js
const box = document.querySelector("#box");

box.classList.toggle("hidden");
```

메뉴 열기/닫기, 다크 모드 전환 같은 기능에서 자주 사용합니다.

```js
button.addEventListener("click", () => {
  menu.classList.toggle("open");
});
```

---

## 🔍 contains: 클래스 존재 여부 확인하기

`contains`는 특정 클래스가 있는지 확인하고 `true` 또는 `false`를 반환합니다.

```js
const box = document.querySelector("#box");

if (box.classList.contains("active")) {
  console.log("현재 활성화 상태입니다.");
}
```

조건에 따라 다른 동작을 해야 할 때 유용합니다.

---

## 📊 className과 classList 비교

| 구분 | className | classList |
| :--- | :--- | :--- |
| 반환 값 | 문자열 | DOMTokenList 객체 |
| 조작 방식 | 클래스 문자열 전체를 교체 | 메서드로 일부 클래스만 조작 |
| 적합한 상황 | 클래스를 통째로 바꿀 때 | 특정 클래스만 추가/삭제할 때 |

보통 특정 클래스 하나를 추가하거나 제거해야 한다면 `classList`가 더 편합니다.

---

## ✅ 정리

`classList`를 사용하면 DOM 요소의 클래스를 쉽게 제어할 수 있습니다.

- `add`: 클래스를 추가합니다.
- `remove`: 클래스를 제거합니다.
- `toggle`: 있으면 제거하고 없으면 추가합니다.
- `contains`: 클래스가 있는지 확인합니다.

UI 상태에 따라 클래스를 바꿔야 한다면 `className` 문자열을 직접 조작하기보다 `classList`를 사용하는 편이 안전하고 읽기 좋습니다.
