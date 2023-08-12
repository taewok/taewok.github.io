---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (수열과 구간 쿼리 1)"
date: 2023-08-09T19:30:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr와 2차원 정수 배열 queries이 주어집니다. queries의 원소는 각각 하나의 query를 나타내며, [s, e] 꼴입니다.

각 query마다 순서대로 s ≤ i ≤ e인 모든 i에 대해 arr[i]에 1을 더합니다.

위 규칙에 따라 queries를 처리한 이후의 arr를 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 1,000
  - 0 ≤ arr의 원소 ≤ 1,000,000
- 1 ≤ queries의 길이 ≤ 1,000
  - 0 ≤ s ≤ e < arr의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| arr             |        queries         |      결과       |
| --------------- | :--------------------: | :-------------: |
| [0, 1, 2, 3, 4] | [[0, 1],[1, 2],[2, 3]] | [1, 3, 4, 4, 4] |

## <b>풀이</b>

```js
function solution(arr, queries) {
  // 각 query에 대해 순서대로 처리합니다.
  for (let query of queries) {
    // query[0]부터 query[1]까지의 범위를 순회하면서 해당 원소에 1을 더합니다.
    for (let i = query[0]; i <= query[1]; i++) {
      arr[i] += 1;
    }
  }
  return arr; // 처리된 배열을 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
