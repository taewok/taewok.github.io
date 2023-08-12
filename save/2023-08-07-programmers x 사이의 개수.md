---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <x 사이의 개수>"
date: 2023-08-07T22:00:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 myString이 주어집니다. myString을 문자 "x"를 기준으로 나눴을 때 나눠진 문자열 각각의 길이를 순서대로 저장한 배열을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString의 길이 ≤ 100,000
  - myString은 알파벳 소문자로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString       |        결과        |
| -------------- | :----------------: |
| "oxooxoxxox"   | [1, 2, 1, 0, 1, 0] |
| "xabcxdefxghi" |    [0, 3, 3, 3]    |

## <b>풀이</b>

```js
function solution(myString) {
  // 문자열을 "x"를 기준으로 나눈 후, 각 나눠진 문자열의 길이를 배열로 변환하여 반환
  return myString.split("x").map((subString) => subString.length);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
