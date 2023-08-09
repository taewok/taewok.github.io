---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <배열 만들기 3>"
date: 2023-08-08T21:40:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr와 2개의 구간이 담긴 배열 intervals가 주어집니다.

intervals는 항상 [[a1, b1], [a2, b2]]의 꼴로 주어지며 각 구간은 닫힌 구간입니다. 닫힌 구간은 양 끝값과 그 사이의 값을 모두 포함하는 구간을 의미합니다.

이때 배열 arr의 첫 번째 구간에 해당하는 배열과 두 번째 구간에 해당하는 배열을 앞뒤로 붙여 새로운 배열을 만들어 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 100,000
  - 1 ≤ arr의 원소 < 100
- 1 ≤ a1 ≤ b1 < arr의 길이
- 1 ≤ a2 ≤ b2 < arr의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| arr             |    intervals     |           결과           |
| --------------- | :--------------: | :----------------------: |
| [1, 2, 3, 4, 5] | [[1, 3], [0, 4]] | [2, 3, 4, 1, 2, 3, 4, 5] |

## <b>풀이</b>

```js
function solution(arr, intervals) {
  const [firstStart, firstEnd] = intervals[0]; // 첫 번째 구간의 시작과 끝
  const [secondStart, secondEnd] = intervals[1]; // 두 번째 구간의 시작과 끝

  // 첫 번째 구간에 해당하는 배열을 구합니다.
  const firstIntervalArr = arr.slice(firstStart, firstEnd + 1);

  // 두 번째 구간에 해당하는 배열을 구합니다.
  const secondIntervalArr = arr.slice(secondStart, secondEnd + 1);

  // 두 구간에 해당하는 배열을 앞뒤로 붙인 새로운 배열을 만들어 반환합니다.
  return [...firstIntervalArr, ...secondIntervalArr];
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
