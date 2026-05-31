---
title: "[React] Query String 이해하기"
date: 2023-07-24T18:00:00
categories: [react]
tags: [react, react-router-dom, query-string, use-search-params]
description: "Query String의 의미와 React Router에서 useSearchParams로 값을 읽고 변경하는 방법을 정리했습니다."
custom_style: true
---

## Query String이란?

Query String은 URL 뒤에 `?`를 붙이고 key-value 형태의 값을 전달하는 방식입니다.

```txt
/search?keyword=react
```

여기서 `keyword=react`가 query string입니다.

검색어, 필터, 정렬 조건처럼 페이지 조회 조건을 URL에 담을 때 자주 사용합니다.

---

## 기본 형태

Query String은 `?` 뒤에 작성합니다.

```txt
/search?keyword=react
```

여러 값을 함께 전달할 때는 `&`로 연결합니다.

```txt
/search?keyword=react&page=2&sort=latest
```

이 URL에는 다음 값들이 들어 있습니다.

| key | value |
| --- | --- |
| `keyword` | `react` |
| `page` | `2` |
| `sort` | `latest` |

---

## 언제 사용할까요?

Query String은 화면의 조회 조건을 URL에 남기고 싶을 때 유용합니다.

예를 들면 이런 상황입니다.

- 검색어
- 페이지 번호
- 정렬 방식
- 필터 조건
- 탭 상태

URL에 조건이 남아 있으면 새로고침해도 같은 화면을 유지하기 쉽고, 다른 사람에게 링크를 공유하기도 좋습니다.

---

## useSearchParams로 값 읽기

React Router에서는 `useSearchParams`를 사용해 query string을 읽을 수 있습니다.

```tsx
import { useSearchParams } from "react-router-dom";

const SearchResult = () => {
  const [searchParams] = useSearchParams();

  const keyword = searchParams.get("keyword");
  const page = searchParams.get("page");

  return (
    <div>
      <p>검색어: {keyword}</p>
      <p>페이지: {page}</p>
    </div>
  );
};

export default SearchResult;
```

`searchParams.get("keyword")`는 `keyword` 값 하나를 가져옵니다.

값이 없으면 `null`이 반환됩니다.

---

## Query String 변경하기

`setSearchParams`를 사용하면 query string을 변경할 수 있습니다.

```tsx
import { useSearchParams } from "react-router-dom";

const SearchPage = () => {
  const [searchParams, setSearchParams] = useSearchParams();

  const handleSearch = () => {
    setSearchParams({
      keyword: "react",
      page: "1",
    });
  };

  return <button onClick={handleSearch}>검색</button>;
};

export default SearchPage;
```

버튼을 누르면 URL이 아래처럼 바뀝니다.

```txt
/search?keyword=react&page=1
```

---

## useLocation으로도 읽을 수 있어요

`useLocation`을 사용하면 query string 문자열 자체를 확인할 수 있습니다.

```tsx
import { useLocation } from "react-router-dom";

const SearchResult = () => {
  const location = useLocation();

  console.log(location.search); // "?keyword=react"

  return <h1>검색 결과</h1>;
};

export default SearchResult;
```

다만 이 경우 직접 파싱해야 하므로, 일반적으로는 `useSearchParams`가 더 편합니다.

---

## Path Parameter와의 차이

Path Parameter는 특정 리소스를 나타낼 때 사용합니다.

```txt
/posts/15
```

Query String은 조회 조건이나 옵션을 나타낼 때 사용합니다.

```txt
/posts?keyword=react&sort=latest
```

상세 페이지 id처럼 리소스 자체를 구분하는 값은 path parameter가 잘 맞고, 검색 조건처럼 바뀔 수 있는 값은 query string이 잘 맞습니다.

---

## 마무리

Query String은 URL에 검색 조건이나 필터 상태를 담을 때 유용합니다.

React Router에서는 `useSearchParams`를 사용하면 query string을 쉽게 읽고 변경할 수 있습니다.

검색, 필터, 페이지네이션처럼 URL에 상태를 남겨야 하는 화면이라면 query string을 활용해보면 좋습니다.
