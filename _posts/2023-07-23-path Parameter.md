---
title: "[React] 패스 파라미터(path parameter)란?"
date: 2023-07-23T16:00:000
categories: [React]
tags: [react] #소문자만 가능
---

<style type="text/css">
    .center{
        text-align:center;
        font-size:1.5rem;
    }
    blockquote{
        font-size:1.2rem;
        background: #F4F4F4;
    }
    blockquote>h3{
        margin:0;
    }
</style>

---

## <b style="border-bottom:2px solid gray" class="h2">패스 파라미터(path parameter)란?</b>

<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">정의
</blockquote></h3>

- 패스 파라미터(Path Parameter)는 URL 경로에 변수를 포함하여 주로 동적인 데이터를 전달하는 방법입니다.

<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">특징
</blockquote></h3>

- URL 경로에 데이터를 포함하기 때문에 직관적이고 읽기 쉬운 URL을 제공합니다.
- URL이 더 의미 있고 SEO(Search Engine Optimization)에 유리합니다.
- URL 구조가 깔끔하며, 검색 엔진에서 잘 색인됩니다.
- 캐싱이나 보안에 유리합니다.

<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">사용상황
</blockquote></h3>

- 블로그 포스트, 상품 정보, 사용자 프로필 등의 개별 리소스를 조회하는 경우에 패스 파라미터를 활용합니다.

---

## <b style="border-bottom:2px solid gray" class="h2">패스 파라미터(path parameter) 사용하기</b>

```jsx
import React from "react";
import { BrowserRouter as Router, Route, useNavigate } from "react-router-dom";
import PostDetail from "./components/PostDetail";

const App = () => {
    const navigate = useNavigate();

  return (
    <BrowserRouter>
      <Routes>
        <Route path="/posts/:postId" element={<PostDetail />} />
      </Routes>
    </BrowserRouter>
    <button onClick={()=>navigate("/posts/15")}></button>
  );
};

export default App;
```

- URL 형식: 일반적으로 콜론(:)을 이용하여 URL 경로에 변수를 포함합니다.
  <br/>

```jsx
import { useParams } from "react-router-dom";

const PostDetail = () => {
  const Param = useParams();
  console.log(Param.postId); // "15"

  return <h3>{Param.postId}</h3>;
};

export default App;
```

위와 같이 router를 설정 해주고 button을 누르고 이동해<br/> useParams을 사용하여 미리 정해둔 변수명을 추출할 수 있다

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
