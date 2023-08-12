---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <주사위 게임 1>"
date: 2023-08-06T16:34:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

1부터 6까지 숫자가 적힌 주사위가 두 개 있습니다. 두 주사위를 굴렸을 때 나온 숫자를 각각 a, b라고 했을 때 얻는 점수는 다음과 같습니다.

- a와 b가 모두 홀수라면 a2 + b2 점을 얻습니다.
- a와 b 중 하나만 홀수라면 2 × (a + b) 점을 얻습니다.
- a와 b 모두 홀수가 아니라면 |a - b| 점을 얻습니다.

두 정수 a와 b가 매개변수로 주어질 때, 얻는 점수를 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- a와 b는 1 이상 6 이하의 정수입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| a   |  b  | 결과 |
| --- | :-: | :--: |
| 3   |  5  |  34  |
| 6   |  1  |  14  |

## <b>풀이</b>

```js
function solution(a, b) {
  // a와 b가 모두 홀수인 경우
  if (a % 2 !== 0 && b % 2 !== 0) {
    return a * a + b * b;
  }
  // a와 b 중 하나만 홀수인 경우
  else if ((a % 2 === 0 && b % 2 !== 0) || (a % 2 !== 0 && b % 2 === 0)) {
    return 2 * (a + b);
  }
  // a와 b 모두 홀수가 아닌 경우
  else {
    return Math.abs(a - b);
  }
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. 먼저, a와 b가 모두 홀수인 경우, a^2 + b^2 점을 반환합니다.

2. 그 다음, a와 b 중 하나만 홀수인 경우, 2 * (a + b) 점을 반환합니다.

3. a와 b 모두 홀수가 아닌 경우, a - b 점을 반환합니다. 이때 Math.abs(a - b)를 사용하여 차이의 절대값을 계산합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
