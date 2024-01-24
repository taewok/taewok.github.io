---
title: "[javascript] alert, prompt, confirm"
date: 2024-01-24T16:01:000
categories: [React, JavaScript]
tags: [react, javascript] #소문자만 가능
---

---

<h3><blockquote>alert()
</blockquote></h3>

<p>사용자에게 간단한 메시지를 보여주는 기능을 수행하는 함수이며, 함수를 호출하면 브라우저는 경고 창을 띄워서 메시지를 표시하며, 사용자가 확인 버튼을 누르기 전까지는 다른 작업을 진행할 수 없습니다.</p>
<br/>

<p>간단한 alert() 함수의 사용 예제는 다음과 같습니다.</p>

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="QWoyqJw" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/QWoyqJw">
  Untitled</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<br/>

<h3><blockquote>prompt()
</blockquote></h3>

<p>사용자에게 입력을 받기 위해 사용되는 간단한 대화상자(Dialog)입니다. 사용자에게 텍스트를 입력받는 데 주로 사용됩니다.</p>
<br/>

<p>간단한 prompt() 함수의 사용 예제는 다음과 같습니다.</p>

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="QWoyqob" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/QWoyqob">
  prompt</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<br/>

```js
const userInput = prompt(message, defaultText);
```

- <b>message</b>: 사용자에게 보여질 메시지 또는 안내문입니다. 이는 일반적으로 입력이 어떤 목적으로 필요한지에 대한 안내를 담고 있습니다.

- <b>defaultText</b> (선택적): 입력 상자에 미리 표시될 기본 텍스트입니다. 사용자는 이 텍스트를 그대로 사용하거나 변경할 수 있습니다.

- <b>userInput</b>: 사용자가 입력한 텍스트를 반환하며 사용자가 '취소' 버튼을 누르면 null이 반환됩니다.

<br/>

<h3><blockquote>confirm()
</blockquote></h3>

<p>사용자에게 확인 대화 상자를 표시하여 "확인" 또는 "취소" 버튼을 클릭하게 하고, 사용자의 선택에 따라 true 또는 false 값을 반환하는 함수입니다.</p>
<br/>

<p>간단한 confirm() 함수의 사용 예제는 다음과 같습니다.</p>

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="Babdebe" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/Babdebe">
  Untitled</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
