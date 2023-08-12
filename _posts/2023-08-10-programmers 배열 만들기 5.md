---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (배열 만들기 5)"
date: 2023-08-10T17:10:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 배열 intStrs와 정수 k, s, l가 주어집니다. intStrs의 원소는 숫자로 이루어져 있습니다.

배열 intStrs의 각 원소마다 s번 인덱스에서 시작하는 길이 l짜리 부분 문자열을 잘라내 정수로 변환합니다. 이때 변환한 정수값이 k보다 큰 값들을 담은 배열을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 0 ≤ s < 100
- 1 ≤ l ≤ 8
- 10l - 1 ≤ k < 10l
- 1 ≤ intStrs의 길이 ≤ 10,000
  - s + l ≤ intStrs의 원소의 길이 ≤ 120

<h3><blockquote>입출력 예
</blockquote></h3>

| intStrs        |   k   |  s  |  l  | 결과           |
| -------------- | :---: | :-: | :-: | -------------- |
| [2021, 12, 28] | 50000 |  5  |  5  | [56789, 99999] |

## <b>풀이</b>

```js
function solution(intStrs, k, s, l) {
  // 정수로 변환한 후, k보다 큰 경우만 필터링하여 반환합니다.
  return intStrs
    .map((intStr) => Number(intStr.slice(s, s + l)))
    .filter((num) => num > k);
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
