---
title: "[HTML/JS] localStorage와 sessionStorage 차이점 정리"
date: 2023-03-02T15:31:00
categories: [html, javascript]
tags: [html, javascript, web-storage, localstorage, sessionstorage]
description: "브라우저에 데이터를 저장하는 Web Storage API인 localStorage와 sessionStorage의 차이점과 기본 사용법을 정리했습니다."
custom_style: true
---

## 🧐 브라우저에 데이터를 저장하려면?

웹 개발을 하다 보면 간단한 데이터를 브라우저에 저장해야 할 때가 있습니다. 예를 들어 테마 설정, 임시 입력값, 장바구니 정보 같은 데이터가 여기에 해당합니다.

이때 사용할 수 있는 대표적인 저장소가 `localStorage`와 `sessionStorage`입니다.

둘 다 브라우저의 Web Storage API에 속하지만, 데이터가 유지되는 시간이 다릅니다.

---

## 💾 localStorage

`localStorage`는 브라우저를 닫아도 데이터가 유지되는 저장소입니다.

사용자가 직접 삭제하거나 코드로 삭제하기 전까지 데이터가 남아 있습니다.

```js
localStorage.setItem("username", "오윤희");

const name = localStorage.getItem("username");
console.log(name);
// "오윤희"

localStorage.removeItem("username");
```

전체 데이터를 비우고 싶다면 `clear`를 사용할 수 있습니다.

```js
localStorage.clear();
```

자동 로그인 여부, 다크 모드 설정, 장바구니 데이터처럼 브라우저를 닫아도 유지되어야 하는 값에 사용할 수 있습니다.

---

## 🧳 sessionStorage

`sessionStorage`는 탭이나 브라우저 창을 닫으면 데이터가 사라지는 저장소입니다.

사용법은 `localStorage`와 거의 같습니다.

```js
sessionStorage.setItem("token", "abc-123");

const token = sessionStorage.getItem("token");
console.log(token);
// "abc-123"

sessionStorage.removeItem("token");
```

전체 데이터를 비울 때는 마찬가지로 `clear`를 사용할 수 있습니다.

```js
sessionStorage.clear();
```

일회성 입력값, 현재 탭에서만 필요한 상태, 임시 데이터에 사용하기 좋습니다.

---

## ⚠️ 문자열만 저장할 수 있어요

Web Storage에는 문자열만 저장할 수 있습니다.

객체나 배열을 그대로 넣으면 원하는 형태로 저장되지 않을 수 있습니다.

```js
const user = {
  name: "오윤희",
  age: 30,
};

localStorage.setItem("user", JSON.stringify(user));
```

가져올 때는 `JSON.parse`로 다시 객체로 변환합니다.

```js
const savedUser = localStorage.getItem("user");

if (savedUser) {
  const user = JSON.parse(savedUser);
  console.log(user.name);
}
```

---

## 📊 localStorage vs sessionStorage

가장 큰 차이는 데이터가 얼마나 오래 유지되는지입니다.

| 구분 | localStorage | sessionStorage |
| :--- | :--- | :--- |
| 데이터 수명 | 직접 삭제하기 전까지 유지 | 탭이나 창을 닫으면 삭제 |
| 데이터 범위 | 같은 도메인에서 공유 | 같은 탭 안에서만 유지 |
| 사용 예시 | 테마 설정, 자동 로그인 여부 | 임시 입력값, 현재 탭 상태 |
| 저장 형식 | 문자열 | 문자열 |

---

## ✅ 정리

`localStorage`와 `sessionStorage`는 브라우저에 간단한 데이터를 저장할 때 사용할 수 있습니다.

- 오래 유지해야 하는 데이터는 `localStorage`에 저장합니다.
- 탭을 닫으면 사라져도 되는 데이터는 `sessionStorage`에 저장합니다.
- 둘 다 문자열만 저장할 수 있습니다.
- 객체나 배열은 `JSON.stringify`, `JSON.parse`를 함께 사용합니다.

데이터가 얼마나 오래 유지되어야 하는지를 기준으로 둘 중 하나를 선택하면 됩니다.
