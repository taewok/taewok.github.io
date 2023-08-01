---
title: "[JavaScript] ES6 const와 let 이해하기"
date: 2023-02-25T19:30:000
categories: [JavaScript]
tags: [JavaScript,let,const] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">const</b>
- constant(상수)를 뜻하며 '항상 같은 수'를 말한다.
- 값을 재할당 할 수 없다.
- 재선언은 할 수 없다.

```js
const name = "독고진";
name = "구애정";
console.log(name);
//실행결과
TypeError: Assignment to constant variable.
```

- const 변수는 반드시 값이 할당되어야 한다.

```js
const age;      // X
const age = 21; // O
```

***

## <b style="border-bottom:2px solid gray">let</b>
- 값을 재할당 할 수 있다.
- 재선언은 할 수 없다.

```js
let name = "독고진";
name = "구애정";
console.log(name);
//실행결과
구애정
```

## <b style="border-bottom:2px solid gray">var</b>
<p>const와 let이 나오기 이전에 var라는 변수가 있었지만 이제는 사용을 지양하는 쪽</p>

- 값 재할당 가능
- 재선언 가능 

```js
var age = 28;       //결과 : 28  
age = 18;           //결과 : 18 
var age = 'old';    //결과 : 'old' 
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>