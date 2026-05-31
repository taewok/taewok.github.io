---
title: "[Next.js] React Query Hydration으로 서버에서 미리 가져온 데이터 재사용하기"
date: 2026-05-02T10:45:00Z
categories: [frontend]
tags: [nextjs, react-query, tanstack-query, hydration, server-components]
description: "Next.js App Router에서 TanStack Query의 prefetchQuery, dehydrate, HydrationBoundary를 사용해 서버에서 가져온 데이터를 클라이언트 캐시로 전달하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 서버에서 가져온 데이터를 클라이언트에서도 다시 요청해야 할까요?

Next.js App Router를 사용하면 서버 컴포넌트에서 데이터를 미리 가져온 뒤 화면을 렌더링할 수 있습니다. 첫 화면에 필요한 데이터를 서버에서 준비할 수 있기 때문에 초기 로딩 경험을 개선하는 데 유리합니다.

하지만 클라이언트 컴포넌트에서 같은 데이터를 `useQuery`로 다시 사용해야 한다면 고민이 생깁니다.

서버에서는 이미 데이터를 가져왔는데, 클라이언트에서 `useQuery`가 마운트되면서 같은 API를 다시 호출할 수 있기 때문입니다. 사용자는 이미 렌더링된 화면을 보고 있지만, 네트워크 탭에는 중복 요청이 찍히고 React Query 캐시도 서버에서 가져온 데이터를 알지 못하는 상황이 생길 수 있어요.

이 문제를 해결하기 위해 사용한 방식이 **React Query Hydration**입니다. 핵심은 서버에서 미리 채운 Query Cache를 직렬화해서 클라이언트로 넘기고, 클라이언트의 `QueryClient`가 그 캐시를 자연스럽게 이어받도록 만드는 것입니다. 이렇게 해두면 첫 화면은 서버에서 빠르게 준비하고, 이후 클라이언트에서는 같은 데이터를 캐시 기반으로 재사용할 수 있습니다.

---

## 💡 Hydration이란?

일반적으로 Hydration은 서버에서 만들어진 HTML에 클라이언트의 JavaScript 이벤트와 상태를 연결하는 과정을 의미합니다.

TanStack Query에서의 Hydration은 조금 더 구체적입니다. 서버에서 `prefetchQuery`로 미리 채운 Query Cache를 `dehydrate`로 직렬화하고, 클라이언트에서 `HydrationBoundary`를 통해 다시 Query Cache로 복원하는 과정입니다.

흐름으로 보면 다음과 같습니다.

1. 서버에서 `QueryClient`를 만듭니다.
2. 서버에서 `prefetchQuery`로 필요한 데이터를 미리 가져옵니다.
3. `dehydrate`로 서버 캐시를 직렬화합니다.
4. 클라이언트에서 `HydrationBoundary`가 직렬화된 캐시를 복원합니다.
5. 클라이언트 컴포넌트는 `useQuery`를 호출해도 이미 캐시에 있는 데이터를 즉시 사용할 수 있습니다.

이렇게 하면 서버 렌더링의 장점과 React Query의 클라이언트 캐싱 장점을 함께 가져갈 수 있습니다.

---

## 🛠️ 1. QueryClientProvider 설정하기

React Query를 사용하려면 클라이언트 영역에서 `QueryClientProvider`가 먼저 필요합니다.

App Router에서는 보통 `app/providers.tsx` 같은 클라이언트 컴포넌트를 만들고, 루트 레이아웃에서 감싸는 방식으로 설정합니다.

```tsx
// app/providers.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useState } from "react";

export default function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000,
          },
        },
      }),
  );

  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx
import Providers from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

여기서 `QueryClient`를 `useState`의 초기 함수로 만드는 이유는 클라이언트 렌더링 중에 매번 새 인스턴스가 생기지 않도록 하기 위해서입니다.

---

## 🧩 2. 서버에서 QueryClient 만들기

서버 컴포넌트에서 데이터를 미리 가져오려면 서버 전용 `QueryClient`도 필요합니다.

페이지나 서버 컴포넌트 안에서 직접 새 `QueryClient`를 만들어도 됩니다. 다만 여러 서버 컴포넌트가 같은 요청 안에서 동일한 `QueryClient`를 공유해야 한다면 React의 `cache`를 사용해 요청 단위로 재사용할 수 있습니다.

```tsx
// lib/getQueryClient.ts
import { QueryClient } from "@tanstack/react-query";
import { cache } from "react";

export const getQueryClient = cache(
  () =>
    new QueryClient({
      defaultOptions: {
        queries: {
          staleTime: 60 * 1000,
        },
      },
    }),
);
```

`cache`는 서버 컴포넌트 환경에서 같은 입력에 대한 결과를 메모이즈합니다. React는 서버 요청이 끝나면 이 캐시를 비우기 때문에 사용자 간 데이터가 섞이지 않습니다.

다만 모든 상황에서 `cache`가 필수는 아닙니다. 한 페이지에서 한 번만 prefetch한다면 해당 서버 컴포넌트 안에서 `new QueryClient()`를 만들어도 충분합니다. 여러 서버 컴포넌트가 같은 요청 안에서 캐시를 공유해야 할 때 `cache`를 고려하면 됩니다.

---

## ⚡ 3. 서버 컴포넌트에서 prefetch 후 HydrationBoundary로 감싸기

이제 서버 컴포넌트에서 데이터를 미리 가져오고, 그 캐시를 클라이언트로 전달해보겠습니다.

```tsx
// app/posts/page.tsx
import { dehydrate, HydrationBoundary } from "@tanstack/react-query";
import { getQueryClient } from "@/lib/getQueryClient";
import PostList from "./_components/PostList";
import { fetchPosts } from "@/services/post";

export default async function PostsPage() {
  const queryClient = getQueryClient();

  await queryClient.prefetchQuery({
    queryKey: ["posts"],
    queryFn: fetchPosts,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <PostList />
    </HydrationBoundary>
  );
}
```

여기서 중요한 것은 서버와 클라이언트에서 사용하는 `queryKey`와 `queryFn`이 같은 의미를 가져야 한다는 점입니다.

서버에서 `["posts"]`라는 키로 데이터를 미리 넣어두었는데, 클라이언트에서 다른 키를 사용하면 React Query는 같은 데이터라고 판단하지 못합니다. 그러면 Hydration을 해도 클라이언트에서 다시 요청하게 됩니다.

---

## 👀 4. 클라이언트 컴포넌트에서는 평소처럼 useQuery 사용하기

`PostList`는 클라이언트 컴포넌트입니다. 이 컴포넌트에서는 평소처럼 `useQuery`를 사용하면 됩니다.

```tsx
// app/posts/_components/PostList.tsx
"use client";

import { useQuery } from "@tanstack/react-query";
import { fetchPosts } from "@/services/post";

export default function PostList() {
  const { data: posts, isLoading } = useQuery({
    queryKey: ["posts"],
    queryFn: fetchPosts,
  });

  if (isLoading) {
    return <div>게시글을 불러오는 중입니다.</div>;
  }

  return (
    <ul>
      {posts?.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Hydration이 적용되어 있다면 `useQuery`는 서버에서 미리 채워진 캐시를 먼저 사용합니다. 그래서 첫 렌더링에서 로딩 상태를 거치지 않고 데이터를 바로 보여줄 수 있습니다.

물론 `staleTime`이 지나거나 사용자가 다시 포커스했을 때처럼 React Query의 정책에 따라 refetch가 일어날 수 있습니다. Hydration은 클라이언트 요청을 영원히 막는 기능이 아니라, **서버에서 이미 가져온 초기 데이터를 클라이언트 캐시의 출발점으로 넘겨주는 기능**에 가깝습니다.

---

## ⚠️ 실수하기 쉬운 부분

### 1. staleTime을 설정하지 않아 바로 refetch되는 경우

React Query의 기본 `staleTime`은 `0`입니다. 이 상태에서는 Hydration으로 데이터를 넘겨도 클라이언트가 데이터를 즉시 stale 상태로 판단할 수 있습니다.

그러면 화면은 빠르게 보이더라도 마운트 직후 다시 API 요청이 발생합니다. 초기 중복 요청을 줄이고 싶다면 서비스 특성에 맞는 `staleTime`을 설정하는 것이 좋습니다.

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,
    },
  },
});
```

### 2. 서버와 클라이언트의 queryKey가 다른 경우

서버에서 `["posts"]`로 prefetch했는데 클라이언트에서 `["post-list"]`로 조회하면 캐시가 연결되지 않습니다.

동일한 데이터라면 query key를 상수나 query option 함수로 분리해 서버와 클라이언트가 같은 정의를 사용하도록 만드는 편이 안전합니다.

```ts
export const postQueries = {
  list: () => ({
    queryKey: ["posts"],
    queryFn: fetchPosts,
  }),
};
```

```tsx
await queryClient.prefetchQuery(postQueries.list());

const { data } = useQuery(postQueries.list());
```

### 3. QueryClient를 전역 싱글톤으로 두는 경우

브라우저에서는 하나의 `QueryClient`를 오래 유지하는 것이 자연스럽습니다. 하지만 서버에서는 전역으로 하나의 `QueryClient`를 공유하면 사용자 요청 간 캐시가 섞일 수 있습니다.

서버에서는 요청마다 독립적인 `QueryClient`를 만들거나, React의 `cache`처럼 요청 단위로 안전하게 재사용되는 방식을 사용해야 합니다.

---

## 📊 Props 전달 방식과 Hydration 방식 비교

서버 컴포넌트에서 가져온 데이터를 클라이언트 컴포넌트에 props로 넘기는 방식도 가능합니다. 간단한 화면이라면 오히려 이 방식이 더 명확할 수 있습니다.

하지만 클라이언트에서 같은 데이터를 React Query로 계속 관리해야 한다면 Hydration 방식이 더 잘 맞습니다.

| 항목 | Props 직접 전달 방식 | React Query Hydration 방식 |
| :--- | :--- | :--- |
| 초기 렌더링 | 서버 데이터로 바로 렌더링 | 서버 데이터로 바로 렌더링 |
| 클라이언트 캐시 | 별도로 직접 연결해야 함 | Query Cache로 자연스럽게 이어짐 |
| refetch | 직접 구현 필요 | React Query 정책으로 관리 |
| 하위 컴포넌트 접근 | props 전달이 필요 | 같은 `queryKey`로 접근 가능 |
| 적합한 상황 | 단순한 읽기 화면 | 캐싱, 갱신, 낙관적 업데이트 필요 |

정리하면, props 전달은 단순하고 직관적입니다. 반면 Hydration은 초기 서버 데이터와 클라이언트 캐시를 이어주는 구조라서, 페이지 이동 후 재방문하거나 여러 컴포넌트가 같은 데이터를 공유하는 화면에 더 유리합니다.

---

## ✅ 언제 이 방식을 쓰면 좋을까요?

React Query Hydration은 다음 상황에서 특히 유용했습니다.

- 첫 화면에서 로딩 스피너보다 실제 콘텐츠를 바로 보여주고 싶을 때
- 클라이언트에서도 같은 데이터를 `useQuery`로 계속 관리해야 할 때
- 여러 컴포넌트가 같은 `queryKey`의 데이터를 공유해야 할 때
- 서버에서 미리 가져온 데이터를 React Query 캐시의 초기값으로 사용하고 싶을 때
- 페이지 이동 후 돌아왔을 때 캐시, refetch, stale 상태를 React Query 정책으로 관리하고 싶을 때

반대로 서버에서 한 번 보여주기만 하면 되는 정적인 데이터라면 굳이 Hydration까지 사용할 필요는 없습니다. 서버 컴포넌트에서 `fetch`하고 props로 내려주는 방식이 더 단순할 수 있습니다.

---

## 🏁 마치며

이번 작업의 핵심은 데이터를 어디서 가져오느냐보다, **서버에서 가져온 데이터를 클라이언트가 어떻게 이어받게 할 것인가**였습니다.

Next.js App Router에서는 서버 컴포넌트로 초기 데이터를 준비하기 쉽습니다. 여기에 React Query Hydration을 더하면 클라이언트 컴포넌트도 같은 데이터를 캐시에서 바로 사용할 수 있습니다.

`prefetchQuery`, `dehydrate`, `HydrationBoundary`는 처음 보면 단계가 많아 보이지만 역할은 명확합니다.

- `prefetchQuery`: 서버에서 Query Cache를 미리 채웁니다.
- `dehydrate`: 서버 Query Cache를 직렬화합니다.
- `HydrationBoundary`: 직렬화된 캐시를 클라이언트 Query Cache로 복원합니다.

이 구조를 적용한 뒤에는 서버 렌더링의 빠른 첫 화면과 React Query의 캐싱, refetch, 상태 관리 장점을 함께 사용할 수 있었습니다. 복잡한 목록 페이지나 대시보드처럼 같은 데이터를 여러 곳에서 재사용하는 화면이라면 충분히 적용할 만한 패턴이었습니다.
