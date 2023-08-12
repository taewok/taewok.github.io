---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <글자 지우기>"
date: 2023-08-10T18:50:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 my_string과 정수 배열 indices가 주어질 때, my_string에서 indices의 원소에 해당하는 인덱스의 글자를 지우고 이어 붙인 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ indices의 길이 < my_string의 길이 ≤ 100
- my_string은 영소문자로만 이루어져 있습니다
- 0 ≤ indices의 원소 < my_string의 길이
- indices의 원소는 모두 서로 다릅니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string             |           indices            | 결과          |
| --------------------- | :--------------------------: | ------------- |
| "apporoograpemmemprs" | [1, 16, 6, 15, 0, 10, 11, 3] | "programmers" |

## <b>풀이</b>

```js
function solution(my_string, indices) {
  return [...my_string] // 문자열을 배열로 변환하여 각 글자를 요소로 가지는 배열 생성
    .filter((v, i) => !indices.includes(i)) // indices 배열에 없는 인덱스의 글자만 필터링
    .join(""); // 필터링된 글자들을 이어붙인 문자열로 변환하여 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
