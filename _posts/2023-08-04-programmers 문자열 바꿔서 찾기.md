---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (문자열 바꿔서 찾기)"
date: 2023-08-04T19:55:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자 "A"와 "B"로 이루어진 문자열 myString과 pat가 주어집니다. myString의 "A"를 "B"로, "B"를 "A"로 바꾼 문자열의 연속하는 부분 문자열 중 pat이 있으면 1을 아니면 0을 return 하는 solution 함수를 완성하세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ myString의 길이 ≤ 100
- 1 ≤ pat의 길이 ≤ 10
  - myString과 pat는 문자 "A"와 "B"로만 이루어진 문자열입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString |  pat   | 결과 |
| -------- | :----: | :--: |
| "ABBAA"  | "AABB" |  1   |
| "ABAB"   | "ABAB" |  0   |

## <b>풀이</b>

```js
// 변환된 문자열 내에 pat이 포함되어 있는지 여부를 반환하는 함수
function solution(myString, pat) {
  // 문자열 변환을 위한 빈 문자열 초기화
  var answer = "";

  // myString의 각 문자를 순회하며 변환한 문자열 생성
  for (let str of myString) {
    if (str === "A") answer += "B"; // "A"를 "B"로 변환하여 추가
    else answer += "A"; // "B"를 "A"로 변환하여 추가
  }

  // 변환된 문자열 내에 pat이 포함되어 있는지 확인하고 결과 반환
  return answer.includes(pat) ? 1 : 0;
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
