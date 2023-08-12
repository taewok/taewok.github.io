---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (특정 문자열로 끝나는 가장 긴 부분 문자열 찾기)"
date: 2023-08-11T17:20:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

문자열 myString과 pat가 주어집니다. myString의 부분 문자열중 pat로 끝나는 가장 긴 부분 문자열을 찾아서 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 5 ≤ myString ≤ 20
- 1 ≤ pat ≤ 5
  - pat은 반드시 myString의 부분 문자열로 주어집니다.
- myString과 pat에 등장하는 알파벳은 대문자와 소문자를 구분합니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| myString   | pat  | 결과       |
| ---------- | :--: | ---------- |
| "AbCdEFG"  | "dE" | "AbCdE"    |
| "AAAAaaaa" | "a"  | "AAAAaaaa" |

## <b>풀이</b>

```js
function solution(myString, pat) {
  // myString에서 pat로 끝나는 부분 문자열의 인덱스를 찾습니다.
  // lastIndexOf() 함수를 사용하여 pat이 마지막으로 등장하는 인덱스를 구합니다.
  // 그 인덱스에 pat의 길이를 더해주면 pat로 끝나는 부분 문자열의 끝 인덱스가 됩니다.
  // 이를 이용하여 slice() 함수로 해당 부분 문자열을 추출합니다.
  var answer = myString.slice(0, myString.lastIndexOf(pat) + pat.length);
  return answer;
}
```

- <strong>lastIndexOf()</strong> 함수는 배열에서 지정된 값과 일치하는 마지막 요소의 인덱스를 반환하는 JavaScript 배열 메서드입니다. 이 메서드는 배열을 뒤에서부터 검색하며, 일치하는 값이 있는 경우 마지막으로 일치하는 요소의 인덱스를 반환하고 일치하는 값이 없는 경우 -1을 반환합니다.

- <strong>slice()</strong> 함수는 배열의 일부분을 선택하여 새로운 배열로 반환하는 JavaScript 배열 메서드입니다. 이 메서드는 원본 배열을 수정하지 않고 원하는 범위의 요소를 복사하여 새로운 배열을 생성합니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
