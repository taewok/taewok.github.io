---
title: "[React Query] QueryClient 완벽 이해하기: prefetchQuery부터 캐시 조작까지"
date: 2026-09-21T00:30:00Z
categories: [frontend]
tags: [react, react-query, tanstack-query, queryclient, prefetch, cache, optimistic-update]
description: "TanStack Query의 QueryClient를 이용해 prefetchQuery, prefetchInfiniteQuery, query, infiniteQuery, setQueryData, getQueryData, invalidateQueries, cancelQueries를 어떻게 사용하고 UX를 개선하는지 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: React Query는 useQuery만 쓰는 도구가 아니에요

React Query를 처음 사용할 때는 보통 `useQuery`와 `useMutation`부터 익힙니다.

```tsx
const { data, isPending } = useQuery({
  queryKey: ["posts"],
  queryFn: fetchPosts,
});
```

```tsx
const mutation = useMutation({
  mutationFn: createPost,
});
```

이 정도만 써도 로딩, 에러, 캐싱, 재요청을 훨씬 편하게 다룰 수 있습니다.

하지만 어느 순간 이런 고민이 생깁니다.

```txt
상세 페이지로 들어가기 전에 데이터를 미리 받아둘 수 없을까?
다음 페이지 데이터를 미리 준비할 수 없을까?
수정 성공 후 목록을 바로 바꿔 보여줄 수 없을까?
서버 응답을 기다리기 전에 UI를 먼저 바꿀 수 없을까?
캐시를 직접 읽거나 수정해야 할 때는 어떻게 해야 할까?
```

이때 필요한 것이 `QueryClient`입니다.

`QueryClient`는 React Query의 캐시를 직접 다루는 객체입니다.

`useQuery`가 컴포넌트에서 데이터를 구독하는 Hook이라면, `QueryClient`는 그 뒤쪽에 있는 캐시 저장소를 조작하는 도구에 가깝습니다.

이번 글에서는 `prefetchQuery`, `prefetchInfiniteQuery`, `setQueryData`, `getQueryData`, `invalidateQueries`, `cancelQueries` 같은 기능을 왜 쓰는지, 어떤 상황에서 사용자 경험을 개선할 수 있는지 정리해보겠습니다.

---

## QueryClient는 무엇을 하는 객체일까?

React Query를 설정할 때 보통 `QueryClient`를 만듭니다.

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

export function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Routes />
    </QueryClientProvider>
  );
}
```

이 `queryClient`는 Query Cache와 Mutation Cache를 가지고 있습니다.

쉽게 말하면 아래 일을 담당합니다.

```txt
서버 데이터 캐시 저장
queryKey 기준으로 데이터 찾기
캐시된 데이터 직접 읽기
캐시된 데이터 직접 수정하기
특정 query 무효화하기
필요한 데이터를 미리 받아두기
진행 중인 요청 취소하기
```

컴포넌트 안에서는 `useQueryClient`로 접근합니다.

```tsx
import { useQueryClient } from "@tanstack/react-query";

function PostButton() {
  const queryClient = useQueryClient();

  return (
    <button
      type="button"
      onClick={() => {
        queryClient.invalidateQueries({ queryKey: ["posts"] });
      }}
    >
      게시글 새로고침
    </button>
  );
}
```

핵심은 이것입니다.

```txt
useQuery
→ 캐시를 구독해서 화면에 보여준다

QueryClient
→ 캐시를 직접 읽고, 채우고, 수정하고, 무효화한다
```

---

## 먼저 queryKey를 잘 설계해야 한다

QueryClient를 잘 쓰려면 먼저 `queryKey`가 안정적으로 설계되어 있어야 합니다.

예를 들어 게시글 목록과 상세를 이렇게 나눌 수 있습니다.

```tsx
const postKeys = {
  all: ["posts"] as const,
  lists: () => [...postKeys.all, "list"] as const,
  list: (filters: PostFilters) => [...postKeys.lists(), filters] as const,
  details: () => [...postKeys.all, "detail"] as const,
  detail: (postId: string) => [...postKeys.details(), postId] as const,
};
```

이렇게 해두면 QueryClient 조작이 명확해집니다.

```tsx
queryClient.invalidateQueries({
  queryKey: postKeys.lists(),
});
```

```tsx
queryClient.setQueryData(postKeys.detail(post.id), post);
```

대충 문자열을 여기저기 직접 쓰면 나중에 캐시 조작이 어려워집니다.

```tsx
queryClient.invalidateQueries({ queryKey: ["post"] });
queryClient.invalidateQueries({ queryKey: ["posts"] });
queryClient.invalidateQueries({ queryKey: ["post-list"] });
```

이런 식으로 흩어지면 어떤 캐시를 무효화해야 하는지 헷갈립니다.

QueryClient를 다루는 글이지만, 출발점은 결국 `queryKey`입니다.

```txt
QueryClient 조작의 정확도는 queryKey 설계에 달려 있다.
```

---

## prefetch란 무엇일까?

prefetch는 사용자가 실제로 데이터를 필요로 하기 전에 미리 받아두는 작업입니다.

예를 들어 게시글 목록에서 상세 링크에 마우스를 올렸다고 해보겠습니다.

```txt
사용자가 상세 링크에 hover
→ 상세 데이터를 미리 요청
→ 캐시에 저장
→ 사용자가 클릭해서 상세 페이지 진입
→ 이미 캐시에 있으므로 더 빠르게 표시
```

사용자는 클릭 후 로딩을 덜 보게 됩니다.

이게 prefetch의 핵심입니다.

```txt
사용자가 곧 필요로 할 가능성이 높은 데이터를
조금 먼저 받아서 캐시에 넣어둔다.
```

React Query v4나 많은 기존 글에서는 보통 `prefetchQuery`를 사용했습니다.

```tsx
await queryClient.prefetchQuery({
  queryKey: ["post", postId],
  queryFn: () => fetchPost(postId),
});
```

하지만 최신 TanStack Query 문서에서는 `queryClient.query`를 사용하는 흐름을 권장합니다.

```tsx
import { noop } from "@tanstack/react-query";

void queryClient
  .query({
    queryKey: ["post", postId],
    queryFn: () => fetchPost(postId),
    staleTime: 60 * 1000,
  })
  .catch(noop);
```

최신 문서 기준으로는 `prefetchQuery`, `prefetchInfiniteQuery`, `ensureQueryData` 같은 기존 imperative 메서드는 다음 메이저 버전에서 제거될 예정입니다.

그래서 새로 학습한다면 아래처럼 이해하는 것이 좋습니다.

| 기존에 많이 보던 방식 | 최신 권장 흐름 |
| --- | --- |
| `prefetchQuery` | `queryClient.query(...).catch(noop)` |
| `prefetchInfiniteQuery` | `queryClient.infiniteQuery(...).catch(noop)` |
| `ensureQueryData` | `queryClient.query({ ..., staleTime: "static" })` |
| `fetchQuery` | `queryClient.query(...)` |
| `fetchInfiniteQuery` | `queryClient.infiniteQuery(...)` |

다만 실무 코드베이스에서는 아직 `prefetchQuery`를 많이 볼 수 있습니다.

그래서 이 글에서는 기존 이름도 함께 설명하되, 예제는 최신 흐름을 중심으로 보겠습니다.

---

## queryClient.query로 상세 페이지 미리 불러오기

게시글 목록에서 상세 페이지로 이동하는 상황을 생각해봅시다.

```tsx
type Post = {
  id: string;
  title: string;
  content: string;
};

async function fetchPost(postId: string): Promise<Post> {
  const response = await fetch(`/api/posts/${postId}`);

  if (!response.ok) {
    throw new Error("게시글을 불러오지 못했습니다.");
  }

  return response.json();
}
```

목록 아이템에 hover나 focus가 들어왔을 때 상세 데이터를 미리 받아올 수 있습니다.

```tsx
import { noop, useQueryClient } from "@tanstack/react-query";

function PostLink({ post }: { post: Post }) {
  const queryClient = useQueryClient();

  const prefetchPost = () => {
    void queryClient
      .query({
        queryKey: ["posts", "detail", post.id],
        queryFn: () => fetchPost(post.id),
        staleTime: 60 * 1000,
      })
      .catch(noop);
  };

  return (
    <a
      href={`/posts/${post.id}`}
      onMouseEnter={prefetchPost}
      onFocus={prefetchPost}
    >
      {post.title}
    </a>
  );
}
```

`staleTime`을 넣은 이유가 중요합니다.

prefetch를 했는데 바로 stale 상태가 되면, 상세 페이지에서 다시 요청이 나갈 수 있습니다.

```tsx
staleTime: 60 * 1000
```

이렇게 하면 1분 동안은 fresh한 데이터로 간주할 수 있습니다.

그리고 상세 페이지에서는 같은 `queryKey`를 사용해야 합니다.

```tsx
function PostDetailPage({ postId }: { postId: string }) {
  const { data, isPending } = useQuery({
    queryKey: ["posts", "detail", postId],
    queryFn: () => fetchPost(postId),
    staleTime: 60 * 1000,
  });

  if (isPending) {
    return <p>게시글을 불러오는 중...</p>;
  }

  return <article>{data.title}</article>;
}
```

prefetch와 `useQuery`의 `queryKey`가 다르면 같은 캐시를 쓰지 못합니다.

```txt
prefetch queryKey
→ ["posts", "detail", postId]

useQuery queryKey
→ ["post", postId]

결과
→ 서로 다른 캐시로 인식
→ prefetch 효과 없음
```

prefetch가 제대로 동작하려면 `queryKey`, `queryFn`, `staleTime` 전략이 함께 맞아야 합니다.

---

## prefetchQuery를 쓰는 기존 코드와 비교하기

기존 코드에서는 아래처럼 많이 작성합니다.

```tsx
await queryClient.prefetchQuery({
  queryKey: ["posts", "detail", postId],
  queryFn: () => fetchPost(postId),
  staleTime: 60 * 1000,
});
```

이 코드는 "데이터를 미리 받아 캐시에 넣는다"는 목적을 분명하게 보여줍니다.

최신 문서의 `queryClient.query` 방식은 조금 더 일반적인 API입니다.

```tsx
await queryClient.query({
  queryKey: ["posts", "detail", postId],
  queryFn: () => fetchPost(postId),
  staleTime: 60 * 1000,
});
```

차이는 에러 처리 방식에서 체감됩니다.

`query`는 데이터를 반환하고 에러를 throw할 수 있습니다.

하지만 hover prefetch처럼 실패해도 화면을 깨면 안 되는 작업에서는 에러를 삼키는 편이 자연스럽습니다.

```tsx
void queryClient
  .query({
    queryKey: ["posts", "detail", postId],
    queryFn: () => fetchPost(postId),
    staleTime: 60 * 1000,
  })
  .catch(noop);
```

이렇게 하면 prefetch가 실패해도 사용자가 실제 상세 페이지에 들어갔을 때 `useQuery`가 다시 요청할 수 있습니다.

정리하면 이렇습니다.

```txt
실패해도 괜찮은 미리 불러오기
→ void queryClient.query(...).catch(noop)

라우트 진입에 반드시 필요한 데이터
→ await queryClient.query(...) 후 에러 처리
```

---

## prefetch는 언제 사용자 경험을 좋게 만들까?

prefetch는 아무 데이터나 미리 받는 기술이 아닙니다.

잘 맞는 상황은 이렇습니다.

- 사용자가 곧 이동할 가능성이 높다
- 데이터 크기가 너무 크지 않다
- 미리 받아도 낭비가 크지 않다
- 클릭 후 로딩 시간이 사용자 경험에 영향을 준다
- 같은 `queryKey`로 이후 화면에서 재사용된다

예를 들면 다음 상황이 좋습니다.

```txt
목록 → 상세 페이지
탭 hover → 탭 내용
다음 페이지 버튼 근처 → 다음 페이지 데이터
검색 결과 카드 focus → 상세 데이터
라우터 loader → 라우트 진입 데이터
```

반대로 이런 경우에는 조심해야 합니다.

- hover만 해도 너무 많은 요청이 나간다
- 데이터가 크고 사용자가 실제로 열 가능성이 낮다
- 권한에 따라 민감한 데이터가 달라진다
- 모바일에서 hover가 없어 효과가 제한적이다
- prefetch 때문에 중요한 요청이 밀린다

prefetch는 "빠르게 보이기"를 위해 네트워크를 조금 앞당겨 쓰는 전략입니다.

그래서 요청 낭비와 UX 개선 사이의 균형을 봐야 합니다.

---

## queryClient.infiniteQuery로 무한 쿼리 미리 불러오기

무한 스크롤이나 커서 기반 목록에서는 `useInfiniteQuery`를 사용합니다.

```tsx
const postsQuery = useInfiniteQuery({
  queryKey: ["posts", "infinite"],
  queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
  initialPageParam: null,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});
```

이런 infinite query도 미리 채울 수 있습니다.

기존에는 `prefetchInfiniteQuery`를 사용했습니다.

```tsx
await queryClient.prefetchInfiniteQuery({
  queryKey: ["posts", "infinite"],
  queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
  initialPageParam: null,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});
```

최신 흐름에서는 `queryClient.infiniteQuery`를 사용합니다.

```tsx
import { noop } from "@tanstack/react-query";

void queryClient
  .infiniteQuery({
    queryKey: ["posts", "infinite"],
    queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  })
  .catch(noop);
```

기본적으로는 첫 페이지를 prefetch합니다.

여러 페이지를 미리 받고 싶다면 `pages` 옵션과 `getNextPageParam`을 함께 사용할 수 있습니다.

```tsx
void queryClient
  .infiniteQuery({
    queryKey: ["posts", "infinite"],
    queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    pages: 3,
  })
  .catch(noop);
```

이렇게 하면 처음 3페이지를 미리 받아둘 수 있습니다.

다만 무한 목록에서 여러 페이지를 미리 받는 것은 네트워크 비용이 큽니다.

실무에서는 보통 아래 기준을 봅니다.

```txt
첫 화면에서 바로 여러 페이지가 필요한가?
사용자가 거의 항상 아래로 스크롤하는가?
각 페이지 데이터가 가벼운가?
모바일 네트워크에서도 부담 없는가?
```

대부분의 경우에는 첫 페이지만 미리 받고, 이후는 `fetchNextPage`로 자연스럽게 이어가는 편이 좋습니다.

---

## queryClient.setQueryData로 캐시 직접 수정하기

`setQueryData`는 캐시 데이터를 직접 바꿉니다.

```tsx
queryClient.setQueryData(queryKey, updater);
```

예를 들어 게시글 상세를 이미 가지고 있다면 목록에서 받은 일부 데이터로 상세 캐시를 미리 채울 수 있습니다.

```tsx
queryClient.setQueryData(["posts", "detail", post.id], post);
```

이런 작업을 manual priming이라고 볼 수 있습니다.

이미 동기적으로 가지고 있는 데이터를 캐시에 넣어두는 것입니다.

```tsx
function PostListItem({ post }: { post: PostSummary }) {
  const queryClient = useQueryClient();

  return (
    <a
      href={`/posts/${post.id}`}
      onMouseEnter={() => {
        queryClient.setQueryData(["posts", "detail", post.id], post);
      }}
    >
      {post.title}
    </a>
  );
}
```

다만 주의할 점이 있습니다.

목록 데이터의 `post`가 상세 데이터보다 정보가 적을 수 있습니다.

```tsx
type PostSummary = {
  id: string;
  title: string;
};

type PostDetail = {
  id: string;
  title: string;
  content: string;
  author: User;
  comments: Comment[];
};
```

이 경우 `PostSummary`를 `PostDetail` 캐시에 그대로 넣으면 상세 페이지에서 `content`가 없어서 문제가 생길 수 있습니다.

이럴 때는 상세 query의 타입과 화면 요구사항을 분리해서 생각해야 합니다.

```tsx
queryClient.setQueryData<PostDetail | undefined>(
  ["posts", "detail", post.id],
  (oldPost) => {
    if (!oldPost) {
      return oldPost;
    }

    return {
      ...oldPost,
      title: post.title,
    };
  },
);
```

`setQueryData`는 강력하지만, 서버 데이터의 모양을 정확히 이해하고 써야 합니다.

---

## setQueryData로 mutation 성공 후 즉시 반영하기

게시글 제목을 수정하는 mutation을 생각해봅시다.

서버 요청이 성공하면 상세 캐시를 바로 바꿀 수 있습니다.

```tsx
function useUpdatePostTitle() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updatePostTitle,
    onSuccess: (updatedPost) => {
      queryClient.setQueryData(
        ["posts", "detail", updatedPost.id],
        updatedPost,
      );
    },
  });
}
```

이렇게 하면 상세 화면은 즉시 최신 데이터로 바뀝니다.

목록에도 같은 제목이 보인다면 목록 캐시도 함께 수정할 수 있습니다.

```tsx
queryClient.setQueriesData<PostPage>(
  { queryKey: ["posts", "list"] },
  (oldPage) => {
    if (!oldPage) {
      return oldPage;
    }

    return {
      ...oldPage,
      posts: oldPage.posts.map((post) =>
        post.id === updatedPost.id
          ? { ...post, title: updatedPost.title }
          : post,
      ),
    };
  },
);
```

`setQueryData`는 하나의 queryKey를 수정하고, `setQueriesData`는 조건에 맞는 여러 query를 한 번에 수정합니다.

필터가 여러 개 있는 목록에서는 `setQueriesData`가 유용할 수 있습니다.

```txt
["posts", "list", { category: "react" }]
["posts", "list", { category: "nextjs" }]
["posts", "list", { category: "all" }]
```

이 목록들 중 같은 게시글이 들어 있을 수 있기 때문입니다.

---

## invalidateQueries는 캐시를 직접 바꾸는 것이 아니다

`invalidateQueries`는 캐시 데이터를 직접 수정하지 않습니다.

대신 해당 query를 stale 상태로 만들고, 활성 상태인 query를 다시 가져오게 할 수 있습니다.

```tsx
queryClient.invalidateQueries({
  queryKey: ["posts"],
});
```

예를 들어 게시글 작성에 성공한 뒤 목록을 다시 가져오고 싶다면 이렇게 씁니다.

```tsx
const createPostMutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    queryClient.invalidateQueries({
      queryKey: ["posts", "list"],
    });
  },
});
```

이 방식의 장점은 단순함입니다.

서버가 최종 진실이고, 우리는 그냥 관련 데이터를 다시 가져옵니다.

```txt
mutation 성공
→ 관련 query 무효화
→ active query background refetch
→ 서버 데이터로 화면 갱신
```

반면 `setQueryData`는 직접 캐시를 수정합니다.

```txt
mutation 성공
→ 캐시 직접 수정
→ 화면 즉시 변경
→ 필요하면 이후 invalidate로 서버와 재동기화
```

둘 중 무엇이 항상 더 좋은 것은 아닙니다.

기준은 이렇습니다.

| 상황 | 추천 |
| --- | --- |
| 서버 응답으로 바꿀 데이터가 명확함 | `setQueryData` |
| 영향받는 데이터가 많고 계산이 복잡함 | `invalidateQueries` |
| 즉시 반영이 중요함 | `setQueryData` 또는 optimistic update |
| 서버 정합성이 더 중요함 | `invalidateQueries` |
| 목록 정렬, 필터, 페이지 위치가 복잡함 | `invalidateQueries`가 더 안전할 수 있음 |

실무에서는 둘을 함께 쓰는 경우도 많습니다.

```tsx
onSuccess: (updatedPost) => {
  queryClient.setQueryData(["posts", "detail", updatedPost.id], updatedPost);

  queryClient.invalidateQueries({
    queryKey: ["posts", "list"],
  });
}
```

상세는 즉시 바꾸고, 목록은 서버 기준으로 다시 맞추는 식입니다.

---

## getQueryData는 캐시를 즉시 읽는다

`getQueryData`는 특정 queryKey의 캐시 데이터를 동기적으로 읽습니다.

```tsx
const post = queryClient.getQueryData<Post>(["posts", "detail", postId]);
```

없으면 `undefined`를 반환합니다.

이 함수는 컴포넌트 렌더링용으로 쓰는 것이 아닙니다.

컴포넌트에서 캐시 변화를 구독해야 한다면 `useQuery`를 써야 합니다.

```tsx
const { data } = useQuery({
  queryKey: ["posts", "detail", postId],
  queryFn: () => fetchPost(postId),
});
```

`getQueryData`는 보통 이런 상황에서 사용합니다.

- optimistic update 전에 이전 값 백업
- 이벤트 핸들러에서 현재 캐시 확인
- 이미 캐시에 있으면 prefetch를 생략
- 다른 캐시를 만들 때 기존 캐시 일부 재사용

예를 들어 이미 상세 데이터가 있으면 prefetch를 생략할 수 있습니다.

```tsx
function prefetchPost(postId: string) {
  const cachedPost = queryClient.getQueryData(["posts", "detail", postId]);

  if (cachedPost) {
    return;
  }

  void queryClient
    .query({
      queryKey: ["posts", "detail", postId],
      queryFn: () => fetchPost(postId),
      staleTime: 60 * 1000,
    })
    .catch(noop);
}
```

다만 이 패턴은 `staleTime`과 중복될 수 있습니다.

React Query 자체가 fresh한 캐시가 있으면 불필요한 fetch를 피할 수 있기 때문입니다.

대부분은 `queryClient.query`에 `staleTime`을 주는 방식으로 충분합니다.

---

## cancelQueries는 optimistic update에서 중요하다

낙관적 업데이트를 할 때는 `cancelQueries`가 중요합니다.

예를 들어 할 일 목록에서 새 할 일을 추가한다고 해보겠습니다.

서버 응답을 기다리지 않고 UI에 먼저 추가하면 사용자는 빠르게 반응한다고 느낍니다.

하지만 동시에 기존 목록 refetch가 진행 중이라면 문제가 생길 수 있습니다.

```txt
1. todos refetch 진행 중
2. 사용자가 새 todo 추가
3. setQueryData로 낙관적 todo 추가
4. 아까 진행 중이던 refetch가 늦게 도착
5. 낙관적 업데이트를 덮어씀
```

그래서 낙관적 업데이트 전에 관련 query를 취소합니다.

```tsx
const addTodoMutation = useMutation({
  mutationFn: addTodo,
  onMutate: async (newTodo, context) => {
    await context.client.cancelQueries({
      queryKey: ["todos"],
    });

    const previousTodos = context.client.getQueryData<Todo[]>(["todos"]);

    context.client.setQueryData<Todo[]>(["todos"], (oldTodos = []) => [
      ...oldTodos,
      {
        id: "temporary-id",
        title: newTodo.title,
      },
    ]);

    return {
      previousTodos,
    };
  },
  onError: (error, newTodo, onMutateResult, context) => {
    context.client.setQueryData(["todos"], onMutateResult?.previousTodos);
  },
  onSettled: (data, error, variables, onMutateResult, context) => {
    context.client.invalidateQueries({
      queryKey: ["todos"],
    });
  },
});
```

흐름은 이렇습니다.

```txt
1. 진행 중인 todos refetch 취소
2. 이전 todos 스냅샷 저장
3. 캐시에 임시 todo 추가
4. mutation 실패 시 이전 스냅샷으로 rollback
5. 성공/실패와 관계없이 invalidate로 서버와 재동기화
```

이 패턴은 사용자 경험을 크게 개선합니다.

사용자는 서버 응답을 기다리지 않고 자신의 행동이 즉시 반영되는 것을 봅니다.

다만 낙관적 업데이트는 서버 규칙을 클라이언트가 어느 정도 흉내 내야 하므로, 복잡한 비즈니스 로직에서는 신중해야 합니다.

---

## ensureQueryData는 어떤 용도였을까?

기존 React Query 코드에서는 `ensureQueryData`를 볼 수 있습니다.

```tsx
const post = await queryClient.ensureQueryData({
  queryKey: ["posts", "detail", postId],
  queryFn: () => fetchPost(postId),
});
```

의도는 이렇습니다.

```txt
캐시에 데이터가 있으면 그것을 반환하고,
없으면 fetch해서 캐시에 넣은 뒤 반환한다.
```

라우터 loader에서 자주 쓰던 패턴입니다.

최신 문서에서는 이 역시 `queryClient.query`와 `staleTime: "static"` 조합으로 대체하는 흐름입니다.

```tsx
const post = await queryClient.query({
  queryKey: ["posts", "detail", postId],
  queryFn: () => fetchPost(postId),
  staleTime: "static",
});
```

`staleTime: "static"`은 캐시에 데이터가 있으면 stale 여부와 관계없이 사용할 수 있게 하는 의도입니다.

라우트 진입 전에 데이터가 반드시 필요하다면 `await`하고 에러를 처리합니다.

```tsx
async function postLoader({ params }: LoaderArgs) {
  const postId = params.postId;

  if (!postId) {
    throw new Error("postId가 없습니다.");
  }

  return queryClient.query({
    queryKey: ["posts", "detail", postId],
    queryFn: () => fetchPost(postId),
    staleTime: "static",
  });
}
```

이 경우는 hover prefetch처럼 실패를 조용히 무시하면 안 됩니다.

라우트가 그 데이터 없이는 렌더링될 수 없기 때문입니다.

```txt
비중요 prefetch
→ void queryClient.query(...).catch(noop)

라우트에 필수인 데이터
→ await queryClient.query(...)
→ 실패 시 에러 경계나 라우터 에러 처리
```

---

## 라우터와 함께 쓰면 더 강력하다

prefetch는 라우터와 잘 어울립니다.

사용자가 어떤 경로로 이동할지 라우터가 알고 있기 때문입니다.

예를 들어 게시글 상세 라우트에 진입하기 전에 데이터를 준비할 수 있습니다.

```tsx
const postRoute = {
  path: "/posts/:postId",
  loader: async ({ params }) => {
    const postId = params.postId;

    await queryClient.query({
      queryKey: ["posts", "detail", postId],
      queryFn: () => fetchPost(postId),
      staleTime: 60 * 1000,
    });
  },
};
```

이렇게 하면 컴포넌트가 마운트된 뒤에야 요청을 시작하는 waterfall을 줄일 수 있습니다.

```txt
나쁜 흐름
→ 라우트 이동
→ 컴포넌트 렌더링
→ useQuery 실행
→ 데이터 요청
→ 로딩
→ 화면 표시

개선된 흐름
→ 라우트 이동 시작
→ loader에서 데이터 요청
→ 캐시 채움
→ 컴포넌트는 캐시 데이터 사용
```

Next.js App Router에서는 서버에서 `QueryClient`를 만들고 `prefetchQuery` 또는 최신 방식의 `query`로 캐시를 채운 뒤 `dehydrate`와 `HydrationBoundary`로 클라이언트에 넘기는 패턴을 사용할 수 있습니다.

이 부분은 Hydration 글에서 다룬 내용과 연결됩니다.

---

## request waterfall 줄이기

QueryClient prefetch는 단순히 "클릭 전에 미리 받기"만 의미하지 않습니다.

컴포넌트 구조 때문에 생기는 request waterfall을 줄이는 데도 사용할 수 있습니다.

예를 들어 아래 구조를 봅시다.

```tsx
function ArticlePage({ articleId }: { articleId: string }) {
  const articleQuery = useQuery({
    queryKey: ["articles", articleId],
    queryFn: () => fetchArticle(articleId),
  });

  if (articleQuery.isPending) {
    return <p>아티클을 불러오는 중...</p>;
  }

  return (
    <>
      <ArticleContent article={articleQuery.data} />
      <ArticleComments articleId={articleId} />
    </>
  );
}

function ArticleComments({ articleId }: { articleId: string }) {
  const commentsQuery = useQuery({
    queryKey: ["articles", articleId, "comments"],
    queryFn: () => fetchComments(articleId),
  });

  // ...
}
```

이 구조에서는 댓글 요청이 아티클 요청 이후에 시작될 수 있습니다.

```txt
1. article 요청
2. article 도착 후 ArticleComments 렌더링
3. comments 요청
```

댓글이 아티클 본문과 독립적으로 가져올 수 있는 데이터라면 부모에서 미리 시작할 수 있습니다.

```tsx
function ArticlePage({ articleId }: { articleId: string }) {
  const queryClient = useQueryClient();

  const articleQuery = useQuery({
    queryKey: ["articles", articleId],
    queryFn: (...args) => {
      void queryClient
        .query({
          queryKey: ["articles", articleId, "comments"],
          queryFn: () => fetchComments(articleId),
        })
        .catch(noop);

      return fetchArticle(articleId, ...args);
    },
  });

  // ...
}
```

이렇게 하면 article 요청과 comments 요청을 더 일찍 병렬로 시작할 수 있습니다.

물론 모든 prefetch가 좋은 것은 아닙니다.

댓글 영역을 사용자가 거의 보지 않는다면 요청 낭비일 수 있습니다.

하지만 대부분의 사용자가 댓글까지 보는 화면이라면 waterfall을 줄여 체감 속도를 개선할 수 있습니다.

---

## setQueryData와 invalidateQueries 중 무엇을 써야 할까?

많이 헷갈리는 지점입니다.

mutation 성공 후에는 두 가지 선택지가 있습니다.

첫 번째는 캐시를 직접 수정하는 것입니다.

```tsx
onSuccess: (updatedPost) => {
  queryClient.setQueryData(
    ["posts", "detail", updatedPost.id],
    updatedPost,
  );
}
```

두 번째는 관련 query를 무효화해서 다시 가져오는 것입니다.

```tsx
onSuccess: () => {
  queryClient.invalidateQueries({
    queryKey: ["posts"],
  });
}
```

판단 기준은 이렇습니다.

```txt
서버 응답만으로 정확히 캐시를 갱신할 수 있는가?
→ setQueryData

어떤 목록에 들어가야 하는지, 정렬이 어떻게 바뀌는지 복잡한가?
→ invalidateQueries

즉시 화면 반영이 중요한가?
→ setQueryData 또는 optimistic update

서버 기준으로 다시 맞추는 것이 더 중요한가?
→ invalidateQueries
```

예를 들어 게시글 상세 제목 수정은 `setQueryData`가 잘 맞습니다.

```tsx
queryClient.setQueryData(["posts", "detail", post.id], post);
```

하지만 새 게시글 작성 후 목록에 어디에 들어가야 하는지는 정렬, 필터, 페이지에 따라 달라집니다.

이 경우에는 목록을 무효화하는 편이 더 안전할 수 있습니다.

```tsx
queryClient.invalidateQueries({
  queryKey: ["posts", "list"],
});
```

---

## QueryClient 조작에서 불변성 지키기

`setQueryData`를 사용할 때는 기존 데이터를 직접 변경하면 안 됩니다.

나쁜 예시입니다.

```tsx
queryClient.setQueryData<Todo[]>(["todos"], (oldTodos = []) => {
  oldTodos.push(newTodo);

  return oldTodos;
});
```

이 코드는 기존 배열을 직접 변경합니다.

React Query 캐시를 다룰 때도 React 상태처럼 불변성을 지키는 것이 좋습니다.

좋은 예시는 새 배열을 반환하는 것입니다.

```tsx
queryClient.setQueryData<Todo[]>(["todos"], (oldTodos = []) => {
  return [...oldTodos, newTodo];
});
```

객체도 마찬가지입니다.

```tsx
queryClient.setQueryData<Post>(
  ["posts", "detail", postId],
  (oldPost) => {
    if (!oldPost) {
      return oldPost;
    }

    return {
      ...oldPost,
      title: nextTitle,
    };
  },
);
```

캐시를 직접 바꾸는 순간부터는 우리가 작은 상태 관리 코드를 작성하는 것과 비슷해집니다.

그래서 불변성, 타입, rollback을 함께 신경 써야 합니다.

---

## 실무에서 자주 쓰는 QueryClient 메서드 정리

| 메서드 | 역할 | 대표 사용처 |
| --- | --- | --- |
| `query` | 데이터를 가져오고 캐시에 저장하고 결과를 반환 | prefetch, loader, 필수 데이터 준비 |
| `infiniteQuery` | infinite query 데이터를 가져오고 캐시에 저장 | 무한 목록 prefetch |
| `setQueryData` | 특정 queryKey의 캐시를 직접 수정 | 상세 데이터 즉시 반영 |
| `setQueriesData` | 조건에 맞는 여러 query 캐시 수정 | 여러 목록에 포함된 항목 갱신 |
| `getQueryData` | 특정 queryKey 캐시를 동기적으로 읽음 | optimistic update 스냅샷 |
| `getQueriesData` | 여러 query 캐시를 읽음 | 여러 목록 상태 확인 |
| `invalidateQueries` | query를 stale로 만들고 refetch 유도 | mutation 성공 후 서버와 동기화 |
| `cancelQueries` | 진행 중인 요청 취소 | optimistic update 덮어쓰기 방지 |
| `removeQueries` | 캐시에서 query 제거 | 로그아웃, 민감 데이터 정리 |
| `clear` | QueryClient의 query/mutation cache 비우기 | 앱 전체 캐시 초기화 |

처음부터 전부 외울 필요는 없습니다.

실무에서는 보통 아래 순서로 익히면 됩니다.

```txt
1. invalidateQueries
2. setQueryData
3. getQueryData
4. cancelQueries
5. query / infiniteQuery
6. setQueriesData
```

---

## UX 개선 패턴 1. 링크 hover prefetch

상세 페이지 이동이 많은 서비스에서 효과적입니다.

```tsx
function UserCard({ user }: { user: User }) {
  const queryClient = useQueryClient();

  const prefetchUser = () => {
    void queryClient
      .query({
        queryKey: ["users", "detail", user.id],
        queryFn: () => fetchUser(user.id),
        staleTime: 30 * 1000,
      })
      .catch(noop);
  };

  return (
    <a
      href={`/users/${user.id}`}
      onMouseEnter={prefetchUser}
      onFocus={prefetchUser}
    >
      {user.name}
    </a>
  );
}
```

`onFocus`를 함께 넣으면 키보드 사용자도 prefetch 혜택을 받을 수 있습니다.

---

## UX 개선 패턴 2. 다음 페이지 미리 준비하기

페이지네이션에서 현재 페이지를 보여주면서 다음 페이지를 미리 받아둘 수 있습니다.

```tsx
function ProductPage({ page }: { page: number }) {
  const queryClient = useQueryClient();

  const productsQuery = useQuery({
    queryKey: ["products", "list", page],
    queryFn: () => fetchProducts(page),
    staleTime: 30 * 1000,
  });

  useEffect(() => {
    if (!productsQuery.data?.hasMore) {
      return;
    }

    void queryClient
      .query({
        queryKey: ["products", "list", page + 1],
        queryFn: () => fetchProducts(page + 1),
        staleTime: 30 * 1000,
      })
      .catch(noop);
  }, [queryClient, page, productsQuery.data?.hasMore]);

  // ...
}
```

이렇게 하면 사용자가 "다음"을 눌렀을 때 이미 다음 페이지 데이터가 캐시에 있을 가능성이 높아집니다.

다만 사용자가 다음 페이지로 가지 않을 수도 있으므로, 데이터 크기와 트래픽 비용을 함께 고려해야 합니다.

---

## UX 개선 패턴 3. 수정 후 즉시 반영하기

사용자 이름을 수정한 뒤 상세 화면을 바로 바꾸고 싶다면 `setQueryData`가 좋습니다.

```tsx
function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateUser,
    onSuccess: (updatedUser) => {
      queryClient.setQueryData(
        ["users", "detail", updatedUser.id],
        updatedUser,
      );

      queryClient.invalidateQueries({
        queryKey: ["users", "list"],
      });
    },
  });
}
```

상세는 즉시 반영하고, 목록은 서버에서 다시 받아와 정렬과 필터까지 정확히 맞춥니다.

---

## UX 개선 패턴 4. 낙관적 좋아요

좋아요 버튼은 optimistic update가 잘 어울립니다.

```tsx
function useLikePost(postId: string) {
  return useMutation({
    mutationFn: () => likePost(postId),
    onMutate: async (variables, context) => {
      await context.client.cancelQueries({
        queryKey: ["posts", "detail", postId],
      });

      const previousPost = context.client.getQueryData<Post>([
        "posts",
        "detail",
        postId,
      ]);

      context.client.setQueryData<Post>(
        ["posts", "detail", postId],
        (oldPost) => {
          if (!oldPost) {
            return oldPost;
          }

          return {
            ...oldPost,
            liked: true,
            likeCount: oldPost.likeCount + 1,
          };
        },
      );

      return {
        previousPost,
      };
    },
    onError: (error, variables, onMutateResult, context) => {
      context.client.setQueryData(
        ["posts", "detail", postId],
        onMutateResult?.previousPost,
      );
    },
    onSettled: (data, error, variables, onMutateResult, context) => {
      context.client.invalidateQueries({
        queryKey: ["posts", "detail", postId],
      });
    },
  });
}
```

사용자는 버튼을 누르자마자 좋아요가 반영된 것처럼 느낍니다.

실패하면 이전 캐시로 rollback합니다.

좋아요, 북마크, 팔로우처럼 단순하고 되돌리기 쉬운 액션에 잘 맞는 패턴입니다.

---

## 언제 QueryClient 조작을 피해야 할까?

QueryClient 조작은 강력하지만, 과하게 쓰면 캐시 상태가 복잡해집니다.

다음 상황에서는 단순한 `invalidateQueries`가 더 나을 수 있습니다.

- 목록 정렬 규칙이 복잡하다
- 서버에서 계산되는 값이 많다
- 필터와 페이지네이션 조합이 많다
- optimistic update 실패 시 rollback이 어렵다
- 클라이언트가 서버 비즈니스 로직을 정확히 재현하기 어렵다

예를 들어 결제 상태, 재고 수량, 권한 변경처럼 정확성이 중요한 데이터는 섣불리 낙관적 업데이트하지 않는 편이 좋습니다.

```txt
사용자가 빠르게 느끼는 것보다
잘못된 상태를 보여주지 않는 것이 더 중요한 화면이 있다.
```

이런 경우에는 mutation 성공 후 `invalidateQueries`로 서버 데이터를 다시 받아오는 편이 안전합니다.

---

## 실무 체크리스트

QueryClient를 사용하기 전에 아래를 확인하면 좋습니다.

```txt
queryKey가 일관적인가?
prefetch와 useQuery가 같은 queryKey를 쓰는가?
prefetch 데이터가 실제로 재사용되는가?
staleTime이 너무 짧아서 바로 refetch되지 않는가?
setQueryData로 바꾸는 데이터 타입이 정확한가?
캐시를 직접 수정할 때 불변성을 지키는가?
optimistic update 실패 시 rollback할 수 있는가?
무효화 범위가 너무 넓거나 좁지 않은가?
민감한 데이터는 로그아웃 시 제거되는가?
```

특히 prefetch는 "성공하면 좋고 실패해도 괜찮은 작업"인지, "라우트 렌더링에 필수인 작업"인지 구분해야 합니다.

```txt
실패해도 괜찮음
→ catch(noop)

실패하면 화면을 보여줄 수 없음
→ await + 에러 처리
```

---

## 정리

`QueryClient`는 React Query 캐시를 직접 다루는 핵심 객체입니다.

`useQuery`가 캐시를 구독해서 화면에 보여주는 도구라면, `QueryClient`는 캐시를 미리 채우고, 읽고, 수정하고, 무효화하는 도구입니다.

기존에는 `prefetchQuery`, `prefetchInfiniteQuery`, `ensureQueryData` 같은 이름을 많이 사용했지만, 최신 TanStack Query 문서에서는 `queryClient.query`, `queryClient.infiniteQuery` 중심의 흐름을 권장합니다.

실무에서 가장 자주 쓰는 목적은 다음과 같습니다.

```txt
상세 페이지 데이터를 미리 받아 로딩 줄이기
다음 페이지 데이터를 미리 준비하기
mutation 성공 후 캐시 즉시 수정하기
낙관적 업데이트로 반응성 높이기
관련 query를 무효화해 서버와 다시 동기화하기
```

다만 QueryClient를 쓰면 캐시를 직접 책임지는 영역이 늘어납니다.

그래서 처음에는 `invalidateQueries`로 단순하게 시작하고, 실제 사용자 경험상 필요한 지점에 `prefetch`, `setQueryData`, `cancelQueries`를 추가하는 편이 좋습니다.

좋은 기준은 이것입니다.

```txt
서버 데이터를 더 빨리 보여주기 위해 캐시를 미리 채울 것인가?
사용자 행동을 즉시 반영하기 위해 캐시를 직접 수정할 것인가?
정확성을 위해 서버와 다시 동기화할 것인가?
```

이 질문에 답할 수 있으면 QueryClient는 단순한 내부 객체가 아니라, 사용자 경험을 설계하는 강력한 도구가 됩니다.

---

## 참고 자료

- [TanStack Query 공식 문서 - QueryClient](https://tanstack.com/query/latest/docs/framework/react/reference/classes/QueryClient)
- [TanStack Query 공식 문서 - Prefetching & Router Integration](https://tanstack.com/query/latest/docs/framework/react/guides/prefetching)
- [TanStack Query 공식 문서 - Infinite Queries](https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries)
- [TanStack Query 공식 문서 - Optimistic Updates](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates)
- [TanStack Query 공식 문서 - Query Invalidation](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation)

{% endraw %}
