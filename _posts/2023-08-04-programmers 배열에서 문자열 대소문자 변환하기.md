---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 <배열에서 문자열 대소문자 변환하기
>"
date: 2023-08-04T20:14:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 배열 strArr가 주어집니다. 모든 원소가 알파벳으로만 이루어져 있을 때, 배열에서 홀수번째 인덱스의 문자열은 모든 문자를 대문자로, 짝수번째 인덱스의 문자열은 모든 문자를 소문자로 바꿔서 반환하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ strArr ≤ 20
  - 1 ≤ strArr의 원소의 길이 ≤ 20
  - strArr의 원소는 알파벳으로 이루어진 문자열 입니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| strArr                    |           결과            |
| ------------------------- | :-----------------------: |
| ["AAA","BBB","CCC","DDD"] | ["aaa","BBB","ccc","DDD"] |
| ["aBc","AbC"]             |       ["abc","ABC"]       |

## <b>풀이</b>

```js
function solution(strArr) {
  // 문자열 배열(strArr)을 map 함수를 사용하여 각 원소에 대해 처리
  // v: 현재 원소의 값, i: 현재 원소의 인덱스
  return strArr.map((v, i) =>
    // 짝수 번째 원소일 경우 (인덱스가 0, 2, 4, ...)
    i % 2 === 0
      ? // 해당 원소의 모든 문자를 소문자로 변환하여 반환
        v.toLowerCase()
      : // 홀수 번째 원소일 경우 (인덱스가 1, 3, 5, ...)
        // 해당 원소의 모든 문자를 대문자로 변환하여 반환
        v.toUpperCase()
  );
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
