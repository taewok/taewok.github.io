---
title: "[JavaScript] 해시맵과 Set으로 알고리즘 문제를 푸는 법"
date: 2026-06-19T15:00:00Z
categories: [javascript]
tags: [javascript, hashmap, set, algorithm, data-structure]
description: "해시맵과 Set이 왜 빠른지, JavaScript의 Map과 Set을 어떻게 쓰는지, 그리고 알고리즘 문제에서 자주 만나는 패턴까지 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 왜 해시맵과 Set이 자꾸 나올까요?

알고리즘 문제를 풀다 보면 이런 말을 자주 보게 됩니다.

```txt
해시맵으로 풀기
Set으로 중복 제거
O(1) 조회
```

처음에는 그냥 외워서 쓰게 되지만, 정확히 왜 좋은지 모르면 비슷한 문제에서 다시 막히기 쉽습니다.

이 글에서는 해시맵과 Set을 단순히 자료구조 이름으로 외우는 대신, **언제 쓰면 좋은지, 왜 빠른지, JavaScript에서는 어떻게 쓰는지**를 차근차근 정리해보겠습니다.

핵심은 이겁니다.

```txt
해시맵
→ 값을 빠르게 찾고, 세고, 기록할 때 좋다

Set
→ 중복 여부를 빠르게 확인하고, 이미 본 값을 관리할 때 좋다
```

---

## 💡 해시맵이란 무엇일까요?

해시맵은 `key`와 `value`를 짝지어서 저장하는 구조입니다.

```txt
사과 → 3개
바나나 → 5개
오렌지 → 2개
```

이런 식으로 “무언가를 찾기 위한 이름”과 “실제 값”을 연결합니다.

JavaScript에서는 주로 `Map`을 해시맵처럼 사용합니다.

```ts
const fruitCount = new Map<string, number>();

fruitCount.set("apple", 3);
fruitCount.set("banana", 5);
fruitCount.set("orange", 2);

console.log(fruitCount.get("banana")); // 5
```

이 구조의 강점은 특정 key를 찾을 때 배열처럼 처음부터 끝까지 훑지 않아도 된다는 점입니다.

```txt
배열
→ 앞에서부터 하나씩 확인

Map
→ key로 바로 접근
```

그래서 값 조회, 존재 확인, 개수 누적 같은 작업에서 자주 등장합니다.

---

## 📌 해시맵이 빠른 이유

해시맵은 내부적으로 key를 어떤 규칙에 따라 저장 위치로 바꿉니다.

이 과정을 보통 해싱이라고 부릅니다.

```txt
key
→ hash function
→ 저장 위치
```

덕분에 대부분의 경우 값을 빠르게 찾을 수 있습니다.

실무나 알고리즘 문제에서는 보통 이렇게 기억하면 충분합니다.

```txt
Map은 평균적으로 빠른 조회와 삽입이 가능하다
```

정확한 내부 구현은 언어와 런타임마다 다르지만, 문제 풀이에서는 보통 “빠른 key-value 저장소” 정도로 생각하면 됩니다.

---

## 🧭 Set은 무엇일까요?

Set은 값을 순서보다는 **존재 여부** 중심으로 저장하는 자료구조입니다.

```ts
const seen = new Set<number>();

seen.add(1);
seen.add(2);
seen.add(3);

console.log(seen.has(2)); // true
console.log(seen.has(4)); // false
```

Set의 가장 큰 장점은 중복을 자동으로 허용하지 않는다는 점입니다.

```ts
const numbers = new Set([1, 1, 2, 2, 3]);

console.log([...numbers]); // [1, 2, 3]
```

그래서 Set은 다음 상황에서 자주 쓰입니다.

- 중복 제거
- 이미 본 값 체크
- 방문 여부 관리
- 교집합/차집합 계산

---

## 🛠️ JavaScript에서는 Map과 Set을 어떻게 쓸까요?

### Map 기본 사용법

```ts
const map = new Map<string, number>();

map.set("a", 1);
map.set("b", 2);

console.log(map.get("a")); // 1
console.log(map.has("b")); // true
console.log(map.delete("a")); // true
console.log(map.size); // 1
```

자주 쓰는 메서드는 다음과 같습니다.

| 메서드 | 역할 |
| --- | --- |
| `set` | key-value 저장 |
| `get` | 값 조회 |
| `has` | key 존재 여부 확인 |
| `delete` | 삭제 |
| `clear` | 전체 삭제 |
| `size` | 개수 확인 |

### Set 기본 사용법

```ts
const set = new Set<string>();

set.add("apple");
set.add("banana");
set.add("apple");

console.log(set.has("banana")); // true
console.log(set.size); // 2
```

Set도 자주 쓰는 메서드가 거의 비슷합니다.

| 메서드 | 역할 |
| --- | --- |
| `add` | 값 추가 |
| `has` | 값 존재 여부 확인 |
| `delete` | 삭제 |
| `clear` | 전체 삭제 |
| `size` | 개수 확인 |

---

## ✅ 왜 알고리즘에서 자주 쓰일까요?

배열만으로도 많은 문제를 풀 수 있습니다.

하지만 배열은 특정 값을 찾을 때 선형 탐색이 필요합니다.

```ts
const numbers = [4, 2, 7, 1];

console.log(numbers.includes(7)); // 찾을 수는 있지만 내부적으로 훑습니다
```

값이 많아질수록 이런 탐색은 느려질 수 있습니다.

반면 Map과 Set은 “이미 있는지”, “몇 번 나왔는지”, “어디에 저장됐는지”를 빠르게 확인하기 좋습니다.

그래서 알고리즘 문제에서는 이런 패턴이 자주 나옵니다.

- 중복 제거
- 빈도수 세기
- 두 수의 합
- 첫 번째로 중복 없는 값 찾기
- 방문 체크
- 교집합 구하기

---

## 🧩 패턴 1: 중복 제거

가장 쉬운 예제부터 보겠습니다.

```ts
const items = ["apple", "banana", "apple", "orange", "banana"];

const uniqueItems = [...new Set(items)];

console.log(uniqueItems); // ["apple", "banana", "orange"]
```

이 패턴은 정말 자주 씁니다.

배열을 직접 순회하면서 중복 여부를 검사할 수도 있지만, Set을 쓰면 훨씬 간단해집니다.

```txt
배열
→ Set으로 한 번 감싸기
→ 다시 배열로 펼치기
```

---

## 🧩 패턴 2: 빈도수 세기

해시맵이 가장 빛나는 곳 중 하나가 빈도수 세기입니다.

예를 들어 문자열에서 각 문자가 몇 번 나왔는지 세고 싶다고 해보겠습니다.

```ts
const text = "banana";
const countMap = new Map<string, number>();

for (const char of text) {
  countMap.set(char, (countMap.get(char) ?? 0) + 1);
}

console.log(countMap.get("a")); // 3
console.log(countMap.get("n")); // 2
```

이 패턴은 정말 중요합니다.

```txt
값이 없으면 0으로 시작
값이 있으면 1 더하기
```

자주 쓰는 형태는 이렇게 외우면 됩니다.

```ts
map.set(key, (map.get(key) ?? 0) + 1);
```

이 한 줄을 익혀두면 빈도수 문제를 많이 쉽게 풀 수 있습니다.

---

## 🧩 패턴 3: 두 수의 합

해시맵 문제의 대표 예시는 `Two Sum`입니다.

배열에서 합이 특정 값이 되는 두 수의 인덱스를 찾는 문제입니다.

```ts
function twoSum(nums: number[], target: number): number[] {
  const seen = new Map<number, number>();

  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];

    if (seen.has(need)) {
      return [seen.get(need)!, i];
    }

    seen.set(nums[i], i);
  }

  return [];
}
```

이 코드의 핵심은 현재 숫자를 보면서 “내가 필요한 값이 이미 있었는가?”를 Map으로 바로 확인하는 것입니다.

```txt
현재 값 = 3
target = 10
필요한 값 = 7

7이 이미 Map에 있으면 정답
```

배열을 매번 다시 훑지 않아도 되기 때문에 효율적입니다.

---

## 🧩 패턴 4: 첫 번째로 중복 없는 문자 찾기

Set과 Map은 문자열 문제에서도 자주 나옵니다.

```ts
function firstUniqueChar(s: string): string | null {
  const count = new Map<string, number>();

  for (const char of s) {
    count.set(char, (count.get(char) ?? 0) + 1);
  }

  for (const char of s) {
    if (count.get(char) === 1) {
      return char;
    }
  }

  return null;
}
```

먼저 Map으로 빈도를 세고, 다시 순회하면서 처음 등장한 1회 문자만 찾습니다.

이 방식은 두 번 순회하지만 구조가 매우 읽기 쉽습니다.

---

## 🧩 패턴 5: 방문 여부 체크

그래프, 트리, BFS, DFS 같은 문제에서는 Set이 자주 등장합니다.

```ts
const visited = new Set<number>();

function dfs(node: number) {
  if (visited.has(node)) return;

  visited.add(node);

  // 인접 노드 탐색
}
```

이미 방문한 노드를 다시 처리하지 않기 위해 Set을 씁니다.

이 패턴은 탐색 문제에서 거의 기본처럼 등장합니다.

```txt
방문했는가?
→ Set으로 체크
```

---

## 📦 Map과 Object는 뭐가 다를까요?

JavaScript에서는 `Object`도 키-값 저장소처럼 쓸 수 있습니다.

```ts
const obj: Record<string, number> = {};

obj["apple"] = 3;
obj["banana"] = 5;
```

그렇다면 왜 Map을 따로 쓸까요?

```txt
Object
→ 일반적인 JSON 구조에 좋음

Map
→ key-value 조회, 삽입, 순회에 더 자연스러움
```

Map이 더 편한 점은 다음과 같습니다.

- key 타입이 문자열로 제한되지 않습니다.
- 반복 순서가 명확합니다.
- `size`, `has`, `delete` 같은 메서드가 직관적입니다.

반대로 단순한 설정 객체나 JSON과 맞닿은 구조라면 Object가 더 자연스러울 수 있습니다.

즉, 둘 중 무엇이 절대적으로 좋다기보다 **쓰임새가 다르다**고 보는 편이 맞습니다.

---

## 🧭 언제 Map을 쓰고 언제 Set을 쓸까요?

간단히 나누면 이렇게 생각할 수 있습니다.

```txt
Map
→ 무엇인가를 key로 빠르게 찾고 싶을 때

Set
→ 값이 이미 있는지, 중복인지 빠르게 확인하고 싶을 때
```

예를 들면:

- 단어별 등장 횟수
- 사용자 ID와 데이터 매핑
- 캐시 저장
- 중복 제거
- 방문 노드 관리
- 교집합 검사

이런 문제에는 Map과 Set이 잘 맞습니다.

---

## 🚧 자주 헷갈리는 부분

### 1. Set은 key-value가 아닙니다

Set은 값만 저장합니다.

```ts
const set = new Set<string>();
set.add("apple");
```

```txt
Map = key-value
Set = value only
```

### 2. Map의 get은 없을 수도 있습니다

```ts
const value = map.get("x");
```

`x`가 없으면 `undefined`가 나올 수 있으니, 바로 숫자 연산을 하기 전에 기본값을 넣는 편이 안전합니다.

```ts
map.set(key, (map.get(key) ?? 0) + 1);
```

### 3. Set은 중복을 자동으로 제거합니다

이건 장점이지만, “중복이 있던 상태”를 따로 보존하고 싶다면 Set만 쓰면 안 됩니다.

### 4. 해시맵은 자료구조 개념이고, JavaScript에서는 주로 Map으로 구현합니다

개념과 언어 API를 분리해서 생각하면 덜 헷갈립니다.

```txt
해시맵 = 개념
Map = JavaScript에서 그 개념을 쓰는 대표 API
```

---

## ✅ 정리

해시맵과 Set은 알고리즘 문제에서 매우 자주 나오는 이유가 분명합니다.

둘 다 배열보다 “존재 여부”와 “빠른 조회”에 유리하기 때문입니다.

핵심만 다시 잡아보면 이렇습니다.

- 해시맵은 key-value를 빠르게 다루는 구조입니다.
- JavaScript에서는 주로 `Map`을 사용합니다.
- Set은 값의 존재 여부와 중복 제거에 강합니다.
- `Map.get(key) ?? 0` 패턴은 빈도수 문제에서 특히 자주 씁니다.
- `new Set(array)`는 중복 제거의 가장 쉬운 시작점입니다.
- 방문 체크, 교집합, 두 수의 합 같은 문제에서 Map과 Set이 자주 등장합니다.

처음에는 “해시맵으로 풀자”라는 말을 그냥 외우게 되지만, 실제로는 다음 질문으로 바꿔 생각하면 훨씬 편합니다.

```txt
지금 필요한 건 빠른 조회인가?
중복 체크인가?
빈도수 누적인가?
방문 여부 관리인가?
```

이 질문에 답이 `예`라면, Map과 Set을 먼저 떠올려보면 좋습니다.

알고리즘 문제에서 해시맵과 Set을 잘 쓰게 되면, 코드도 짧아지고 사고도 한결 단순해집니다.
