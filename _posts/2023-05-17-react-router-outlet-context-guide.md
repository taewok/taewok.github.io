---
title: "[React] Outlet으로 자식 컴포넌트에 Props 전달하기"
date: 2023-05-17T20:27:00
categories: [react]
tags: [react, react-router-dom, outlet, useoutletcontext, props-drilling]
description: "React Router의 Outlet을 사용할 때 자식 컴포넌트로 데이터를 전달하지 못해 고민인가요? useOutletContext를 활용한 실무 해결법을 소개합니다."
custom_style: true
---

---

## 🚀 도입: 왜 Outlet은 Props 전달이 안 될까?

리액트로 프로젝트를 하다 보면 중첩 라우팅(Nested Routing)을 자주 사용하게 됩니다. 공통 레이아웃을 잡고 내부 컨텐츠만 바꿔주는 방식이죠. 그런데 여기서 한 가지 문제에 직면했습니다.

"부모 라우트에서 가지고 있는 상태값을 자식 라우트 컴포넌트에게 전달하고 싶은데, `<Outlet />`에는 직접 Props를 내릴 수가 없네?"

보통의 컴포넌트라면 `<Child param={data} />`와 같이 작성하면 되지만, 라우터가 렌더링을 제어하는 `Outlet`은 구조가 다릅니다. 이 문제를 해결하기 위해 제가 실무에서 적용한 `useOutletContext` 활용법을 정리했습니다.

---

## 💡 개념 설명: Outlet과 Context의 만남

React Router Dom v6에서 제공하는 `Outlet`은 자식 라우트가 렌더링될 위치를 지정하는 역할을 합니다. 이때 부모의 데이터를 공유하기 위해 별도의 Context API를 직접 설정할 수도 있지만, 라우터 자체에서 지원하는 `context` prop을 사용하면 훨씬 깔끔하게 해결됩니다.

---

## 💻 코드 예시: 실전 적용하기

실제로 제가 회원가입 프로세스를 구현하면서 적용했던 구조를 예시로 들어보겠습니다.

### 1. 부모 컴포넌트 (데이터 전달)

부모인 `Register` 컴포넌트에서 공통으로 쓰일 `id`나 상태 변경 함수를 `context`에 담아 보냅니다.

{% raw %}

```jsx
import React, { useState } from "react";
import { Outlet } from "react-router-dom";
import styled from "styled-components";

const Register = () => {
  // 자식 컴포넌트들과 공유할 상태
  const [id, setId] = useState("gemini_user");

  return (
    &lt;RegisterContainer&gt;
      &lt;RegisterWrapper&gt;
        &lt;h1&gt;회원가입 단계&lt;/h1&gt;
        {/* context prop을 통해 데이터를 객체 형태로 전달 */}
        &lt;Outlet context={{ id, setId }} /&gt;
      &lt;/RegisterWrapper&gt;
    &lt;/RegisterContainer&gt;
  );
};

export default Register;
```

{% endraw %}

### 2. 자식 컴포넌트 (데이터 수신)

자식인 `EmailConfirm`에서는 `useOutletContext` 훅을 사용하여 간편하게 꺼내 씁니다.

```jsx
import { useOutletContext } from 'react-router-dom';

function EmailConfirm() {
  // 부모가 전달한 context를 구조 분해 할당으로 가져옴
  const { id, setId } = useOutletContext();

  const handleChange = () => {
    setId("updated_user");
  };

  return (
    &lt;div&gt;
      &lt;p&gt;현재 접속 아이디: {id}&lt;/p&gt;
      &lt;button onClick={handleChange}&gt;아이디 변경&lt;/button&gt;
    &lt;/div&gt;
  );
}
```

### 3. TypeScript를 사용한다면?

타입 안정성을 위해 인터페이스를 정의하여 제네릭으로 넘겨주는 것이 좋습니다.

```jsx
import { useOutletContext } from 'react-router-dom';

interface UserContextType {
  id: string;
  setId: React.Dispatch&lt;React.SetStateAction&lt;string&gt;&gt;;
}

function Child() {
  const { id } = useOutletContext&lt;UserContextType&gt;();

  return &lt;div&gt;User ID: {id}&lt;/div&gt;;
}
```

---

## 🛠️ 실전 포인트: 삽질 방지 가이드

<div style="background-color: #f8f9fa; border-left: 5px solid #2196F3; padding: 15px; margin-bottom: 20px; color: #91512c;">
    <strong>✅ 언제 써야 할까?</strong><br/>
    전역 상태(Redux, Recoil)로 관리하기에는 너무 지엽적이고, Props로 하나하나 내리기엔 라우터 구조상 불가능할 때 '중첩 라우트 간의 로컬 공유 상태'를 위해 사용하세요.
</div>

<div style="background-color: #fff3cd; border-left: 5px solid #ffc107; padding: 15px; margin-bottom: 20px; color: #91512c;">
    <strong>⚠️ 실수하기 쉬운 부분</strong><br/>
    <code>useOutletContext</code>는 반드시 <code>&lt;Outlet /&gt;</code> 내부에서 렌더링되는 자식 컴포넌트에서만 작동합니다. 라우트 외부 컴포넌트에서 호출하면 <code>undefined</code>를 반환하니 주의하세요!
</div>

<div style="background-color: #fce4ec; border-left: 5px solid #e91e63; padding: 15px; color: #91512c;">
    <strong>🔥 실제 개발 경험</strong><br/>
    회원가입 단계를 1단계(정보입력), 2단계(인증)로 나눌 때 1단계에서 입력한 데이터를 2단계로 넘기기 위해 사용했습니다. 페이지 이동 시에도 부모가 살아있어 데이터 유지가 매우 간편했습니다.
</div>

---

## 📊 방식 비교

| 비교 항목 | Props Drilling | Context API       | useOutletContext |
| :-------- | :------------- | :---------------- | :--------------- |
| 복잡도    | 매우 높음      | 중간              | 낮음             |
| 가독성    | 나쁨           | 설정 코드 필요    | 매우 깔끔        |
| 적용 범위 | 일반 컴포넌트  | 앱 전체/특정 범위 | 중첩 라우트 내부 |

---

## 🏁 마무리하며

`Outlet`과 `useOutletContext`를 조합하면 복잡한 중첩 라우팅 구조에서도 데이터 흐름을 명확하게 관리할 수 있습니다. 무분별한 전역 상태 사용을 줄이고, 리액트 라우터가 제공하는 기능을 100% 활용해 보세요!

궁금하신 점이나 더 좋은 아이디어가 있다면 댓글로 남겨주세요! 😊
