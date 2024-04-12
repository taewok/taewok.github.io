---
title: "[Css] css만으로 Scroll Snap 구현하기"
date: 2024-04-13T00:35:000
categories: [Css]
tags: [css] #소문자만 가능
---

---

<p>BUCLIN 프로젝트 진행중 상품을 하나씩 스크롤 되도록 구현이 필요해서 구글링 중 알게된 css만으로 scroll snap을 구현할 수 있다고하여 기록해본다.</p>

## <b>scroll snap이란?</b>

<p>스크롤 스냅(scroll snap)은 사용자가 웹 페이지를 스크롤할 때 요소가 일정한 간격으로 자동으로 스냅되는 기능을 말하며, 용자가 스크롤할 때 페이지가 자연스럽게 움직이고, 각 섹션 또는 이미지가 스냅되어 정확한 위치에 배치된다.</p>

#### 예시

<img src="https://i.ibb.co/XyLtz87/Animation.gif"/>

<!-- ## <b>구현 코드</b>

```html
<div class="scroll-box">
  <div class="item" style="background: coral;">1</div>
  <div class="item" style="background: rgb(121, 241, 215);">2</div>
  <div class="item" style="background: rgb(206, 255, 187);">3</div>
  <div class="item" style="background: rgb(247, 179, 216);">4</div>
  <div class="item" style="background: rgb(167, 202, 255);">5</div>
</div>
``` -->

<br/>

<h3><blockquote>scroll snap 구현의 핵심
</blockquote></h3>

<p>scroll snap 구현의 핵심은 부모 element에는 'scroll-snap-type' 속성 필수로 적용하고 자식 element는 'scroll-snap-align' 속성을 필수로 적용해주어야 한다는 것 이다.</p>

#### 💡 scroll-snap-type 속성에 따른 차이

<p>주로 쓰이는 scroll-snap-type: x mandatory와 scroll-snap-type: y mandatory가 예시이다.</p>

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="ExJRJbQ" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/ExJRJbQ">
  scroll-snap-type</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

#### 💡 scroll-snap-align 속성에 따른 차이

<p>차이를 쉽게 보기 위해 자식 요소의 높이를 조절해줬다.</p>

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="BaEVbox" data-user="taewok" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/taewok/pen/BaEVbox">
  scroll snap</a> by taewok (<a href="https://codepen.io/taewok">@taewok</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

<strong>start</strong>: 요소를 스크롤 스냅 지점에서 시작 위치에 정렬합니다. 즉, 요소의 시작 부분이 스냅 지점에 맞춰 정렬됩니다.

<strong>center</strong>: 요소를 스크롤 스냅 지점에서 중앙에 정렬합니다. 즉, 요소가 스냅 지점을 중심으로 가운데에 위치하도록 정렬됩니다.

<strong>end</strong>: 요소를 스크롤 스냅 지점에서 끝 위치에 정렬합니다. 즉, 요소의 끝 부분이 스냅 지점에 맞춰 정렬됩니다.

<p>이렇게 css만으로 scroll snap을 구현해봤다. 이제부터 간단한 scroll snap을 구현할 때는 css만으로도 충분할 것 같다. 좀 더 세세한 기능이 추가되면 javascript를 사용해야겠지만....</p>

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
