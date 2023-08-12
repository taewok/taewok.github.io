---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <수열과 구간 쿼리 1>"
date: 2023-08-10T17:00:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 date1과 date2가 주어집니다. 두 배열은 각각 날짜를 나타내며 [year, month, day] 꼴로 주어집니다. 각 배열에서 year는 연도를, month는 월을, day는 날짜를 나타냅니다.

만약 date1이 date2보다 앞서는 날짜라면 1을, 아니면 0을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- date1의 길이 = date2의 길이 = 3
  - 0 ≤ year ≤ 10,000
  - 1 ≤ month ≤ 12
  - day는 month에 따라 가능한 날짜로 주어집니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| date1          |     date2      | 결과 |
| -------------- | :------------: | :--: |
| [2021, 12, 28] | [2021, 12, 29] |  1   |
| [1024, 10, 24] | [1024, 10, 24] |  0   |

## <b>풀이</b>

```js
function solution(date1, date2) {
  // JavaScript의 내장 Date 객체를 활용하여 날짜를 비교합니다.
  // new Date(date1)과 new Date(date2)로 날짜 객체를 생성하고 비교합니다.
  // 생성된 날짜 객체끼리의 비교는 자연스럽게 날짜 순서를 확인할 수 있습니다.
  // date1이 date2보다 앞선 경우 1을 반환하고, 그렇지 않으면 0을 반환합니다.
  return new Date(date1) < new Date(date2) ? 1 : 0;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
