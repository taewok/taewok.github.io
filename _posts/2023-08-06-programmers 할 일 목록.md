---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <할 일 목록>"
date: 2023-08-06T23:30:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

오늘 해야 할 일이 담긴 문자열 배열 todo_list와 각각의 일을 지금 마쳤는지를 나타내는 boolean 배열 finished가 매개변수로 주어질 때, todo_list에서 아직 마치지 못한 일들을 순서대로 담은 문자열 배열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ todo_list의 길이 1 ≤ 100
- 2 ≤ todo_list의 원소의 길이 ≤ 20
  - todo_list의 원소는 영소문자로만 이루어져 있습니다.
  - todo_list의 원소는 모두 서로 다릅니다.
- finished[i]는 true 또는 false이고 true는 todo_list[i]를 마쳤음을, false는 아직 마치지 못했음을 나타냅니다.
- 아직 마치지 못한 일이 적어도 하나 있습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| todo_list                                                  |          finished          | 결과                             |
| ---------------------------------------------------------- | :------------------------: | -------------------------------- |
| ["problemsolving", "practiceguitar", "swim", "studygraph"] | [true, false, true, false] | ["practiceguitar", "studygraph"] |

## <b>풀이</b>

```js
function solution(todo_list, finished) {
  // 아직 마치지 못한 일들을 필터링하여 새로운 배열을 반환
  return todo_list.filter((value, index) => !finished[index]);
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. <strong>filter()</strong>로 todo_list 배열을 순회하면서, 해당 작업의 인덱스에 해당하는 finished 배열의 값이 false인 경우에만 필터링합니다. 즉, 아직 마치지 못한 일들만 필터링합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
