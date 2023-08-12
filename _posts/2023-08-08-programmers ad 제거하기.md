---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (ad 제거하기)"
date: 2023-08-08T21:54:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 배열 strArr가 주어집니다. 배열 내의 문자열 중 "ad"라는 부분 문자열을 포함하고 있는 모든 문자열을 제거하고 남은 문자열을 순서를 유지하여 배열로 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ strArr의 길이 ≤ 1,000
  - 1 ≤ strArr의 원소의 길이 ≤ 20
  - strArr의 원소는 알파벳 소문자로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| strArr                        |             결과              |
| ----------------------------- | :---------------------------: |
| ["and","notad","abcd"]        |        ["and","abcd"]         |
| ["there","are","no","a","ds"] | ["there","are","no","a","ds"] |

## <b>풀이</b>

```js
function solution(strArr) {
  // 문자열 배열 strArr에서 "ad"를 포함하는 문자열은 필터링하여 제거하고, 남은 문자열들을 반환합니다.
  return strArr.filter((v) => !v.includes("ad"));
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
