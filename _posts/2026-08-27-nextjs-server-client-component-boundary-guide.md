---
title: "[Next.js] Server Component와 Client Component 경계 이해하기"
date: 2026-08-27T00:20:00Z
categories: [frontend]
tags: [nextjs, react, server-components, client-components, app-router, use-client]
description: "Next.js App Router에서 Server Component와 Client Component의 차이, use client 경계, 직렬화 가능한 props, children 패턴, 실무 분리 기준을 정리했습니다."
custom_style: true
---

## 들어가며: "여기는 서버일까, 클라이언트일까?"

Next.js App Router를 사용하다 보면 처음에는 이런 질문을 자주 만나게 됩니다.

```txt
이 컴포넌트에 use client를 붙여야 할까?
서버 컴포넌트에서 useState를 왜 못 쓰지?
클라이언트 컴포넌트로 함수를 넘기면 왜 에러가 나지?
서버 컴포넌트를 클라이언트 컴포넌트 안에 넣을 수는 없을까?
```

Pages Router 시절에는 대부분의 React 컴포넌트가 브라우저에서 실행된다고 생각해도 크게 어색하지 않았습니다.

하지만 App Router에서는 Server Component와 Client Component가 함께 등장합니다.

이 둘을 제대로 구분하지 못하면 다음과 같은 문제가 생깁니다.

- 필요 없는 컴포넌트까지 클라이언트 번들에 포함됩니다.
- 서버에서만 써야 하는 코드를 브라우저로 보내려 합니다.
- `useState`, `useEffect`, `onClick`을 어디서 써야 하는지 헷갈립니다.
- props 직렬화 에러를 만납니다.
- 데이터 fetching 위치가 애매해집니다.

이번 글에서는 Server Component와 Client Component를 단순히 외우는 것이 아니라, **둘 사이의 경계가 어디에 생기고 어떻게 설계해야 하는지**를 중심으로 정리해보겠습니다.

---

## 한 줄로 먼저 이해하기

Next.js App Router에서 기본값은 Server Component입니다.

```txt
아무것도 쓰지 않으면 Server Component
"use client"를 파일 맨 위에 쓰면 Client Component 경계 시작
```

예를 들어 아래 컴포넌트는 Server Component입니다.

```tsx
// app/page.tsx
export default function Page() {
  return <main>홈 페이지</main>;
}
```

반대로 파일 맨 위에 `"use client"`를 쓰면 Client Component가 됩니다.

```tsx
// components/Counter.tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button type="button" onClick={() => setCount((prev) => prev + 1)}>
      {count}
    </button>
  );
}
```

`useState`나 `onClick`처럼 브라우저에서 상호작용이 필요한 코드는 Client Component에서 다룹니다.

---

## Server Component는 무엇일까?

Server Component는 서버에서 렌더링되는 컴포넌트입니다.

Next.js App Router에서는 기본적으로 모든 컴포넌트가 Server Component입니다.

```tsx
// app/products/page.tsx
async function getProducts() {
  const response = await fetch("https://example.com/api/products");
  return response.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <main>
      <h1>상품 목록</h1>

      <ul>
        {products.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </main>
  );
}
```

Server Component에서는 서버에서 데이터를 가져오고, 그 결과로 UI를 만들 수 있습니다.

이때 컴포넌트 코드 자체가 브라우저 JavaScript 번들에 그대로 포함되지 않는다는 점이 중요합니다.

그래서 Server Component는 이런 작업에 잘 어울립니다.

- 데이터베이스 조회
- 서버 API 호출
- 파일 시스템 접근
- 인증된 사용자 정보 조회
- API key나 secret이 필요한 작업
- 정적인 UI 렌더링
- 큰 라이브러리를 서버에서만 사용하기

하지만 Server Component는 브라우저에서 실행되는 컴포넌트가 아니므로 다음 기능은 사용할 수 없습니다.

- `useState`
- `useEffect`
- `useRef`를 이용한 브라우저 DOM 조작
- `onClick`, `onChange` 같은 이벤트 핸들러
- `window`, `document`, `localStorage`

즉, Server Component는 **데이터를 준비하고 정적인 UI를 만드는 영역**에 가깝습니다.

---

## Client Component는 무엇일까?

Client Component는 브라우저에서 상호작용을 담당하는 컴포넌트입니다.

파일 맨 위에 `"use client"`를 붙이면 해당 파일이 Client Component의 진입점이 됩니다.

```tsx
"use client";

import { useState } from "react";

export function LikeButton() {
  const [liked, setLiked] = useState(false);

  return (
    <button type="button" onClick={() => setLiked((prev) => !prev)}>
      {liked ? "좋아요 취소" : "좋아요"}
    </button>
  );
}
```

Client Component는 이런 기능이 필요할 때 사용합니다.

- 클릭, 입력, 드래그 같은 이벤트 처리
- `useState`, `useReducer` 기반 상태 관리
- `useEffect`로 브라우저 API 사용
- `localStorage`, `window`, `document` 접근
- 모달, 드롭다운, 탭, 캐러셀 같은 상호작용 UI
- Zustand, React Query 같은 클라이언트 상태 도구 사용

중요한 점은 `"use client"`가 **이 파일부터 클라이언트 영역이 시작된다**는 선언이라는 것입니다.

컴포넌트 하나만 바뀌는 것이 아니라, 그 파일에서 import한 하위 모듈들도 클라이언트 번들에 들어갈 수 있습니다.

그래서 `"use client"`는 가능하면 필요한 작은 컴포넌트에만 붙이는 것이 좋습니다.

---

## 경계는 어디에서 생길까?

경계는 `"use client"`가 붙은 파일에서 생깁니다.

```txt
Server Component
↓
Client Component 경계
↓
Client Component 하위 트리
```

예를 들어 이런 구조가 있다고 해보겠습니다.

```txt
app/page.tsx
components/ProductList.tsx
components/AddToCartButton.tsx
```

`page.tsx`와 `ProductList.tsx`는 서버에서 데이터를 렌더링하고, 버튼만 클라이언트에서 동작하게 만들 수 있습니다.

```tsx
// app/page.tsx
import { ProductList } from "@/components/ProductList";

export default async function Page() {
  const products = await getProducts();

  return <ProductList products={products} />;
}
```

```tsx
// components/ProductList.tsx
import { AddToCartButton } from "./AddToCartButton";

type Product = {
  id: string;
  name: string;
};

type ProductListProps = {
  products: Product[];
};

export function ProductList({ products }: ProductListProps) {
  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>
          {product.name}
          <AddToCartButton productId={product.id} />
        </li>
      ))}
    </ul>
  );
}
```

```tsx
// components/AddToCartButton.tsx
"use client";

type AddToCartButtonProps = {
  productId: string;
};

export function AddToCartButton({ productId }: AddToCartButtonProps) {
  const handleClick = () => {
    console.log(`${productId} 상품을 장바구니에 담습니다.`);
  };

  return (
    <button type="button" onClick={handleClick}>
      장바구니 담기
    </button>
  );
}
```

이 구조에서 경계는 `AddToCartButton.tsx`에서 생깁니다.

```txt
Page                 Server Component
ProductList          Server Component
AddToCartButton      Client Component
```

이렇게 하면 상품 목록 전체를 클라이언트 컴포넌트로 만들지 않아도 됩니다.

상호작용이 필요한 버튼만 클라이언트로 내려보내면 됩니다.

---

## "use client"는 최대한 아래로 내리기

가장 흔한 실수는 페이지 전체에 `"use client"`를 붙이는 것입니다.

```tsx
// app/products/page.tsx
"use client";

import { useState } from "react";

export default function ProductsPage() {
  const [keyword, setKeyword] = useState("");

  return (
    <main>
      <input
        value={keyword}
        onChange={(event) => setKeyword(event.target.value)}
      />
      <ProductList keyword={keyword} />
    </main>
  );
}
```

물론 이렇게 하면 에러는 줄어들 수 있습니다.

하지만 페이지 전체가 클라이언트 경계 안으로 들어갑니다.

그 결과 서버에서만 처리해도 되는 UI나 로직까지 브라우저 JavaScript 번들에 포함될 수 있습니다.

더 나은 방향은 상호작용이 필요한 부분만 분리하는 것입니다.

```tsx
// app/products/page.tsx
import { ProductSearch } from "@/components/ProductSearch";
import { ProductList } from "@/components/ProductList";

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <main>
      <h1>상품 목록</h1>
      <ProductSearch />
      <ProductList products={products} />
    </main>
  );
}
```

```tsx
// components/ProductSearch.tsx
"use client";

import { useState } from "react";

export function ProductSearch() {
  const [keyword, setKeyword] = useState("");

  return (
    <input
      value={keyword}
      onChange={(event) => setKeyword(event.target.value)}
      placeholder="상품 검색"
    />
  );
}
```

이렇게 하면 페이지와 상품 목록은 서버 컴포넌트로 남고, 검색 input만 클라이언트 컴포넌트가 됩니다.

```txt
서버에서 가능한 일은 서버에 남기고
브라우저 상호작용만 클라이언트로 분리한다
```

이 감각이 App Router 구조에서 중요합니다.

---

## Server에서 Client로 props 넘기기

Server Component는 Client Component를 import해서 렌더링할 수 있습니다.

```tsx
// app/page.tsx
import { LikeButton } from "@/components/LikeButton";

export default function Page() {
  return <LikeButton initialLiked={false} />;
}
```

다만 Server Component에서 Client Component로 넘기는 props는 React가 직렬화할 수 있어야 합니다.

쉽게 말하면 네트워크를 통해 보낼 수 있는 값이어야 합니다.

```tsx
// 가능
<LikeButton productId="p_123" initialLiked={false} count={10} />
```

문자열, 숫자, boolean, 배열, 일반 객체처럼 JSON으로 표현 가능한 값은 보통 문제가 없습니다.

하지만 함수는 넘길 수 없습니다.

```tsx
// Server Component
export default function Page() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <ClientButton onClick={handleClick} />;
}
```

이런 코드는 문제가 됩니다.

함수는 서버에서 만들어진 실행 로직이고, 브라우저로 직렬화해서 넘길 수 없기 때문입니다.

대신 이벤트 핸들러는 Client Component 안에서 정의합니다.

```tsx
// ClientButton.tsx
"use client";

type ClientButtonProps = {
  label: string;
};

export function ClientButton({ label }: ClientButtonProps) {
  const handleClick = () => {
    console.log("clicked");
  };

  return (
    <button type="button" onClick={handleClick}>
      {label}
    </button>
  );
}
```

Server Component는 실행 함수를 넘기는 대신, 클라이언트가 동작하는 데 필요한 데이터만 넘기는 편이 좋습니다.

```tsx
// Server Component
<ClientButton label="저장" />
```

---

## Client에서 Server Component를 직접 import할 수 있을까?

이 부분이 가장 헷갈립니다.

원칙적으로 Client Component 안에서 Server Component를 직접 import해서 사용하는 구조는 피해야 합니다.

```tsx
// ClientShell.tsx
"use client";

import { ServerProfile } from "./ServerProfile";

export function ClientShell() {
  const [open, setOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setOpen((prev) => !prev)}>열기</button>
      {open && <ServerProfile />}
    </div>
  );
}
```

이런 식으로 쓰면 `ServerProfile`이 클라이언트 경계 안으로 들어오려고 합니다.

서버에서만 실행되어야 하는 코드가 클라이언트 번들에 섞일 수 있고, 데이터 fetching 구조도 이상해집니다.

대신 **Server Component를 children으로 넘기는 패턴**을 사용합니다.

```tsx
// app/page.tsx
import { ClientShell } from "@/components/ClientShell";
import { ServerProfile } from "@/components/ServerProfile";

export default function Page() {
  return (
    <ClientShell>
      <ServerProfile />
    </ClientShell>
  );
}
```

```tsx
// components/ClientShell.tsx
"use client";

import { useState, type ReactNode } from "react";

type ClientShellProps = {
  children: ReactNode;
};

export function ClientShell({ children }: ClientShellProps) {
  const [open, setOpen] = useState(false);

  return (
    <section>
      <button type="button" onClick={() => setOpen((prev) => !prev)}>
        열기
      </button>

      {open && <div>{children}</div>}
    </section>
  );
}
```

```tsx
// components/ServerProfile.tsx
export async function ServerProfile() {
  const profile = await getProfile();

  return <div>{profile.name}</div>;
}
```

이 구조에서는 `ServerProfile`이 서버에서 먼저 렌더링되고, 그 결과가 `ClientShell`의 `children`으로 전달됩니다.

즉, 클라이언트 컴포넌트가 서버 컴포넌트를 직접 import하는 것이 아니라, 서버 쪽에서 이미 만들어진 UI를 구멍에 끼워 넣는 방식입니다.

이 패턴은 모달, 탭, 아코디언, 레이아웃 shell 같은 컴포넌트에서 자주 사용됩니다.

---

## children 패턴이 유용한 예시

예를 들어 탭 UI는 클릭 상태가 필요하므로 Client Component가 자연스럽습니다.

하지만 탭 안의 내용은 서버에서 데이터를 가져와 렌더링하고 싶을 수 있습니다.

```tsx
// app/dashboard/page.tsx
import { Tabs } from "@/components/Tabs";
import { RecentOrders } from "@/components/RecentOrders";
import { UserStats } from "@/components/UserStats";

export default function DashboardPage() {
  return (
    <Tabs
      tabs={[
        {
          id: "orders",
          label: "최근 주문",
          content: <RecentOrders />,
        },
        {
          id: "stats",
          label: "사용자 통계",
          content: <UserStats />,
        },
      ]}
    />
  );
}
```

```tsx
// components/Tabs.tsx
"use client";

import { useState, type ReactNode } from "react";

type TabItem = {
  id: string;
  label: string;
  content: ReactNode;
};

type TabsProps = {
  tabs: TabItem[];
};

export function Tabs({ tabs }: TabsProps) {
  const [activeTabId, setActiveTabId] = useState(tabs[0]?.id);
  const activeTab = tabs.find((tab) => tab.id === activeTabId);

  return (
    <section>
      <div>
        {tabs.map((tab) => (
          <button
            key={tab.id}
            type="button"
            onClick={() => setActiveTabId(tab.id)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      <div>{activeTab?.content}</div>
    </section>
  );
}
```

`Tabs`는 Client Component지만, `RecentOrders`와 `UserStats`는 Server Component로 둘 수 있습니다.

핵심은 Client Component가 Server Component를 직접 import하지 않고, `content`라는 prop으로 이미 준비된 ReactNode를 받는다는 점입니다.

---

## 데이터 fetching은 어디서 할까?

기본 원칙은 이렇습니다.

```txt
첫 화면에 필요한 데이터는 Server Component에서 가져온다.
사용자 상호작용 이후 바뀌는 데이터는 Client Component에서 가져올 수 있다.
```

예를 들어 상품 상세 페이지의 초기 데이터는 서버에서 가져오는 것이 자연스럽습니다.

```tsx
// app/products/[id]/page.tsx
type ProductPageProps = {
  params: Promise<{
    id: string;
  }>;
};

export default async function ProductPage({ params }: ProductPageProps) {
  const { id } = await params;
  const product = await getProduct(id);

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <AddToCartButton productId={product.id} />
    </main>
  );
}
```

하지만 사용자가 필터를 바꾸거나, 검색어를 입력하거나, 무한 스크롤을 하는 데이터는 클라이언트에서 다루는 편이 자연스러울 수 있습니다.

```tsx
"use client";

import { useState } from "react";

export function ProductFilter() {
  const [category, setCategory] = useState("all");

  return (
    <select
      value={category}
      onChange={(event) => setCategory(event.target.value)}
    >
      <option value="all">전체</option>
      <option value="book">책</option>
      <option value="desk">책상</option>
    </select>
  );
}
```

즉, 데이터 fetching 위치는 단순히 "서버가 좋다" 또는 "클라이언트가 좋다"로 나누기보다 데이터가 필요한 시점을 기준으로 정하면 됩니다.

| 상황 | 추천 위치 |
| --- | --- |
| 페이지 진입 시 바로 필요한 데이터 | Server Component |
| SEO에 노출되어야 하는 데이터 | Server Component |
| API key나 DB 접근이 필요한 데이터 | Server Component |
| 사용자 입력 이후 바뀌는 데이터 | Client Component |
| 실시간으로 자주 갱신되는 데이터 | Client Component 또는 별도 실시간 구독 |
| React Query 캐시와 함께 쓸 데이터 | Server prefetch + Hydration 또는 Client query |

---

## Provider는 왜 Client Component일까?

App Router에서 React Query, Zustand Provider, Theme Provider 같은 코드를 설정할 때 이런 구조를 자주 봅니다.

```tsx
// app/providers.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useState, type ReactNode } from "react";

type ProvidersProps = {
  children: ReactNode;
};

export function Providers({ children }: ProvidersProps) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx
import { Providers } from "./providers";

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

Provider는 보통 context, 상태, effect를 사용하므로 Client Component가 필요합니다.

그렇다고 `layout.tsx` 전체를 Client Component로 만들 필요는 없습니다.

`Providers`만 클라이언트 컴포넌트로 분리하고, `layout.tsx`는 Server Component로 유지할 수 있습니다.

이것도 경계를 아래로 내리는 좋은 예시입니다.

```txt
layout.tsx는 서버에 남긴다
Provider만 client boundary로 분리한다
children은 그 안으로 통과시킨다
```

---

## 자주 만나는 에러: props는 직렬화 가능해야 합니다

Server Component에서 Client Component로 props를 넘길 때 이런 류의 에러를 볼 수 있습니다.

```txt
Props must be serializable for components in the "use client" entry file
```

대부분 원인은 함수, 클래스 인스턴스, 복잡한 객체를 넘기려고 했기 때문입니다.

예를 들어 다음 코드는 위험합니다.

```tsx
// Server Component
export default function Page() {
  return (
    <ClientForm
      onSubmit={(value) => {
        console.log(value);
      }}
    />
  );
}
```

`onSubmit`은 함수이므로 서버에서 클라이언트로 그대로 보낼 수 없습니다.

대신 클라이언트에서 핸들러를 정의하거나, 서버에서 실행되어야 하는 작업이라면 Server Action 구조를 사용해야 합니다.

```tsx
// ClientForm.tsx
"use client";

export function ClientForm() {
  const handleSubmit = (value: string) => {
    console.log(value);
  };

  return <form>{/* ... */}</form>;
}
```

또는 서버 작업을 별도 action으로 분리합니다.

```tsx
// actions.ts
"use server";

export async function saveProfile(formData: FormData) {
  const name = formData.get("name");

  await updateProfile({ name });
}
```

```tsx
// Server Component
import { saveProfile } from "./actions";

export default function Page() {
  return (
    <form action={saveProfile}>
      <input name="name" />
      <button type="submit">저장</button>
    </form>
  );
}
```

이렇게 하면 클라이언트로 일반 함수를 넘기는 것이 아니라, 서버에서 실행될 action을 명시적으로 연결할 수 있습니다.

---

## Server Component에서 못 하는 것

Server Component에서는 브라우저 환경이 없습니다.

그래서 다음 코드는 사용할 수 없습니다.

```tsx
// Server Component
export default function Page() {
  const width = window.innerWidth;

  return <div>{width}</div>;
}
```

`window`는 브라우저에만 있는 객체입니다.

서버에서 실행되는 컴포넌트에서는 존재하지 않습니다.

이런 코드는 Client Component로 분리해야 합니다.

```tsx
"use client";

import { useEffect, useState } from "react";

export function WindowWidth() {
  const [width, setWidth] = useState<number | null>(null);

  useEffect(() => {
    const updateWidth = () => {
      setWidth(window.innerWidth);
    };

    updateWidth();
    window.addEventListener("resize", updateWidth);

    return () => {
      window.removeEventListener("resize", updateWidth);
    };
  }, []);

  return <div>{width}</div>;
}
```

브라우저 API가 필요하면 Client Component입니다.

이 기준은 꽤 단순하고 강력합니다.

---

## Client Component에서 조심해야 하는 것

Client Component는 브라우저에서 실행될 수 있으므로 서버 전용 코드를 직접 import하면 안 됩니다.

예를 들어 DB client, secret key, 서버 파일 시스템을 다루는 코드는 Client Component에 들어가면 안 됩니다.

```tsx
// Client Component
"use client";

import { db } from "@/lib/db";

export function BadComponent() {
  // 브라우저에서 DB에 직접 접근하려는 구조가 됩니다.
  return <div>위험한 구조</div>;
}
```

이런 로직은 Server Component, Route Handler, Server Action 쪽에 둬야 합니다.

클라이언트 컴포넌트는 서버에 요청하거나, 서버에서 미리 내려준 데이터를 받아서 UI 상호작용을 처리하는 역할에 집중하는 편이 좋습니다.

---

## 실무에서 나누는 기준

컴포넌트를 만들 때는 아래 질문으로 판단하면 편합니다.

```txt
1. useState, useEffect, event handler가 필요한가?
   → 필요하면 Client Component

2. window, document, localStorage가 필요한가?
   → 필요하면 Client Component

3. DB, file system, secret, 서버 API가 필요한가?
   → 필요하면 Server Component 또는 Server Action

4. 첫 화면에 필요한 데이터를 렌더링해야 하는가?
   → Server Component 우선

5. 사용자의 조작 이후에만 바뀌는 UI인가?
   → Client Component 고려
```

조금 더 구체적으로 보면 이렇게 나눌 수 있습니다.

| UI | 추천 |
| --- | --- |
| 페이지 레이아웃 | Server Component |
| 목록, 상세 본문 | Server Component |
| SEO가 중요한 콘텐츠 | Server Component |
| 좋아요 버튼 | Client Component |
| 모달 열기/닫기 | Client Component |
| 드롭다운, 탭, 아코디언 | Client Component |
| 폼 입력 상태 | Client Component |
| 폼 제출 후 서버 저장 | Server Action 또는 API 호출 |
| Provider | 별도 Client Component |

핵심은 페이지 전체를 클라이언트로 바꾸는 것이 아니라, **상호작용하는 작은 섬만 클라이언트로 분리하는 것**입니다.

---

## 흔히 하는 실수

### 1. 에러가 나면 무조건 page.tsx에 "use client" 붙이기

이렇게 하면 당장은 해결된 것처럼 보이지만, 서버 컴포넌트의 장점을 잃기 쉽습니다.

상호작용이 필요한 부분만 별도 컴포넌트로 분리하는 편이 좋습니다.

### 2. Server Component에서 이벤트 핸들러 넘기기

Server Component에서 만든 일반 함수는 Client Component props로 넘길 수 없습니다.

클라이언트 이벤트는 Client Component 안에서 만들고, 서버 작업은 Server Action이나 API로 연결합니다.

### 3. Client Component에서 서버 전용 모듈 import하기

DB client, secret, file system 코드는 브라우저로 가면 안 됩니다.

서버 전용 로직은 서버 영역에 둬야 합니다.

### 4. children 패턴을 모르고 구조를 억지로 뒤집기

Client Component 안에서 Server Component를 직접 import하려고 하기보다, Server Component에서 Client shell을 렌더링하고 그 안에 서버 렌더링 결과를 `children`으로 넣는 패턴을 사용하면 훨씬 자연스럽습니다.

### 5. props로 Date, Map, class instance를 그대로 넘기기

Server에서 Client로 넘기는 값은 직렬화 가능해야 합니다.

필요하다면 문자열이나 일반 객체로 변환해서 넘기는 편이 안전합니다.

```tsx
// 서버에서 Date를 문자열로 변환
<ClientDate createdAt={post.createdAt.toISOString()} />
```

---

## 추천 구조

App Router 프로젝트에서 자주 사용하는 구조는 다음과 같습니다.

```txt
app
├─ layout.tsx              Server Component
├─ providers.tsx           Client Component
└─ products
   └─ page.tsx             Server Component

components
├─ ProductList.tsx         Server Component
├─ ProductCard.tsx         Server Component
├─ AddToCartButton.tsx     Client Component
├─ ProductFilter.tsx       Client Component
└─ Modal.tsx               Client Component

lib
├─ db.ts                   server only
├─ api.ts                  server or shared
└─ format.ts               shared
```

이런 식으로 생각하면 역할이 분명해집니다.

```txt
app/page.tsx
→ 데이터를 가져오고 큰 화면 구조를 만든다

server component
→ 데이터 기반 UI를 렌더링한다

client component
→ 클릭, 입력, 상태, 브라우저 API를 담당한다

server action / route handler
→ 서버에서 실행되어야 하는 변경 작업을 담당한다
```

---

## 정리

Server Component와 Client Component의 경계를 이해하려면 먼저 기본값을 기억하면 됩니다.

```txt
Next.js App Router의 컴포넌트는 기본적으로 Server Component다.
"use client"는 클라이언트 경계를 여는 선언이다.
```

그리고 실무에서는 이렇게 판단하면 됩니다.

- 데이터 fetching, DB 접근, secret 사용은 Server Component 쪽에 둡니다.
- `useState`, `useEffect`, 이벤트 핸들러, 브라우저 API는 Client Component에서 사용합니다.
- `"use client"`는 가능한 한 작은 컴포넌트에 붙입니다.
- Server에서 Client로 넘기는 props는 직렬화 가능한 값이어야 합니다.
- Client Component 안에 Server Component를 직접 import하기보다 `children` 패턴을 사용합니다.
- Provider는 별도 Client Component로 분리하고, layout은 Server Component로 유지할 수 있습니다.

한 줄로 요약하면 이렇습니다.

```txt
서버에서 할 수 있는 일은 서버에 남기고
사용자와 직접 상호작용하는 부분만 클라이언트로 보낸다
```

이 기준이 잡히면 App Router의 구조가 훨씬 덜 낯설어집니다.

처음에는 `"use client"`를 어디에 붙일지 계속 고민하게 되지만, 점점 컴포넌트를 볼 때 자연스럽게 구분할 수 있게 됩니다.

```txt
이 컴포넌트는 데이터를 준비하는가?
아니면 사용자의 행동에 반응하는가?
```

이 질문 하나만 잘 붙잡아도 Server Component와 Client Component의 경계를 꽤 안정적으로 잡을 수 있습니다.

## 참고

- [Next.js 공식 문서: Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Next.js 공식 문서: use client directive](https://nextjs.org/docs/app/api-reference/directives/use-client)
- [React 공식 문서: use client](https://react.dev/reference/rsc/use-client)
- [React 공식 문서: use server](https://react.dev/reference/rsc/use-server)
