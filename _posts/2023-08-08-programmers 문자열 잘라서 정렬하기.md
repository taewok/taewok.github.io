---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <문자열 잘라서 정렬하기>"
date: 2023-08-08T20:30:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 myString이 주어집니다. "x"를 기준으로 해당 문자열을 잘라내 배열을 만든 후 사전순으로 정렬한 배열을 return 하는 solution 함수를 완성해 주세요.

단, 빈 문자열은 반환할 배열에 넣지 않습니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString ≤ 100,000
  - myString은 알파벳 소문자로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString        |          결과           |
| --------------- | :---------------------: |
| "axbxcxdx"      |    ["a","b","c","d"]    |
| "dxccxbbbxaaaa" | ["aaaa","bbb","cc","d"] |

## <b>풀이</b>

```js
function solution(myString) {
  // 문자열 myString을 "x"를 기준으로 잘라서 배열로 변환
  // 그리고 잘린 문자열 중 빈 문자열은 제외하고 남은 배열을 얻습니다.
  const resultArray = myString.split("x").filter((v) => v !== "");

  // 사전순으로 정렬하여 반환
  return resultArray.sort();
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
