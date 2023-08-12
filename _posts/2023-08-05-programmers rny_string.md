---
title: "[Programmers] 프로그래머스 코딩테스트 연습 Lv.0 (rny_string)"
date: 2023-08-05T20:22:000
categories: [Programmers]
tags: [programmers] #소문자만 가능
---

---

## <b>문제</b>

<h3><blockquote>문제설명
</blockquote></h3>

'm'과 "rn"이 모양이 비슷하게 생긴 점을 활용해 문자열에 장난을 하려고 합니다. 문자열 rny_string이 주어질 때, rny_string의 모든 'm'을 "rn"으로 바꾼 문자열을 return 하는 solution 함수를 작성해 주세요.

<h3><blockquote>제한사항
</blockquote></h3>

- 1 ≤ rny_string의 길이 ≤ 100
- rny_string은 영소문자로만 이루어져 있습니다.

<h3><blockquote>입출력 예
</blockquote></h3>

| rny_string    |      결과       |
| ------------- | :-------------: |
| "masterpiece" | "rnasterpiece"  |
| "programmers" | "prograrnrners" |

## <b>풀이</b>

```js
function solution(rny_string) {
  // 문자열 rny_string의 모든 'm'을 "rn"으로 바꾸고 결과 반환
  return rny_string.replaceAll("m", "rn");
}
```

- <strong>replaceAll()</strong> 메서드는 첫 번째 인자로 지정한 문자 또는 정규식을 두 번째 인자로 지정한 문자열로 모두 바꿉니다.

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
