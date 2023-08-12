---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <조건에 맞게 수열 변환하기 3>"
date: 2023-08-04T19:45:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr와 자연수 k가 주어집니다.

만약 k가 홀수라면 arr의 모든 원소에 k를 곱하고, k가 짝수라면 arr의 모든 원소에 k를 더합니다.

이러한 변환을 마친 후의 arr를 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 1,000,000
  - 1 ≤ arr의 원소의 값 ≤ 100
- 1 ≤ k ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| arr                    |  k  |           결과           |
| ---------------------- | :-: | :----------------------: |
| [1, 2, 3, 100, 99, 98] |  3  | [3, 6, 9, 300, 297, 294] |
| [1, 2, 3, 100, 99, 98] |  2  | [3, 4, 5, 102, 101, 100] |

## <b>풀이</b>

```js
// 주어진 배열 arr과 정수 k에 대한 변환을 수행하는 함수
function solution(arr, k) {
  // k가 짝수인지 홀수인지 확인하여 조건에 따라 처리
  // 삼항 연산자를 사용하여 조건에 맞는 처리를 선택
  return k % 2 === 0 ? arr.map((v) => v + k) : arr.map((v) => v * k);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
