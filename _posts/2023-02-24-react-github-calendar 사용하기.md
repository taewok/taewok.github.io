---
title: "[React] react-github-calendar 사용하기"
date: 2023-02-24T18:34:000
categories: [React]
tags: [react, react-github-calendar] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">사용하게된 이유</b>
<p>백엔드에서 user에 github 정보를 받아오는데 잔디까지 가져오는데 어려움이 있어 직접 알아보니 프론트에서 <strong style="color:#ff526f">react-github-calendar</strong>라는 라이브러리를 사용하면 간편하게 잔디를 가져올 수 있다고 한다.</p>

***

## <b style="border-bottom:2px solid gray">설치</b>
```
npm install react-github-calendar //npm 사용시
yarn add react-github-calendar //yarn 사용시
```

***

## <b style="border-bottom:2px solid gray">기본 사용</b>
- username 부분에 원하는 유저에 이름을 입력해놓는다

```jsx
import React from 'react';
import GitHubCalendar from "react-github-calendar"; 

const App = () => {
  return (
    <div>
      <GitHubCalendar username='유저이름'/>
    </div>
  );
};

export default App;
```
<p>코드를 실행하면 아래 이미지와 같이 원하는 유저에 github 잔디를 가져온다</p>
<img src="https://user-images.githubusercontent.com/88264006/221151045-f4bd2e6c-e759-4fd3-be4e-467d33b4db10.png" alt="react-github-calendar 사용완료"/>

## <b style="border-bottom:2px solid gray">그외 옵션</b>
### <b>year</b>
- 원하는 년도를 설정 default는 작년

```jsx
<GitHubCalendar year='2023'/>
```

### <b>color</b>
- 잔디에 색깔 설정(가장 진한 색을 지정)

```jsx
<GitHubCalendar color='#ffffff'/>
```

### <b>fontSize</b>
- font 크기를 지정

```jsx
<GitHubCalendar fontSize={19}/>
```

### <b>blockSize</b>
- 잔디 블록의 크기를 지정

```jsx
<GitHubCalendar blockSize={19}/>
```

### <b>blockMargin</b>
- 잔디 블록 끼리의 margin 지정

```jsx
<GitHubCalendar blockMargin={19}/>
```

### <b>blockRadius</b>
- 잔디 블록의 border-radius 지정

```jsx
<GitHubCalendar blockRadius={19}/>
```