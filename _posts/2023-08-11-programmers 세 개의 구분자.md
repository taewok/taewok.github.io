---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (세 개의 구분자)"
date: 2023-08-11T19:39:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

임의의 문자열이 주어졌을 때 문자 "a", "b", "c"를 구분자로 사용해 문자열을 나누고자 합니다.

예를 들어 주어진 문자열이 "baconlettucetomato"라면 나눠진 문자열 목록은 ["onlettu", "etom", "to"] 가 됩니다.

문자열 myStr이 주어졌을 때 위 예시와 같이 "a", "b", "c"를 사용해 나눠진 문자열을 순서대로 저장한 배열을 return 하는 solution 함수를 완성해 주세요.

단, 두 구분자 사이에 다른 문자가 없을 경우에는 아무것도 저장하지 않으며, return할 배열이 빈 배열이라면 ["EMPTY"]를 return 합니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myStr의 길이 ≤ 1,000,000
  - myStr은 알파벳 소문자로 이루어진 문자열 입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myStr                |           결과            |
| -------------------- | :-----------------------: |
| "baconlettucetomato" | ["onlettu", "etom", "to"] |
| "abcd"               |           ["d"]           |
| "cabab"              |         ["EMPTY"]         |

## <b>풀이</b>

```js
function solution(myStr) {
  // "a", "b", "c"를 구분자로 사용하여 문자열을 나누고 배열로 반환
  const reg = /a|b|c/g;
  myStr = myStr
    .replaceAll(reg, " ")
    .split(" ")
    .filter((v) => v != "");

  // 나눠진 문자열 배열이 비어있으면 ["EMPTY"] 반환, 그렇지 않으면 배열 반환
  return myStr.length ? myStr : ["EMPTY"];
}
```

<h3><blockquote>문제풀이</blockquote></h3>

1. 정규 표현식 /a|b|c/g은 "a", "b", "c" 중 하나에 일치하는 모든 문자를 찾습니다.
2. myStr.replaceAll(reg, " ")는 정규 표현식에 일치하는 문자들을 공백으로 대체하여 문자열을 변환합니다.
3. split(" ") 메소드는 변환된 문자열을 공백을 기준으로 분할하여 배열로 만듭니다.
4. filter(v => v != "")는 빈 문자열을 필터링하여 배열의 요소 중 빈 요소를 제거합니다.
5. 배열에 값이 남아있으면 그 배열을 반환하고, 빈 배열이면 ["EMPTY"] 배열을 반환합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
