---
title: "[JavaScript] slice/splice 함수?"
date: 2023-03-30T11:10:000
categories: [JavaScript]
tags: [JavaScript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">slice() 함수란?</b>
- <b style="color:#ff526f">slice()</b> 함수는 배열에서 원하는 범위를 복사한 <strong style="color:#ff526f">새로운 배열</strong> 을 만든다.
- 원본 배열이 그대로 보존되야 하는 상황에서 주로 사용한다.

<p>배열에 0번쨰부터 2번쨰 index를 복사하는 코드</p>

```js
const array = ["a","b","c","d","e"];
const start = 0;
const last = 3;
console.log(array.slice(start,last));  //["a","b","c"]
```

<b>start</b>: 어디서 부터 자를지에 따른 시작점을 지정 <br/>
<b>last</b>: 어디까지 자를지에 따른 맺음점을 지정 

### <b>start 인자만 지정해줬을 때</b>
- 지정 순번부터 마지막까지 배열을 복사

```js
const array = ["a","b","c","d","e"];
console.log(array.slice(3));  //["d","e"]
```

### <b>인자가 없을 떄</b>
- 처음부터 마지막까지 배열을 복사

```js
const array = ["a","b","c","d","e"];
console.log(array.slice());  //["a","b","c","d","e"]
```

***

## <b style="border-bottom:2px solid gray">splice() 함수란?</b>
- <b style="color:#ff526f">splice()</b> 함수는 배열에서 원하는 범위를 삭제하거나 새로운 값을 추가한다.
- 원본 배열이 변경이 된다.

```js
const array = ["a", "b", "c", "d", "e"];
const start = 1;
const num = 3;
const newArray = array.splice(start, num);
console.log(newArray)  // ["b","c","d"]
console.log(array);    // ["a","e"] 
```
<p>기존 array변수에는 값이 사라졌고 그 값은 newArray에 담겼다.</p>

<b>start</b>: 어디서 부터 자를지에 따른 시작점을 지정 <br/>
<b>num</b>: start 앞에 몇개에 요소를 선택할지

### <b>삭제X 값추가</b>
- 2번쨰부터 아무값도 삭제하지 않고 값을 추가해보자

```js
const array = ["a", "b", "c", "d", "e"];
array.splice(2,0,-1,-2)
console.log(array);  // ['a', 'b', -1, -2, 'c', 'd', 'e']
```

<p>시작 인자와 마지막 인자 이후로는 그 자리에 추가해주고 싶은 값을 인자로 넘기게 된다.</p>

### <b>삭제O 값추가</b>
- 원하는 값을 삭제하고 그 자리에 다른 값을 넣어보자

```js
const array = ["a", "b", "c", "d", "e"];
array.splice(2,2,-1,-2,-3)
console.log(array);  // ['a', 'b', -1, -2, -3, 'e']
```

<p>시작 인자와 마지막 인자 이후로는 그 자리에 추가해주고 싶은 값을 인자로 넘기게 된다.</p>

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>