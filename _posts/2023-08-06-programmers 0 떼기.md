---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <0 떼기>"
date: 2023-08-06T21:04:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수로 이루어진 문자열 n_str이 주어질 때, n_str의 가장 왼쪽에 처음으로 등장하는 0들을 뗀 문자열을 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 2 ≤ n_str ≤ 10
- n_str이 "0"으로만 이루어진 경우는 없습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| n_str    |   결과   |
| -------- | :------: |
| "0010"   |   "10"   |
| "854020" | "854020" |

## <b>풀이</b>

```js
function solution(n_str) {
  // 문자열을 배열로 변환한 후, 가장 왼쪽에 처음으로 등장하는 0이 아닌 원소의 인덱스를 찾습니다.
  const answer = [...n_str].findIndex((v) => v !== "0");

  // 해당 인덱스부터 끝까지의 부분 문자열을 반환합니다.
  return n_str.slice(answer);
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. n_str 문자열을 배열로 변환한 후, <strong>findIndex()</strong>을 사용해 가장 왼쪽에 처음으로 등장하는 0이 아닌 원소의 인덱스를 찾습니다.

2. <strong>slice()</strong>을 사용하여 해당 인덱스부터 끝까지의 부분 문자열을 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
