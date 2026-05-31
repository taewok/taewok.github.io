---
title: "[Error] No QueryClient set, use QueryClientProvider to set one 해결하기"
date: 2023-06-22T12:40:00
categories: [error]
tags: [error, react-query, tanstack-query, react]
description: "React Query에서 QueryClientProvider를 설정하지 않았거나 패키지 import가 섞였을 때 발생하는 에러를 정리했습니다."
custom_style: true
---

## 발생한 에러

React Query를 사용하던 중 다음 에러가 발생했습니다.

```txt
No QueryClient set, use QueryClientProvider to set one
```

이 에러는 React Query 훅이 사용할 `QueryClient`를 찾지 못할 때 발생합니다.

대표적으로 `useQuery`, `useMutation`을 사용하는 컴포넌트가 `QueryClientProvider` 바깥에 있을 때 볼 수 있습니다.

---

## 기본 해결 방법

앱 최상단을 `QueryClientProvider`로 감싸야 합니다.

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

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

이렇게 하면 `App` 아래에서 사용하는 React Query 훅들이 같은 `queryClient`를 사용할 수 있습니다.

---

## import 경로가 섞여도 발생할 수 있어요

제가 만난 문제는 Provider가 없는 것이 아니라 import 경로가 섞인 경우였습니다.

React Query v3에서는 보통 `react-query`에서 import했습니다.

```tsx
import { useQuery } from "react-query";
```

하지만 v4부터는 패키지 이름이 TanStack Query로 바뀌면서 `@tanstack/react-query`를 사용합니다.

```tsx
import { useQuery } from "@tanstack/react-query";
```

Provider는 `@tanstack/react-query`에서 가져왔는데, 훅은 `react-query`에서 가져오면 서로 다른 context를 바라보게 됩니다.

그래서 Provider로 감쌌는데도 QueryClient가 없다는 에러가 날 수 있습니다.

---

## 확인해야 할 부분

에러가 발생하면 아래를 순서대로 확인해보면 좋습니다.

1. 앱 최상단이 `QueryClientProvider`로 감싸져 있는지 확인합니다.
2. `QueryClientProvider`에 `client={queryClient}`가 전달되어 있는지 확인합니다.
3. `useQuery`, `useMutation` import 경로가 Provider와 같은 패키지인지 확인합니다.
4. `react-query`와 `@tanstack/react-query`가 동시에 설치되어 있지 않은지 확인합니다.

특히 v3에서 v4로 마이그레이션한 프로젝트라면 import 경로가 남아 있을 가능성이 큽니다.

---

## 패키지 정리하기

TanStack Query v4를 사용한다면 `react-query` import를 모두 `@tanstack/react-query`로 바꿔줍니다.

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from "@tanstack/react-query";
```

필요 없다면 예전 패키지는 제거합니다.

```bash
npm uninstall react-query
```

---

## 마무리

`No QueryClient set` 에러는 대부분 Provider 설정 문제이지만, import 경로가 섞인 경우에도 발생할 수 있습니다.

Provider를 제대로 감쌌는데도 에러가 계속된다면 `react-query`와 `@tanstack/react-query`를 함께 사용하고 있지 않은지 확인해보는 것이 좋습니다.

React Query는 Provider와 훅이 같은 패키지의 context를 공유해야 정상적으로 동작합니다.
