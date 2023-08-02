---
title: "[JavaScript] setTimeout() setInterval() 사용하기"
date: 2023-03-11T21:01:000
categories: [JavaScript]
tags: [javascript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">setTimeout() 이란?</b>
<p>일정 시간 후에 특정 코드, 함수를 실행하고 싶을 때 사용하는 함수</p>
- 1000 = 1초 뒤에 실행

```js
setTimeout(()=>{
    console.log("HI~");
},1000);
//1초뒤
HI~
```

### <b>clearTimeout()</b>
- <strong style="color:#ff526f">setTimeout()</strong> 함수를 사용하고 취소할 떄 사용하는 함수

```js
let timerVar = setTimeout(()=>{
    console.log("")
},3000);

clearTimeout(timerVar);
```

***

## <b style="border-bottom:2px solid gray">setInterval() 이란?</b>
<p>일정 시간마다  특정 코드, 함수를 반복해서 실행하고 싶을 때 사용하는 함수</p>
- 1000 = 1초 뒤에 실행

```js
setInterval(()=>{
    console.log("HI~");
},1000);
//1초뒤
HI~
//2초뒤
HI~
//3초뒤
HI~
......무한
```

### <b>clearInterval()</b>
- <strong style="color:#ff526f">setInterval()</strong> 함수를 사용하고 반복을 중지시킬 떄 사용하는 함수

```js
let timerVar = setInterval(()=>{
    console.log("")
},3000);

clearInterval(timerVar);
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>