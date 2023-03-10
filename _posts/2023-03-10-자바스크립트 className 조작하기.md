---
title: "[JavaScript] className 읽기/추가/변경/삭제"
date: 2023-03-10T17:01:000
categories: [JavaScript]
tags: [JavaScript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">사용 상황</b>
<p>Html 요소에 css animation을 주고 싶을떄 className을 추가해서 animation을 동작시킨다.</p>

***

## <b style="border-bottom:2px solid gray">classList</b>
<p>공백으로 구분된 문자열인 element.className을 통해 엘리먼트의 클래스 목록에 접근하는 방식을 대체하는 간편한 방법이다.</p>

### <b>add</b>
- 이미 있는 class명은 자동으로 중복제외 된다.

```js
const box = document.getElementById("#box");
box.classList.add("추가");
box.classList.add("또추가", "또또추가");
console.log(box.classList);
//실행결과
추가 또추가 또또추가
```

### <b>읽기</b>
```js
const box = document.getElementById("#box");
console.log(box.classList);
//실행결과
추가 또추가 또또추가
```

### <b>remove</b>
```js
const box = document.getElementById("#box");
box.classList.remove("또추가");
console.log(box.className);
//실행결과
추가 또또추가
```

### <b>toggle</b>
- class명이 없으면 넣고 있으면 빼주는 간편토글

```js
//첫번쨰 토글 실행
const box = document.getElementById("#box");
box.classList.toggle("또추가");
//실행결과
추가 또추가 또또추가

//두번쨰 토글 실행
box.classList.toggle("또추가");
console.log(box.className);
//실행결과
추가 또또추가
```

***

## <b style="border-bottom:2px solid gray">className</b>
<p>클래스명이 단일값일 경우 사용</p>

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>