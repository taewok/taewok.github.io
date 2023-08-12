---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (5명씩)"
date: 2023-08-05T20:50:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

최대 5명씩 탑승가능한 놀이기구를 타기 위해 줄을 서있는 사람들의 이름이 담긴 문자열 리스트 names가 주어질 때, 앞에서 부터 5명씩 묶은 그룹의 가장 앞에 서있는 사람들의 이름을 담은 리스트를 return하도록 solution 함수를 완성해주세요. 마지막 그룹이 5명이 되지 않더라도 가장 앞에 있는 사람의 이름을 포함합니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 5 ≤ names의 길이 ≤ 30
- 1 ≤ names의 원소의 길이 ≤ 10
- names의 원소는 영어 알파벳 소문자로만 이루어져 있습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| names                                                      | 결과            |
| ---------------------------------------------------------- | --------------- |
| ["nami", "ahri", "jayce", "garen", "ivern", "vex", "jinx"] | ["nami", "vex"] |

## <b>풀이</b>

```js
function solution(names) {
  return names.filter((v, i) => i % 5 === 0);
  // filter 함수를 사용하여 배열의 각 요소를 확인하면서 조건을 만족하는 요소들을 선택하여 새 배열로 반환
  // (v, i)는 각 요소의 값과 인덱스를 나타냄
  // i % 5 === 0는 인덱스가 0, 5, 10, ... 인 요소들을 선택하는 조건
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
