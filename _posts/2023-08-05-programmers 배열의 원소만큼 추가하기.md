---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (배열의 원소만큼 추가하기)"
date: 2023-08-05T18:25:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

아무 원소도 들어있지 않은 빈 배열 X가 있습니다. 양의 정수 배열 arr가 매개변수로 주어질 때, arr의 앞에서부터 차례대로 원소를 보면서 원소가 a라면 X의 맨 뒤에 a를 a번 추가하는 일을 반복한 뒤의 배열 X를 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 ≤ 100
- 1 ≤ arr의 원소 ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| arr       |                 결과                 |
| --------- | :----------------------------------: |
| [5, 1, 4] |    [5, 5, 5, 5, 5, 1, 4, 4, 4, 4]    |
| [6, 6]    | [6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6] |
| [1]       |                 [1]                  |

## <b>풀이</b>

```js
function solution(arr) {
  var answer = [];

  // arr 배열의 각 원소에 대해 반복
  for (let i = 0; i < arr.length; i++) {
    // 현재 원소의 값 만큼 반복하여 answer 배열에 추가
    for (let j = 0; j < arr[i]; j++) {
      answer.push(arr[i]);
    }
  }

  // 결과 배열 반환
  return answer;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
