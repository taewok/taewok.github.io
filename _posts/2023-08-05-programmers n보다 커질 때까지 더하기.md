---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (n보다 커질 때까지 더하기)"
date: 2023-08-05T17:15:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 numbers와 정수 n이 매개변수로 주어집니다. numbers의 원소를 앞에서부터 하나씩 더하다가 그 합이 n보다 커지는 순간 이때까지 더했던 원소들의 합을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ numbers의 길이 ≤ 100
- 1 ≤ numbers의 원소 ≤ 100
- 0 ≤ n < numbers의 모든 원소의 합

<h3><blockquote>입출력 예
</blockquote></h3>

| numbers                  |  n  | 결과 |
| ------------------------ | :-: | :--: |
| [34, 5, 71, 29, 100, 34] | 123 | 139  |
| [58, 44, 27, 10, 100]    | 139 | 239  |

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
