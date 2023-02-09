---
title: "[React] react-icons 사용하기"
date: 2023-02-07T17:4:000
categories: [react]
tags: [react] #소문자만 가능
---

---

<p>react 프로젝트에서 간편하고 퀄리티있는 아이콘을 사용하게 해주는 라이브러리~~</p>
[https://react-icons.github.io/react-icons/](https://react-icons.github.io/react-icons/) 리액트 아이콘!!!!<br/>

## <b style="border-bottom:2px solid gray">사용전 준비</b>
<span>우선 해당 react 프로젝트 터미널에 라이브러리를 설치 해준다.</span>
```
npm install react-icons --save // npm 사용시
yarn add react-icons // yarn 사용시
```
![image](https://user-images.githubusercontent.com/88264006/217190949-69a02f1c-1b6f-45cf-8429-8d0fbf7d8ccb.png)<br/>
<br/>
<span>위에 보이는 react-icons 사이트에 들어가서</span><br/>
<span>왼쪽 상단에 검색창에 자물쇠 icon을 찾기 위해 lock를 검색하면 나오는 <b>AiFillLock</b>를 한번 사용해보겠습니다.</span>
![image](https://user-images.githubusercontent.com/88264006/217192731-0d9b5885-8736-4da3-851d-5a6b5a412ce2.png)<br/>

***

## <b style="border-bottom:2px solid gray">App.css</b>
```react
body{
    height:100vh;
}

#root{
    display: flex;
    justify-content: center;
    height: 100%;
}

svg{
    margin-top: 400px;
    scale: 6;
}
```

## <b style="border-bottom:2px solid gray">App.js</b>
```react
import {AiFillLock} from "react-icons/ai";
import "./App.css";

function App() {
  return (
    <div>
      <AiFillLock/>
    </div>
  );
}
export default App;
```

<p><b style="color:orange">import {AiFillLock} from "react-icons/ai"</b>이런 식으로 불러오는데 2가지만 기억하자</p>

1. 가져오고 싶은 아이콘 이름을 {}안에 넣는다.
2. "react-icons"이 기본 주소인데 아이콘 이름 맨 앞 두글자를 소문자로 추가해줘야 한다. ex)"react-icons/ai"

<span>그러면 이렇게 아이콘이 잘 나온다!</span><br/>
![image](https://user-images.githubusercontent.com/88264006/217201288-116989e4-84c7-4055-ae65-65bf5e922f44.png)
<br/>

## <b style="border-bottom:2px solid gray">마치며</b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>

