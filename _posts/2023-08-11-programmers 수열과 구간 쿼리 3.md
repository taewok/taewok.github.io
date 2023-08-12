---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (수열과 구간 쿼리 3)"
date: 2023-08-11T16:55:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr와 2차원 정수 배열 queries이 주어집니다. queries의 원소는 각각 하나의 query를 나타내며, [i, j] 꼴입니다.

각 query마다 순서대로 arr[i]의 값과 arr[j]의 값을 서로 바꿉니다.

위 규칙에 따라 queries를 처리한 이후의 arr를 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 1,000
  - 0 ≤ arr의 원소 ≤ 1,000,000
- 1 ≤ queries의 길이 ≤ 1,000
  - 0 ≤ i < j < arr의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| arr             |        queries         | 결과            |
| --------------- | :--------------------: | --------------- |
| [0, 1, 2, 3, 4] | [[0, 3],[1, 2],[1, 4]] | [3, 4, 1, 0, 2] |

## <b>풀이</b>

```js
function solution(arr, queries) {
  for (let value of queries) { // queries 배열의 각 query에 대해서 반복
    const i = arr[value[0]]; // query의 첫 번째 원소에 해당하는 값 i 저장
    const j = arr[value[1]]; // query의 두 번째 원소에 해당하는 값 j 저장
    arr[value[0]] = j; // 배열 arr에서 첫 번째 원소를 j로 업데이트
    arr[value[1]] = i; // 배열 arr에서 두 번째 원소를 i로 업데이트
  }
  return arr; // 업데이트된 배열 arr 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
