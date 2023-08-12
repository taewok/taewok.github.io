---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.1 <크기가 작은 부분 문자열
>"
date: 2023-08-03T16:25:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

숫자로 이루어진 문자열 t와 p가 주어질 때, t에서 p와 길이가 같은 부분문자열 중에서, 이 부분문자열이 나타내는 수가 p가 나타내는 수보다 작거나 같은 것이 나오는 횟수를 return하는 함수 solution을 완성하세요.

예를 들어, t="3141592"이고 p="271" 인 경우, t의 길이가 3인 부분 문자열은 314, 141, 415, 159, 592입니다. 이 문자열이 나타내는 수 중 271보다 작거나 같은 수는 141, 159 2개 입니다.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ p의 길이 ≤ 18
- p의 길이 ≤ t의 길이 ≤ 10,000
- t와 p는 숫자로만 이루어진 문자열이며, 0으로 시작하지 않습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| t              |   p   | 결과 |
| -------------- | :---: | :--: |
| "3141592"      | "271" |  2   |
| "500220839878" |  "7"  |  8   |

## <b>풀이</b>

| t         |  p  |
| --------- | :-: |
| "3141592" |  2  |

```js
function solution(t, p) {
  var answer = 0;
  var count = t.length; // 반복문을 실행할 횟수

  for (let i = 0; i < count; i++) {
    if (t.length >= p.length) {
      // t문자열에 남은 length가 p.length 보다 작다면 실행이 안된다.
      if (Number(t.slice(0, p.length)) <= Number(p)) answer += 1; // 문자열 t의 맨 앞에서부터 p의 길이만큼 숫자를 자르고 정수로 변환한 값이 p보다 작거나 같다면 answer+1
      t = t.slice(1); // 문자열 t에서 맨 앞 글자 자르기
    } else break;
  }

  return answer;
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. 반복문을 실행할 횟수인 t.length를 count 변수에 담아준다. 그 이유는 t = t.slice 코드에 의해 t.length가 수시로 변하기 때문이다. 그리고 for문으로 count만큼 반목하게 한다.

2. for문 안에 t문자열의 길이가 p문자열의 길이보다 크거나 같은 경우에만 다음 코드를 실행하게 하며 조건에 부합하지 않다면 break로 반복문을 멈춘다.

3. 문자열 t의 맨 앞부터 p의 길이만큼 숫자를 자르고 정수로 변환한 값이 p를 정수로 변환한 값보다 작거나 같다면 answer+=1을 해준다. 

4. 다음 반복문을 위해 t = t.slice(1)로 맨 앞 문자열을 잘라준다. (한 칸씩 이동하여) 다음 부분 문자열을 비교할 수 있도록 준비

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
