---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (공백으로 구분하기 2)"
date: 2023-08-06T18:20:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

단어가 공백 한 개 이상으로 구분되어 있는 문자열 my_string이 매개변수로 주어질 때, my_string에 나온 단어를 앞에서부터 순서대로 담은 문자열 배열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- my_string은 영소문자와 공백으로만 이루어져 있습니다.
- 1 ≤ my_string의 길이 ≤ 1,000
- my_string의 맨 앞과 맨 뒤에도 공백이 있을 수 있습니다.
- my_string에는 단어가 하나 이상 존재합니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| my_string       |         결과         |
| --------------- | :------------------: |
| " i love you"   | ["i", "love", "you"] |
| " programmers " |   ["programmers"]    |

## <b>풀이</b>

```js
function solution(my_string) {
  // 주어진 문자열을 공백을 기준으로 분리한 후, 빈 문자열이 아닌 단어만 필터링하여 새로운 배열 생성
  return my_string.split(" ").filter((v) => v !== "");
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
