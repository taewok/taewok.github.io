---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <문자열 섞기>"
date: 2023-08-05T17:50:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

길이가 같은 두 문자열 str1과 str2가 주어집니다.

두 문자열의 각 문자가 앞에서부터 서로 번갈아가면서 한 번씩 등장하는 문자열을 만들어 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ str1의 길이 = str2의 길이 ≤ 10
  - str1과 str2는 알파벳 소문자로 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| str1    |  str2   |     결과     |
| ------- | :-----: | :----------: |
| "aaaaa" | "bbbbb" | "ababababab" |

## <b>풀이</b>

```js
function solution(numbers, n) {
  let answer = 0;

  // numbers 배열의 각 원소에 대해 반복
  for (let v of numbers) {
    // 현재 원소를 결과에 더함
    answer += v;

    // 결과가 주어진 값 n을 초과하는지 검사
    if (answer > n) {
      // 초과하는 순간까지 더한 원소들의 합을 반환하고 함수 종료
      return answer;
    }
  }

  // 모든 원소를 더한 결과를 반환
  return answer;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
