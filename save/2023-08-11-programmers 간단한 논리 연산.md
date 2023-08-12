---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <간단한 논리 연산>"
date: 2023-08-11T18:05:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

boolean 변수 x1, x2, x3, x4가 매개변수로 주어질 때, 다음의 식의 true/false를 return 하는 solution 함수를 작성해 주세요.

- (x1 ∨ x2) ∧ (x3 ∨ x4)

<h3><blockquote>입출력 예
</blockquote></h3>

| x1    |  x2   | x3    | x4    | 결과  |
| ----- | :---: | ----- | ----- | ----- |
| false | true  | true  | true  | true  |
| true  | false | false | false | false |

## <b>풀이</b>

```js
function solution(x1, x2, x3, x4) {
  // 주어진 논리식을 평가하여 결과 반환
  return (x1 || x2) && (x3 || x4);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
