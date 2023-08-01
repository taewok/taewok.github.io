---
title: "[React] Outlet을 이용해 자손 컴포넌트에 props 전달하기"
date: 2023-05-17T16:30:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">사용 예시</b>

<p>다음의 router 구조가 있다고 가정하자</p>

```jsx
<Route path="/register" element={<Register />}>
  <Route path="/register/:RegisterType" element={<EmailConfirm />} />
</Route>
```

### 부모 컴포넌트

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
    <p>부모 컴포넌트안에 Outlet으로 원하는 자손 컴포넌트 위치를 정한다. 그리고 context prop으로 자손 컴포넌트에게에게 넘기고 싶은 값을 넣는다.</p>
</blockquote>

```jsx
import React, { useState } from "react";
import { Outlet } from "react-router";
import styled from "styled-components";

const Register = () => {
  const [id, setId] = useState("taewok");

  return (
    <RegisterContainer>
      <RegisterWrraper>
        <Title>회원가입</Title>
        <Outlet context={{ id }} />
      </RegisterWrraper>
    </RegisterContainer>
  );
};
export default Register;
```

### 자손 컴포넌트

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
    <p>자손 컴포넌트에서 위의 Outlet의 context prop을 가지려면 react-router-dom의 useOutletContext라는 Hook을 사용해야한다.</p>
</blockquote>

```jsx
import { useOutletContext } from 'react-router-dom';

function EmailConfirm() {
…
    const {id} = useOutletContext();
    console.lof(id);
…
}

//console 결과
taewok
```

<p>TypeScript 사용</p>

```jsx
import { useOutletContext } from 'react-router-dom';

interface UserInfoConText {
  id: string;
}

function Child() {
…
    const {id} = useOutletContext<UserInfoConText>();
    console.lof(id);
…
}

//console 결과
taewok
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
