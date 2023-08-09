---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <9로 나눈 나머지>"
date: 2023-08-08T22:07:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

음이 아닌 정수를 9로 나눈 나머지는 그 정수의 각 자리 숫자의 합을 9로 나눈 나머지와 같은 것이 알려져 있습니다.
이 사실을 이용하여 음이 아닌 정수가 문자열 number로 주어질 때, 이 정수를 9로 나눈 나머지를 return 하는 solution 함수를 작성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ number의 길이 ≤ 100,000
- number의 원소는 숫자로만 이루어져 있습니다.
- number는 정수 0이 아니라면 숫자 '0'으로 시작하지 않습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| number                 | 결과 |
| ---------------------- | :--: |
| "123"                  |  6   |
| "78720646226947352489" |  2   |

## <b>풀이</b>

```js
function solution(number) {
  // 문자열 number를 split 함수를 사용하여 각 자리 숫자로 분리하고, 각 숫자를 Number로 변환하여 배열로 만듭니다.
  const numberArray = number.split("").map((v) => Number(v));

  // 배열의 요소들을 모두 더하여 총 합을 구합니다. 초기값으로 0을 사용하여 reduce 함수를 호출합니다.
  const sum = numberArray.reduce((a, b) => a + b, 0);

  // 총 합을 9로 나눈 나머지를 구하여 반환합니다.
  return sum % 9;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
