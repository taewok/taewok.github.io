---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 \<꼬리 문자열
\>"
date: 2023-08-02T19:06:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열들이 담긴 리스트가 주어졌을 때, 모든 문자열들을 순서대로 합친 문자열을 꼬리 문자열이라고 합니다. 꼬리 문자열을 만들 때 특정 문자열을 포함한 문자열은 제외시키려고 합니다. 예를 들어 문자열 리스트 ["abc", "def", "ghi"]가 있고 문자열 "ef"를 포함한 문자열은 제외하고 꼬리 문자열을 만들면 "abcghi"가 됩니다.

문자열 리스트 str_list와 제외하려는 문자열 ex가 주어질 때, str_list에서 ex를 포함한 문자열을 제외하고 만든 꼬리 문자열을 return하도록 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 2 ≤ str_list의 길이 ≤ 10
- 1 ≤ str_list의 원소의 길이 ≤ 10
- 1 ≤ ex의 길이 ≤ 5

<h3><blockquote>입출력 예
</blockquote></h3>

| str_list              |  ex  |   결과   |
| --------------------- | :--: | :------: |
| ["abc", "def", "ghi"] | "ef" | "abcghi" |
| ["abc", "bbc", "cbc"] | "c"  |    ""    |

## <b>풀이</b>

| str_list              |  ex  |
| --------------------- | :--: |
| ["abc", "def", "ghi"] | "ef" |

```js
function solution(str_list, ex) {
  var answer = ""; // 결과 문자열을 담을 변수를 선언

  // 주어진 문자열 리스트에서 ex를 포함하지 않는 문자열들을 걸러내고, 그들을 빈칸 없이 연결하여 answer에 저장
  answer = str_list.filter((v) => !v.includes(ex)).join("");

  return answer;
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. <strong>filter()</strong>를 사용하여 str_list 배열의 각 원소를 검사하며 <strong>includes()</strong>를 사용해서 원소에 ex가 포함되어 있는지 없는지를 boolean값으로 판별합니다.

2. 만약 원소에 ex가 포함되어 <strong>includes()</strong>가 true 반환한다면 !연산자를 사용하여 false로 바꾸어 filter에 들어가지 않도록한다.

3. `str_list.filter((v) => !v.includes(ex))`는 ["abc", "ghi"]가 되어있고 배열의 모든 요소를 문자열로 변환하고, 이들을 하나의 문자열로 연결하는 <strong>join()</strong>을 사용하면 answer값은 "abcghi"가 된다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
