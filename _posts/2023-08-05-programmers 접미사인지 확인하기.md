---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (접미사인지 확인하기)"
date: 2023-08-05T18:15:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

어떤 문자열에 대해서 접미사는 특정 인덱스부터 시작하는 문자열을 의미합니다. 예를 들어, "banana"의 모든 접미사는 "banana", "anana", "nana", "ana", "na", "a"입니다.
문자열 my_string과 is_suffix가 주어질 때, is_suffix가 my_string의 접미사라면 1을, 아니면 0을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ my_string의 길이 ≤ 100
- 1 ≤ is_suffix의 길이 ≤ 100
- my_string과 is_suffix는 영소문자로만 이루어져 있습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string | is_suffix | 결과 |
| --------- | :-------: | :--: |
| "banana"  |   "ana"   |  1   |
| "banana"  |   "nan"   |  0   |
| "banana"  |  "wxyz"   |  0   |

## <b>풀이</b>

```js
function solution(my_string, is_suffix) {
  // is_suffix가 my_string의 끝에 위치하는지 검사하여 결과 반환
  return my_string.endsWith(is_suffix) ? 1 : 0;
}
```

- endsWith() 메서드는 문자열이 특정 문자열로 끝나는지 여부를 확인하는 JavaScript 문자열 메서드이며 지정한 값으로 끝나면 true를 반환하고 그렇지 않으면 false를 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
