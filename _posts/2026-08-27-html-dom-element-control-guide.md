---
title: "[HTML/JavaScript] DOM이란? 요소 선택과 제어 방법 정리"
date: 2026-08-27T01:10:00Z
categories: [html, javascript]
tags: [html, javascript, dom, queryselector, element, event]
description: "HTML DOM이 무엇인지, 브라우저가 HTML을 DOM 트리로 해석하는 방식과 JavaScript로 요소를 선택하고 내용, 스타일, 클래스, 속성, 이벤트를 제어하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며: HTML과 화면 사이에는 DOM이 있어요

HTML을 처음 배울 때는 보통 이런 식으로 생각합니다.

```txt
HTML 파일을 작성한다
↓
브라우저가 화면에 보여준다
```

틀린 말은 아니지만, 중간에 중요한 과정이 하나 있습니다.

브라우저는 HTML 코드를 그대로 화면에 붙이는 것이 아니라, HTML을 해석해서 **DOM**이라는 구조로 바꿉니다.

그리고 JavaScript는 이 DOM을 통해 화면의 요소를 찾고, 바꾸고, 삭제하고, 이벤트를 연결할 수 있습니다.

예를 들어 버튼을 클릭했을 때 문구가 바뀌는 기능은 결국 이런 흐름입니다.

```txt
버튼 클릭
↓
JavaScript가 DOM 요소를 찾음
↓
요소의 textContent를 변경
↓
화면의 글자가 바뀜
```

이번 글에서는 HTML DOM이 무엇인지, 그리고 JavaScript로 DOM 요소를 어떻게 제어하는지 기초부터 정리해보겠습니다.

---

## DOM이란?

DOM은 `Document Object Model`의 줄임말입니다.

한국어로 풀어보면 문서 객체 모델 정도로 이해할 수 있습니다.

조금 더 쉽게 말하면 이렇습니다.

```txt
DOM은 브라우저가 HTML 문서를 JavaScript로 다룰 수 있게 만든 객체 구조입니다.
```

예를 들어 이런 HTML이 있다고 해보겠습니다.

```html
<!doctype html>
<html>
  <head>
    <title>DOM 예제</title>
  </head>
  <body>
    <h1>안녕하세요</h1>
    <button>클릭</button>
  </body>
</html>
```

브라우저는 이 HTML을 대략 이런 트리 구조로 이해합니다.

```txt
document
└─ html
   ├─ head
   │  └─ title
   └─ body
      ├─ h1
      └─ button
```

이런 구조를 DOM 트리라고 부릅니다.

HTML 태그 하나하나는 DOM에서 노드 또는 요소 객체로 표현됩니다.

그래서 JavaScript에서는 다음처럼 요소를 찾고 제어할 수 있습니다.

```js
const title = document.querySelector("h1");

title.textContent = "반갑습니다";
```

HTML 파일을 직접 다시 쓰지 않아도 화면의 글자가 바뀌는 이유는 JavaScript가 DOM을 수정했기 때문입니다.

---

## document는 무엇일까?

DOM을 제어할 때 가장 자주 만나는 객체가 `document`입니다.

`document`는 현재 브라우저에 로드된 HTML 문서를 나타냅니다.

```js
console.log(document);
```

`document`를 통해 문서 안의 요소를 찾거나 새 요소를 만들 수 있습니다.

```js
document.querySelector("h1");
document.createElement("div");
```

즉, `document`는 JavaScript에서 HTML 문서로 들어가는 입구라고 볼 수 있습니다.

```txt
JavaScript
↓
document
↓
DOM tree
↓
HTML 요소
```

DOM을 제어한다는 말은 대부분 `document`를 통해 요소를 찾고, 그 요소 객체의 속성이나 메서드를 사용하는 것을 의미합니다.

---

## 요소 선택하기

DOM 제어의 첫 단계는 요소를 선택하는 것입니다.

요소를 선택해야 내용을 바꾸거나, 클래스를 추가하거나, 이벤트를 연결할 수 있습니다.

가장 많이 사용하는 메서드는 `querySelector`와 `querySelectorAll`입니다.

---

## querySelector: 하나의 요소 선택하기

`querySelector`는 CSS 선택자와 일치하는 첫 번째 요소를 가져옵니다.

```html
<h1 id="title">DOM 배우기</h1>
<p class="description">첫 번째 설명입니다.</p>
<p class="description">두 번째 설명입니다.</p>
```

```js
const title = document.querySelector("#title");
const description = document.querySelector(".description");

console.log(title);
console.log(description);
```

여기서 `#title`은 id 선택자이고, `.description`은 class 선택자입니다.

`querySelector(".description")`는 class가 `description`인 요소 중 첫 번째 요소만 가져옵니다.

```txt
첫 번째 설명입니다.
```

만약 선택자와 일치하는 요소가 없으면 `null`이 반환됩니다.

```js
const button = document.querySelector(".unknown-button");

console.log(button); // null
```

그래서 실제 코드에서는 요소가 있는지 확인한 뒤 사용하는 것이 안전합니다.

```js
const title = document.querySelector("#title");

if (title) {
  title.textContent = "제목을 변경했습니다";
}
```

---

## querySelectorAll: 여러 요소 선택하기

여러 요소를 한 번에 가져오고 싶다면 `querySelectorAll`을 사용합니다.

```html
<ul>
  <li class="item">HTML</li>
  <li class="item">CSS</li>
  <li class="item">JavaScript</li>
</ul>
```

```js
const items = document.querySelectorAll(".item");

console.log(items);
```

`querySelectorAll`은 여러 요소를 담은 `NodeList`를 반환합니다.

반복문으로 하나씩 제어할 수 있습니다.

```js
items.forEach((item) => {
  item.classList.add("active");
});
```

이렇게 하면 class가 `item`인 모든 요소에 `active` 클래스가 추가됩니다.

```html
<li class="item active">HTML</li>
<li class="item active">CSS</li>
<li class="item active">JavaScript</li>
```

---

## getElementById도 자주 볼 수 있어요

예전 코드나 간단한 예제에서는 `getElementById`도 자주 보입니다.

```html
<h1 id="title">제목</h1>
```

```js
const title = document.getElementById("title");
```

`getElementById`는 id로 요소를 찾습니다.

`querySelector`로도 같은 요소를 찾을 수 있습니다.

```js
const title = document.querySelector("#title");
```

둘 다 사용할 수 있지만, 처음에는 CSS 선택자 문법을 그대로 사용할 수 있는 `querySelector`와 `querySelectorAll`에 익숙해지는 것이 좋습니다.

---

## 요소의 텍스트 바꾸기

요소를 선택했다면 이제 내용을 바꿔볼 수 있습니다.

텍스트를 바꿀 때는 `textContent`를 자주 사용합니다.

```html
<h1 id="title">기존 제목</h1>
```

```js
const title = document.querySelector("#title");

if (title) {
  title.textContent = "새로운 제목";
}
```

결과는 이렇게 바뀝니다.

```html
<h1 id="title">새로운 제목</h1>
```

`textContent`는 요소 안의 텍스트를 변경합니다.

HTML 태그를 넣어도 태그로 해석하지 않고 글자로 처리합니다.

```js
title.textContent = "<span>새 제목</span>";
```

화면에는 `<span>새 제목</span>`이라는 문자열이 그대로 보입니다.

---

## innerHTML은 조심해서 사용하기

HTML 구조까지 넣고 싶다면 `innerHTML`을 사용할 수 있습니다.

```html
<div id="box"></div>
```

```js
const box = document.querySelector("#box");

if (box) {
  box.innerHTML = "<strong>강조된 문장</strong>";
}
```

그러면 실제 HTML 요소가 만들어집니다.

```html
<div id="box">
  <strong>강조된 문장</strong>
</div>
```

하지만 `innerHTML`은 조심해야 합니다.

사용자가 입력한 값을 그대로 `innerHTML`에 넣으면 보안 문제가 생길 수 있습니다.

예를 들어 댓글 입력값을 그대로 HTML로 넣는 구조는 위험합니다.

```js
commentBox.innerHTML = userInput;
```

단순 텍스트를 넣을 때는 가능하면 `textContent`를 사용하는 것이 안전합니다.

```js
commentBox.textContent = userInput;
```

초보 단계에서는 이렇게 기억해도 좋습니다.

```txt
글자만 바꿀 때는 textContent
HTML 구조를 넣어야 할 때만 innerHTML
```

---

## input 값 제어하기

`input`이나 `textarea`의 값은 `textContent`가 아니라 `value`로 다룹니다.

```html
<input id="nickname" type="text" value="taewok" />
<button id="submit-button">확인</button>
```

```js
const input = document.querySelector("#nickname");
const button = document.querySelector("#submit-button");

button.addEventListener("click", () => {
  console.log(input.value);
});
```

위 코드는 버튼을 클릭했을 때 input에 입력된 값을 출력합니다.

다만 `querySelector`의 반환 타입은 넓기 때문에 TypeScript에서는 `HTMLInputElement`인지 확인하거나 타입을 좁혀야 합니다.

JavaScript만 사용할 때도 요소가 존재하는지 먼저 확인하는 습관이 좋습니다.

```js
const input = document.querySelector("#nickname");
const button = document.querySelector("#submit-button");

if (input && button) {
  button.addEventListener("click", () => {
    console.log(input.value);
  });
}
```

---

## 클래스 제어하기

DOM 요소의 클래스를 다룰 때는 `classList`를 사용합니다.

```html
<div id="box" class="box">박스</div>
```

```js
const box = document.querySelector("#box");

if (box) {
  box.classList.add("active");
}
```

`classList`에는 자주 쓰는 메서드가 있습니다.

| 메서드 | 의미 |
| --- | --- |
| `add` | 클래스 추가 |
| `remove` | 클래스 제거 |
| `toggle` | 있으면 제거, 없으면 추가 |
| `contains` | 클래스가 있는지 확인 |

예시로 보면 더 쉽습니다.

```js
box.classList.add("active");
box.classList.remove("hidden");
box.classList.toggle("open");

if (box.classList.contains("active")) {
  console.log("활성 상태입니다.");
}
```

메뉴 열기/닫기 같은 UI는 `toggle`을 많이 사용합니다.

```html
<button id="menu-button">메뉴</button>
<nav id="menu" class="menu">메뉴 내용</nav>
```

```js
const button = document.querySelector("#menu-button");
const menu = document.querySelector("#menu");

if (button && menu) {
  button.addEventListener("click", () => {
    menu.classList.toggle("open");
  });
}
```

CSS에서는 `open` 클래스가 있을 때의 스타일을 정의합니다.

```css
.menu {
  display: none;
}

.menu.open {
  display: block;
}
```

JavaScript는 클래스를 붙이고 떼는 역할만 하고, 실제 스타일은 CSS가 담당하게 만드는 방식입니다.

이 구조가 깔끔합니다.

---

## 스타일 직접 제어하기

요소의 스타일을 JavaScript로 직접 바꿀 수도 있습니다.

```html
<div id="box">박스</div>
```

```js
const box = document.querySelector("#box");

if (box) {
  box.style.backgroundColor = "royalblue";
  box.style.color = "white";
  box.style.padding = "16px";
}
```

CSS에서는 `background-color`처럼 하이픈을 쓰지만, JavaScript의 `style`에서는 camelCase를 사용합니다.

| CSS | JavaScript |
| --- | --- |
| `background-color` | `backgroundColor` |
| `font-size` | `fontSize` |
| `margin-top` | `marginTop` |
| `border-radius` | `borderRadius` |

하지만 많은 스타일을 JavaScript로 직접 넣는 것은 유지보수가 어려울 수 있습니다.

```js
box.style.backgroundColor = "royalblue";
box.style.color = "white";
box.style.padding = "16px";
box.style.borderRadius = "8px";
box.style.fontWeight = "700";
```

이럴 때는 클래스를 제어하는 편이 더 좋습니다.

```css
.box.active {
  border-radius: 8px;
  padding: 16px;
  background-color: royalblue;
  color: white;
  font-weight: 700;
}
```

```js
box.classList.add("active");
```

초보 단계에서는 이렇게 나누면 좋습니다.

```txt
일시적인 숫자 값 변경
→ style 사용 가능

여러 스타일 묶음 변경
→ classList 사용 추천
```

---

## 속성 제어하기

HTML 요소에는 다양한 속성이 있습니다.

```html
<img id="profile-image" src="/default.png" alt="기본 이미지" />
<a id="link" href="/">홈으로</a>
```

이런 속성은 `getAttribute`, `setAttribute`, `removeAttribute`로 다룰 수 있습니다.

```js
const image = document.querySelector("#profile-image");

if (image) {
  image.setAttribute("src", "/profile.png");
  image.setAttribute("alt", "프로필 이미지");
}
```

속성 값을 읽을 수도 있습니다.

```js
const link = document.querySelector("#link");

if (link) {
  const href = link.getAttribute("href");
  console.log(href);
}
```

속성을 제거할 수도 있습니다.

```js
link.removeAttribute("href");
```

자주 다루는 속성은 프로퍼티로 직접 접근할 수도 있습니다.

```js
const image = document.querySelector("#profile-image");

if (image) {
  image.src = "/profile.png";
  image.alt = "프로필 이미지";
}
```

다만 요소 종류에 따라 사용할 수 있는 프로퍼티가 다릅니다.

`img`에는 `src`, `alt`가 자연스럽지만, 일반 `div`에는 `src`가 없습니다.

---

## 요소 만들기

DOM에서는 JavaScript로 새 요소를 만들 수 있습니다.

가장 기본은 `document.createElement`입니다.

```js
const item = document.createElement("li");

item.textContent = "새로운 항목";
```

이렇게 만들기만 하면 아직 화면에는 보이지 않습니다.

DOM 트리에 추가해야 합니다.

```html
<ul id="list"></ul>
```

```js
const list = document.querySelector("#list");
const item = document.createElement("li");

item.textContent = "새로운 항목";

if (list) {
  list.append(item);
}
```

`append`는 선택한 요소의 마지막 자식으로 새 요소를 추가합니다.

결과는 이렇게 됩니다.

```html
<ul id="list">
  <li>새로운 항목</li>
</ul>
```

---

## 요소 삭제하기

요소를 삭제할 때는 `remove`를 사용할 수 있습니다.

```html
<p id="message">삭제할 문장입니다.</p>
```

```js
const message = document.querySelector("#message");

if (message) {
  message.remove();
}
```

버튼을 클릭하면 리스트 항목을 삭제하는 예시도 만들 수 있습니다.

```html
<ul id="todo-list">
  <li>
    JavaScript 공부하기
    <button class="delete-button">삭제</button>
  </li>
  <li>
    DOM 정리하기
    <button class="delete-button">삭제</button>
  </li>
</ul>
```

```js
const deleteButtons = document.querySelectorAll(".delete-button");

deleteButtons.forEach((button) => {
  button.addEventListener("click", () => {
    const item = button.closest("li");

    if (item) {
      item.remove();
    }
  });
});
```

여기서 `closest("li")`는 현재 버튼에서 가장 가까운 부모 `li` 요소를 찾습니다.

삭제 버튼을 누르면 해당 버튼이 들어 있는 리스트 항목만 삭제됩니다.

---

## 이벤트 연결하기

DOM 제어에서 빠질 수 없는 것이 이벤트입니다.

이벤트는 사용자의 행동이나 브라우저의 변화입니다.

예를 들어 다음과 같은 것들이 이벤트입니다.

- 클릭
- 키보드 입력
- 마우스 이동
- input 값 변경
- 페이지 로드
- 스크롤

이벤트를 연결할 때는 `addEventListener`를 사용합니다.

```html
<button id="button">클릭</button>
```

```js
const button = document.querySelector("#button");

if (button) {
  button.addEventListener("click", () => {
    console.log("버튼을 클릭했습니다.");
  });
}
```

구조는 이렇게 이해하면 됩니다.

```txt
button.addEventListener("click", 실행할 함수)
```

즉, 버튼에서 click 이벤트가 발생하면 두 번째 인자로 넘긴 함수가 실행됩니다.

---

## 이벤트 객체 사용하기

이벤트 핸들러 함수는 이벤트 객체를 받을 수 있습니다.

```html
<input id="search-input" type="text" />
```

```js
const input = document.querySelector("#search-input");

if (input) {
  input.addEventListener("input", (event) => {
    console.log(event.target.value);
  });
}
```

여기서 `event.target`은 이벤트가 실제로 발생한 요소입니다.

input 이벤트에서는 `event.target.value`를 통해 현재 입력값을 확인할 수 있습니다.

다만 조금 더 안전하게 쓰려면 요소 타입을 확인할 수 있습니다.

```js
input.addEventListener("input", (event) => {
  const target = event.target;

  if (target instanceof HTMLInputElement) {
    console.log(target.value);
  }
});
```

이런 확인은 TypeScript를 사용할 때 특히 자주 보게 됩니다.

---

## 이벤트 제거하기

이벤트를 등록한 뒤 필요 없어지면 제거할 수도 있습니다.

이때는 `removeEventListener`를 사용합니다.

```js
const button = document.querySelector("#button");

const handleClick = () => {
  console.log("클릭했습니다.");
};

if (button) {
  button.addEventListener("click", handleClick);
  button.removeEventListener("click", handleClick);
}
```

주의할 점은 등록할 때와 제거할 때 같은 함수 참조를 사용해야 한다는 것입니다.

다음 코드는 제거가 잘 되지 않습니다.

```js
button.addEventListener("click", () => {
  console.log("클릭했습니다.");
});

button.removeEventListener("click", () => {
  console.log("클릭했습니다.");
});
```

두 함수는 모양은 같지만 서로 다른 함수입니다.

그래서 제거하려면 함수에 이름을 붙여두는 것이 좋습니다.

```js
const handleClick = () => {
  console.log("클릭했습니다.");
};

button.addEventListener("click", handleClick);
button.removeEventListener("click", handleClick);
```

---

## 간단한 실습: 카운터 만들기

지금까지 배운 내용을 조합해서 카운터를 만들어보겠습니다.

```html
<section class="counter">
  <p id="count">0</p>
  <button id="increase-button">증가</button>
  <button id="decrease-button">감소</button>
</section>
```

```js
const countElement = document.querySelector("#count");
const increaseButton = document.querySelector("#increase-button");
const decreaseButton = document.querySelector("#decrease-button");

let count = 0;

const renderCount = () => {
  countElement.textContent = String(count);
};

if (countElement && increaseButton && decreaseButton) {
  increaseButton.addEventListener("click", () => {
    count += 1;
    renderCount();
  });

  decreaseButton.addEventListener("click", () => {
    count -= 1;
    renderCount();
  });
}
```

이 예제에는 DOM 제어의 기본 흐름이 모두 들어 있습니다.

```txt
1. 요소를 선택한다.
2. 상태로 사용할 값을 만든다.
3. 화면을 갱신하는 함수를 만든다.
4. 버튼 이벤트를 연결한다.
5. 이벤트가 발생하면 값을 바꾸고 DOM을 업데이트한다.
```

React의 `useState`도 결국 이런 문제를 더 편하게 다루기 위한 도구라고 볼 수 있습니다.

순수 JavaScript로 DOM 제어를 이해해두면 React를 배울 때도 화면이 어떻게 바뀌는지 훨씬 잘 보입니다.

---

## 간단한 실습: Todo 추가하기

이번에는 input에 입력한 값을 리스트에 추가해보겠습니다.

```html
<form id="todo-form">
  <input id="todo-input" type="text" placeholder="할 일을 입력하세요" />
  <button type="submit">추가</button>
</form>

<ul id="todo-list"></ul>
```

```js
const form = document.querySelector("#todo-form");
const input = document.querySelector("#todo-input");
const list = document.querySelector("#todo-list");

if (form && input && list) {
  form.addEventListener("submit", (event) => {
    event.preventDefault();

    const value = input.value.trim();

    if (!value) {
      return;
    }

    const item = document.createElement("li");
    item.textContent = value;

    list.append(item);
    input.value = "";
  });
}
```

여기서 `event.preventDefault()`는 form의 기본 제출 동작을 막습니다.

기본 동작을 막지 않으면 브라우저가 페이지를 새로고침할 수 있습니다.

이 예제에서 사용한 DOM 기능은 다음과 같습니다.

- `querySelector`로 요소 선택
- `addEventListener`로 submit 이벤트 연결
- `event.preventDefault`로 기본 동작 방지
- `input.value`로 입력값 읽기
- `document.createElement`로 새 `li` 만들기
- `textContent`로 내용 넣기
- `append`로 리스트에 추가하기

작지만 DOM 제어의 핵심이 잘 들어 있는 예제입니다.

---

## DOM 제어할 때 자주 하는 실수

### 1. 요소를 찾기 전에 스크립트가 먼저 실행됨

HTML 요소가 만들어지기 전에 JavaScript가 실행되면 요소를 찾지 못할 수 있습니다.

```html
<script src="./main.js"></script>

<button id="button">클릭</button>
```

이 경우 script가 먼저 실행되고 버튼은 아직 없을 수 있습니다.

해결 방법은 보통 script를 body 끝에 두는 것입니다.

```html
<button id="button">클릭</button>

<script src="./main.js"></script>
```

또는 `defer`를 사용할 수 있습니다.

```html
<script src="./main.js" defer></script>
```

`defer`를 사용하면 HTML 파싱이 끝난 뒤 스크립트가 실행됩니다.

### 2. null 확인 없이 바로 사용하기

`querySelector`는 요소를 찾지 못하면 `null`을 반환합니다.

```js
const button = document.querySelector("#button");

button.addEventListener("click", () => {
  console.log("클릭");
});
```

요소가 없으면 위 코드는 에러가 납니다.

안전하게는 먼저 확인합니다.

```js
const button = document.querySelector("#button");

if (button) {
  button.addEventListener("click", () => {
    console.log("클릭");
  });
}
```

### 3. 스타일을 전부 JavaScript로 처리하기

JavaScript로 모든 스타일을 직접 바꾸면 코드가 금방 복잡해집니다.

```js
menu.style.display = "block";
menu.style.opacity = "1";
menu.style.transform = "translateY(0)";
```

이보다는 CSS 클래스를 준비하고 JavaScript에서는 클래스만 토글하는 편이 좋습니다.

```js
menu.classList.toggle("open");
```

```css
.menu {
  opacity: 0;
  transform: translateY(-8px);
}

.menu.open {
  opacity: 1;
  transform: translateY(0);
}
```

### 4. innerHTML에 사용자 입력을 그대로 넣기

사용자 입력은 신뢰할 수 없는 값입니다.

```js
content.innerHTML = userInput;
```

단순 텍스트라면 `textContent`를 사용합니다.

```js
content.textContent = userInput;
```

---

## React를 쓸 때도 DOM을 알아야 할까?

React를 사용하면 직접 `document.querySelector`를 쓰는 일이 줄어듭니다.

React에서는 보통 상태를 바꾸면 UI가 다시 렌더링됩니다.

```tsx
const [count, setCount] = useState(0);
```

그래서 React 코드에서 DOM을 직접 수정하는 방식은 자주 권장되지 않습니다.

하지만 DOM 개념은 여전히 중요합니다.

React에서도 다음 상황에서는 DOM 이해가 필요합니다.

- `useRef`로 input에 focus 주기
- `getBoundingClientRect`로 요소 위치 측정하기
- 모달 focus trap 만들기
- 바깥 클릭 감지하기
- 스크롤 위치 제어하기
- 파일 input 다루기
- canvas, video 같은 브라우저 API 사용하기

즉, React가 DOM 제어를 대신 도와주더라도 브라우저가 실제로 화면을 어떻게 구성하고 조작하는지 이해하는 것은 여전히 중요합니다.

---

## 전체 흐름 다시 정리하기

DOM 제어는 크게 다음 흐름으로 볼 수 있습니다.

```txt
1. HTML이 브라우저에 의해 DOM 트리로 변환된다.
2. JavaScript는 document를 통해 DOM에 접근한다.
3. querySelector로 원하는 요소를 찾는다.
4. textContent, classList, style, attribute 등으로 요소를 바꾼다.
5. addEventListener로 사용자 행동에 반응한다.
6. 필요하면 createElement, append, remove로 요소를 추가하거나 삭제한다.
```

이 흐름이 잡히면 대부분의 기본 인터랙션을 이해할 수 있습니다.

예를 들어 메뉴 열기, 탭 전환, 모달 닫기, Todo 추가, 검색어 입력 같은 기능은 모두 이 기본 원리 위에서 만들어집니다.

---

## 정리

DOM은 HTML 문서를 JavaScript가 다룰 수 있도록 브라우저가 만들어둔 객체 구조입니다.

HTML을 작성하면 브라우저는 그것을 DOM 트리로 만들고, JavaScript는 `document`를 통해 그 트리에 접근합니다.

DOM 제어에서 자주 사용하는 기능은 다음과 같습니다.

- `document.querySelector`: 조건에 맞는 첫 번째 요소 선택
- `document.querySelectorAll`: 조건에 맞는 여러 요소 선택
- `textContent`: 텍스트 변경
- `innerHTML`: HTML 구조 변경
- `value`: input 값 읽기와 변경
- `classList`: 클래스 추가, 삭제, 토글
- `style`: 인라인 스타일 변경
- `setAttribute`: 속성 변경
- `createElement`: 새 요소 생성
- `append`: 요소 추가
- `remove`: 요소 삭제
- `addEventListener`: 이벤트 연결
- `removeEventListener`: 이벤트 제거

초보 단계에서는 이 정도만 익혀도 많은 UI를 만들 수 있습니다.

한 줄로 정리하면 이렇습니다.

```txt
DOM을 안다는 것은
JavaScript가 HTML 화면을 어떻게 찾고 바꾸는지 이해한다는 뜻입니다.
```

이 기초가 잡히면 이후 React, Next.js 같은 프레임워크를 배울 때도 화면 업데이트 원리를 훨씬 자연스럽게 이해할 수 있습니다.

## 참고

- [MDN: Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [MDN: Document](https://developer.mozilla.org/en-US/docs/Web/API/Document)
- [MDN: querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector)
- [MDN: querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll)
- [MDN: classList](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList)
- [MDN: addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
