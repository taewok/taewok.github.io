---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <n보다 커질 때까지 더하기>"
date: 2023-08-05T20:15:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 my_string과 정수 k가 주어질 때, my_string을 k번 반복한 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ my_string의 길이 ≤ 100
- my_string은 영소문자로만 이루어져 있습니다.
- 1 ≤ k ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string |  k  |                    결과                    |
| --------- | :-: | :----------------------------------------: |
| "string"  |  3  |            "stringstringstring"            |
| "love"    | 10  | "lovelovelovelovelovelovelovelovelovelove" |

## <b>풀이</b>

```js
function solution(my_string, k) {
  // 문자열 my_string을 k번 반복하여 결과 반환
  return my_string.repeat(k);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
