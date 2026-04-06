---
title: "[JavaScript] 자바스크립트 배열 filter 함수 완벽 정리 (사용법, 예제)"
date: 2023-02-01
categories: [javascript]
tags: [javascript, filter, array, es6]
description: "자바스크립트 배열 메서드인 filter의 기본 문법, 3가지 매개변수(value, index, array), 그리고 원본 배열 보존 특성까지 예제를 통해 쉽게 알아봅니다."
custom_style: true
---

JavaScript에서 배열(Array)을 다룰 때 가장 자주 사용하고 유용한 메서드 중 하나가 바로 `filter`입니다.  
이름 그대로 특정 조건을 만족하는 요소들만 **걸러내어** 새로운 배열을 만들 때 필수적으로 사용됩니다. 😊

이번 글에서는 `filter` 함수의 기본 문법부터 매개변수 활용, 그리고 실무에서 중요한 불변성(Immutability) 특징까지 예제와 함께 정리해보겠습니다.

---

## 1. filter 함수 기본 문법

`filter` 메서드는 배열의 각 요소를 순회하면서 콜백 함수를 실행합니다. 이때 콜백 함수의 반환 값이 `true`인 요소만 모아서 **새로운 배열**을 반환합니다.

```javascript
const newArray = array.filter((value, index, array) => {
  return 조건식; // 조건이 true면 유지, false면 버림
});
```

**핵심 요약:** 조건에 맞는 요소만 '필터링'하여 추출합니다.

<br/>

## 2. filter의 세 가지 매개변수 (Parameters)

`filter`의 콜백 함수는 다음과 같은 세 가지 인자를 받을 수 있습니다. 필요에 따라 골라서 사용하면 됩니다.

1.  `value`: 현재 처리 중인 요소의 값
2.  `index`: 현재 처리 중인 요소의 인덱스 (선택 사항)
3.  `array`: `filter`를 호출한 배열 자체 (선택 사항)

예제 코드로 각 매개변수가 무엇을 출력하는지 확인해 볼까요?

```javascript
const array = [1, 2, 3, 4, 5];
```

### 1) value (요소 값)

가장 많이 사용하는 매개변수입니다. 배열의 각 값을 순서대로 받아옵니다.

```javascript
array.filter((value) => console.log(value));

// 실행 결과
// 1
// 2
// 3
// 4
// 5
```

### 2) index (인덱스 번호)

현재 요소가 배열의 몇 번째인지 알려주는 인덱스(0부터 시작)입니다.

```javascript
array.filter((value, index) => console.log(index));

// 실행 결과
// 0
// 1
// 2
// 3
// 4
```

### 3) array (원본 배열 참조)

현재 `filter`를 돌고 있는 배열 그 자체를 가져옵니다. 5개의 요소가 있으므로 배열 전체가 5번 출력됩니다.

```javascript
array.filter((value, index, array) => console.log(array));

// 실행 결과
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
// [1, 2, 3, 4, 5]
```

---

## 3. 실전 예제: 원하는 조건의 값만 추출하기

`filter`의 진가는 조건식과 함께할 때 발휘됩니다.  
예를 들어, 배열에서 **3 이상인 숫자만** 뽑아내고 싶다면 어떻게 해야 할까요?

```javascript
const array = [1, 2, 3, 4, 5];

// value가 3보다 크거나 같은 경우만 true를 반환
const newArray = array.filter((value) => value >= 3);

console.log(newArray);

// 실행 결과
// [3, 4, 5]
```

위와 같이 조건식이 `true`인 값들만 모아 새로운 배열인 `[3, 4, 5]`가 생성되었습니다.

---

## 4. 중요: 원본 배열은 변하지 않습니다 (불변성)

`filter` 함수를 사용할 때 가장 중요한 특징 중 하나는 **기존 배열(Original Array)을 변경하지 않는다**는 점입니다. 이를 **불변성(Immutability)**을 지킨다고 표현합니다.

`filter`는 항상 **새로운 배열(New Array)**을 반환합니다.

```javascript
const array = [1, 2, 3, 4, 5];
const newArray = array.filter((value) => value >= 3);

console.log("원본 배열:", array);
console.log("새로운 배열:", newArray);

// 실행 결과
// 원본 배열: [1, 2, 3, 4, 5]  <-- 변하지 않음!
// 새로운 배열: [3, 4, 5]
```

이러한 특성 덕분에 리액트(React)와 같은 라이브러리에서 상태를 관리할 때 매우 안전하고 유용하게 사용됩니다.

---

## 마치며

오늘은 자바스크립트의 필수 메서드인 `filter`에 대해 알아보았습니다. 데이터를 원하는 대로 가공할 때 정말 많이 쓰이니 꼭 익혀두시길 바랍니다!

혹시 내용 중 수정이 필요하거나 궁금한 점이 있다면 편하게 댓글 남겨주세요.  
피드백은 언제나 환영입니다! 👋
