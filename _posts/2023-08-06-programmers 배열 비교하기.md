---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <배열 비교하기>"
date: 2023-08-06T18:40:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

이 문제에서 두 정수 배열의 대소관계를 다음과 같이 정의합니다.

- 두 배열의 길이가 다르다면, 배열의 길이가 긴 쪽이 더 큽니다.
- 배열의 길이가 같다면 각 배열에 있는 모든 원소의 합을 비교하여 다르다면 더 큰 쪽이 크고, 같다면 같습니다.

두 정수 배열 arr1과 arr2가 주어질 때, 위에서 정의한 배열의 대소관계에 대하여 arr2가 크다면 -1, arr1이 크다면 1, 두 배열이 같다면 0을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr1의 길이 ≤ 100
- 1 ≤ arr2의 길이 ≤ 100
- 1 ≤ arr1의 원소 ≤ 100
- 1 ≤ arr2의 원소 ≤ 100
- 문제에서 정의한 배열의 대소관계가 일반적인 프로그래밍 언어에서 정의된 배열의 대소관계와 다를 수 있는 점에 유의해주세요.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString         |     myString     | 결과 |
| ---------------- | :--------------: | :--: |
| [49, 13]         |   [70, 11, 2]    |  -1  |
| [100, 17, 84, 1] | [55, 12, 65, 36] |  1   |
| [1, 2, 3, 4, 5]  | [3, 3, 3, 3, 3]  |  0   |

## <b>풀이</b>

```js
function solution(arr1, arr2) {
  if (arr1.length !== arr2.length) {
    // 배열의 길이가 다른 경우, 길이가 긴 배열이 크다고 판단
    return arr1.length > arr2.length ? 1 : -1;
  } else {
    const sum1 = arr1.reduce((a, b) => a + b);
    const sum2 = arr2.reduce((a, b) => a + b);

    // 각 배열의 원소 합을 비교하여 크기를 판단
    if (sum1 > sum2) return 1;
    else if (sum1 < sum2) return -1;
    else return 0; // 두 배열의 합이 같다면 같다고 판단
  }
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
