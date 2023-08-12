---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <두 수의 연산값 비교하기>"
date: 2023-08-07T22:30:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

연산 ⊕는 두 정수에 대한 연산으로 두 정수를 붙여서 쓴 값을 반환합니다. 예를 들면 다음과 같습니다.

- 12 ⊕ 3 = 123
- 3 ⊕ 12 = 312

양의 정수 a와 b가 주어졌을 때, a ⊕ b와 2 _ a _ b 중 더 큰 값을 return하는 solution 함수를 완성해 주세요.

단, a ⊕ b와 2 _ a _ b가 같으면 a ⊕ b를 return 합니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ a, b < 10,000

<h3><blockquote>입출력 예
</blockquote></h3>

| a   |  b  | 결과 |
| --- | :-: | :--: |
| 2   | 91  | 364  |
| 91  |  2  | 912  |

## <b>풀이</b>

```js
function solution(a, b) {
  // 두 정수를 문자열로 변환한 후 이어 붙여서 숫자로 만든 값 (a ⊕ b)
  const num1 = Number(String(a) + String(b));

  // 두 정수의 곱에 2를 곱한 값 (2 * a * b)
  const num2 = 2 * a * b;

  // a ⊕ b와 2 * a * b 중 더 큰 값을 반환. 같으면 a ⊕ b를 반환
  return num1 === num2 ? num1 : Math.max(num1, num2);
}+
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
