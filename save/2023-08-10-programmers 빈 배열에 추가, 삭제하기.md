---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <빈 배열에 추가, 삭제하기>"
date: 2023-08-10T18:36:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

아무 원소도 들어있지 않은 빈 배열 X가 있습니다. 길이가 같은 정수 배열 arr과 boolean 배열 flag가 매개변수로 주어질 때, flag를 차례대로 순회하며 flag[i]가 true라면 X의 뒤에 arr[i]를 arr[i] × 2 번 추가하고, flag[i]가 false라면 X에서 마지막 arr[i]개의 원소를 제거한 뒤 X를 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ arr의 길이 = flag의 길이 ≤ 100
- arr의 모든 원소는 1 이상 9 이하의 정수입니다.
- 현재 X의 길이보다 더 많은 원소를 빼는 입력은 주어지지 않습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| arr             |               flag                | 결과                     |
| --------------- | :-------------------------------: | ------------------------ |
| [3, 2, 4, 1, 3] | [true, false, true, false, false] | [3, 3, 3, 3, 4, 4, 4, 4] |

## <b>풀이</b>

```js
function solution(arr, flag) {
  var answer = []; // 결과를 저장할 배열

  for (let i = 0; i < flag.length; i++) {
    if (flag[i]) {
      // flag[i]가 true인 경우
      for (let j = 0; j < arr[i] * 2; j++) {
        answer.push(arr[i]); // arr[i]를 arr[i]*2번 추가
      }
    } else {
      // flag[i]가 false인 경우
      for (let k = 0; k < arr[i]; k++) {
        answer.pop(); // 마지막 arr[i]개의 원소를 제거
      }
    }
  }
  return answer; // 결과 배열 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
