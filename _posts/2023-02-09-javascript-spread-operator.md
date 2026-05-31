---
title: "[JavaScript] 스프레드 연산자(...) 정리"
date: 2023-02-09T13:40:00
categories: [javascript]
tags: [javascript, es6, spread, syntax]
description: "ES6의 스프레드 연산자(...)가 무엇인지, 배열과 객체를 복사하거나 병합할 때 어떻게 사용하는지 예제로 정리했습니다."
custom_style: true
---

## 🧐 스프레드 연산자는 언제 사용할까요?

JavaScript를 작성하다 보면 배열이나 객체를 복사하거나, 기존 값에 새로운 값을 추가해야 하는 경우가 많습니다.

이때 자주 사용하는 문법이 스프레드 연산자입니다.

```js
...
```

점 세 개로 이루어진 이 문법은 배열, 객체, 문자열처럼 여러 값을 가진 데이터를 펼쳐서 사용할 수 있게 해줍니다.

이번 글에서는 스프레드 연산자가 무엇인지, 배열과 객체에서 어떻게 사용하는지 차근차근 정리해보겠습니다.

---

## 💡 스프레드 연산자란?

스프레드(Spread)는 "펼치다"라는 뜻을 가지고 있습니다.

이름처럼 스프레드 연산자는 배열이나 객체처럼 묶여 있는 값을 개별 요소로 펼쳐줍니다.

```js
const numbers = [1, 2, 3];

console.log(...numbers);
// 1 2 3
```

배열 안에 있던 `1`, `2`, `3`이 하나씩 펼쳐져서 출력됩니다.

---

## 🧱 객체 복사와 속성 추가하기

스프레드 연산자는 객체를 복사하거나, 기존 객체에 새로운 속성을 추가할 때 자주 사용합니다.

```js
const user = {
  name: "오윤희",
  job: "vocalist",
};

const nextUser = {
  ...user,
  age: 44,
};

console.log(nextUser);
// { name: "오윤희", job: "vocalist", age: 44 }
```

`...user`는 `user` 객체 안에 있는 속성들을 새 객체 안으로 펼쳐줍니다.

그래서 `nextUser`는 기존 `name`, `job`을 유지하면서 `age`를 추가한 새 객체가 됩니다.

---

## ✏️ 객체 속성 덮어쓰기

같은 이름의 속성을 뒤에 작성하면 기존 값이 덮어써집니다.

```js
const user = {
  name: "오윤희",
  job: "vocalist",
};

const nextUser = {
  ...user,
  job: "teacher",
};

console.log(nextUser);
// { name: "오윤희", job: "teacher" }
```

객체는 뒤에 있는 속성이 우선 적용됩니다.

이 패턴은 React에서 상태 객체의 일부 값만 변경할 때도 자주 사용합니다.

```js
setUser((prev) => ({
  ...prev,
  job: "teacher",
}));
```

---

## 📦 배열 복사와 요소 추가하기

배열에서도 스프레드 연산자를 사용할 수 있습니다.

```js
const array = ["진천댁", "주단태", "주석훈"];

const newArray = [...array, "주석경"];

console.log(newArray);
// ["진천댁", "주단태", "주석훈", "주석경"]
```

`...array`가 기존 배열의 요소들을 펼쳐주고, 뒤에 `"주석경"`을 추가합니다.

원본 배열은 그대로 유지되고, 새로운 배열이 만들어집니다.

```js
console.log(array);
// ["진천댁", "주단태", "주석훈"]
```

---

## 🔗 배열 합치기

여러 배열을 합칠 때도 스프레드 연산자를 사용할 수 있습니다.

```js
const front = ["HTML", "CSS"];
const script = ["JavaScript", "TypeScript"];

const skills = [...front, ...script];

console.log(skills);
// ["HTML", "CSS", "JavaScript", "TypeScript"]
```

`concat`을 사용해도 되지만, 스프레드 문법을 사용하면 더 직관적으로 읽히는 경우가 많습니다.

---

## 🔤 문자열을 배열로 바꾸기

문자열도 펼칠 수 있습니다.

```js
const name = "심수련";

console.log([...name]);
// ["심", "수", "련"]
```

문자열을 배열 안에서 펼치면 글자 하나하나가 배열의 요소가 됩니다.

---

## ⚠️ 얕은 복사에 주의하기

스프레드 연산자는 원본을 직접 수정하지 않고 새 배열이나 객체를 만들어줍니다.

하지만 깊은 복사가 아니라 **얕은 복사**라는 점은 주의해야 합니다.

```js
const user = {
  name: "오윤희",
  profile: {
    age: 44,
  },
};

const copiedUser = { ...user };

copiedUser.profile.age = 45;

console.log(user.profile.age);
// 45
```

겉의 객체는 새로 만들어졌지만, 안쪽의 `profile` 객체는 같은 참조를 공유하고 있습니다.

중첩 객체까지 안전하게 복사해야 한다면 `structuredClone`이나 별도의 깊은 복사 방법을 고려해야 합니다.

---

## ✅ 정리

스프레드 연산자는 배열이나 객체를 펼쳐서 사용할 수 있게 해주는 문법입니다.

- 배열을 복사하거나 합칠 때 사용할 수 있습니다.
- 객체를 복사하거나 일부 속성을 변경할 때 사용할 수 있습니다.
- 문자열을 글자 배열로 만들 수도 있습니다.
- 원본 데이터를 직접 수정하지 않는 패턴을 만들기 좋습니다.
- 다만 중첩 객체는 얕은 복사만 된다는 점을 조심해야 합니다.

React에서 상태를 다룰 때도 정말 자주 쓰이는 문법이니 익숙해져두면 좋습니다.
