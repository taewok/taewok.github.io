---
title: "[JavaScript] 배열 조작: push, pop, unshift, shift 성능과 실전 팁"
date: 2023-03-22T11:30:000
categories: [javascript]
tags: [javascript] #소문자만 가능
---

자바스크립트에서 배열(Array)은 가장 많이 다루는 데이터 구조입니다. 단순히 값을 넣고 빼는 것을 넘어, 각 메서드가 배열의 어느 위치에 영향을 주는지와 그에 따른 성능 차이를 이해하는 것이 중요합니다.

오늘은 배열의 앞뒤를 조작하는 4가지 핵심 메서드와 실무 환경에서의 주의점을 정리해 보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. 배열 뒤쪽 조작: push() & pop()</b>

배열의 **끝부분**을 조작하며, 기존 요소들의 인덱스를 건드리지 않기 때문에 속도가 매우 빠릅니다.

### push(): 끝에 추가

```js
const stack = [1, 2, 3];
stack.push(4);
console.log(stack); // [1, 2, 3, 4]
```

### pop(): 끝에서 삭제 및 추출

```js
const stack = [1, 2, 3];
const lastValue = stack.pop();

console.log(lastValue); // 3 (삭제된 값 반환)
console.log(stack); // [1, 2]
```

---

## <b style="border-bottom:2px solid gray" class="h2">2. 배열 앞쪽 조작: unshift() & shift()</b>

배열의 **맨 앞**을 조작합니다. 이 작업은 뒤에 있는 모든 요소의 인덱스를 하나씩 뒤로 밀거나 앞으로 당겨야 하므로, 데이터 양이 많을 경우 성능에 영향을 줄 수 있습니다.

### unshift(): 앞에 추가

```js
const queue = [1, 2, 3];
queue.unshift(0);
console.log(queue); // [0, 1, 2, 3]
```

### shift(): 앞에서 삭제 및 추출

```js
const queue = [1, 2, 3];
const firstValue = queue.shift();

console.log(firstValue); // 1 (삭제된 값 반환)
console.log(queue); // [2, 3]
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. 실전 개발 경험: 성능과 불변성</b>

### ① 성능 최적화 팁

만약 수만 개의 데이터를 다루는 루프 안에서 배열을 조작해야 한다면, 가급적 `unshift/shift`보다는 `push/pop`을 사용하는 것이 유리합니다. 앞쪽 조작은 $O(n)$의 시간이 걸리지만, 뒤쪽 조작은 $O(1)$로 처리가 가능하기 때문입니다.

- $O(1)$ : 상수 시간 (Constant Time)데이터가 1개든 100만 개든 상관없이 항상 일정한 시간이 걸리는 경우입니다. 가장 이상적이고 빠른 속도죠.
- $O(n)$ : 선형 시간 (Linear Time)데이터의 양($n$)에 비례해서 시간도 정비례하게 늘어나는 경우입니다.

### ② 원본 배열을 지키고 싶다면? (Immutability)

위의 메서드들은 모두 **원본 배열을 직접 수정(Mutable)**합니다. 리액트(React)와 같은 프레임워크에서 상태를 업데이트할 때는 원본을 유지해야 하므로 스프레드 연산자(`...`)를 권장합니다.

```js
const original = [1, 2, 3];

// push 대신 새 배열 생성
const added = [...original, 4]; // [1, 2, 3, 4]

// unshift 대신 새 배열 생성
const prepended = [0, ...original]; // [0, 1, 2, 3]
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>큐(Queue)와 스택(Stack) 구조를 구현할 때 이 메서드들의 차이를 꼭 기억해 두기</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
