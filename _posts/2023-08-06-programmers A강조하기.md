---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <A 강조하기>"
date: 2023-08-06T16:25:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 myString이 주어집니다. myString에서 알파벳 "a"가 등장하면 전부 "A"로 변환하고, "A"가 아닌 모든 대문자 알파벳은 소문자 알파벳으로 변환하여 return 하는 solution 함수를 완성하세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString의 길이 ≤ 20
  - myString은 알파벳으로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString           |        결과        |
| ------------------ | :----------------: |
| "abstract algebra" | "AbstrAct AlgebrA" |
| "PrOgRaMmErS"      |   "progrAmmers"    |

## <b>풀이</b>

```js
function solution(myString) {
  // 문자열을 문자 배열로 변환하여 각 문자에 접근하고 변환 작업을 수행
  return [...myString]
    .map((v) => (v === "a" || v === "A" ? "A" : v.toLowerCase()))
    .join("");
  // map 함수를 사용하여 각 문자를 변환하고, 변환된 문자들을 합쳐서 새로운 문자열로 생성
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. [...myString]를 사용하여 문자열 myString을 문자 배열로 변환합니다.

2. map 함수를 사용하여 문자 배열의 각 문자에 대해 변환 작업을 수행합니다. 만약 문자가 "a"나 "A"라면 "A"로 변환하고, 그렇지 않은 경우에는 해당 문자를 소문자로 변환합니다.

3. join("")을 사용하여 변환된 문자들을 하나의 문자열로 합칩니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
