---
title: "[JavaScript] 배열 자르기 완벽 비교: slice() vs splice() 차이점"
date: 2023-03-30T11:10:000
categories: [javascript]
tags: [javascript] #소문자만 가능
---

자바스크립트에서 배열의 일부분을 추출하거나 수정할 때 사용하는 `slice()`와 `splice()`. 이름은 비슷하지만 동작 방식은 완전히 다릅니다. 특히 **원본 배열을 건드리느냐**의 차이는 실무에서 버그를 줄이는 핵심 포인트입니다.

오늘은 이 두 함수의 차이점을 명확하게 정리해 보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. slice(): 원본은 그대로, 필요한 만큼만 복사</b>

`slice()`는 배열에서 원하는 범위를 복사하여 **새로운 배열**을 만듭니다. 원본 배열이 변하지 않기 때문에 데이터의 불변성을 유지해야 하는 상황(예: React State)에서 주로 사용합니다.

### 기본 사용법

```js
const array = ["a", "b", "c", "d", "e"];
const start = 0;
const end = 3;

// 인덱스 0부터 3 직전(2번 인덱스)까지 복사
const newArray = array.slice(start, end);

console.log(newArray); // ["a", "b", "c"]
console.log(array); // ["a", "b", "c", "d", "e"] (원본 보존)
```

- start: 시작점 인덱스
- end: 종료 지점 인덱스 (**해당 인덱스 직전**까지만 포함됨에 주의!)
- 팁: 인자를 비우고 `slice()`를 실행하면 전체 배열을 복사합니다.

---

## <b style="border-bottom:2 border-bottom:2px solid gray" class="h2">2. splice(): 원본을 직접 수정 (삭제 및 추가)</b>

`splice()`는 배열의 요소를 삭제하거나, 교체하거나, 새 요소를 추가하여 **원본 배열 자체를 변경**합니다.

### 삭제와 동시에 값 추출

```js
const array = ["a", "b", "c", "d", "e"];
const start = 1;
const deleteCount = 3;

// 인덱스 1부터 3개를 삭제하고 삭제된 요소를 반환
const deletedItems = array.splice(start, deleteCount);

console.log(deletedItems); // ["b", "c", "d"]
console.log(array); // ["a", "e"] (원본이 변경됨)
```

### 삭제 없이 값만 추가하기

`deleteCount`를 0으로 설정하면 특정 위치에 값을 삽입할 수 있습니다.

```js
const array = ["a", "b", "e"];
// 인덱스 2번 위치에 아무것도 지우지 않고 "c", "d" 추가
array.splice(2, 0, "c", "d");

console.log(array); // ["a", "b", "c", "d", "e"]
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. 핵심 비교 요약</b>

| 구분         | slice()                | splice()              |
| :----------- | :--------------------- | :-------------------- |
| 원본 배열    | 보존 (Immutable)       | 변경 (Mutable)        |
| 반환값       | 잘라낸 새 배열         | 삭제된 요소들의 배열  |
| 주요 용도    | 단순 복사 및 부분 추출 | 요소 삭제, 삽입, 교체 |
| 두 번째 인자 | 종료 인덱스 (미포함)   | 삭제할 개수           |

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>원본 데이터를 보호해야 하는 상황인지, 혹은 원본을 직접 수정해서 관리해야 하는 상황인지에 따라 <code>slice</code>와 <code>splice</code>를 적절히 선택하는 것이 중요합니다. 특히 의도치 않게 원본 데이터가 바뀌어 버그를 찾는 일이 없도록 주의하세요!</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
