---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (순서 바꾸기)"
date: 2023-08-06T21:04:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 리스트 num_list와 정수 n이 주어질 때, num_list를 n 번째 원소 이후의 원소들과 n 번째까지의 원소들로 나눠 n 번째 원소 이후의 원소들을 n 번째까지의 원소들 앞에 붙인 리스트를 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 2 ≤ num_list의 길이 ≤ 30
- 1 ≤ num_list의 원소 ≤ 9
- 1 ≤ n ≤ num_list의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| num_list        |  n  | 결과            |
| --------------- | :-: | --------------- |
| [2, 1, 6]       |  1  | [1, 6, 2]       |
| [5, 2, 1, 7, 5] |  2  | [7, 5, 5, 2, 1] |

## <b>풀이</b>

```js
function solution(num_list, n) {
  // n 번째 원소 이후의 원소들과 n 번째까지의 원소들로 나눈 두 부분을 합쳐서 반환
  return num_list.slice(n).concat(num_list.slice(0, n));
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. <strong>slice()</strong>를 사용하여 num_list의 n번째 원소부터 끝까지의 부분을 추출합니다.

2. 두 부분을 <strong>concat()</strong> 함수를 사용하여 합쳐서 새로운 배열을 생성한 후 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
