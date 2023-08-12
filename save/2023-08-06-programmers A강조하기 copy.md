---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <뒤에서 5등까지>"
date: 2023-08-06T17:15:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수로 이루어진 리스트 num_list가 주어집니다. num_list에서 가장 작은 5개의 수를 오름차순으로 담은 리스트를 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 6 ≤ num_list의 길이 ≤ 30
- 1 ≤ num_list의 원소 ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| num_list                   |        결과        |
| -------------------------- | :----------------: |
| [12, 4, 15, 46, 38, 1, 14] | [1, 4, 12, 14, 15] |

## <b>풀이</b>

```js
function solution(num_list) {
  // 주어진 리스트를 오름차순으로 정렬한 후, 앞에서부터 5개의 원소를 선택하여 새로운 배열을 생성
  return num_list.sort((a, b) => a - b).slice(0, 5);
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. num_list를 sort 함수를 사용하여 오름차순으로 정렬합니다. 

2. 정렬된 배열에서 slice(0, 5)를 사용하여 앞에서부터 5개의 원소를 선택하여 새로운 배열을 생성합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
