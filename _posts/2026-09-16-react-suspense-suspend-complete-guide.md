---
title: "[React] Suspense 완벽 이해하기: suspend 개념부터 fallback 동작까지"
date: 2026-09-16T00:30:00Z
categories: [frontend]
tags: [react, suspense, suspend, fallback, lazy, streaming, performance]
description: "React Suspense가 무엇을 기다리는지, suspend가 어떤 의미인지, fallback이 언제 보이는지, lazy, use, Error Boundary, Next.js streaming까지 연결해서 정리했습니다."
custom_style: true
---

## 들어가며: Suspense는 로딩 컴포넌트가 아니에요

React에서 `Suspense`를 처음 보면 보통 이렇게 이해합니다.

```tsx
<Suspense fallback={<p>로딩 중...</p>}>
  <SomeComponent />
</Suspense>
```

그래서 이런 느낌이 들 수 있습니다.

```txt
Suspense는 로딩 UI를 보여주는 컴포넌트구나
데이터 가져올 때 감싸면 자동으로 로딩이 나오겠구나
fallback은 그냥 isLoading일 때 보여주는 화면이구나
```

반은 맞고, 반은 부족합니다.

`Suspense`는 단순히 로딩 UI를 보여주는 컴포넌트가 아닙니다.

더 정확히 말하면 **자식 컴포넌트가 아직 렌더링을 완료할 수 없을 때, 그 영역의 렌더링을 잠시 보류하고 대신 fallback을 보여주는 경계**입니다.

여기서 중요한 단어가 바로 `suspend`입니다.

```txt
suspend
→ 컴포넌트가 렌더링 도중 "아직 준비가 안 됐으니 기다려줘"라고 React에게 알리는 상태
```

이번 글에서는 `Suspense`를 단순 사용법이 아니라 동작 원리까지 연결해서 이해해보겠습니다.

---

## Suspense를 한 문장으로 이해하기

`Suspense`는 아래 문장으로 이해하면 좋습니다.

```txt
자식 컴포넌트가 렌더링 도중 suspend되면,
가장 가까운 Suspense가 fallback을 보여주고,
준비가 끝나면 다시 렌더링을 시도한다.
```

흐름으로 보면 이렇습니다.

```txt
1. React가 컴포넌트를 렌더링한다.
2. 어떤 컴포넌트가 아직 준비되지 않은 값을 읽는다.
3. 그 컴포넌트가 suspend된다.
4. React는 가장 가까운 Suspense boundary를 찾는다.
5. 해당 boundary 안의 UI 대신 fallback을 보여준다.
6. 기다리던 작업이 끝나면 React가 다시 렌더링한다.
7. 이번에는 값이 준비되어 있으므로 실제 UI가 보여진다.
```

여기서 `Suspense boundary`는 `<Suspense>`로 감싼 영역을 말합니다.

```tsx
<Suspense fallback={<UserSkeleton />}>
  <UserProfile />
</Suspense>
```

이 코드에서 `UserProfile`이 suspend되면, React는 `UserSkeleton`을 보여줍니다.

---

## suspend란 무엇일까?

React 컴포넌트는 기본적으로 렌더링할 때 값을 바로 가지고 있어야 합니다.

```tsx
function UserName({ name }: { name: string }) {
  return <p>{name}</p>;
}
```

이 경우 `name`이 이미 있으므로 바로 렌더링할 수 있습니다.

그런데 어떤 값은 아직 준비되지 않았을 수 있습니다.

```txt
컴포넌트 코드를 아직 다운로드하지 못함
서버에서 데이터를 아직 가져오지 못함
스트리밍 중인 HTML 조각이 아직 도착하지 않음
```

이때 컴포넌트가 렌더링을 계속할 수 없다면 React에게 이렇게 알려야 합니다.

```txt
지금은 렌더링을 완료할 수 없어.
이 작업이 끝나면 다시 시도해줘.
```

이 상태가 `suspend`입니다.

중요한 점은 `suspend`가 일반적인 `isLoading` 상태와 다르다는 것입니다.

`isLoading`은 컴포넌트 안에서 직접 분기합니다.

```tsx
function UserProfile() {
  const { data, isLoading } = useUser();

  if (isLoading) {
    return <UserSkeleton />;
  }

  return <p>{data.name}</p>;
}
```

반면 `Suspense`는 컴포넌트 바깥의 boundary가 로딩 UI를 담당합니다.

```tsx
<Suspense fallback={<UserSkeleton />}>
  <UserProfile />
</Suspense>
```

`UserProfile`은 "로딩이면 무엇을 보여줄지"를 직접 결정하지 않습니다.

대신 렌더링 도중 준비되지 않은 것을 만나면 suspend되고, 가장 가까운 `Suspense`가 fallback을 보여줍니다.

---

## Suspense를 활성화하는 것들

`Suspense`로 감싸기만 한다고 모든 비동기 작업이 자동으로 잡히는 것은 아닙니다.

React 공식 문서 기준으로 Suspense boundary를 활성화하는 대표적인 경우는 다음과 같습니다.

| 경우 | 예시 |
| --- | --- |
| 컴포넌트 코드를 늦게 불러올 때 | `React.lazy` |
| Promise를 렌더링 중에 읽을 때 | `use(promise)` |
| Suspense를 지원하는 프레임워크나 라이브러리에서 데이터를 읽을 때 | Next.js, Relay, 일부 query 라이브러리 |
| 서버 렌더링 중 HTML 조각이 아직 도착하지 않았을 때 | streaming SSR |

반대로 아래 코드는 `Suspense`를 활성화하지 않습니다.

```tsx
function UserProfile() {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetch("/api/me")
      .then((res) => res.json())
      .then(setUser);
  }, []);

  if (!user) {
    return <p>로딩 중...</p>;
  }

  return <p>{user.name}</p>;
}
```

이 코드는 `useEffect`에서 데이터를 가져옵니다.

`useEffect`는 렌더링이 끝난 뒤 실행됩니다. 즉, 렌더링 도중에 컴포넌트가 suspend되는 것이 아닙니다.

그래서 이 컴포넌트를 `Suspense`로 감싸도 fallback이 자동으로 뜨지 않습니다.

```tsx
<Suspense fallback={<UserSkeleton />}>
  <UserProfile />
</Suspense>
```

위처럼 감싸도 `UserProfile` 내부의 `if (!user)` 분기가 그대로 로딩 UI를 담당합니다.

핵심은 이 차이입니다.

```txt
useEffect fetch
→ 렌더링 후에 비동기 작업 시작
→ Suspense boundary를 활성화하지 않음

Suspense 기반 fetch
→ 렌더링 도중 준비되지 않은 값을 읽음
→ 컴포넌트가 suspend됨
→ Suspense fallback이 보임
```

---

## 가장 익숙한 예: React.lazy

`Suspense`를 가장 쉽게 만나는 곳은 `React.lazy`입니다.

```tsx
import { lazy, Suspense } from "react";

const Chart = lazy(() => import("./Chart"));

export default function Dashboard() {
  return (
    <Suspense fallback={<p>차트를 불러오는 중...</p>}>
      <Chart />
    </Suspense>
  );
}
```

이 코드에서 `Chart` 컴포넌트 코드는 처음부터 바로 불러오지 않습니다.

`Chart`가 실제로 렌더링되어야 하는 순간 동적으로 import됩니다.

```txt
Chart 렌더링 시도
→ Chart 코드가 아직 없음
→ Chart가 suspend됨
→ Suspense fallback 표시
→ Chart 코드 다운로드 완료
→ 다시 렌더링
→ Chart 표시
```

여기서 중요한 점은 `fallback`이 `Chart`만의 로딩 UI가 아니라는 것입니다.

`fallback`은 **해당 Suspense boundary 안쪽 전체가 아직 준비되지 않았을 때 보여주는 대체 UI**입니다.

```tsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Chart />
  <RecentOrders />
</Suspense>
```

이 경우 `Chart` 하나만 suspend되어도 `Header`, `Chart`, `RecentOrders` 전체가 `PageSkeleton`으로 대체될 수 있습니다.

그래서 boundary를 어디에 두는지가 중요합니다.

---

## Suspense boundary는 로딩 경험의 단위다

아래 코드를 봅시다.

```tsx
<Suspense fallback={<PageSkeleton />}>
  <Profile />
  <ActivityFeed />
  <RecommendationList />
</Suspense>
```

이 구조에서는 세 영역 중 하나라도 suspend되면 전체가 `PageSkeleton`으로 대체됩니다.

즉, 사용자는 세 영역이 모두 준비될 때까지 하나의 큰 로딩 화면을 봅니다.

이게 나쁜 것은 아닙니다.

화면 전체가 하나의 완성된 덩어리로 보여야 한다면 좋은 선택입니다.

하지만 프로필은 빨리 보여주고, 활동 내역과 추천 목록은 따로 로딩되어도 괜찮다면 boundary를 나누는 편이 좋습니다.

```tsx
<Profile />

<Suspense fallback={<ActivityFeedSkeleton />}>
  <ActivityFeed />
</Suspense>

<Suspense fallback={<RecommendationSkeleton />}>
  <RecommendationList />
</Suspense>
```

이렇게 하면 각 영역이 준비되는 대로 따로 나타날 수 있습니다.

정리하면 이렇습니다.

```txt
Suspense boundary는 컴포넌트 구조의 단위가 아니라
사용자가 경험할 로딩 흐름의 단위다.
```

너무 크게 감싸면 화면 전체가 자주 fallback으로 바뀔 수 있습니다.

너무 잘게 나누면 여기저기 스켈레톤이 깜빡이는 산만한 화면이 될 수 있습니다.

좋은 기준은 이것입니다.

```txt
어떤 UI들이 함께 나타나야 자연스러운가?
어떤 UI들은 늦게 따로 나타나도 괜찮은가?
```

---

## Promise를 throw한다는 말은 무슨 뜻일까?

Suspense를 깊게 공부하다 보면 이런 설명을 만납니다.

```txt
Suspense는 Promise를 throw해서 동작한다.
```

처음 보면 꽤 이상합니다.

보통 `throw`는 에러를 던질 때 사용합니다.

```ts
throw new Error("문제가 발생했습니다.");
```

그런데 Suspense의 내부 모델에서는 "아직 준비되지 않았다"는 신호로 Promise가 사용됩니다.

아주 단순화하면 이런 느낌입니다.

```tsx
function readUser() {
  if (status === "pending") {
    throw promise;
  }

  if (status === "error") {
    throw error;
  }

  return data;
}

function UserProfile() {
  const user = readUser();

  return <p>{user.name}</p>;
}
```

`readUser()`가 아직 준비되지 않은 Promise를 던지면 React는 이것을 일반적인 에러처럼 바로 터뜨리지 않고, 가까운 `Suspense` boundary에서 잡습니다.

```tsx
<Suspense fallback={<UserSkeleton />}>
  <UserProfile />
</Suspense>
```

그 뒤 Promise가 resolve되면 React는 다시 렌더링을 시도합니다.

두 번째 렌더링에서는 `readUser()`가 데이터를 반환할 수 있으므로 실제 UI가 그려집니다.

흐름은 이렇습니다.

```txt
readUser()
→ 아직 데이터 없음
→ Promise throw
→ Suspense가 잡음
→ fallback 표시
→ Promise resolve
→ React가 다시 렌더링
→ readUser()가 data 반환
→ 실제 UI 표시
```

다만 실무에서 직접 `throw promise` 패턴을 손으로 만드는 일은 많지 않습니다.

대부분은 아래 방식을 사용합니다.

- `React.lazy`
- React의 `use(promise)`
- Next.js 같은 Suspense 지원 프레임워크
- Suspense 모드를 지원하는 데이터 fetching 라이브러리

직접 만든다면 Promise 캐싱, 에러 처리, 재시도, 중복 요청 방지까지 챙겨야 해서 금방 복잡해집니다.

---

## React 19의 use와 Suspense

React 19 기준으로는 `use` API를 통해 Promise를 읽을 수 있습니다.

예를 들어 Server Component에서 Promise를 만들고, Client Component로 전달한다고 해보겠습니다.

```tsx
// app/page.tsx
import { Suspense } from "react";
import { UserName } from "./UserName";
import { getUser } from "./api";

export default function Page() {
  const userPromise = getUser();

  return (
    <Suspense fallback={<p>사용자 정보를 불러오는 중...</p>}>
      <UserName userPromise={userPromise} />
    </Suspense>
  );
}
```

```tsx
// app/UserName.tsx
"use client";

import { use } from "react";

type User = {
  name: string;
};

export function UserName({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);

  return <p>{user.name}</p>;
}
```

`use(userPromise)`는 Promise가 아직 pending이면 컴포넌트를 suspend시킵니다.

그러면 가장 가까운 `Suspense`가 fallback을 보여줍니다.

Promise가 resolve되면 `use(userPromise)`는 값을 반환하고 실제 UI가 렌더링됩니다.

여기서 중요한 주의점이 있습니다.

Promise는 렌더링마다 새로 만들면 안 됩니다.

```tsx
function BadExample() {
  const user = use(fetch("/api/me").then((res) => res.json()));

  return <p>{user.name}</p>;
}
```

이런 식으로 렌더링 중에 매번 새 Promise를 만들면 React가 재시도할 때도 또 새로운 Promise가 생길 수 있습니다.

그러면 계속 suspend되거나 중복 요청이 발생할 수 있습니다.

그래서 Suspense에서 Promise를 읽을 때는 같은 Promise 인스턴스가 재사용되도록 캐싱되어야 합니다.

프레임워크나 데이터 fetching 라이브러리를 쓰면 이 부분을 대신 관리해주는 경우가 많습니다.

---

## Error Boundary와 Suspense는 역할이 다르다

`Suspense`는 pending 상태를 다룹니다.

`Error Boundary`는 error 상태를 다룹니다.

둘은 비슷해 보이지만 역할이 다릅니다.

| 상황 | 처리하는 경계 |
| --- | --- |
| Promise가 아직 pending | `Suspense` |
| Promise가 reject됨 | `Error Boundary` |
| 렌더링 중 에러 발생 | `Error Boundary` |

그래서 데이터 로딩 UI와 에러 UI가 모두 필요하다면 보통 둘을 함께 둡니다.

```tsx
<ErrorBoundary fallback={<ErrorMessage />}>
  <Suspense fallback={<UserSkeleton />}>
    <UserProfile />
  </Suspense>
</ErrorBoundary>
```

이 구조에서는 다음처럼 동작합니다.

```txt
UserProfile이 pending Promise를 읽음
→ Suspense가 UserSkeleton 표시

Promise가 정상 resolve됨
→ UserProfile 표시

Promise가 reject되거나 렌더링 에러 발생
→ ErrorBoundary가 ErrorMessage 표시
```

`Suspense`는 실패를 해결해주는 장치가 아닙니다.

로딩은 `Suspense`, 실패는 `Error Boundary`가 담당한다고 나눠서 이해하면 좋습니다.

---

## fallback은 언제 다시 나타날까?

처음 로딩 때 fallback이 보이는 것은 자연스럽습니다.

문제는 이미 화면에 보이던 UI가 다시 suspend되는 경우입니다.

예를 들어 검색 화면을 생각해봅시다.

```txt
검색어: react
→ 결과 목록 표시됨

검색어를 react suspense로 변경
→ 새 결과를 가져오는 동안 다시 fallback 표시
```

이때 기존 결과가 갑자기 사라지고 큰 스켈레톤으로 바뀌면 사용자는 화면이 깜빡인다고 느낄 수 있습니다.

이런 경우에는 `useDeferredValue`나 `startTransition` 같은 기능을 함께 고려할 수 있습니다.

목표는 단순합니다.

```txt
이미 보여준 UI를 바로 지우지 말고
새 데이터가 준비될 때까지 이전 UI를 잠시 유지한다.
```

예를 들어 검색어 입력값은 즉시 바꾸되, 결과 목록은 조금 늦게 따라오게 만들 수 있습니다.

```tsx
function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <input value={query} onChange={(event) => setQuery(event.target.value)} />

      <Suspense fallback={<SearchResultSkeleton />}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

이렇게 하면 입력은 즉시 반응하고, 결과 목록은 이전 값을 조금 더 유지할 수 있습니다.

즉, `Suspense`는 fallback을 보여주는 장치이고, `useDeferredValue`나 `startTransition`은 이미 보이는 UI가 갑자기 사라지는 경험을 줄이는 데 도움을 줍니다.

---

## Next.js에서 Suspense가 더 중요해지는 이유

Next.js App Router를 사용하면 `Suspense`는 단순한 클라이언트 로딩 UI를 넘어 서버 렌더링과도 연결됩니다.

예를 들어 App Router의 `loading.tsx`는 내부적으로 route segment를 `Suspense` boundary로 감싸는 역할을 합니다.

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

```tsx
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const data = await getDashboardData();

  return <Dashboard data={data} />;
}
```

이 경우 사용자가 `/dashboard`로 이동하면 Next.js는 먼저 `loading.tsx`의 UI를 보여줄 수 있습니다.

그리고 `page.tsx`의 데이터와 HTML이 준비되면 실제 화면으로 교체합니다.

직접 `Suspense`를 둘 수도 있습니다.

```tsx
export default function DashboardPage() {
  return (
    <main>
      <Summary />

      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<OrderListSkeleton />}>
        <RecentOrders />
      </Suspense>
    </main>
  );
}
```

이 구조에서는 `Summary`는 먼저 보여주고, `RevenueChart`와 `RecentOrders`는 준비되는 대로 따로 스트리밍할 수 있습니다.

그래서 Next.js에서 `Suspense`는 다음 질문과 연결됩니다.

```txt
어떤 부분을 먼저 보여줄 것인가?
어떤 부분은 늦게 도착해도 괜찮은가?
사용자가 기다리는 동안 어떤 skeleton을 볼 것인가?
```

즉, `Suspense`는 성능 기능이면서 동시에 UX 설계 도구입니다.

---

## Suspense를 잘못 이해했을 때 생기는 실수

### 1. useEffect fetch를 Suspense가 잡아줄 거라고 생각한다

```tsx
<Suspense fallback={<Loading />}>
  <UserProfile />
</Suspense>
```

이렇게 감쌌다고 해서 `UserProfile` 안의 모든 비동기 작업이 자동으로 fallback으로 연결되지는 않습니다.

`useEffect`는 렌더링 이후 실행되므로 Suspense를 활성화하지 않습니다.

### 2. 모든 컴포넌트를 Suspense로 감싼다

```tsx
<Suspense fallback={<Loading />}>
  <Button />
</Suspense>
```

작고 항상 필요한 컴포넌트까지 전부 감싸면 구조만 복잡해질 수 있습니다.

Suspense boundary는 "로딩 경험을 나누고 싶은 지점"에 두는 것이 좋습니다.

### 3. fallback을 너무 빈약하게 만든다

```tsx
<Suspense fallback={<p>Loading...</p>}>
  <ProductGrid />
</Suspense>
```

기능적으로는 동작하지만, 실제 서비스에서는 화면 구조를 예측할 수 있는 skeleton이 더 자연스러운 경우가 많습니다.

```tsx
<Suspense fallback={<ProductGridSkeleton />}>
  <ProductGrid />
</Suspense>
```

fallback은 단순히 기다리라는 표시가 아니라, 사용자가 "어떤 화면이 곧 올지" 이해하게 해주는 힌트입니다.

### 4. 에러 처리를 Suspense에 맡긴다

`Suspense`는 pending을 처리합니다.

실패한 요청이나 렌더링 에러는 `Error Boundary`가 필요합니다.

```tsx
<ErrorBoundary fallback={<ErrorMessage />}>
  <Suspense fallback={<ProductSkeleton />}>
    <ProductDetail />
  </Suspense>
</ErrorBoundary>
```

### 5. 렌더링 중 Promise를 매번 새로 만든다

```tsx
function UserProfile() {
  const user = use(fetchUser());

  return <p>{user.name}</p>;
}
```

`fetchUser()`가 호출될 때마다 새 Promise를 만든다면 Suspense가 안정적으로 동작하기 어렵습니다.

Promise는 캐싱되어 같은 작업에 대해 같은 Promise가 재사용되어야 합니다.

---

## Suspense를 어디에 쓰면 좋을까?

실무에서 Suspense가 잘 어울리는 상황은 다음과 같습니다.

- 페이지나 큰 기능 단위의 코드 스플리팅
- 차트, 에디터, 지도처럼 무거운 컴포넌트 lazy loading
- Next.js App Router의 route loading UI
- Server Component streaming에서 일부 영역을 늦게 보여주기
- Suspense 지원 데이터 fetching 라이브러리와 함께 로딩 경계를 설계하기
- 검색 결과처럼 일부 UI만 늦게 갱신되는 화면

반대로 이런 곳에는 신중해야 합니다.

- 아주 작은 버튼, 아이콘, 텍스트 컴포넌트
- 첫 화면에서 반드시 즉시 보여야 하는 핵심 콘텐츠
- fallback이 오히려 사용자 경험을 방해하는 영역
- `useEffect` 기반 로딩 상태를 그대로 쓰는 컴포넌트

핵심은 이것입니다.

```txt
Suspense를 어디에 둘 것인가
= 사용자가 어떤 순서로 화면을 보게 할 것인가
```

---

## 정리

`Suspense`는 단순한 로딩 컴포넌트가 아닙니다.

컴포넌트가 렌더링 도중 준비되지 않은 것을 만나 `suspend`되면, 가장 가까운 `Suspense` boundary가 fallback을 보여주고 준비가 끝난 뒤 다시 렌더링을 시도합니다.

`suspend`는 "아직 렌더링을 완료할 수 없으니 기다려달라"는 신호입니다.

이 신호는 `React.lazy`, `use(promise)`, Suspense 지원 프레임워크나 라이브러리, streaming SSR 같은 상황에서 발생할 수 있습니다.

반대로 `useEffect` 안에서 데이터를 가져오는 일반적인 방식은 Suspense를 자동으로 활성화하지 않습니다.

Suspense를 잘 쓰려면 API보다 먼저 boundary의 의미를 이해해야 합니다.

```txt
이 UI는 함께 나타나야 하는가?
이 UI는 따로 늦게 나타나도 되는가?
로딩 중 사용자는 무엇을 봐야 자연스러운가?
실패했을 때는 어디서 처리할 것인가?
```

이 질문에 답할 수 있으면 `Suspense`는 더 이상 낯선 기능이 아니라, 로딩 경험과 렌더링 흐름을 설계하는 도구가 됩니다.

---

## 참고 자료

- [React 공식 문서 - Suspense](https://react.dev/reference/react/Suspense)
- [React 공식 문서 - lazy](https://react.dev/reference/react/lazy)
- [React 공식 문서 - use](https://react.dev/reference/react/use)
- [React 공식 문서 - renderToPipeableStream](https://react.dev/reference/react-dom/server/renderToPipeableStream)
- [Next.js 공식 문서 - loading.js](https://nextjs.org/docs/app/api-reference/file-conventions/loading)
