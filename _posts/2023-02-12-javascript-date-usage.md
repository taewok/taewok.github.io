---
title: "[JavaScript] 자바스크립트 날짜(Date) 객체와 주요 메서드 정리"
date: 2023-02-12T18:33:00
categories: [javascript]
tags: [javascript, date, time, frontend]
description: "JavaScript에서 날짜와 시간을 다루는 Date 객체의 생성 방법부터 getFullYear, getMonth 등 자주 사용하는 필수 메서드들을 예제와 함께 정리합니다."
custom_style: true
---

웹 개발을 하다 보면 회원가입 일자, 게시글 작성 시간, 디데이 계산 등 **날짜와 시간**을 다뤄야 하는 경우가 정말 많습니다.  
JavaScript에서는 **`Date` 객체**를 통해 이러한 년, 월, 일, 시, 분, 초, 밀리초 단위의 시간 정보를 쉽게 처리할 수 있습니다. 🕒

이번 글에서는 `Date` 객체의 생성 방법과 실무에서 자주 쓰이는 주요 메서드들을 정리해 보겠습니다.

---

## 1. Date 객체 생성하기 (기본 사용법)

### 1) 현재 시간 가져오기 (`new Date()`)

아무런 매개변수 없이 `new Date()`를 호출하면, 코드가 실행되는 시점의 **현재 날짜와 시간**을 반환합니다.

```javascript
const date = new Date();
console.log(date);

// 실행 결과 (현재 시간에 따라 다름)
// Sun Feb 12 2023 18:44:15 GMT+0900 (한국 표준시)
```

### 2) 특정 날짜 지정하기

`new Date()`의 괄호 안에 문자열이나 숫자로 특정 날짜를 넣어주면 해당 날짜 객체가 생성됩니다.

- `"MM/DD/YYYY"`
- `"YYYY-MM-DD"` (가장 권장하는 ISO 형식)
- `"Month DD YYYY"`

```javascript
const date1 = new Date("02/12/2023");
const date2 = new Date("2023-02-12"); // 권장
const date3 = new Date("February 12, 2023");

console.log(date1);
// 실행 결과: Sun Feb 12 2023 00:00:00 GMT+0900 (한국 표준시)
```

---

## 2. 날짜 및 시간 정보 가져오기 (Get Methods)

`Date` 객체에서 원하는 정보(년, 월, 일 등)만 쏙쏙 뽑아내는 메서드들입니다.  
**주의해야 할 점(함정)이 몇 가지 있으니 꼭 확인하세요!**

### 1) getFullYear() : 연도

현재 연도를 **4자리 정수**로 반환합니다.

```javascript
const date = new Date(); // 2023년 2월 12일 기준
const year = date.getFullYear();

console.log(year);
// 실행 결과: 2023
```

### 2) getMonth() : 월 (⭐ 주의)

**가장 많이 실수하는 부분입니다.** `getMonth()`는 1월이 0부터 시작하여 12월이 11로 끝납니다. 따라서 **실제 월을 구하려면 반드시 `+1`을 해줘야 합니다.**

- 0: 1월
- 11: 12월

```javascript
const date = new Date();
const month = date.getMonth();

console.log(month); // 2월인 경우 1이 출력됨
console.log(month + 1); // 실제 월을 쓰려면 +1 필수! (결과: 2)
```

### 3) getDate() : 일

현재 날짜(일)를 정수로 반환합니다.

```javascript
const date = new Date();
const day = date.getDate();

console.log(day);
// 실행 결과: 12
```

### 4) getDay() : 요일

요일을 **0부터 6까지의 정수**로 반환합니다. 일요일부터 시작합니다.

- **0: 일요일**
- 1: 월요일
- ...
- **6: 토요일**

```javascript
const date = new Date();
const dayOfWeek = date.getDay();

console.log(dayOfWeek);
// 실행 결과: 0 (일요일인 경우)
```

---

## 3. 시간 정보 가져오기

시간, 분, 초 단위도 손쉽게 가져올 수 있습니다.

### 1) getHours() : 시

0부터 23까지의 정수(24시간제)를 반환합니다.

```javascript
const hours = new Date().getHours();
console.log(hours); // 예: 18
```

### 2) getMinutes() : 분

0부터 59까지의 정수를 반환합니다.

```javascript
const minutes = new Date().getMinutes();
console.log(minutes); // 예: 44
```

### 3) getSeconds() : 초

0부터 59까지의 정수를 반환합니다.

```javascript
const seconds = new Date().getSeconds();
console.log(seconds); // 예: 15
```

---

## 마치며

오늘은 자바스크립트의 기본인 `Date` 객체와 주요 메서드에 대해 알아보았습니다.
특히 **`getMonth()`는 0부터 시작한다**는 점, **`getDay()`는 요일을 숫자로 반환한다**는 점을 꼭 기억해 주세요!

혹시 궁금한 점이나 피드백이 있다면 언제든 댓글 남겨주세요. 👋
