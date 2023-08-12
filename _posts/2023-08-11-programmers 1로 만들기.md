---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <1로 만들기>"
date: 2023-08-11T18:30:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수가 있을 때, 짝수라면 반으로 나누고, 홀수라면 1을 뺀 뒤 반으로 나누면, 마지막엔 1이 됩니다. 예를 들어 10이 있다면 다음과 같은 과정으로 1이 됩니다.

- 10 / 2 = 5
- (5 - 1) / 2 = 4
- 4 / 2 = 2
- 2 / 2 = 1
  위와 같이 4번의 나누기 연산으로 1이 되었습니다.

정수들이 담긴 리스트 num_list가 주어질 때, num_list의 모든 원소를 1로 만들기 위해서 필요한 나누기 연산의 횟수를 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 3 ≤ num_list의 길이 ≤ 15
- 1 ≤ num_list의 원소 ≤ 30

<h3><blockquote>입출력 예
</blockquote></h3>

| num_list           | 결과 |
| ------------------ | :--: |
| [12, 4, 15, 1, 14] |  11  |

## <b>풀이</b>

```js
function solution(num_list) {
  var answer = 0;

  // num_list의 각 정수에 대해 반복
  for (let num of num_list) {
    // 정수를 1로 만들기 위한 나누기 연산 횟수 계산
    while (num != 1) {
      answer++; // 나누기 연산 횟수 증가

      // 짝수인 경우 반으로 나누고, 홀수인 경우 1을 뺀 뒤 반으로 나눔
      if (num % 2 == 0) {
        num = num / 2;
      } else {
        num = (num - 1) / 2;
      }
    }
  }

  // 나누기 연산 횟수 반환
  return answer;
}
```

<h3><blockquote>문제풀이</blockquote></h3>

1. for (let num of num_list) 반복문은 num_list 배열의 각 정수에 대해 반복합니다.
2. while (num != 1) 반복문은 현재 정수가 1이 아닐 때까지 반복하여 나누기 연산 횟수를 계산합니다.
3. answer++는 나누기 연산 횟수를 증가시킵니다.
4. if (num % 2 == 0) 조건문은 현재 정수가 짝수인지 여부를 검사합니다. 짝수라면 반으로 나누고, 홀수라면 1을 빼고 반으로 나누어 나누기 연산을 수행합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
