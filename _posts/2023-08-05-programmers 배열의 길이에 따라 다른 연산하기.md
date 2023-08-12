---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (배열의 길이에 따라 다른 연산하기)"
date: 2023-08-05T20:38:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr과 정수 n이 매개변수로 주어집니다. arr의 길이가 홀수라면 arr의 모든 짝수 인덱스 위치에 n을 더한 배열을, arr의 길이가 짝수라면 arr의 모든 홀수 인덱스 위치에 n을 더한 배열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 1,000
- 1 ≤ arr의 원소 ≤ 1,000
- 1 ≤ n ≤ 1,000

<h3><blockquote>입출력 예
</blockquote></h3>

| arr                    |  n  | 결과                   |
| ---------------------- | :-: | ---------------------- |
| [49, 12, 100, 276, 33] | 27  | [76, 12, 127, 276, 60] |
| [444, 555, 666, 777]   | 100 | [444, 655, 666, 877]   |

## <b>풀이</b>

```js
function solution(arr, n) {
  // 만약 배열의 길이가 짝수라면
  if (arr.length % 2 === 0) {
    // 배열의 모든 요소에 대해 map을 사용하여 새로운 배열을 생성
    return arr.map((v, i) => (i % 2 === 0 ? v : v + n));
  } else {
    // 배열의 모든 요소에 대해 map을 사용하여 새로운 배열을 생성
    return arr.map((v, i) => (i % 2 === 0 ? v + n : v));
  }
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
