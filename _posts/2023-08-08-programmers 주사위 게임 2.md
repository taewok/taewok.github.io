---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <주사위 게임 2>"
date: 2023-08-08T18:40:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

1부터 6까지 숫자가 적힌 주사위가 세 개 있습니다. 세 주사위를 굴렸을 때 나온 숫자를 각각 a, b, c라고 했을 때 얻는 점수는 다음과 같습니다.

- 세 숫자가 모두 다르다면 a + b + c 점을 얻습니다.
- 세 숫자 중 어느 두 숫자는 같고 나머지 다른 숫자는 다르다면 (a + b + c) × (a2 + b2 + c2 )점을 얻습니다.
- 세 숫자가 모두 같다면 (a + b + c) × (a2 + b2 + c2 ) × (a3 + b3 + c3 )점을 얻습니다.

세 정수 a, b, c가 매개변수로 주어질 때, 얻는 점수를 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- a, b, c는 1이상 6이하의 정수입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| a   |  b  |  c  |  결과  |
| --- | :-: | :-: | :----: |
| 2   |  6  |  1  |   9    |
| 5   |  3  |  3  |  473   |
| 4   |  4  |  4  | 110592 |

## <b>풀이</b>

```js
function solution(a, b, c) {
  // 세 숫자가 모두 같은 경우
  if (a == b && b == c) {
    // 점수 계산: (a + b + c) × (a^2 + b^2 + c^2) × (a^3 + b^3 + c^3)
    return (
      (a + b + c) *
      (a * a + b * b + c * c) *
      (a * a * a + b * b * b + c * c * c)
    );
  }
  // 두 숫자가 같고 나머지 하나는 다른 경우
  else if (a == b || b == c || c == a) {
    // 점수 계산: (a + b + c) × (a^2 + b^2 + c^2)
    return (a + b + c) * (a * a + b * b + c * c);
  }
  // 세 숫자가 모두 다른 경우
  else {
    // 점수 계산: a + b + c
    return a + b + c;
  }
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
