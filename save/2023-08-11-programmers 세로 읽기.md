---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <세로 읽기>"
date: 2023-08-11T15:24:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 my_string과 두 정수 m, c가 주어집니다. my_string을 한 줄에 m 글자씩 가로로 적었을 때 왼쪽부터 세로로 c번째 열에 적힌 글자들을 문자열로 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- my_string은 영소문자로 이루어져 있습니다.
- 1 ≤ m ≤ my_string의 길이 ≤ 1,000
- m은 my_string 길이의 약수로만 주어집니다.
- 1 ≤ c ≤ m

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string              |  m  | c   | 결과          |
| ---------------------- | :-: | --- | ------------- |
| "ihrhbakrfpndopljhygc" |  4  | 2   | "happy"       |
| "programmers"          |  1  | 1   | "programmers" |

## <b>풀이</b>

```js
function solution(my_string, m, c) {
  const array = []; // 빈 배열 array 생성
  for (let i = 0; i < my_string.length / m; i++) { // my_string을 m 글자씩 나누는 루프
    array.push([...my_string].splice(i * m, m)); // m 글자씩 잘라서 array 배열에 추가
  }
  return array.map((v) => v[c - 1]).join(""); // array의 c번째 열의 글자들을 이어붙인 문자열로 변환하여 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
