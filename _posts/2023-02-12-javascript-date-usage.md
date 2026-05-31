---
title: "[JavaScript] 날짜와 시간을 다루는 Date 객체 정리"
date: 2023-02-12T18:33:00
categories: [javascript]
tags: [javascript, date, time, frontend]
description: "JavaScript에서 날짜와 시간을 다루는 Date 객체의 생성 방법과 getFullYear, getMonth, getDate 같은 주요 메서드를 정리했습니다."
custom_style: true
---

## 🧐 JavaScript에서 날짜는 어떻게 다룰까요?

웹 개발을 하다 보면 날짜와 시간을 다뤄야 하는 경우가 많습니다.

예를 들어 회원가입 일자, 게시글 작성 시간, 디데이 계산, 예약 시간 표시 같은 기능이 모두 날짜와 관련되어 있습니다.

JavaScript에서는 이런 날짜와 시간을 `Date` 객체로 다룰 수 있습니다.

이번 글에서는 `Date` 객체를 만드는 방법과 자주 사용하는 메서드를 정리해보겠습니다.

---

## 🧱 Date 객체 생성하기

### 현재 날짜와 시간 가져오기

아무 값 없이 `new Date()`를 호출하면 현재 날짜와 시간이 담긴 Date 객체가 만들어집니다.

```js
const date = new Date();

console.log(date);
```

실행 결과는 현재 시간에 따라 달라집니다.

```txt
Sun Feb 12 2023 18:44:15 GMT+0900 (한국 표준시)
```

### 특정 날짜 지정하기

`new Date()` 안에 문자열이나 숫자를 넣어 특정 날짜를 만들 수도 있습니다.

```js
const date1 = new Date("02/12/2023");
const date2 = new Date("2023-02-12");
const date3 = new Date("February 12, 2023");
```

여러 형식이 가능하지만, 보통은 `YYYY-MM-DD` 형태의 ISO 형식을 사용하는 편이 안전합니다.

```js
const date = new Date("2023-02-12");
```

---

## 📅 날짜 정보 가져오기

`Date` 객체에서는 연도, 월, 일, 요일 같은 값을 메서드로 꺼낼 수 있습니다.

### getFullYear

`getFullYear()`는 연도를 4자리 숫자로 반환합니다.

```js
const date = new Date("2023-02-12");

console.log(date.getFullYear());
// 2023
```

### getMonth

`getMonth()`는 월을 반환합니다.

여기서 주의할 점이 있습니다. `getMonth()`는 1월을 `0`으로 반환하고, 12월을 `11`로 반환합니다.

```js
const date = new Date("2023-02-12");

console.log(date.getMonth());
// 1
```

2월인데 `1`이 나오는 이유는 월이 0부터 시작하기 때문입니다.

그래서 실제 월을 표시하려면 `+1`을 해주어야 합니다.

```js
const month = date.getMonth() + 1;

console.log(month);
// 2
```

### getDate

`getDate()`는 날짜, 즉 며칠인지를 반환합니다.

```js
const date = new Date("2023-02-12");

console.log(date.getDate());
// 12
```

### getDay

`getDay()`는 요일을 숫자로 반환합니다.

```js
const date = new Date("2023-02-12");

console.log(date.getDay());
// 0
```

요일 숫자는 다음과 같이 매핑됩니다.

```txt
0: 일요일
1: 월요일
2: 화요일
3: 수요일
4: 목요일
5: 금요일
6: 토요일
```

---

## ⏰ 시간 정보 가져오기

시간, 분, 초도 비슷한 방식으로 가져올 수 있습니다.

```js
const date = new Date();

console.log(date.getHours());
console.log(date.getMinutes());
console.log(date.getSeconds());
```

각 메서드는 다음 값을 반환합니다.

- `getHours()`: 0부터 23까지의 시각입니다.
- `getMinutes()`: 0부터 59까지의 분입니다.
- `getSeconds()`: 0부터 59까지의 초입니다.

---

## 🧩 날짜 표시 포맷 만들기

가져온 값을 조합하면 원하는 형태의 날짜 문자열을 만들 수 있습니다.

```js
const date = new Date("2023-02-12");

const year = date.getFullYear();
const month = date.getMonth() + 1;
const day = date.getDate();

const formattedDate = `${year}-${month}-${day}`;

console.log(formattedDate);
// 2023-2-12
```

월과 일이 한 자리일 때 `0`을 붙이고 싶다면 `padStart`를 사용할 수 있습니다.

```js
const month = String(date.getMonth() + 1).padStart(2, "0");
const day = String(date.getDate()).padStart(2, "0");

const formattedDate = `${year}-${month}-${day}`;

console.log(formattedDate);
// 2023-02-12
```

---

## ✅ 정리

JavaScript에서는 `Date` 객체로 날짜와 시간을 다룰 수 있습니다.

- `new Date()`는 현재 날짜와 시간을 만듭니다.
- `new Date("2023-02-12")`처럼 특정 날짜를 만들 수도 있습니다.
- `getFullYear()`는 연도를 가져옵니다.
- `getMonth()`는 월을 가져오지만 0부터 시작하므로 `+1`이 필요합니다.
- `getDate()`는 날짜를 가져옵니다.
- `getDay()`는 요일을 숫자로 가져옵니다.
- `getHours()`, `getMinutes()`, `getSeconds()`로 시간 정보도 가져올 수 있습니다.

특히 `getMonth()`가 0부터 시작한다는 점은 자주 헷갈리니 꼭 기억해두면 좋습니다.
