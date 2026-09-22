---
title: "[React Query] keepPreviousData 이해하기: 페이지 전환 중 이전 데이터를 유지하는 방법"
date: 2026-09-20T00:30:00Z
categories: [frontend]
tags: [react, react-query, tanstack-query, keepPreviousData, placeholderData, pagination]
description: "TanStack Query에서 keepPreviousData가 필요한 이유, v4와 v5 사용법 차이, 페이지네이션과 필터 전환에서 이전 데이터를 유지할 때의 장점과 주의사항을 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: 페이지를 넘길 때 화면이 깜빡이는 이유

React Query로 페이지네이션을 만들 때 보통 `page` 값을 `queryKey`에 넣습니다.

```tsx
const { data, isPending } = useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
});
```

이 구조는 맞습니다.

`page`가 바뀌면 React Query는 다른 데이터를 가져와야 한다고 판단합니다.

```txt
page = 1
→ queryKey: ["projects", 1]

page = 2
→ queryKey: ["projects", 2]
```

문제는 사용자가 1페이지에서 2페이지로 넘어갈 때입니다.

2페이지 데이터가 아직 도착하지 않았다면 화면은 잠시 로딩 상태로 바뀔 수 있습니다.

```txt
1페이지 목록 표시
→ 다음 버튼 클릭
→ 2페이지 queryKey로 변경
→ 2페이지 데이터 없음
→ 로딩 UI 표시
→ 2페이지 데이터 도착
→ 2페이지 목록 표시
```

사용자 입장에서는 목록이 잠깐 사라졌다가 다시 나타나는 것처럼 보입니다.

이 깜빡임을 줄이기 위해 사용하는 기능이 `keepPreviousData`입니다.

---

## keepPreviousData가 해결하는 문제

`keepPreviousData`의 목적은 단순합니다.

```txt
queryKey가 바뀌어서 새 데이터를 가져오는 동안
기존에 보여주던 데이터를 잠시 유지한다.
```

즉, 1페이지에서 2페이지로 이동할 때 2페이지 데이터가 아직 없더라도 화면을 빈 로딩 상태로 보내지 않습니다.

대신 1페이지 데이터를 계속 보여줍니다.

```txt
1페이지 목록 표시
→ 다음 버튼 클릭
→ 2페이지 데이터 요청 시작
→ 새 데이터가 올 때까지 1페이지 목록 유지
→ 2페이지 데이터 도착
→ 2페이지 목록으로 교체
```

이렇게 하면 화면이 덜 흔들립니다.

사용자는 "목록이 사라졌다"가 아니라 "다음 데이터를 불러오는 중이구나"라고 느낄 수 있습니다.

---

## v4와 v5에서 사용법이 다르다

여기서 가장 헷갈리는 부분이 있습니다.

React Query v4에서는 보통 이렇게 사용했습니다.

```tsx
const { data, isPreviousData } = useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  keepPreviousData: true,
});
```

하지만 TanStack Query v5에서는 `keepPreviousData: true` 옵션이 아니라 `placeholderData`를 사용합니다.

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";

const { data, isPlaceholderData } = useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
});
```

v5 기준으로 정리하면 이렇습니다.

| 버전 | 사용 방식 | 상태 플래그 |
| --- | --- | --- |
| v4 | `keepPreviousData: true` | `isPreviousData` |
| v5 | `placeholderData: keepPreviousData` | `isPlaceholderData` |

현재 새 프로젝트라면 v5 방식으로 이해하는 것이 좋습니다.

다만 실무 코드나 오래된 글에서는 `keepPreviousData: true`를 볼 수 있으니, 두 방식이 같은 문제를 해결한다는 점을 알고 있으면 됩니다.

---

## v5에서 keepPreviousData 사용하기

기본 예제를 보겠습니다.

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";
import { useState } from "react";

type Project = {
  id: string;
  name: string;
};

type ProjectPage = {
  projects: Project[];
  hasMore: boolean;
};

async function fetchProjects(page: number): Promise<ProjectPage> {
  const response = await fetch(`/api/projects?page=${page}`);

  if (!response.ok) {
    throw new Error("프로젝트 목록을 불러오지 못했습니다.");
  }

  return response.json();
}

export function ProjectList() {
  const [page, setPage] = useState(1);

  const {
    data,
    isPending,
    isFetching,
    isError,
    isPlaceholderData,
  } = useQuery({
    queryKey: ["projects", page],
    queryFn: () => fetchProjects(page),
    placeholderData: keepPreviousData,
  });

  if (isPending) {
    return <p>처음 데이터를 불러오는 중...</p>;
  }

  if (isError) {
    return <p>프로젝트 목록을 불러오지 못했습니다.</p>;
  }

  return (
    <section>
      <ul>
        {data.projects.map((project) => (
          <li key={project.id}>{project.name}</li>
        ))}
      </ul>

      {isFetching && <p>새 페이지를 불러오는 중...</p>}

      <button
        type="button"
        disabled={page === 1}
        onClick={() => setPage((currentPage) => currentPage - 1)}
      >
        이전
      </button>

      <button
        type="button"
        disabled={isPlaceholderData || !data.hasMore}
        onClick={() => setPage((currentPage) => currentPage + 1)}
      >
        다음
      </button>
    </section>
  );
}
```

이 코드의 핵심은 이 부분입니다.

```tsx
placeholderData: keepPreviousData
```

새 `queryKey`의 데이터가 아직 없을 때, 이전 query의 데이터를 placeholder처럼 사용합니다.

그리고 그 상태인지 확인할 때는 `isPlaceholderData`를 봅니다.

```tsx
disabled={isPlaceholderData || !data.hasMore}
```

이렇게 하면 아직 2페이지가 도착하지 않았는데 사용자가 3페이지로 빠르게 넘어가는 상황을 막을 수 있습니다.

---

## isPending, isFetching, isPlaceholderData 차이

`keepPreviousData`를 이해하려면 상태값 차이를 알아야 합니다.

| 값 | 의미 |
| --- | --- |
| `isPending` | 아직 보여줄 데이터가 없는 첫 로딩 상태 |
| `isFetching` | 요청이 진행 중인 상태 |
| `isPlaceholderData` | 현재 data가 진짜 새 데이터가 아니라 placeholder 데이터인 상태 |

처음 1페이지에 들어왔을 때는 아직 아무 데이터도 없습니다.

```txt
첫 진입
→ data 없음
→ isPending: true
→ 로딩 UI 표시
```

하지만 1페이지를 본 뒤 2페이지로 넘어갈 때는 이전 데이터가 있습니다.

```txt
1페이지 표시 중
→ 2페이지로 이동
→ 2페이지 요청 시작
→ data는 일단 1페이지 데이터
→ isPending: false
→ isFetching: true
→ isPlaceholderData: true
```

즉, `keepPreviousData`는 첫 로딩을 없애는 기능이 아닙니다.

첫 로딩에는 이전 데이터가 없기 때문입니다.

이 기능은 **이미 성공한 query가 있고, queryKey가 바뀌는 전환 구간**에서 의미가 있습니다.

---

## placeholderData와 keepPreviousData의 관계

TanStack Query v5에서 `placeholderData`는 query가 아직 실제 데이터를 갖고 있지 않을 때 임시 데이터를 넣어주는 옵션입니다.

```tsx
useQuery({
  queryKey: ["post", postId],
  queryFn: () => fetchPost(postId),
  placeholderData: {
    id: postId,
    title: "불러오는 중...",
    body: "",
  },
});
```

이렇게 직접 placeholder 값을 넣을 수도 있습니다.

하지만 이전 query의 데이터를 그대로 이어서 쓰고 싶다면 `keepPreviousData`를 사용합니다.

```tsx
useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
});
```

이 코드는 개념적으로 아래와 비슷합니다.

```tsx
useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  placeholderData: (previousData) => previousData,
});
```

`previousData`를 그대로 반환하는 identity function을 사용한다고 이해하면 됩니다.

그래서 `keepPreviousData`는 특별한 캐시 전략이라기보다, `placeholderData`에 넣기 좋은 헬퍼 함수에 가깝습니다.

---

## 언제 사용하면 좋을까?

`keepPreviousData`는 queryKey가 바뀌지만 화면 구조는 유지되는 상황에 잘 맞습니다.

대표적인 예시는 페이지네이션입니다.

```tsx
queryKey: ["products", page]
```

검색 필터 전환에도 사용할 수 있습니다.

```tsx
queryKey: ["products", { page, category, sort }]
```

탭 전환에서도 경우에 따라 사용할 수 있습니다.

```tsx
queryKey: ["orders", status]
```

잘 어울리는 화면은 이런 특징이 있습니다.

- 이전 데이터와 새 데이터가 같은 형태다
- 새 데이터가 올 때까지 이전 데이터를 보여줘도 어색하지 않다
- 빈 로딩 화면보다 기존 목록을 유지하는 편이 자연스럽다
- 페이지, 필터, 정렬, 커서처럼 queryKey 일부가 자주 바뀐다
- 사용자가 현재 화면 맥락을 잃지 않는 것이 중요하다

예를 들어 관리자 테이블에서 1페이지에서 2페이지로 넘어갈 때, 목록이 매번 통째로 사라지는 것보다 기존 목록 위에 작은 로딩 표시를 보여주는 편이 더 안정적입니다.

---

## 장점 1. 화면 깜빡임을 줄인다

가장 큰 장점은 UI가 덜 흔들린다는 것입니다.

`keepPreviousData`가 없으면 queryKey가 바뀔 때 새 query가 pending 상태가 되면서 로딩 UI로 바뀔 수 있습니다.

```txt
목록
→ 로딩
→ 새 목록
```

`keepPreviousData`를 사용하면 전환이 더 부드럽습니다.

```txt
목록
→ 기존 목록 유지 + 불러오는 중 표시
→ 새 목록
```

특히 테이블, 카드 목록, 검색 결과처럼 화면 면적이 큰 UI에서 차이가 큽니다.

사용자는 화면이 비었다고 느끼지 않고, 데이터가 갱신 중이라는 느낌을 받습니다.

---

## 장점 2. 레이아웃 점프를 줄인다

목록이 로딩 UI로 바뀌면 화면 높이가 줄어들거나 스켈레톤 구조가 달라질 수 있습니다.

그러면 버튼 위치, 스크롤 위치, 카드 높이가 흔들립니다.

`keepPreviousData`를 사용하면 기존 목록을 그대로 유지하기 때문에 레이아웃 점프가 줄어듭니다.

```txt
기존 rows 유지
→ 테이블 높이 유지
→ 페이지네이션 버튼 위치 유지
→ 사용자가 시선을 잃지 않음
```

관리자 페이지나 대시보드처럼 데이터를 반복해서 탐색하는 화면에서는 이런 차이가 꽤 크게 느껴집니다.

---

## 장점 3. 로딩 상태를 더 세밀하게 나눌 수 있다

`keepPreviousData`를 사용하면 로딩 상태를 둘로 나눠서 생각할 수 있습니다.

```txt
처음 들어왔는데 아무 데이터도 없음
→ hard loading

이미 보여줄 데이터가 있고 새 데이터만 가져오는 중
→ soft loading
```

코드로는 이렇게 나눌 수 있습니다.

```tsx
if (isPending) {
  return <PageSkeleton />;
}

return (
  <section>
    {isFetching && <LoadingBar />}
    <ProjectTable projects={data.projects} />
  </section>
);
```

처음 로딩에서는 큰 스켈레톤을 보여주고, 페이지 전환에서는 작은 로딩 바나 버튼 비활성화 정도만 보여줄 수 있습니다.

이게 사용자 경험을 훨씬 자연스럽게 만듭니다.

---

## 장점 4. 다음 페이지 중복 클릭을 막기 쉽다

새 페이지 데이터가 아직 도착하지 않았는데 사용자가 다음 버튼을 연속으로 누르면 의도치 않은 페이지로 넘어갈 수 있습니다.

`isPlaceholderData`를 사용하면 현재 화면이 이전 데이터인지 알 수 있습니다.

```tsx
<button
  type="button"
  disabled={isPlaceholderData || !data.hasMore}
  onClick={() => setPage((currentPage) => currentPage + 1)}
>
  다음
</button>
```

이렇게 하면 새 페이지가 도착하기 전까지 다음 이동을 잠시 막을 수 있습니다.

`isFetching`만으로 막을 수도 있지만, `isPlaceholderData`는 "현재 data가 이전 query의 임시 데이터인가?"를 더 직접적으로 표현합니다.

---

## 주의할 점 1. 이전 데이터는 새 데이터가 아니다

`keepPreviousData`를 사용하면 새 queryKey로 이동했는데도 `data`에 이전 query의 데이터가 들어올 수 있습니다.

예를 들어 현재 `page`는 2인데, `data`는 아직 1페이지 데이터일 수 있습니다.

```txt
page = 2
data = 1페이지 데이터
isPlaceholderData = true
```

그래서 UI에서 이 상태를 구분해주는 것이 좋습니다.

```tsx
{isPlaceholderData && <p>새 페이지를 불러오는 중입니다.</p>}
```

또는 테이블 위에 작은 dimmed overlay를 줄 수도 있습니다.

```tsx
<div className={isPlaceholderData ? "table is-loading" : "table"}>
  <ProjectTable projects={data.projects} />
</div>
```

사용자에게 아무 표시 없이 이전 데이터를 보여주면, 새 필터가 적용됐는데 왜 결과가 그대로인지 헷갈릴 수 있습니다.

---

## 주의할 점 2. 완전히 다른 데이터에는 어울리지 않을 수 있다

`keepPreviousData`는 이전 데이터와 새 데이터가 같은 맥락일 때 좋습니다.

예를 들어 상품 목록의 1페이지와 2페이지는 같은 목록의 다른 페이지입니다.

이 경우 이전 페이지를 잠시 보여주는 것이 자연스럽습니다.

하지만 전혀 다른 데이터라면 오히려 혼란스러울 수 있습니다.

```tsx
queryKey: ["profile", userId]
```

사용자 A의 프로필에서 사용자 B의 프로필로 이동했는데, B의 데이터가 도착하기 전까지 A의 프로필을 보여주면 위험할 수 있습니다.

특히 아래 정보는 이전 데이터를 유지하면 안 되는 경우가 많습니다.

- 사용자 개인정보
- 결제 정보
- 권한이나 역할 정보
- 계좌, 주문, 의료, 보안 관련 정보
- 서로 다른 사용자의 상세 페이지

이런 경우에는 명확한 로딩 UI를 보여주는 편이 더 안전합니다.

```tsx
if (isPending) {
  return <ProfileSkeleton />;
}
```

핵심은 이것입니다.

```txt
이전 데이터를 잠시 보여줘도 사용자가 오해하지 않는가?
```

오해할 수 있다면 `keepPreviousData`를 쓰지 않는 편이 좋습니다.

---

## 주의할 점 3. placeholderData는 캐시에 저장되는 진짜 데이터가 아니다

`placeholderData`는 query가 성공한 것처럼 UI를 렌더링하게 해주지만, 그 자체가 새 query의 진짜 캐시 데이터로 저장되는 것은 아닙니다.

즉, `page = 2` query가 아직 성공하지 않았는데 `data`에 1페이지 데이터가 보이는 것은 placeholder 동작입니다.

2페이지 요청이 성공하면 그때 2페이지 데이터가 해당 queryKey의 캐시에 들어갑니다.

이 차이를 이해해야 합니다.

```txt
placeholderData
→ 화면을 임시로 채우는 데이터
→ 새 query의 진짜 응답은 아님

queryFn 성공 결과
→ queryKey에 저장되는 실제 캐시 데이터
```

그래서 `isPlaceholderData` 플래그를 함께 확인하는 습관이 중요합니다.

---

## keepPreviousData와 staleTime은 다르다

`keepPreviousData`와 `staleTime`을 헷갈릴 수 있습니다.

둘 다 "캐시된 데이터를 보여준다"는 느낌이 있기 때문입니다.

하지만 역할이 다릅니다.

| 옵션 | 역할 |
| --- | --- |
| `staleTime` | 같은 queryKey의 데이터를 얼마 동안 fresh로 볼지 정함 |
| `keepPreviousData` | queryKey가 바뀌는 동안 이전 query의 데이터를 임시로 보여줌 |

예를 들어 같은 1페이지로 다시 돌아오는 경우를 생각해봅시다.

```tsx
queryKey: ["projects", 1]
```

이미 1페이지 캐시가 있다면 React Query는 그 캐시를 사용할 수 있습니다.

이것은 `keepPreviousData` 때문이라기보다 React Query의 캐시 동작입니다.

반면 1페이지에서 2페이지로 이동할 때는 queryKey가 달라집니다.

```tsx
["projects", 1] → ["projects", 2]
```

이때 2페이지 데이터가 아직 없는데 1페이지 데이터를 임시로 보여주는 것이 `keepPreviousData`입니다.

정리하면 이렇습니다.

```txt
같은 queryKey의 캐시 재사용
→ staleTime, cache, refetch 정책과 관련

다른 queryKey로 넘어가는 중 이전 데이터 유지
→ keepPreviousData와 관련
```

---

## keepPreviousData와 initialData도 다르다

`initialData`는 query의 초기 데이터를 캐시에 넣는 데 사용합니다.

```tsx
useQuery({
  queryKey: ["todos"],
  queryFn: fetchTodos,
  initialData: [],
});
```

반면 `placeholderData`는 임시 표시용 데이터입니다.

```tsx
useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
});
```

차이는 이렇습니다.

| 옵션 | 캐시에 저장 여부 | 주 사용 목적 |
| --- | --- | --- |
| `initialData` | 캐시에 저장됨 | 진짜 초기 데이터가 있을 때 |
| `placeholderData` | 캐시에 저장되지 않음 | 실제 데이터가 오기 전 UI를 임시로 채울 때 |
| `keepPreviousData` | 이전 데이터를 placeholder로 사용 | queryKey 전환 중 깜빡임 줄이기 |

서버에서 미리 가져온 진짜 데이터라면 `initialData`나 Hydration을 고려할 수 있습니다.

이전 페이지 데이터를 잠깐 보여주려는 목적이라면 `placeholderData: keepPreviousData`가 더 잘 맞습니다.

---

## 필터 검색에서 사용하는 예시

페이지네이션뿐 아니라 검색 필터에서도 사용할 수 있습니다.

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";
import { useState } from "react";

type ProductFilters = {
  keyword: string;
  category: string;
  page: number;
};

export function ProductSearchPage() {
  const [filters, setFilters] = useState<ProductFilters>({
    keyword: "",
    category: "all",
    page: 1,
  });

  const {
    data,
    isPending,
    isFetching,
    isPlaceholderData,
  } = useQuery({
    queryKey: ["products", filters],
    queryFn: () => fetchProducts(filters),
    placeholderData: keepPreviousData,
  });

  if (isPending) {
    return <ProductListSkeleton />;
  }

  return (
    <section>
      <SearchFilters
        value={filters}
        onChange={(nextFilters) => setFilters(nextFilters)}
      />

      {isFetching && <p>검색 결과를 업데이트하는 중...</p>}

      <ProductList
        products={data.products}
        dimmed={isPlaceholderData}
      />
    </section>
  );
}
```

검색어, 카테고리, 페이지가 바뀔 때마다 queryKey가 바뀝니다.

새 검색 결과가 오기 전까지 이전 결과를 유지하면 화면이 덜 흔들립니다.

다만 검색 조건이 완전히 달라졌는데 이전 결과를 그대로 보여주면 사용자가 헷갈릴 수 있습니다.

그래서 `isPlaceholderData`일 때 흐리게 처리하거나 "업데이트 중" 표시를 넣는 것이 좋습니다.

---

## cursor 기반 페이지네이션에서도 사용할 수 있다

페이지 번호가 아니라 cursor를 사용하는 API도 많습니다.

```tsx
const { data, isPlaceholderData } = useQuery({
  queryKey: ["posts", cursor],
  queryFn: () => fetchPosts(cursor),
  placeholderData: keepPreviousData,
});
```

이 경우에도 원리는 같습니다.

```txt
cursor A의 데이터 표시
→ cursor B로 이동
→ cursor B 데이터 요청
→ 요청 중에는 cursor A 데이터 유지
→ cursor B 데이터 도착 후 교체
```

다만 무한 스크롤처럼 이전 페이지들을 누적해서 보여주는 UI라면 `useInfiniteQuery`가 더 적합할 수 있습니다.

`keepPreviousData`는 "현재 페이지를 다른 페이지로 교체하는 UI"에 잘 맞습니다.

계속 아래로 붙여나가는 UI라면 `useInfiniteQuery`를 먼저 고려하는 편이 자연스럽습니다.

---

## 언제 쓰지 않는 게 좋을까?

`keepPreviousData`가 항상 좋은 것은 아닙니다.

다음 상황에서는 신중해야 합니다.

- 이전 데이터와 새 데이터가 전혀 다른 의미를 가진다
- 이전 데이터를 보여주면 사용자가 잘못된 판단을 할 수 있다
- 보안, 권한, 결제, 개인정보처럼 정확성이 중요한 화면이다
- 상세 페이지처럼 "다른 대상"으로 이동하는 느낌이 강하다
- 새 조건이 적용되면 결과를 비우는 편이 더 명확하다

예를 들어 사용자 상세 페이지에서는 이전 사용자의 정보를 잠깐 보여주는 것이 오해를 만들 수 있습니다.

```tsx
queryKey: ["user", userId]
```

이런 경우에는 명확하게 스켈레톤이나 로딩 상태를 보여주는 편이 낫습니다.

반대로 같은 목록의 페이지 전환이라면 `keepPreviousData`가 잘 어울립니다.

```tsx
queryKey: ["users", { page, role, sort }]
```

---

## 실무 판단 기준

`keepPreviousData`를 쓸지 말지 고민된다면 아래 질문을 해보면 됩니다.

```txt
queryKey가 바뀌는가?
이전 데이터와 새 데이터의 형태가 같은가?
이전 데이터를 잠시 보여줘도 사용자가 오해하지 않는가?
빈 로딩 화면보다 이전 데이터를 유지하는 편이 자연스러운가?
isPlaceholderData 상태를 UI에 표시할 계획이 있는가?
```

대부분 "예"라면 사용해도 좋습니다.

반대로 아래 질문에 "예"라면 조심해야 합니다.

```txt
이전 데이터를 보여주면 잘못된 정보처럼 보이는가?
사용자나 권한이 바뀌는 화면인가?
상세 페이지처럼 완전히 다른 대상을 보여주는가?
```

이 경우에는 `keepPreviousData`보다 명확한 로딩 UI가 더 낫습니다.

---

## 정리

`keepPreviousData`는 queryKey가 바뀌는 동안 이전 데이터를 잠시 유지해주는 기능입니다.

TanStack Query v5에서는 아래처럼 사용합니다.

```tsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";

useQuery({
  queryKey: ["projects", page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
});
```

React Query v4의 `keepPreviousData: true`와 같은 문제를 해결하지만, v5에서는 `placeholderData`를 통해 표현합니다.

이 기능의 장점은 명확합니다.

```txt
화면 깜빡임 감소
레이아웃 점프 감소
첫 로딩과 전환 로딩 분리
페이지 전환 UX 개선
다음 페이지 중복 클릭 제어
```

하지만 이전 데이터는 어디까지나 이전 데이터입니다.

새 queryKey의 진짜 데이터가 도착하기 전까지 임시로 보여주는 값이므로, `isPlaceholderData`를 사용해 사용자에게 업데이트 중이라는 힌트를 주는 것이 좋습니다.

잘 맞는 곳은 페이지네이션, 필터링된 목록, 정렬 가능한 테이블처럼 같은 맥락의 데이터를 전환하는 화면입니다.

반대로 사용자 상세, 결제, 권한, 개인정보처럼 잘못된 이전 데이터가 오해를 만들 수 있는 화면에서는 쓰지 않는 편이 더 안전합니다.

---

## 참고 자료

- [TanStack Query 공식 문서 - Paginated / Lagged Queries](https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries)
- [TanStack Query 공식 문서 - Placeholder Query Data](https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data)
- [TanStack Query 공식 문서 - Migrating to v5](https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5)

{% endraw %}
