---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (수 조작하기 2)"
date: 2023-08-09T17:13:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 numLog가 주어집니다. 처음에 numLog[0]에서 부터 시작해 "w", "a", "s", "d"로 이루어진 문자열을 입력으로 받아 순서대로 다음과 같은 조작을 했다고 합시다.

- "w" : 수에 1을 더한다.
- "s" : 수에 1을 뺀다.
- "d" : 수에 10을 더한다.
- "a" : 수에 10을 뺀다.

그리고 매번 조작을 할 때마다 결괏값을 기록한 정수 배열이 numLog입니다. 즉, numLog[i]는 numLog[0]로부터 총 i번의 조작을 가한 결과가 저장되어 있습니다.

주어진 정수 배열 numLog에 대해 조작을 위해 입력받은 문자열을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 2 ≤ log의 길이 ≤ 100,000
  - -100,000 ≤ log[0] ≤ 100,000
  - 1 ≤ i ≤ log의 길이인 모든 i에 대해 |log[i] - log[i - 1]|의 값은 1 또는 10입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| log                                       |     결과      |
| ----------------------------------------- | :-----------: |
| [0, 1, 0, 10, 0, 1, 0, 10, 0, -1, -2, -1] | "wsdawsdassw" |

## <b>풀이</b>

```js
function solution(numLog) {
  let answer = ""; // 결과 문자열을 저장할 변수
  let prevNum; // 이전 조작 결과를 저장할 변수

  // 주어진 numLog 배열을 순회하면서 조작 결과를 계산합니다.
  for (let i = 1; i <= numLog.length; i++) {
    prevNum = numLog[i - 1]; // 이전 조작 결과를 저장

    // 현재 조작 결과와 이전 조작 결과의 차이를 계산하여 해당하는 문자를 결과 문자열에 추가합니다.
    const num = numLog[i] - prevNum;
    if (num == 1) answer += "w";
    else if (num == -1) answer += "s";
    else if (num == 10) answer += "d";
    else if (num == -10) answer += "a";
  }

  return answer; // 결과 문자열을 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
