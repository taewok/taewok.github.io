---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (배열의 원소 삭제하기)"
date: 2023-08-06T21:04:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

정수 배열 arr과 delete_list가 있습니다. arr의 원소 중 delete_list의 원소를 모두 삭제하고 남은 원소들은 기존의 arr에 있던 순서를 유지한 배열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 100
- 1 ≤ arr의 원소 ≤ 1,000
- arr의 원소는 모두 서로 다릅니다.
- 1 ≤ delete_list의 길이 ≤ 100
- 1 ≤ delete_list의 원소 ≤ 1,000
- delete_list의 원소는 모두 서로 다릅니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| arr                       |         delete_list         |          결과          |
| ------------------------- | :-------------------------: | :--------------------: |
| [293, 1000, 395, 678, 94] | [94, 777, 104, 1000, 1, 12] |    [293, 395, 678]     |
| [110, 66, 439, 785, 1]    |     [377, 823, 119, 43]     | [110, 66, 439, 785, 1] |

## <b>풀이</b>

```js
function solution(arr, delete_list) {
  // delete_list에 포함되지 않은 원소들로 구성된 배열을 반환
  return arr.filter((v) => !delete_list.includes(v));
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. arr 배열을 순회하면서, <strong>filter()</strong>와 <strong>includes()</strong>를 사용하여 delete_list 배열에 포함된 원소가 아닌 경우만 필터링하여 새로운 배열을 생성합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
