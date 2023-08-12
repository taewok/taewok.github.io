---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <문자열 뒤집기>"
date: 2023-08-11T17:44:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 my_string과 정수 s, e가 매개변수로 주어질 때, my_string에서 인덱스 s부터 인덱스 e까지를 뒤집은 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- my_string은 숫자와 알파벳으로만 이루어져 있습니다.
- 1 ≤ my_string의 길이 ≤ 1,000
- 0 ≤ s ≤ e < my_string의 길이

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string         | pat |  e  | 결과              |
| ----------------- | :-: | :-: | ----------------- |
| "Progra21Sremm3"  |  6  | 12  | "ProgrammerS123"  |
| "Stanley1yelnatS" |  4  | 10  | "Stanley1yelnatS" |

## <b>풀이</b>

```js
function solution(my_string, s, e) {
  // s 이전의 문자열
  const frontStr = my_string.slice(0, s);

  // s부터 e까지의 문자열을 뒤집어 생성한 문자열
  const midStr = my_string
    .split("")
    .slice(s, e + 1)
    .reverse()
    .join("");

  // e 이후의 문자열
  const backStr = my_string.slice(e + 1);

  // 결과 문자열 반환
  return frontStr + midStr + backStr;
}
```

<h3><blockquote>문제풀이</blockquote></h3>

1. my_string.slice(0, s)는 문자열 my_string의 인덱스 0부터 s 이전까지의 부분 문자열을 추출합니다.
2. my_string.split("").slice(s, e + 1).reverse().join("")는 문자열 my_string을 배열로 변환한 뒤 s부터 e까지의 요소를 추출하여 뒤집고, 다시 문자열로 변환합니다.
3. my_string.slice(e + 1)는 문자열 my_string의 e 이후의 부분 문자열을 추출합니다.
4. 최종적으로 추출한 세 부분을 결합하여 원하는 결과 문자열을 생성합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
