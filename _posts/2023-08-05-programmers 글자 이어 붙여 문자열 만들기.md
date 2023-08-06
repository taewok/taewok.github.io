---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <글자 이어 붙여 문자열 만들기>"
date: 2023-08-05T20:28:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 my_string과 정수 배열 index_list가 매개변수로 주어집니다. my_string의 index_list의 원소들에 해당하는 인덱스의 글자들을 순서대로 이어 붙인 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ my_string의 길이 ≤ 1,000
- my_string의 원소는 영소문자로 이루어져 있습니다.
- 1 ≤ index_list의 길이 ≤ 1,000
- 0 ≤ index_list의 원소 < my_string의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string            |                index_list                | 결과          |
| -------------------- | :--------------------------------------: | ------------- |
| "cvsgiorszzzmrpaqpe" | [16, 6, 5, 3, 12, 14, 11, 11, 17, 12, 7] | "programmers" |
| "zpiaz"              |             [1, 2, 0, 0, 3]              | "pizza"       |

## <b>풀이</b>

```js
function solution(my_string, index_list) {
  // index_list의 각 원소에 대해 해당 인덱스의 글자를 추출하고 이어붙인 문자열 반환
  return index_list.map((v) => my_string[v]).join("");
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

- index_list.map((v) => my_string[v])는 index_list 배열의 각 원소에 대해 해당 인덱스의 글자를 추출하여 새로운 배열을 생성합니다.
- <strong>join("")</strong>로 배열의 모든 원소를 빈 문자열을 구분자로 이어붙여 하나의 문자열로 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
