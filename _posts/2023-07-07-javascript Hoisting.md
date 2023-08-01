---
title: "[JavaScript] 호이스팅(Hoisting)이란?"
date: 2023-07-07T18:35:000
categories: [JavaScript]
tags: [javaScript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray" class="h2">호이스팅(Hoisting)이란?</b>

함수 안에 있는 선언들을 모두 끌어올려서 해당 함수 유효 범위의 최상단에 선언하는 것을 말하며, JS의 독특한 특징이다

- 실제로 코드가 끌어올려지는 것은 아니다.

```js
//눈에 보이는 실제 코드
console.log(a); // undefined
var a = "A"; // var 변수 
```

```js
//호이스팅 된 코드
var a; 
console.log(a);
a = "A";
```

var a를 상단으로 호이스팅했기 때문에 변수 선언 전에 console.log로 출력을 해도 에러가 발생하지 않고, undefined 가 출력된다.

모든 선언(function, var, let, const 및 class)은 JavaScript에서 호이스팅되며, var 선언은 undefined로 초기화되지만 let 및 const 선언은 초기화되지 않은 상태로 유지된다.

<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">var, let, const</blockquote></h3>

- var

변수의 재선언/재할당 모두 가능, Hoisting이 되어 변수를 선언하기 전에 참조할 수 있다. 하지만 let,const를 더 권한다

```js
console.log(b) // undefined
var b = "something"
```

- let

변수의 재선언은 불가능하나 재할당은 가능, 변수를 선언하기 전에 참조할 수 없으나, 변수를 선언할 시 할당을 반드시 할 필요는 없다.

```js
console.log(b) // Error: Uncaught ReferenceError: b is not defined
let b = "something"
```

- const

변수의 재선언/재할당 모두 불가능, 변수를 선언하기 전에 참조할 수 없으며, 변수를 선언할 시 할당을 반드시 해야 한다.

```js
console.log(c); // ReferenceError: c is not defined
const c = 0;
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>

