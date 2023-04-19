---
title: "[JavaScript] Gsap ScrollTrigger사용하기"
date: 2023-04-19T14:12:000
categories: [JavaScript]
tags: [JavaScript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">ScrollTrigger란?</b>

<p>scroll 위치에 맞춰서 애니메이션을 실행시키고 싶은 경우 gsap 의 scrollTrigger을 사용하면 보다 쉽게 구현이 가능하다!</p>

---

## <b style="border-bottom:2px solid gray">기본 Setting</b>

## ✅ gsap setting

### cdn 파일로 불러오기

```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.11.3/gsap.min.js"></script>
```

### npm 설치

```js
npm install gsap
```

## ✅ ScrollTrigger setting

<p>gsap 속성에서 scrollTrigger 속성을 사용할 수 있게 된다.</p>

```js
gsap.registerPlugin(ScrollTrigger);
```

---

## <b style="border-bottom:2px solid gray">기본 속성들</b>

### <b style="font-size:22px">✅ trigger</b>

```js
gsap.to(".box", {
  scrollTrigger: {
    trigger: "",
  },
});
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
    <p>scrollTrigger: {trigger: "target"}</p>
    <li style="color:#ff526f">target : 스크롤의 기준이 될 개체를 class명,id로 선택가능</li>
</blockquote>

### <b style="font-size:22px">✅ start,end</b>

```js
gsap.to(".box", {
  scrollTrigger: {
    trigger: ".box",
    start: ""
  },
});
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
<p>값이 하나인 경우는 선택한 요소에 높이를 기준으로 시작선과 끝맺음 선을 정한다.</p>
<li style="color:#ff526f">top,center,bottom,px,%등의 단위 사용가능</li>
</blockquote>

<p class="codepen" data-height="300" data-default-tab="js,result" data-slug-hash="jOeMaex" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/jOeMaex">
  Untitled</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<br/>
<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
<span>값이 2개인 경우는 첫번쨰는 선택한 요소에 높이를 기준으로 시작선과 끝맺음 선을 정하고 두번째 값은 viewPort 높이를 기준으로 시작선과 끝맺음 선을 정한다.</span>
</blockquote>

<p class="codepen" data-height="300" data-default-tab="js,result" data-slug-hash="poxEdXx" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/poxEdXx">
  2.scrollTrigger start,end</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

### <b style="font-size:22px">✅ markers</b>

<p>기본적으론 false이며 true로 설정시 시작선과 끝맺음선이 보이게 된다.</p>

```js
gsap.to(".box", {
      scrollTrigger: {
      trigger: ".box",
      start: "top",
      end: "bottom",
      markers: true,
    }
});
```

<p>스타일링도 설정해 줄 수 있다.</p>

```js
markers: {
    startColor: "red",
    endColor: "yellow",
    fontSize: "3rem"
}
```

|속성명|설명|
|:---:|:---|
|`startColor`|시작선의 색깔변경|
|`endColor`|끝맺음 선의 색깔변경|
|`fontSize`|가이드 문자에 font-size 변경|

### <b style="font-size:22px">✅ toggleActions</b>

<p>toggleActions:"none none none none"</p>

<span>왼쪽에서 순서대로</span><br/>

|속성명|설명|
|:---:|:---|
|`onEnter`|scroller-start선이 start선을 지나갔을 떄|
|`onLeave`|scroller-end 선이 end 선을 지날 떄|
|`onEnterBack`|end 선을 지나쳤던 scroller-end 선이 다시 돌아올 때|
|`onLeaveBack`|start 선을 지나쳤던 scroller-start 선이 다시 돌아올 때|

<span>총 4개의 action을 간편한 문자열로 정해줄 수 있다.</span><br/>

|속성명|설명|
|:---:|:---|
|`play`|애니메이션이 실행중이라면 계속 실행|
|`pause`|애니메이션이 진행중이라면 그대로 멈춘다.|
|`resume`|pause에 의해 멈춘 애니메이션을 그자리에서 재개|
|`reverse`|처음 위치로 되돌아간다. duration을 적용한 만큼 걸린다.|
|`restart`|애니메이션을 처음 위치에서 다시 시작|
|`reset`|처음 위치로|
|`complete`|바로 애니메이션의 끝나는 지점으로|
|`none`|아무것도 없다.|

<p class="codepen" data-height="300" data-default-tab="js,result" data-slug-hash="MWPjQyy" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/MWPjQyy">
  Untitled</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
<br/>
<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
<span>toggleActions이 아닌 세부적인 속성으로 사용도 가능하다. 함수가능</span>
</blockquote>

<p class="codepen" data-height="300" data-default-tab="js,result" data-slug-hash="ExdgEdz" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/ExdgEdz">
  scrollTrigger onEnter,onLeave,onEnterBack,onLeaveBack</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

### <b style="font-size:22px">✅ scrub</b>