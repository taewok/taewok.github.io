---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (l로 만들기)"
date: 2023-08-06T17:36:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

알파벳 소문자로 이루어진 문자열 myString이 주어집니다. 알파벳 순서에서 "l"보다 앞서는 모든 문자를 "l"로 바꾼 문자열을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString ≤ 100,000
  - myString은 알파벳 소문자로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString     |     결과     |
| ------------ | :----------: |
| "abcdevwxyz" | "lllllvwxyz" |
| "jjnnllkkmm" | "llnnllllmm" |

## <b>풀이</b>

```js
function solution(myString) {
  // 문자열을 문자 배열로 변환하여 각 문자에 접근하고 변환 작업을 수행
  return [...myString].map((v) => (v < "l" ? "l" : v)).join("");
  // map 함수를 사용하여 각 문자를 확인하고 조건에 따라 변환하며,
  // 변환된 문자들을 합쳐서 새로운 문자열로 생성
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. [...myString]를 사용하여 문자열 myString을 문자 배열로 변환합니다.

- map 함수를 사용하여 각 문자에 대해 변환 작업을 수행하고. 만약 문자가 "l"보다 앞서는 문자라면 "l"로 변환하고, 그렇지 않은 경우에는 해당 문자를 그대로 유지합니다. 문자열을 비교하면 아스키코드를 기준으로 참/거짓을 반환하는 것 같다.

- join("")을 사용하여 변환된 문자들을 하나의 문자열로 합칩니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
