---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (이차원 배열 대각선 순회하기)"
date: 2023-08-10T17:24:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

2차원 정수 배열 board와 정수 k가 주어집니다.

i + j <= k를 만족하는 모든 (i, j)에 대한 board[i][j]의 합을 return 하는 solution 함수를 완성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 0 ≤ s < 100
- 1 ≤ l ≤ 8
- 10l - 1 ≤ k < 10l
- 1 ≤ intStrs의 길이 ≤ 10,000
  - s + l ≤ intStrs의 원소의 길이 ≤ 120

<h3><blockquote>입출력 예
</blockquote></h3>

| board                                     |  k  | 결과 |
| ----------------------------------------- | :-: | ---- |
| [[0, 1, 2],[1, 2, 3],[2, 3, 4],[3, 4, 5]] |  2  | 8    |

## <b>풀이</b>

```js
function solution(board, k) {
    var answer = 0; // 결과 값을 저장할 변수

    // 2차원 배열의 행에 대해 반복
    for(let i = 0; i < board.length; i++) {
        // 현재 행의 열에 대해 반복
        for(let j = 0; j < board[i].length; j++) {
            // i + j가 k 이하인 조건을 만족하는 경우
            if(i + j <= k) {
                answer += board[i][j]; // 해당 위치의 값을 결과에 더해줌
            }
        }
    }

    return answer; // 최종 결과 반환
}
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
