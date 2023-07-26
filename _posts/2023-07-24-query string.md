---
title: "[React] 쿼리 스트링(Query String)이란?"
date: 2023-07-24T18:00:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b>쿼리 스트링(Query String)이란?</b>

<h3><blockquote>정의
</blockquote></h3>

- 쿼리 스트링은 URL의 끝에 물음표(?)를 사용하여 URL에 매개변수를 추가하는 방법입니다.

<h3><blockquote>특징
</blockquote></h3>

- URL에 매개변수를 쉽게 추가하거나 변경할 수 있어 유연하게 데이터를 전달할 수 있습니다.
- URL이 더 길어지고 읽기에 어려워질 수 있으며, SEO에는 적합하지 않을 수 있습니다.
- 쿼리 파라미터는 URL에서 일반적으로 캐싱되지 않습니다. 동일한 URL이라도 쿼리 파라미터의 값이 다르면 서버로 요청이 새로 발생합니다.

<h3><blockquote>사용상황
</blockquote></h3>

- 검색 및 필터링과 같은 다양한 매개변수를 전달할 때 유용합니다.

---

## <b>쿼리 스트링(Query String) 형태</b>

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/235e794f-72de-4bdb-8271-a314519f6648">

- ? : 이후의 작성된 내용은 쿼리 스트링입니다
  - ex) https://example.com/search?search=react
- & : Key와 value가 한쌍이라고 하고 두쌍이상 사용하려면 &로 연결해줘야 한다
  - ex) https://example.com/search?search=react&location=서울

---

## <b>사용 예시</b>

<h3><blockquote>Routing 하기
</blockquote></h3>

패스 파라미터와 달리 router 설정은 따로 필요 없으며 이동할 url만 신경 써주면 된다.<br/>
react-router-dom에 useNavigate 혹은 Link 훅을 사용하여 이동할 수 있다

```jsx
<Link to="/search?search=react" />;

navigate("/search?search=react");
```

<h3><blockquote>값 가져오기
</blockquote></h3>

쿼리 스트링에 값을 가져오려면 useLocation, useSearchParams와 같은 hook을 사용해야 한다.

### useLocation 사용

- useLocation의 search를 통해 쿼리 스트링 값을 가져올 수 있지만 자바스크립을 통해 parsing하는 과정이 필요하다 그렇기에 useSearchParams를 추천한다

// url 주소가 http://localhost:3000/SearchResult?search=react 일 경우의 예시

```jsx
import React from "react";
import { useLocation } from "react-router-dom";

const SearchResult = () => {
  const location = useLocation();
  console.log(location.search); //?search=react

  return <h1>This is SearchResult</h1>;
};

export default SearchResult;
```

### useSearchParams 사용

- 사용 형태는 useState와 흡사하며 get메소드를 통해 쿼리 스트링에 Key값으로 value를 가져 올 수 있다

// url 주소가 http://localhost:3000/SearchResult?search=react 일 경우의 예시

```jsx
import React from "react";
import { useSearchParams } from "react-router-dom";

const SearchResult = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const search = searchParams.get("search");

  console.log(search); //react

  return <h1>This is SearchResult</h1>;
};

export default SearchResult;
```

- searchParams 메소드
  - searchParams.get(key) : 특정 key의 value 가져오기
  - searchParams.getAll(key) : 특정 key의 모든 value 가져오기

---

## <b>마치며</b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
