---
title: "[React] React Query 기본 사용법 정리"
date: 2023-07-28T16:19:00
categories: [react]
tags: [react, react-query, tanstack-query, server-state]
description: "React Query의 QueryClientProvider, useQuery, useMutation 기본 사용법과 서버 상태 관리의 장점을 정리했습니다."
custom_style: true
---

## React Query란?

React Query는 서버에서 가져온 데이터를 관리하기 위한 라이브러리입니다.

React의 `useState`와 `useEffect`만으로도 API 데이터를 가져올 수 있지만, 로딩 상태, 에러 상태, 캐싱, 재요청, 동기화까지 직접 처리하려면 코드가 금방 복잡해집니다.

React Query를 사용하면 이런 서버 상태 관리 로직을 훨씬 편하게 다룰 수 있습니다.

---

## 설치하기

최신 TanStack Query 기준으로 설치하면 다음과 같습니다.

```bash
npm install @tanstack/react-query
```

예전 버전의 `react-query`와 패키지 이름이 다르기 때문에 import 경로를 함께 확인해야 합니다.

---

## QueryClientProvider 설정하기

React Query를 사용하려면 앱 최상단에 `QueryClientProvider`를 설정해야 합니다.

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import App from "./App";

const queryClient = new QueryClient();

const Root = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  );
};

export default Root;
```

이 Provider 아래에서 `useQuery`, `useMutation` 같은 훅을 사용할 수 있습니다.

---

## useQuery로 데이터 가져오기

`useQuery`는 서버 데이터를 조회할 때 사용합니다.

```tsx
import { useQuery } from "@tanstack/react-query";

const fetchUser = async () => {
  const response = await fetch("/api/user");
  return response.json();
};

const UserProfile = () => {
  const { data, isLoading, isError } = useQuery({
    queryKey: ["user"],
    queryFn: fetchUser,
  });

  if (isLoading) return <p>불러오는 중...</p>;
  if (isError) return <p>사용자 정보를 불러오지 못했습니다.</p>;

  return <p>{data.name}</p>;
};

export default UserProfile;
```

`queryKey`는 이 요청을 구분하는 키입니다.

`queryFn`은 실제로 데이터를 가져오는 함수입니다.

---

## queryKey가 중요한 이유

React Query는 `queryKey`를 기준으로 캐시를 관리합니다.

```tsx
useQuery({
  queryKey: ["post", postId],
  queryFn: () => fetchPost(postId),
});
```

`postId`가 바뀌면 queryKey도 바뀌고, React Query는 새로운 데이터가 필요하다고 판단합니다.

그래서 상세 페이지나 필터링된 목록처럼 조건이 있는 요청에서는 queryKey에 조건 값을 함께 넣는 것이 중요합니다.

---

## useMutation으로 데이터 변경하기

`useMutation`은 생성, 수정, 삭제처럼 서버 데이터를 변경하는 작업에 사용합니다.

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

const createPost = async (title: string) => {
  const response = await fetch("/api/posts", {
    method: "POST",
    body: JSON.stringify({ title }),
  });

  return response.json();
};

const CreatePostButton = () => {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });

  return (
    <button onClick={() => mutation.mutate("새 게시글")}>
      게시글 생성
    </button>
  );
};

export default CreatePostButton;
```

데이터 변경이 성공한 뒤 `invalidateQueries`를 호출하면 관련 목록을 다시 가져올 수 있습니다.

---

## 자주 사용하는 상태값

| 값 | 의미 |
| --- | --- |
| `data` | 요청 성공 후 받은 데이터 |
| `isLoading` | 첫 요청이 진행 중인지 |
| `isFetching` | 백그라운드 재요청 중인지 |
| `isError` | 에러가 발생했는지 |
| `error` | 발생한 에러 객체 |
| `refetch` | 수동으로 다시 요청하는 함수 |

처음 로딩과 백그라운드 refetch를 구분하고 싶다면 `isLoading`과 `isFetching` 차이를 보면 됩니다.

---

## 마무리

React Query는 서버 상태를 관리할 때 반복되는 코드를 크게 줄여줍니다.

조회는 `useQuery`, 변경은 `useMutation`을 사용하고, `queryKey`를 기준으로 캐시를 관리합니다.

API 데이터가 많은 프로젝트라면 로딩, 에러, 캐싱, 재요청 흐름을 직접 만들기보다 React Query를 사용하는 편이 훨씬 안정적입니다.
