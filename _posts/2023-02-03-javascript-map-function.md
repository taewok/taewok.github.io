---
title: "[JavaScript] 자바스크립트 배열 map 함수 완벽 정리 (사용법, 예제, 객체 조작)"
date: 2023-02-03T19:10:00
categories: [javascript]
tags: [javascript, map, array, es6]
description: "JavaScript 배열 메서드인 map의 핵심 원리, 3가지 매개변수(value, index, array), 그리고 실무에서 자주 쓰이는 객체 배열 조작 예제까지 상세히 알아봅니다."
custom_style: true
---

JavaScript에서 배열을 다룰 때 `filter`만큼이나, 아니 그보다 더 자주 사용되는 메서드가 바로 `map`입니다.  
`filter`가 조건을 만족하는 요소를 '걸러내는' 역할이라면, `map`은 배열의 모든 요소를 입맛대로 '변환(Mapping)'하여 새로운 배열을 만들어냅니다. 😊

이번 글에서는 `map` 함수의 기본 문법과 매개변수, 그리고 실무에서 유용한 활용 예제들을 정리해 보겠습니다.

---

## 1. map 함수 기본 문법과 매개변수

`map` 메서드는 배열의 요소를 하나씩 순회하면서 콜백 함수를 실행하고, **콜백 함수가 리턴(Return)하는 값들로 이루어진 새로운 배열**을 반환합니다.

```javascript
const newArray = array.map((value, index, array) => {
  return 반환할_값; // 이 값이 새로운 배열의 요소가 됩니다.
});
```

`map`의 콜백 함수는 `filter`와 마찬가지로 3가지 매개변수를 가집니다.

1.  **value**: 현재 처리 중인 요소의 값
2.  **index**: 현재 요소의 인덱스 (0부터 시작)
3.  **array**: `map`을 호출한 원본 배열

각 매개변수가 어떻게 동작하는지 예제로 확인해 볼까요?

```javascript
const array = [1, 2, 3, 4, 5];
```

### 1) value (요소 값)

가장 기본이 되는 인자입니다. 배열의 요소들을 순서대로 받아옵니다.

```javascript
array.map((value) => console.log(value));

// 실행 결과
// 1
// 2
// 3
// 4
// 5
```

### 2) index (인덱스)

현재 요소가 배열의 몇 번째인지 알려주는 순서(Index)입니다.

```javascript
array.map((value, index) => console.log(index));

// 실행 결과
// 0
// 1
// 2
// 3
// 4
```

### 3) array (원본 배열)

현재 순회 중인 배열 전체를 참조합니다.

```javascript
array.map((value, index, array) => console.log(array));

// 실행 결과
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// ... (총 5번 출력)
```

---

## 2. map 함수 실전 활용 예시

`map`은 `filter`와 달리 **조건(True/False)**이 아니라, **변형된 값(Result)**을 반환한다는 점이 핵심입니다.

### 1) 기존 값에 연산 적용하기 (2배 만들기)

모든 요소에 2를 곱한 새로운 배열을 만들 수 있습니다.

```javascript
const array = [1, 2, 3, 4, 5];
const newArray = array.map((value) => value * 2);

console.log(newArray);

// 실행 결과
// [2, 4, 6, 8, 10]
```

### 2) 조건부 데이터 변환 (if문 활용)

특정 조건에 맞는 요소만 다른 값으로 바꿀 수도 있습니다. 예를 들어 3 이상인 숫자는 `*`로 바꿔볼까요?

```javascript
const array = [1, 2, 3, 4, 5];

const newArray = array.map((value) => {
  if (value >= 3) {
    return "*";
  } else {
    return value;
  }
});

console.log(newArray);

// 실행 결과
// [1, 2, '*', '*', '*']
```

### 3) 객체(Object) 배열에서 특정 데이터만 뽑기 ⭐

실무(React 등)에서 가장 많이 쓰이는 패턴입니다. 객체로 이루어진 배열에서 특정 키(Key)값만 추출하여 문자열 배열로 만들 수 있습니다.

```javascript
const users = [
  { name: "최혜정", age: 27 },
  { name: "박연진", age: 21 },
  { name: "손명오", age: 25 },
  { name: "문동은", age: 23 },
];

// user 객체에서 name만 추출
const names = users.map((user) => user.name);

console.log(names);

// 실행 결과
// ['최혜정', '박연진', '손명오', '문동은']
```

---

## 3. 중요: 원본 배열 불변성 (Immutability)

`map` 함수 역시 `filter`와 마찬가지로 **기존 배열(Original Array)을 변경하지 않습니다.** 변형된 결과는 오직 **새로운 배열(New Array)**에만 담깁니다.

```javascript
const array = [1, 2, 3, 4, 5];
const newArray = array.map((value) => value * 2);

console.log("원본 배열:", array);
console.log("새로운 배열:", newArray);

// 실행 결과
// 원본 배열: [1, 2, 3, 4, 5]  <-- 변하지 않음!
// 새로운 배열: [2, 4, 6, 8, 10]
```

이러한 특성 덕분에 데이터의 원본을 안전하게 지키면서 자유롭게 데이터를 가공할 수 있습니다.

---

## 마치며

오늘은 자바스크립트에서 데이터를 변환할 때 필수적인 `map` 함수에 대해 알아보았습니다.
배열의 모든 요소를 가공하거나, 객체 배열에서 원하는 정보만 쏙 뽑아낼 때 정말 유용하게 쓰이니 꼭 익혀두시길 바랍니다!

내용 중 궁금한 점이나 피드백이 있다면 언제든 댓글 남겨주세요. 👋
