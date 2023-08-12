---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (부분 문자열 이어 붙여 문자열 만들기)"
date: 2023-08-06T19:14:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

길이가 같은 문자열 배열 my_strings와 이차원 정수 배열 parts가 매개변수로 주어집니다. parts[i]는 [s, e] 형태로, my_string[i]의 인덱스 s부터 인덱스 e까지의 부분 문자열을 의미합니다. 각 my_strings의 원소의 parts에 해당하는 부분 문자열을 순서대로 이어 붙인 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ my_strings의 길이 = parts의 길이 ≤ 100
- 1 ≤ my_strings의 원소의 길이 ≤ 100
- parts[i]를 [s, e]라 할 때, 다음을 만족합니다.
  - 0 ≤ s ≤ e < my_strings[i]의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| my_strings                                            |              parts               |     결과      |
| ----------------------------------------------------- | :------------------------------: | :-----------: |
| ["progressive", "hamburger", "hammer", "ahocorasick"] | [[0, 4], [1, 2], [3, 5], [7, 7]] | "programmers" |

## <b>풀이</b>

```js
function solution(my_strings, parts) {
  // 각 문자열에 대해 해당하는 부분 문자열을 추출하여 배열로 생성한 후, join을 사용하여 하나의 문자열로 결합
  return my_strings
    .map((v, i) => v.slice(parts[i][0], parts[i][1] + 1))
    .join("");
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. my_strings 배열을 순회하면서, 각 문자열에 대해 parts 배열에 저장된 [s, e] 형태의 범위를 활용하여 부분 문자열을 추출합니다.

2. <strong>map()</strong> 함수를 사용하여 추출한 부분 문자열을 배열로 생성합니다.

3. <strong>join()</strong> 함수를 사용하여 생성된 부분 문자열 배열을 하나의 문자열로 결합합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
