---
title: "[JavaScript] 배열 앞뒤를 조작하는 push, pop, unshift, shift 정리"
date: 2023-03-22T11:30:00
categories: [javascript]
tags: [javascript, array, push, pop, shift, unshift]
description: "JavaScript 배열의 앞뒤를 조작하는 push, pop, unshift, shift의 차이와 원본 배열 변경 여부, 성능상 주의점을 정리했습니다."
custom_style: true
---

## 🧐 배열 앞뒤에 값을 넣고 빼는 방법

JavaScript에서 배열은 정말 자주 사용하는 자료구조입니다. 단순히 값을 저장하는 것뿐 아니라, 앞이나 뒤에 값을 추가하고 제거하는 일도 많습니다.

이때 사용하는 대표적인 메서드가 `push`, `pop`, `unshift`, `shift`입니다.

이번 글에서는 네 가지 메서드가 배열의 어느 위치를 조작하는지, 그리고 사용할 때 어떤 점을 주의해야 하는지 정리해보겠습니다.

---

## ➕ push: 배열 끝에 추가하기

`push`는 배열의 끝에 값을 추가합니다.

```js
const stack = [1, 2, 3];

stack.push(4);

console.log(stack);
// [1, 2, 3, 4]
```

기존 배열의 뒤에 값이 붙습니다.

---

## ➖ pop: 배열 끝에서 제거하기

`pop`은 배열의 마지막 값을 제거하고, 제거한 값을 반환합니다.

```js
const stack = [1, 2, 3];

const lastValue = stack.pop();

console.log(lastValue);
// 3

console.log(stack);
// [1, 2]
```

스택처럼 마지막에 들어간 값을 먼저 꺼내는 구조에서 자주 사용합니다.

---

## ⬅️ unshift: 배열 앞에 추가하기

`unshift`는 배열의 맨 앞에 값을 추가합니다.

```js
const queue = [1, 2, 3];

queue.unshift(0);

console.log(queue);
// [0, 1, 2, 3]
```

다만 앞에 값을 추가하면 기존 요소들의 인덱스가 모두 뒤로 밀립니다.

---

## ➡️ shift: 배열 앞에서 제거하기

`shift`는 배열의 첫 번째 값을 제거하고, 제거한 값을 반환합니다.

```js
const queue = [1, 2, 3];

const firstValue = queue.shift();

console.log(firstValue);
// 1

console.log(queue);
// [2, 3]
```

큐처럼 먼저 들어간 값을 먼저 꺼내는 구조에서 사용할 수 있습니다.

---

## ⚠️ 원본 배열을 직접 변경합니다

`push`, `pop`, `unshift`, `shift`는 모두 원본 배열을 직접 변경합니다.

```js
const original = [1, 2, 3];

original.push(4);

console.log(original);
// [1, 2, 3, 4]
```

React 상태를 업데이트할 때는 원본 배열을 직접 수정하지 않는 것이 중요합니다. 그래서 스프레드 연산자로 새 배열을 만드는 패턴을 자주 사용합니다.

```js
const original = [1, 2, 3];

const added = [...original, 4];
const prepended = [0, ...original];
```

---

## 📊 성능 관점에서 보기

배열의 끝을 조작하는 `push`, `pop`은 보통 빠르게 처리됩니다. 기존 요소들의 인덱스를 바꿀 필요가 거의 없기 때문입니다.

반면 배열의 앞을 조작하는 `unshift`, `shift`는 기존 요소들의 인덱스를 다시 계산해야 합니다.

```txt
push / pop
→ 배열 끝 조작, 비교적 빠름

unshift / shift
→ 배열 앞 조작, 많은 요소의 인덱스가 바뀜
```

데이터가 적을 때는 큰 차이를 느끼기 어렵지만, 매우 큰 배열을 자주 조작한다면 앞쪽 조작은 주의하는 것이 좋습니다.

---

## ✅ 정리

배열의 앞뒤를 조작할 때는 다음 메서드를 사용할 수 있습니다.

- `push`: 배열 끝에 추가합니다.
- `pop`: 배열 끝에서 제거합니다.
- `unshift`: 배열 앞에 추가합니다.
- `shift`: 배열 앞에서 제거합니다.

네 메서드는 모두 원본 배열을 직접 변경합니다. React 상태처럼 불변성이 중요한 데이터라면 스프레드 연산자 등을 사용해 새 배열을 만드는 방식을 고려하면 좋습니다.
