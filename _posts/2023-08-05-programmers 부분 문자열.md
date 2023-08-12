---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (부분 문자열)"
date: 2023-08-05T17:27:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

어떤 문자열 A가 다른 문자열 B안에 속하면 A를 B의 부분 문자열이라고 합니다. 예를 들어 문자열 "abc"는 문자열 "aabcc"의 부분 문자열입니다.

문자열 str1과 str2가 주어질 때, str1이 str2의 부분 문자열이라면 1을 부분 문자열이 아니라면 0을 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ str1 ≤ str2 ≤ 20
- str1과 str2는 영어 소문자로만 이루어져 있습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| str1  |   str2   | 결과 |
| ----- | :------: | :--: |
| "abc" | "aabcc"  |  1   |
| "tbt" | "tbbttb" |  0   |

## <b>풀이</b>

```js
function solution(str1, str2) {
  // str2 문자열에 str1이 포함되어 있는지 검사
  // 포함되어 있다면 1을, 포함되어 있지 않다면 0을 반환
  return str2.includes(str1) ? 1 : 0;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
