---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (뒤에서 5등 위로)"
date: 2023-08-05T17:40:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수로 이루어진 리스트 num_list가 주어집니다. num_list에서 가장 작은 5개의 수를 제외한 수들을 오름차순으로 담은 리스트를 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 6 ≤ num_list의 길이 ≤ 30
- 1 ≤ num_list의 원소 ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| num_list                               |         결과         |
| -------------------------------------- | :------------------: |
| [12, 4, 15, 46, 38, 1, 14, 56, 32, 10] | [15, 32, 38, 46, 56] |

## <b>풀이</b>

```js
function solution(num_list) {
  // 배열을 오름차순으로 정렬
  num_list.sort((a, b) => a - b);

  // 가장 작은 5개의 수를 제외한 나머지 수들을 새 배열에 저장
  var result = num_list.slice(5);

  // 결과 배열 반환
  return result;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
