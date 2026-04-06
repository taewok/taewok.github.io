---
title: "[JavaScript] 자바스크립트 this의 동작 원리 정리 (일반 함수 vs 화살표 함수)"
date: 2023-02-18T19:10:00
categories: [javascript]
tags: [javascript, this, es6, arrow-function]
description: "JavaScript에서 this가 결정되는 원리(호출 방식)와 일반 함수 내부의 this, 그리고 화살표 함수에서의 this 차이점을 예제 코드를 통해 상세히 알아봅니다."
custom_style: true
---

자바스크립트를 공부하다 보면 가장 헷갈리는 개념 중 하나가 바로 **`this`**입니다.  
MDN 공식 문서를 보면 "함수를 호출한 방법에 의해 결정된다"라고 적혀있지만, 글로만 봐서는 이해하기 어렵습니다. 😵

오늘은 코드를 직접 실행해보며 상황에 따라 변하는 `this`의 정체를 파헤쳐 보겠습니다.

---

## 1. 실습 준비 (기본 세팅)

테스트를 위해 HTML 파일과 JS 파일을 준비합니다.

### index.js

객체 안에 메서드를 만들고, 그 안에 또 다른 내부 함수를 만들어 `this`를 출력해 봅니다.

```javascript
const name = {
  name: "오윤희",
  getName: function () {
    console.log("1. 메서드 호출:", this); // (A)

    const second = function () {
      console.log("2. 내부 함수 호출:", this); // (B)
    };
    second();
  },
};

name.getName();
console.log("3. 객체 확인:", name);
```

### index.html

자바스크립트 파일을 불러옵니다. 여기서 중요한 점은 **`type="module"`**을 사용했다는 점입니다.

**주의:** 모듈(`type="module"`) 환경은 자동으로 **Strict Mode(엄격 모드)**가 적용됩니다. 이는 `this`의 동작에 영향을 줍니다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>JS This Test</title>
  </head>
  <body>
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

---

## 2. 실행 결과 분석

위 코드를 브라우저에서 실행하면 콘솔에 다음과 같이 출력됩니다. 하나씩 뜯어볼까요?

![this 실행 결과](https://user-images.githubusercontent.com/88264006/219857844-54b8c0dd-3683-4200-87e7-19dc36078f01.png)

### 1) 첫 번째 콘솔: `name.getName()`

```javascript
// 출력: {name: "오윤희", getName: f} (name 객체)
```

- **이유:** `getName` 함수를 **호출한 주체**가 바로 `name` 객체이기 때문입니다. (`name.`으로 호출했죠?)
- 메서드 호출 시 `this`는 해당 메서드를 소유하고 호출한 객체를 가리킵니다.

### 2) 두 번째 콘솔: `second()`

```javascript
// 출력: undefined
```

- **이유:** `second` 함수는 `name.getName()` 안에서 실행되었지만, 호출될 때는 `second()` 처럼 **단독으로 호출**되었습니다.
- 누가 호출했는지 명시되지 않았으므로, Strict Mode에서는 `this`가 **`undefined`**가 됩니다.
  - _(참고: Strict Mode가 아니라면 전역 객체인 `window`가 출력됩니다.)_

### 3) 세 번째 콘솔: `name`

- 단순히 `name` 객체 자체를 출력한 것이므로 첫 번째 결과와 동일합니다.

---

## 3. 해결책: 화살표 함수 (Arrow Function)

그렇다면 내부 함수 `second`에서도 상위 객체인 `name`을 `this`로 쓰고 싶다면 어떻게 해야 할까요?  
ES6에서 등장한 **화살표 함수**를 사용하면 됩니다.

```javascript
const name = {
  name: "오윤희",
  getName: function () {
    console.log("1. 메서드 this:", this);

    // 일반 함수(function) -> 화살표 함수(=>)로 변경
    const second = () => {
      console.log("2. 화살표 함수 this:", this);
    };
    second();
  },
};

name.getName();
```

### 변경 후 실행 결과

![화살표 함수 실행 결과](https://user-images.githubusercontent.com/88264006/219870520-3e3f5479-f269-43bc-a1ba-acbb057e850d.png)

아까 `undefined`였던 두 번째 값이 이제는 **`name` 객체**로 잘 출력됩니다! 🎉

### 왜 그럴까요? (Lexical this)

화살표 함수는 자신만의 `this`를 가지지 않습니다. 대신 **자신이 정의된 위치의 상위 스코프(Lexical Scope)의 `this`를 그대로 물려받습니다.**

즉, `second`가 정의된 곳(`getName`)의 `this`가 `name` 객체였기 때문에, `second`의 `this`도 자연스럽게 `name` 객체가 되는 것입니다.

---

## 마치며

- **일반 함수:** 호출하는 시점에 **"누가 호출했느냐"**에 따라 `this`가 변한다. (동적 바인딩)
- **화살표 함수:** 호출과 상관없이 **"어디에 정의되었느냐"**에 따라 상위 스코프의 `this`를 쓴다. (정적 바인딩)

이 두 가지 차이점만 기억해도 자바스크립트 `this`의 80%는 정복한 것입니다!

혹시 내용 중 이해가 안 가거나 궁금한 점이 있다면 댓글로 남겨주세요. 👋
