---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.1 (추억 점수)"
date: 2023-08-03T16:58:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

사진들을 보며 추억에 젖어 있던 루는 사진별로 추억 점수를 매길려고 합니다. 사진 속에 나오는 인물의 그리움 점수를 모두 합산한 값이 해당 사진의 추억 점수가 됩니다. 예를 들어 사진 속 인물의 이름이 ["may", "kein", "kain"]이고 각 인물의 그리움 점수가 [5점, 10점, 1점]일 때 해당 사진의 추억 점수는 16(5 + 10 + 1)점이 됩니다. 다른 사진 속 인물의 이름이 ["kali", "mari", "don", "tony"]이고 ["kali", "mari", "don"]의 그리움 점수가 각각 [11점, 1점, 55점]이고, "tony"는 그리움 점수가 없을 때, 이 사진의 추억 점수는 3명의 그리움 점수를 합한 67(11 + 1 + 55)점입니다.

그리워하는 사람의 이름을 담은 문자열 배열 name, 각 사람별 그리움 점수를 담은 정수 배열 yearning, 각 사진에 찍힌 인물의 이름을 담은 이차원 문자열 배열 photo가 매개변수로 주어질 때, 사진들의 추억 점수를 photo에 주어진 순서대로 배열에 담아 return하는 solution 함수를 완성해주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 3 ≤ name의 길이 = yearning의 길이≤ 100
  - 3 ≤ name의 원소의 길이 ≤ 7
  - name의 원소들은 알파벳 소문자로만 이루어져 있습니다.
  - name에는 중복된 값이 들어가지 않습니다.
  - 1 ≤ yearning[i] ≤ 100
  - yearning[i]는 i번째 사람의 그리움 점수입니다.
- 3 ≤ photo의 길이 ≤ 100
  - 1 ≤ photo[i]의 길이 ≤ 100
  - 3 ≤ photo[i]의 원소(문자열)의 길이 ≤ 7
  - photo[i]의 원소들은 알파벳 소문자로만 이루어져 있습니다.
  - photo[i]의 원소들은 중복된 값이 들어가지 않습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| name                            |   yearning    |                                               photo                                               |    결과     |
| ------------------------------- | :-----------: | :-----------------------------------------------------------------------------------------------: | :---------: |
| ["may", "kein", "kain", "radi"] | [5, 10, 1, 3] | [["may", "kein", "kain", "radi"],["may", "kein", "brin", "deny"], ["kon", "kain", "may", "coni"]] | [19, 15, 6] |
| ["may", "kein", "kain", "radi"] | [5, 10, 1, 3] |                        [["may"],["kein", "deny", "may"], ["kon", "coni"]]                         | [5, 15, 0]  |

## <b>풀이</b>

| name                            |   yearning    |                       photo                        |    결과    |
| ------------------------------- | :-----------: | :------------------------------------------------: | :--------: |
| ["may", "kein", "kain", "radi"] | [5, 10, 1, 3] | [["may"],["kein", "deny", "may"], ["kon", "coni"]] | [5, 15, 0] |

```js
function solution(name, yearning, photo) {
  var answer = [];

  for (let value of photo) {
    // photo 배열의 length만큼 반복
    let sum = 0; // 사진별 추억 점수 합산을 위한 변수 선언

    for (let v of value) {
      // 사진에 찍힌 인물 이름에 대해 반복
      if (name.includes(v)) {
        // 현재 인물 이름이 name 배열에 포함되어 있는지 확인
        sum += yearning[name.indexOf(v)]; // 그리움 점수를 더하여 합산
      }
    }

    answer.push(sum); // 사진별 추억 점수를 answer 배열에 추가
  }

  return answer;
}
```

<h3><blockquote>문제풀이
</blockquote></h3>

1. 첫번쨰 for of문을 사용하여 photo사진첩에 사진 개수 만큼 반복문 실행

2. 사진별 추억 점수 합산을 위한 sum변수 선언

3. 두번쨰 for of문은 첫번쨰 for of문의 value로 현재 사진에 찍힌 인물만큼 반목문 실행

4. <strong>includes()</strong>를 사용하여 name에 v라는 인물이 있는지 확인 후 <strong>indexOf()</strong>로 index를 구하여 점수 확인 후 sum에 더해준다.

5. 첫번쨰 for of문이 반복 될 떄마다 answer배열에 push로 사진별 추억 점수 추가

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
