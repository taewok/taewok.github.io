---
title: "[React] Jest(2) Matcher"
date: 2023-10-02T21:05:000
categories: [React]
tags: [react, jest] #소문자만 가능
---

---

## Matcher(매처)란?

<p>
 테스트 케이스에서 예상 결과와 실제 결과를 비교하고 검증하는 데 사용되는 함수들을 뜻하며 <br/>
 Matchers는 expect() 함수와 함께 사용되며, expect() 함수로 값에 대한 예상 결과와 실제 결과를 검사하는 역할을 합니다. <br/>
 Jest는 다양한 Matcher를 제공하여 다양한 유형의 검증을 수행할 수 있도록 합니다.
</p>

## 다양한 Matcher

<h3><blockquote>toBe()
</blockquote></h3>

- 값을 엄격하게(정확히) 비교합니다. 값과 데이터 유형이 정확히 일치해야 합니다.

```js
expect(result).toBe(3); // result의 값이 3과 정확히 일치해야 함
```

<br/>

<h3><blockquote>toEqual()
</blockquote></h3>

- 객체나 배열과 같은 복합 데이터 유형을 깊게 비교합니다.

```js
expect(result).toEqual(objectB); // result의 객체 내용이 깊게 일치해야 함
```

<br/>

<h3><blockquote>toMatch()
</blockquote></h3>

- 정규 표현식을 사용하여 문자열을 비교합니다.

```js
expect(result).toMatch(/pattern/); // result의 문자열이 정규 표현식과 일치해야 함
```

<br/>

<h3><blockquote>toContain()
</blockquote></h3>

- 배열 또는 문자열 내에 특정 요소가 존재하는지 확인합니다.

```js
expect(result).toContain(item); // result 배열에 특정 항목이 포함되어야 함
```

<br/>

<h3><blockquote>toBeNull()
</blockquote></h3>

- 값이 null인지 확인합니다.

```js
expect(result).toBeNull(); // 값이 null이어야 함
```

<br/>

<h3><blockquote>toBeDefined()
</blockquote></h3>

- 값이 undefined가 아닌지 확인합니다.

```js
expect(result).toBeDefined(); // 값이 undefined가 아니어야 함
```

<br/>

<h3><blockquote>toBeTruthy() / toBeFalsy()
</blockquote></h3>

- 값이 true 또는 false인지 확인합니다.

```js
expect(result).toBeTruthy(); // 값이 true이어야 함
expect(result).toBeFalsy(); // 값이 false이어야 함
```

<br/>

<h3><blockquote>toBeGreaterThan() / toBeLessThan()
</blockquote></h3>

- 값이 특정 값보다 큰지 또는 작은지 확인합니다.

```js
expect(result).toBeGreaterThan(5); // 값이 5보다 커야 함
expect(result).toBeLessThan(10); // 값이 10보다 작아야 함
```

<br/>

<h3><blockquote>toHaveLength()
</blockquote></h3>

- 배열 또는 문자열의 길이를 확인합니다.

```js
expect(result).toHaveLength(3); // 배열의 길이가 3이어야 함
expect(result).toHaveLength(10); // 문자열의 길이가 10이어야 함
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
