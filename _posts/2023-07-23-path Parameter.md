---
title: "[React] Path Parameter 이해하기"
date: 2023-07-23T16:00:00
categories: [react]
tags: [react, react-router-dom, path-parameter, useparams]
description: "React Router에서 path parameter를 정의하고 useParams로 값을 읽는 방법을 정리했습니다."
custom_style: true
---

## Path Parameter란?

Path Parameter는 URL 경로 안에 들어가는 동적인 값입니다.

예를 들어 게시글 상세 페이지 URL이 아래처럼 생겼다고 해볼게요.

```txt
/posts/15
```

여기서 `15`는 게시글 id입니다.

이처럼 URL 경로 일부를 변수처럼 사용하는 값을 path parameter라고 부릅니다.

---

## 언제 사용할까요?

Path Parameter는 특정 리소스 하나를 조회할 때 자주 사용합니다.

예를 들면 이런 페이지입니다.

- 게시글 상세: `/posts/15`
- 상품 상세: `/products/3`
- 사용자 프로필: `/users/taewok`

URL만 봐도 어떤 리소스를 보고 있는지 알 수 있어 직관적입니다.

---

## 라우트 정의하기

React Router에서는 콜론 `:`을 사용해 path parameter를 정의합니다.

```tsx
import { BrowserRouter, Route, Routes } from "react-router-dom";
import PostDetail from "./PostDetail";

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/posts/:postId" element={<PostDetail />} />
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

`/posts/:postId`에서 `:postId`가 동적인 값입니다.

`/posts/15`로 접근하면 `postId` 값은 `"15"`가 됩니다.

---

## useParams로 값 읽기

자식 컴포넌트에서는 `useParams`를 사용해 path parameter 값을 읽을 수 있습니다.

```tsx
import { useParams } from "react-router-dom";

const PostDetail = () => {
  const { postId } = useParams();

  return <h1>게시글 ID: {postId}</h1>;
};

export default PostDetail;
```

`useParams`가 반환하는 값은 문자열이거나 `undefined`일 수 있습니다.

숫자로 사용해야 한다면 직접 변환해야 합니다.

```tsx
const id = Number(postId);
```

---

## navigate로 이동하기

버튼 클릭으로 특정 게시글 상세 페이지로 이동할 수도 있습니다.

```tsx
import { useNavigate } from "react-router-dom";

const PostCard = ({ postId }: { postId: number }) => {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate(`/posts/${postId}`)}>
      상세 보기
    </button>
  );
};

export default PostCard;
```

템플릿 문자열을 사용하면 동적인 id를 URL에 쉽게 넣을 수 있습니다.

---

## Query String과의 차이

Path Parameter는 특정 리소스를 나타낼 때 잘 어울립니다.

```txt
/posts/15
```

Query String은 검색어, 필터, 정렬처럼 옵션을 나타낼 때 잘 어울립니다.

```txt
/posts?keyword=react&sort=latest
```

리소스 자체를 구분하는 값이라면 path parameter를, 조회 조건이라면 query string을 사용하는 식으로 구분하면 이해하기 쉽습니다.

---

## 마무리

Path Parameter는 URL 경로 안에 동적인 값을 넣는 방식입니다.

React Router에서는 `:postId`처럼 콜론을 붙여 정의하고, 컴포넌트에서는 `useParams`로 값을 읽습니다.

상세 페이지처럼 특정 데이터 하나를 조회하는 화면에서 자주 사용되는 패턴입니다.
