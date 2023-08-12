---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (문자열 정수의 합)"
date: 2023-08-04T20:22:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

한 자리 정수로 이루어진 문자열 num_str이 주어질 때, 각 자리수의 합을 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 3 ≤ num_str ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| num_str     | 결과 |
| ----------- | :--: |
| "123456789" |  45  |
| "1000000"   |  1   |

## <b>풀이</b>

```js
function solution(num_str) {
  // 문자열(num_str)을 split 함수를 사용하여 각 자리수별로 분리한 배열을 생성하고,
  // map 함수를 사용하여 문자열을 숫자로 변환한 배열을 생성
  var digitArray = num_str.split("").map((v) => Number(v));

  // reduce 함수를 사용하여 배열 내 숫자들을 모두 더하여 합을 반환
  return digitArray.reduce((a, b) => a + b);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
