---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <원하는 문자열 찾기>"
date: 2023-08-06T16:15:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

알파벳으로 이루어진 문자열 myString과 pat이 주어집니다. myString의 연속된 부분 문자열 중 pat이 존재하면 1을 그렇지 않으면 0을 return 하는 solution 함수를 완성해 주세요.

단, 알파벳 대문자와 소문자는 구분하지 않습니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString의 길이 ≤ 100,000
- 1 ≤ pat의 길이 ≤ 300
- myString과 pat은 모두 알파벳으로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString  |   pat   | 결과 |
| --------- | :-----: | :--: |
| "AbCdEfG" |  "aBc"  |  1   |
| "aaAA"    | "aaaaa" |  0   |

## <b>풀이</b>

```js
function solution(myString, pat) {
  // 주어진 문자열과 부분 문자열을 모두 대문자로 변환하여 비교
  // 대소문자를 구분하지 않고 부분 문자열이 존재하는지 확인
  // 존재하면 1을, 그렇지 않으면 0을 반환
  return myString.toUpperCase().includes(pat.toUpperCase()) ? 1 : 0;
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. myString과 pat을 모두 대문자로 변환합니다. 이렇게 하면 대소문자를 구분하지 않고 문자열을 비교할 수 있습니다.

2. <strong>includes()</strong> 함수를 사용하여 대문자로 변환한 myString 내에 대문자로 변환한 pat이 포함되어 있는지를 확인합니다.

3. 만약 pat이 myString에 포함되어 있다면 1을 반환하고, 그렇지 않다면 0을 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
