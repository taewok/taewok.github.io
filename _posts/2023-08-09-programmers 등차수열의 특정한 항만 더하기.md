---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (등차수열의 특정한 항만 더하기)"
date: 2023-08-09T19:00:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

두 정수 a, d와 길이가 n인 boolean 배열 included가 주어집니다. 첫째항이 a, 공차가 d인 등차수열에서 included[i]가 i + 1항을 의미할 때, 이 등차수열의 1항부터 n항까지 included가 true인 항들만 더한 값을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ a ≤ 100
- 1 ≤ d ≤ 100
- 1 ≤ included의 길이 ≤ 100
- included에는 true가 적어도 하나 존재합니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| a   |  d  | included                                         | 결과 |
| --- | :-: | ------------------------------------------------ | :--: |
| 3   |  4  | [true, false, false, true, true]                 |  37  |
| 7   |  1  | [false, false, false, true, false, false, false] |  10  |

## <b>풀이</b>

```js
function solution(a, d, included) {
  var answer = 0; // 결과 값을 저장할 변수

  let num = a; // 등차수열 항의 값
  for (let i = 0; i < included.length; i++) {
    if (included[i]) {
      answer += num; // included가 true이면 해당 항의 값을 결과에 더함
    }
    num += d; // 다음 항의 값을 계산하기 위해 d를 더해줌
  }
  return answer; // 결과 값 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
