---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <더 크게 합치기>"
date: 2023-08-06T17:05:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

연산 ⊕는 두 정수에 대한 연산으로 두 정수를 붙여서 쓴 값을 반환합니다. 예를 들면 다음과 같습니다.

12 ⊕ 3 = 123
3 ⊕ 12 = 312
양의 정수 a와 b가 주어졌을 때, a ⊕ b와 b ⊕ a 중 더 큰 값을 return 하는 solution 함수를 완성해 주세요.

단, a ⊕ b와 b ⊕ a가 같다면 a ⊕ b를 return 합니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ a, b < 10,000

<h3><blockquote>입출력 예
</blockquote></h3>

| a   |  b  | 결과 |
| --- | :-: | :--: |
| 9   | 91  | 991  |
| 89  |  8  | 898  |

## <b>풀이</b>

```js
function solution(a, b) {
  // a와 b를 문자열로 변환하여 ⊕ 연산을 수행하기 위해 합친 문자열을 만듦
  const num1 = parseInt(String(a) + String(b));

  // a와 b를 바꿔서 문자열로 변환하여 ⊕ 연산을 수행하기 위해 합친 문자열을 만듦
  const num2 = parseInt(String(b) + String(a));

  // 두 개의 ⊕ 연산 결과 중 더 큰 값을 반환
  return Math.max(num1, num2);
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. num1 변수: 주어진 정수 a와 b를 문자열로 변환하여 ⊕ 연산을 수행하기 위해 두 문자열을 합칩니다. 그리고 parseInt 함수를 사용하여 문자열을 숫자로 변환하여 저장합니다.

2. num2 변수: a와 b의 순서를 바꾼 뒤 문자열로 변환하여 ⊕ 연산을 수행하기 위해 두 문자열을 합칩니다. 그리고 parseInt 함수를 사용하여 문자열을 숫자로 변환하여 저장합니다.

3. <strong>Math.max()</strong>을 사용하여 num1과 num2 중에서 더 큰 값을 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
