---
title: "[React] 'value' prop on 'input' should not be null 에러발생"
date: 2023-02-18T14:31:000
categories: [React,Error]
tags: [react,error] #소문자만 가능
---

---

<img src="https://user-images.githubusercontent.com/88264006/219843422-940642bd-e678-4e64-be5e-03ee602cac37.png" alt="`value` prop on `input` should not be null 에러발생"/>
<p>팀 프로젝트를 진행하던 중 잘 동작하던 코드에 콘솔에 위와 같은 에러가 생겼다. 아무래도 백하고 통신 중 input 값에 null이나 undefined 값이 들어가서 그런 것 같다.</p>
<p>동작하는 데는 아무 문제가 없지만 에러 콘솔은 없을수록 좋으니...</p>

## <b style="border-bottom:2px solid gray">해결방법</b>
<p>input에 value 부분에 null 혹은 undefined 값일 때의 대처를 작성해 준다.</p>

```js
<input value={text || " "} onChange={(value)=>textOnChange(value)}/>
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>