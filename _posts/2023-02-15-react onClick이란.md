---
title: "[React] onClick 사용하기"
date: 2023-02-15T11:25:000
categories: [react]
tags: [react,onClick] #소문자만 가능
---

---
## <b style="border-bottom:2px solid gray">react onClick이란?</b>
<p>- react에서 자주 사용되는 이벤트로 html 요소를 클릭했을 떄 실행할 함수를 말한다.</p>

***
## <b style="border-bottom:2px solid gray">onClick 기본 사용법</b>
### App.js
- onClick은 클릭했을떄 함수를 실행하는 것 이기 떄문에 <strong style="color:orange">()=></strong>를 사용해 함수를 만들어준다.
```js
function App() {
  return (
    <div>
      <button onClick={() => console.log("click!")}>클릭!!!</button>
    </div>
  );
}
export default App;
```
<img src="https://user-images.githubusercontent.com/88264006/218914559-c76a3e80-c6cc-4fd2-aa02-8467ecff54a7.png" alt="onClcik함수 실행"/>
<p>위 코드를 실행하면 이미지와 같이 button을 누르기 전엔 콘솔에 아무것도 안찍혀 있다.<br/>
클릭하면 콘솔에 원했던 "click!"가 찍힌다.</p>
<img src="https://user-images.githubusercontent.com/88264006/218915181-13ed377a-b73e-49a7-9c85-b971f14d366d.png" alt="onClick함수 실행"/>

***

## <b style="border-bottom:2px solid gray">react onClick함수 분리해서 관리하기</b>
<p>- 지금이야 단순한 콘솔을 찍는거라 더럽지 않지만 실전으로 들어가면 100% 더러워진다.</p>

### <b>1. 매개변수가 있을떄</b>
```js
function App() {
  const click = (e) => {
    console.log(e);
  };
  return (
    <div>
      <button onClick={(e)=>click(e)}>클릭!!!</button>
    </div>
  );
}
export default App;
```
<p>요소 정보 e를 매개변수로 넘겨서 콘솔에 찍으면 아래와 같이 많은 정보를 얻을 수 있다.</p>
<img src="https://user-images.githubusercontent.com/88264006/218948840-203b9607-a732-48a2-bcfa-2e6f9b8ccd3d.png"/>

### <b>2. 매개변수가 없을떄</b>
- 딱히 html 요소가 전달해야할 매개변수가 없다면 가장 깔끔한 코드
```js
function App() {
  const click = () => {
    console.log("click");
  };
  return (
    <div>
      <button onClick={click}>클릭!!!</button>
    </div>
  );
}
export default App;
//클릭 후 콘솔
click
```

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>