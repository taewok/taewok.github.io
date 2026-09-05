---
title: "[Next.js] dynamic 완벽하게 이해하고 사용하기"
date: 2026-09-05T00:30:00Z
categories: [frontend]
tags: [nextjs, react, dynamic, lazy-loading, code-splitting, performance]
description: "Next.js의 next/dynamic을 왜 사용하는지, React.lazy와의 차이, loading, ssr false, named export, 브라우저 전용 라이브러리 처리, App Router에서의 주의사항까지 정리했습니다."
custom_style: true
---

## 들어가며: dynamic은 성능 최적화 버튼이 아니에요

Next.js를 사용하다 보면 `dynamic`이라는 코드를 자주 보게 됩니다.

```tsx
import dynamic from "next/dynamic";

const Chart = dynamic(() => import("./Chart"));
```

처음 보면 이런 느낌이 듭니다.

```txt
컴포넌트를 dynamic으로 불러오면 성능이 좋아지는 건가?
그럼 모든 컴포넌트를 dynamic으로 바꾸면 되는 걸까?
ssr: false는 언제 쓰는 걸까?
React.lazy랑은 뭐가 다를까?
```

`dynamic`은 분명 유용한 기능입니다.

하지만 무조건 많이 쓴다고 좋은 것은 아닙니다.

오히려 잘못 사용하면 로딩 상태가 많아지고, 화면이 늦게 뜨고, 서버 렌더링 장점을 잃을 수도 있습니다.

이번 글에서는 Next.js의 `next/dynamic`을 제대로 이해하고, 어떤 상황에서 어떻게 사용해야 하는지 정리해보겠습니다.

---

## dynamic이 해결하려는 문제

일반적인 import는 페이지가 로드될 때 함께 묶입니다.

```tsx
import Chart from "./Chart";

export default function DashboardPage() {
  return <Chart />;
}
```

이렇게 작성하면 `DashboardPage`가 로드될 때 `Chart` 코드도 함께 필요해집니다.

만약 `Chart` 컴포넌트가 무거운 차트 라이브러리를 사용한다면 첫 화면에서 받아야 하는 JavaScript가 커질 수 있습니다.

하지만 차트가 처음부터 꼭 필요한 것이 아니라, 사용자가 버튼을 눌렀을 때만 보여도 된다면 어떨까요?

```txt
처음 페이지 진입
→ 차트 코드까지 미리 받을 필요 없음

사용자가 "차트 보기" 클릭
→ 그때 차트 코드 로드
```

이때 사용하는 것이 dynamic import입니다.

Next.js에서는 `next/dynamic`을 사용해 React 컴포넌트를 동적으로 불러올 수 있습니다.

```tsx
import dynamic from "next/dynamic";

const Chart = dynamic(() => import("./Chart"));
```

핵심은 이렇습니다.

```txt
처음부터 모든 JavaScript를 내려보내지 말고
필요해지는 순간에 일부 코드를 나중에 불러온다
```

---

## lazy loading과 code splitting

`dynamic`을 이해하려면 두 단어를 먼저 알아야 합니다.

```txt
lazy loading
→ 필요한 시점까지 로딩을 미루는 것

code splitting
→ JavaScript 코드를 여러 덩어리로 나누는 것
```

예를 들어 쇼핑몰 페이지에 리뷰 모달이 있다고 해보겠습니다.

모든 사용자가 리뷰 모달을 열지는 않습니다.

그런데 첫 화면부터 리뷰 모달 코드와 별점 라이브러리, 이미지 업로드 UI까지 모두 내려보내면 낭비가 될 수 있습니다.

```txt
상품 상세 페이지
├─ 상품 정보: 처음부터 필요
├─ 구매 버튼: 처음부터 필요
├─ 추천 상품: 처음부터 필요할 수 있음
└─ 리뷰 작성 모달: 열 때만 필요
```

이 경우 리뷰 작성 모달은 `dynamic`으로 나중에 불러오기 좋은 후보입니다.

```tsx
const ReviewModal = dynamic(() => import("./ReviewModal"));
```

이렇게 하면 리뷰 모달 코드는 별도 chunk로 나뉘고, 실제 렌더링이 필요할 때 로드됩니다.

---

## 기본 사용법

가장 기본적인 사용법은 다음과 같습니다.

```tsx
import dynamic from "next/dynamic";

const HeavyComponent = dynamic(() => import("./HeavyComponent"));

export default function Page() {
  return <HeavyComponent />;
}
```

`dynamic`에는 함수를 넘깁니다.

그 함수 안에서 `import()`를 호출합니다.

```tsx
dynamic(() => import("./HeavyComponent"));
```

여기서 `import("./HeavyComponent")`는 Promise를 반환합니다.

Next.js는 이 Promise가 해결되면 해당 컴포넌트를 렌더링합니다.

React의 `lazy`와 비슷하게 동작하지만, Next.js 환경에 맞는 옵션을 함께 제공합니다.

---

## loading 옵션으로 로딩 UI 보여주기

동적으로 불러오는 컴포넌트는 코드가 아직 로드되지 않았을 수 있습니다.

이때 사용자에게 빈 화면을 보여주기보다 로딩 UI를 보여주는 것이 좋습니다.

```tsx
import dynamic from "next/dynamic";

const HeavyComponent = dynamic(() => import("./HeavyComponent"), {
  loading: () => <p>불러오는 중...</p>,
});

export default function Page() {
  return <HeavyComponent />;
}
```

`loading` 옵션은 컴포넌트 코드가 로드되는 동안 보여줄 UI입니다.

실무에서는 단순 텍스트보다 skeleton UI를 자주 사용합니다.

```tsx
const ProductChart = dynamic(() => import("./ProductChart"), {
  loading: () => <div className="h-64 rounded-lg bg-gray-100" />,
});
```

다만 로딩 UI를 너무 많이 만들면 화면이 오히려 산만해질 수 있습니다.

중요한 컴포넌트라면 로딩 UI를 의미 있게 만들고, 아주 작은 컴포넌트라면 굳이 dynamic으로 나누지 않는 편이 낫습니다.

---

## 조건부 렌더링과 함께 쓰기

`dynamic`의 장점은 조건부 렌더링과 함께 사용할 때 더 잘 드러납니다.

```tsx
"use client";

import { useState } from "react";
import dynamic from "next/dynamic";

const ReviewModal = dynamic(() => import("./ReviewModal"), {
  loading: () => <p>리뷰 작성창을 불러오는 중...</p>,
});

export default function ProductActions() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button type="button" onClick={() => setIsOpen(true)}>
        리뷰 작성
      </button>

      {isOpen && (
        <ReviewModal onClose={() => setIsOpen(false)} />
      )}
    </>
  );
}
```

이 구조에서는 사용자가 `리뷰 작성` 버튼을 누르기 전까지 `ReviewModal`이 렌더링되지 않습니다.

그래서 모달 코드도 필요한 순간까지 미룰 수 있습니다.

```txt
페이지 진입
→ ProductActions 로드

리뷰 작성 클릭
→ ReviewModal chunk 로드
→ 모달 렌더링
```

모달, 차트, 에디터, 지도처럼 무겁고 자주 사용되지 않는 UI에 잘 어울립니다.

---

## ssr: false는 언제 사용할까?

`dynamic`을 검색하면 가장 많이 보이는 옵션이 `ssr: false`입니다.

```tsx
const Map = dynamic(() => import("./Map"), {
  ssr: false,
});
```

이 옵션은 해당 Client Component를 서버에서 미리 렌더링하지 않고, 브라우저에서만 렌더링하겠다는 뜻입니다.

주로 이런 경우에 사용합니다.

- `window`를 직접 사용하는 라이브러리
- `document`를 직접 사용하는 라이브러리
- 브라우저에서만 동작하는 지도 라이브러리
- 에디터 라이브러리
- 차트 라이브러리 중 SSR을 지원하지 않는 경우

예를 들어 어떤 지도 컴포넌트가 내부에서 `window`를 바로 사용한다고 해보겠습니다.

```tsx
// Map.tsx
"use client";

export default function Map() {
  console.log(window.innerWidth);

  return <div>지도</div>;
}
```

서버에는 `window`가 없습니다.

그래서 서버 렌더링 중 에러가 날 수 있습니다.

이럴 때 `ssr: false`를 사용합니다.

```tsx
"use client";

import dynamic from "next/dynamic";

const Map = dynamic(() => import("./Map"), {
  ssr: false,
  loading: () => <p>지도를 불러오는 중...</p>,
});

export default function MapSection() {
  return <Map />;
}
```

하지만 `ssr: false`는 조심해서 사용해야 합니다.

서버에서 렌더링하지 않으므로 초기 HTML에는 해당 컴포넌트 내용이 들어가지 않습니다.

SEO가 중요한 콘텐츠나 첫 화면 핵심 정보에는 어울리지 않습니다.

```txt
브라우저 전용 라이브러리 때문에 서버에서 렌더링할 수 없다
→ ssr: false 고려

그냥 귀찮아서 서버 에러를 피하고 싶다
→ 구조를 먼저 점검
```

---

## ssr: false는 Client Component 안에서 사용하기

App Router에서는 `ssr: false`를 Server Component에서 사용할 수 없습니다.

이 옵션은 Client Component 쪽으로 옮겨야 합니다.

```tsx
// components/MapSection.tsx
"use client";

import dynamic from "next/dynamic";

const Map = dynamic(() => import("./Map"), {
  ssr: false,
});

export function MapSection() {
  return <Map />;
}
```

그리고 Server Component에서는 이 Client Component를 렌더링합니다.

```tsx
// app/page.tsx
import { MapSection } from "@/components/MapSection";

export default function Page() {
  return (
    <main>
      <h1>매장 위치</h1>
      <MapSection />
    </main>
  );
}
```

이렇게 하면 서버 컴포넌트 구조를 유지하면서, 브라우저 전용 UI만 클라이언트에서 렌더링할 수 있습니다.

---

## named export를 dynamic으로 불러오기

컴포넌트가 default export가 아니라 named export라면 `.then`으로 꺼내면 됩니다.

```tsx
// components/Chart.tsx
"use client";

export function Chart() {
  return <div>차트</div>;
}
```

```tsx
import dynamic from "next/dynamic";

const Chart = dynamic(() =>
  import("./Chart").then((mod) => mod.Chart),
);
```

이 패턴은 공식 문서에서도 소개되는 방식입니다.

다만 개인적으로는 dynamic으로 자주 불러올 컴포넌트라면 default export로 두는 편이 읽기 쉽다고 느낍니다.

```tsx
const Chart = dynamic(() => import("./Chart"));
```

팀의 export 스타일에 맞춰 선택하면 됩니다.

---

## 외부 라이브러리는 import()로 직접 불러올 수도 있어요

`next/dynamic`은 React 컴포넌트를 동적으로 불러올 때 사용하는 도구입니다.

컴포넌트가 아니라 일반 JavaScript 라이브러리를 이벤트 시점에 불러오고 싶다면 `import()`를 직접 사용할 수 있습니다.

예를 들어 검색어를 입력했을 때만 `fuse.js`를 불러와 검색한다고 해보겠습니다.

```tsx
"use client";

import { useState } from "react";

const names = ["React", "Next.js", "TypeScript", "Zustand"];

export default function SearchBox() {
  const [results, setResults] = useState<string[]>([]);

  const handleChange = async (
    event: React.ChangeEvent<HTMLInputElement>,
  ) => {
    const keyword = event.currentTarget.value;

    if (!keyword) {
      setResults([]);
      return;
    }

    const Fuse = (await import("fuse.js")).default;
    const fuse = new Fuse(names);
    const searchResults = fuse.search(keyword);

    setResults(searchResults.map((result) => result.item));
  };

  return (
    <div>
      <input type="search" onChange={handleChange} />

      <ul>
        {results.map((result) => (
          <li key={result}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

여기서는 `dynamic`을 사용하지 않았습니다.

불러오는 대상이 React 컴포넌트가 아니라 라이브러리이기 때문입니다.

정리하면 이렇습니다.

```txt
React 컴포넌트를 늦게 불러오기
→ next/dynamic

이벤트 시점에 일반 라이브러리 불러오기
→ import()
```

---

## React.lazy와 next/dynamic의 차이

React에는 `lazy`가 있습니다.

```tsx
import { lazy, Suspense } from "react";

const Chart = lazy(() => import("./Chart"));

export default function Page() {
  return (
    <Suspense fallback={<p>불러오는 중...</p>}>
      <Chart />
    </Suspense>
  );
}
```

React 공식 문서에 따르면 `lazy`는 컴포넌트 코드가 처음 렌더링될 때까지 로딩을 미룰 수 있게 해줍니다.

Next.js의 `dynamic`은 `React.lazy`와 `Suspense`를 조합한 기능처럼 동작하면서, Next.js에서 필요한 옵션을 제공합니다.

대표적으로 `ssr: false` 같은 옵션은 `next/dynamic`에서 자주 사용합니다.

| 구분 | React.lazy | next/dynamic |
| --- | --- | --- |
| 제공 주체 | React | Next.js |
| 목적 | 컴포넌트 lazy loading | Next.js 환경의 컴포넌트 lazy loading |
| fallback | `Suspense` 사용 | `loading` 옵션 또는 Suspense |
| SSR 제어 | 직접 옵션 없음 | `ssr: false` 제공 |
| Next.js 사용성 | 가능 | Next.js에서 더 자주 사용 |

Next.js 프로젝트에서는 보통 `next/dynamic`을 우선 고려하면 됩니다.

---

## dynamic을 쓰면 무조건 좋은 걸까?

아닙니다.

작은 컴포넌트까지 모두 dynamic으로 쪼개면 오히려 복잡해질 수 있습니다.

예를 들어 아주 작은 버튼 컴포넌트를 dynamic으로 불러오는 것은 보통 의미가 크지 않습니다.

```tsx
const SmallButton = dynamic(() => import("./SmallButton"));
```

이 경우 얻는 이점보다 비동기 로딩과 chunk 요청 비용이 더 거슬릴 수 있습니다.

`dynamic`이 잘 어울리는 대상은 보통 이런 컴포넌트입니다.

- 처음 화면에 꼭 필요하지 않은 컴포넌트
- 크기가 큰 컴포넌트
- 무거운 외부 라이브러리를 포함한 컴포넌트
- 사용자가 특정 행동을 해야만 필요한 컴포넌트
- 브라우저에서만 동작하는 컴포넌트

반대로 이런 경우에는 굳이 dynamic을 쓰지 않는 편이 좋습니다.

- 첫 화면에서 반드시 보여야 하는 핵심 콘텐츠
- SEO에 중요한 콘텐츠
- 아주 작은 공통 버튼, 아이콘, 텍스트 컴포넌트
- 대부분의 사용자가 항상 보는 컴포넌트
- 로딩 UI가 더 어색한 컴포넌트

핵심은 다음 질문입니다.

```txt
이 코드를 처음 페이지 진입 시점에 꼭 받아야 할까?
```

아니라면 dynamic을 고려할 수 있습니다.

---

## App Router에서 알아야 할 주의사항

Next.js App Router에서는 Server Component와 Client Component가 함께 동작합니다.

이때 `dynamic`을 사용할 때 몇 가지 주의할 점이 있습니다.

### 1. lazy loading은 주로 Client Component에 적용됩니다

Next.js 공식 문서에 따르면 Server Component는 기본적으로 code split될 수 있고, streaming을 통해 UI를 점진적으로 보낼 수 있습니다.

그리고 lazy loading은 Client Component에 적용됩니다.

즉, dynamic으로 큰 효과를 기대하는 대상은 보통 Client Component입니다.

예를 들어 모달, 차트, 에디터처럼 브라우저 JavaScript가 필요한 컴포넌트입니다.

### 2. Server Component에서 Client Component를 dynamic import할 때 자동 code splitting 제한이 있습니다

공식 문서에서는 Server Component가 Client Component를 dynamic import하는 경우 automatic code splitting이 현재 지원되지 않는다고 안내합니다.

그래서 클라이언트 code splitting을 의도한다면 Client Component 경계 안에서 dynamic import를 사용하는 구조가 더 명확합니다.

```tsx
// components/ChartSection.tsx
"use client";

import dynamic from "next/dynamic";

const Chart = dynamic(() => import("./Chart"));

export function ChartSection() {
  return <Chart />;
}
```

```tsx
// app/dashboard/page.tsx
import { ChartSection } from "@/components/ChartSection";

export default function DashboardPage() {
  return (
    <main>
      <h1>대시보드</h1>
      <ChartSection />
    </main>
  );
}
```

### 3. Server Component 자체를 dynamic으로 불러오는 것은 기대와 다를 수 있습니다

Server Component를 dynamic import하면 Server Component 자체가 클라이언트에서 늦게 다운로드되는 방식으로 동작하는 것은 아닙니다.

공식 문서에서는 Server Component를 동적으로 import하면 그 Server Component의 자식인 Client Component만 lazy load된다고 설명합니다.

그래서 Server Component의 데이터 fetching을 늦게 하고 싶다면 `dynamic`만으로 해결하려고 하기보다 route 분리, Suspense, streaming, 조건부 UI 설계를 함께 봐야 합니다.

---

## 브라우저 전용 라이브러리 처리 예시

실무에서 `dynamic`을 가장 많이 쓰는 상황 중 하나는 브라우저 전용 라이브러리입니다.

예를 들어 에디터 컴포넌트가 있다고 해보겠습니다.

```tsx
// components/RichEditor.tsx
"use client";

import Editor from "some-browser-editor";

export default function RichEditor() {
  return <Editor />;
}
```

이 라이브러리가 내부에서 `window`나 `document`를 사용한다면 서버 렌더링 중 문제가 생길 수 있습니다.

이때 감싸는 컴포넌트를 만들고 `ssr: false`를 적용합니다.

```tsx
// components/RichEditorClient.tsx
"use client";

import dynamic from "next/dynamic";

const RichEditor = dynamic(() => import("./RichEditor"), {
  ssr: false,
  loading: () => <div className="h-80 rounded-lg bg-gray-100" />,
});

export function RichEditorClient() {
  return <RichEditor />;
}
```

이제 페이지에서는 wrapper를 사용합니다.

```tsx
// app/write/page.tsx
import { RichEditorClient } from "@/components/RichEditorClient";

export default function WritePage() {
  return (
    <main>
      <h1>글 작성</h1>
      <RichEditorClient />
    </main>
  );
}
```

이 구조의 장점은 명확합니다.

```txt
페이지 구조는 서버 컴포넌트로 유지
브라우저 전용 에디터만 클라이언트에서 로드
로딩 중에는 skeleton 표시
```

---

## 모달을 dynamic으로 분리하기

모달은 dynamic과 잘 어울리는 대표적인 UI입니다.

처음 화면에서 보이지 않고, 사용자가 버튼을 눌러야 열리는 경우가 많기 때문입니다.

```tsx
"use client";

import { useState } from "react";
import dynamic from "next/dynamic";

const LoginModal = dynamic(() => import("./LoginModal"), {
  loading: () => null,
});

export function LoginButton() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button type="button" onClick={() => setIsOpen(true)}>
        로그인
      </button>

      {isOpen && (
        <LoginModal onClose={() => setIsOpen(false)} />
      )}
    </>
  );
}
```

여기서 `loading: () => null`을 사용한 이유는 모달 chunk가 아주 빨리 로드될 수도 있고, 애매한 로딩 문구가 순간적으로 깜빡이는 것이 더 어색할 수 있기 때문입니다.

반면 모달이 무겁고 로딩 시간이 느껴진다면 작은 skeleton이나 spinner를 줄 수 있습니다.

```tsx
const LoginModal = dynamic(() => import("./LoginModal"), {
  loading: () => <p>로그인 창을 준비하고 있어요.</p>,
});
```

로딩 UI는 항상 필요하다기보다 사용자가 기다림을 느낄 만한 상황에서 의미가 있습니다.

---

## 차트를 dynamic으로 분리하기

차트 라이브러리는 번들 크기가 커질 수 있습니다.

그리고 대시보드에서 모든 차트를 처음부터 볼 필요가 없는 경우도 많습니다.

```tsx
"use client";

import dynamic from "next/dynamic";

const SalesChart = dynamic(() => import("./SalesChart"), {
  loading: () => <div className="h-72 rounded-lg bg-gray-100" />,
});

export function SalesChartSection() {
  return (
    <section>
      <h2>매출 추이</h2>
      <SalesChart />
    </section>
  );
}
```

차트가 첫 화면 아래쪽에 있다면 dynamic을 고려할 수 있습니다.

다만 첫 화면의 핵심 정보가 차트라면 무조건 늦게 불러오는 것이 답은 아닙니다.

사용자가 가장 먼저 봐야 하는 정보라면 빠르게 보여주는 쪽이 더 중요할 수 있습니다.

---

## dynamic 컴포넌트는 컴포넌트 밖에서 선언하기

`dynamic`으로 만든 컴포넌트는 보통 파일 최상단에 선언합니다.

```tsx
import dynamic from "next/dynamic";

const Chart = dynamic(() => import("./Chart"));

export default function Page() {
  return <Chart />;
}
```

컴포넌트 함수 안에서 매번 선언하는 것은 피하는 편이 좋습니다.

```tsx
export default function Page() {
  const Chart = dynamic(() => import("./Chart"));

  return <Chart />;
}
```

이렇게 하면 렌더링할 때마다 dynamic 컴포넌트 선언이 다시 만들어질 수 있고, 상태 유지나 로딩 동작을 이해하기 어려워집니다.

React 공식 문서에서도 lazy component는 컴포넌트 바깥에서 선언하라고 안내합니다.

`dynamic`도 같은 감각으로 다루면 됩니다.

```txt
dynamic 선언은 module scope
렌더링 함수 안에서는 사용만 하기
```

---

## 여러 컴포넌트를 한 번에 dynamic으로 불러와도 될까?

가능은 하지만 신중해야 합니다.

```tsx
const Chart = dynamic(() => import("./Chart"));
const Editor = dynamic(() => import("./Editor"));
const Map = dynamic(() => import("./Map"));
```

이렇게 나누면 각각 필요한 시점에 로드될 수 있습니다.

하지만 너무 잘게 나누면 요청이 많아지고 로딩 상태도 많아집니다.

다음 기준으로 판단하면 좋습니다.

```txt
무겁고 사용 시점이 다르다
→ 따로 dynamic

항상 함께 사용된다
→ 하나의 컴포넌트로 묶거나 일반 import
```

예를 들어 차트와 차트 필터가 항상 함께 보인다면 굳이 따로 쪼갤 필요가 없을 수 있습니다.

반대로 에디터, 지도, 이미지 크롭 모달처럼 서로 사용 시점이 다르다면 따로 dynamic으로 분리할 수 있습니다.

---

## dynamic을 쓰기 전 체크리스트

`dynamic`을 적용하기 전에 아래 질문을 해보면 좋습니다.

```txt
1. 이 컴포넌트는 첫 화면에 꼭 필요한가?
2. 이 컴포넌트가 무거운 라이브러리를 포함하는가?
3. 대부분의 사용자가 항상 보는 컴포넌트인가?
4. 사용자의 클릭 이후에만 필요한가?
5. 서버 렌더링이 꼭 필요한 콘텐츠인가?
6. SEO에 중요한 내용인가?
7. 로딩 중 보여줄 UI가 자연스러운가?
8. 브라우저 전용 API 때문에 SSR이 깨지는가?
```

추천 판단은 이렇게 정리할 수 있습니다.

| 상황 | dynamic 추천 |
| --- | --- |
| 클릭해야 열리는 모달 | 좋음 |
| 무거운 차트 | 좋음 |
| 브라우저 전용 지도 | `ssr: false`와 함께 고려 |
| 글 본문 | 보통 비추천 |
| SEO가 중요한 상품명/가격 | 비추천 |
| 작은 버튼 컴포넌트 | 보통 비추천 |
| 페이지 하단의 무거운 위젯 | 좋음 |
| 항상 첫 화면에 보이는 핵심 UI | 신중하게 판단 |

---

## 자주 하는 실수

### 1. 모든 컴포넌트에 dynamic 사용하기

`dynamic`은 코드를 나누는 도구입니다.

모든 컴포넌트를 나누면 오히려 로딩과 요청 관리가 복잡해질 수 있습니다.

작고 항상 필요한 컴포넌트는 일반 import가 더 낫습니다.

### 2. ssr: false를 에러 숨기기 용도로 사용하기

서버에서 에러가 난다고 무조건 `ssr: false`를 붙이면 서버 렌더링 장점을 잃을 수 있습니다.

브라우저 전용 코드인지, 컴포넌트 분리가 필요한지 먼저 확인해야 합니다.

### 3. SEO가 중요한 콘텐츠를 ssr: false로 숨기기

상품명, 가격, 글 제목, 본문처럼 검색 엔진과 초기 사용자에게 중요한 콘텐츠는 서버에서 렌더링되는 것이 좋습니다.

이런 영역을 `ssr: false`로 늦게 렌더링하면 초기 HTML이 빈약해질 수 있습니다.

### 4. loading UI를 너무 크게 만들기

dynamic component의 loading UI가 실제 컴포넌트보다 더 눈에 띄면 사용자 경험이 어색합니다.

가능하면 실제 컴포넌트 크기와 비슷한 skeleton을 사용하는 것이 좋습니다.

### 5. 일반 라이브러리 import에 next/dynamic 사용하기

`next/dynamic`은 React 컴포넌트를 위한 도구입니다.

일반 라이브러리는 이벤트 핸들러 안에서 `import()`를 직접 사용하는 편이 자연스럽습니다.

---

## 실무에서 추천하는 사용 패턴

개인적으로는 다음 순서로 판단하는 편이 좋다고 생각합니다.

```txt
1. 일단 일반 import로 작성한다.
2. 번들 크기나 초기 로딩 문제가 보이면 후보를 찾는다.
3. 처음 화면에 필요 없는 무거운 컴포넌트를 dynamic으로 분리한다.
4. 브라우저 전용 라이브러리라면 Client Component 안에서 ssr: false를 적용한다.
5. 로딩 UI가 필요한지 확인한다.
6. 적용 전후의 빌드 결과와 사용자 경험을 비교한다.
```

처음부터 모든 것을 dynamic으로 만들기보다, 실제로 늦게 불러올 가치가 있는 컴포넌트에만 적용하는 것이 좋습니다.

특히 Next.js에서는 Server Component, streaming, route 단위 code splitting 같은 기능도 함께 동작합니다.

`dynamic`은 그중 하나의 도구입니다.

---

## 정리

`next/dynamic`은 Next.js에서 React 컴포넌트를 동적으로 불러오는 기능입니다.

핵심 목적은 처음에 필요한 JavaScript를 줄이고, 특정 컴포넌트 코드를 필요한 시점까지 미루는 것입니다.

다시 정리하면 다음과 같습니다.

- `dynamic(() => import("./Component"))`로 컴포넌트를 동적으로 불러옵니다.
- `loading` 옵션으로 로딩 중 UI를 보여줄 수 있습니다.
- `ssr: false`는 브라우저에서만 렌더링해야 하는 Client Component에 사용합니다.
- named export는 `.then((mod) => mod.Component)` 형태로 꺼낼 수 있습니다.
- 일반 라이브러리는 `next/dynamic`보다 `import()`를 직접 사용하는 것이 자연스럽습니다.
- 작은 컴포넌트나 첫 화면 핵심 콘텐츠에는 굳이 dynamic을 쓰지 않는 편이 좋습니다.
- App Router에서는 Client Component lazy loading이라는 관점으로 이해하는 것이 중요합니다.

한 줄로 요약하면 이렇습니다.

```txt
dynamic은 컴포넌트를 늦게 불러오는 도구이고
진짜 목적은 필요한 JavaScript를 필요한 순간에만 보내는 것이다
```

`dynamic`을 잘 쓰려면 문법보다 판단이 더 중요합니다.

```txt
이 컴포넌트는 지금 당장 필요한가?
아니면 사용자가 어떤 행동을 한 뒤에 필요해지는가?
```

이 질문에 답할 수 있으면 `dynamic`을 꽤 안정적으로 사용할 수 있습니다.

## 참고

- [Next.js 공식 문서: Lazy Loading](https://nextjs.org/docs/app/guides/lazy-loading)
- [Next.js Learn: Dynamic Imports](https://nextjs.org/learn/seo/dynamic-imports)
- [React 공식 문서: lazy](https://react.dev/reference/react/lazy)
- [MDN: Dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import)
