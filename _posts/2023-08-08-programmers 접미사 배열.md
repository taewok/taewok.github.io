---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (접미사 배열)"
date: 2023-08-08T21:10:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

어떤 문자열에 대해서 접미사는 특정 인덱스부터 시작하는 문자열을 의미합니다. 예를 들어, "banana"의 모든 접미사는 "banana", "anana", "nana", "ana", "na", "a"입니다.
문자열 my_string이 매개변수로 주어질 때, my_string의 모든 접미사를 사전순으로 정렬한 문자열 배열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- my_string은 알파벳 소문자로만 이루어져 있습니다.
- 1 ≤ my_string의 길이 ≤ 100

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string     |                                                      결과                                                      |
| ------------- | :------------------------------------------------------------------------------------------------------------: |
| "banana"      |                                 ["a", "ana", "anana", "banana", "na", "nana"]                                  |
| "programmers" | ["ammers", "ers", "grammers", "mers", "mmers", "ogrammers", "programmers", "rammers", "rogrammers", "rs", "s"] |

## <b>풀이</b>

```js
function solution(my_string) {
  var answer = []; // 결과를 담을 배열

  // 문자열의 각 인덱스마다 접미사를 구하여 answer 배열에 추가
  for (let index in my_string) {
    answer.push(my_string.slice(index));
  }

  // 사전순으로 정렬하여 반환
  return answer.sort();
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
