---
title: "[React] react-virtual 완벽 이해하기: TanStack Virtual로 긴 리스트 최적화하기"
date: 2026-09-19T00:30:00Z
categories: [frontend]
tags: [react, react-virtual, tanstack-virtual, virtualization, performance, list]
description: "react-virtual의 현재 이름인 TanStack Virtual을 기준으로 가상 스크롤의 원리, useVirtualizer 사용법, overscan, 동적 높이, scrollToIndex, 무한 스크롤, 실무 주의사항까지 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: 리스트가 많아지면 왜 느려질까?

React에서 목록을 렌더링할 때 처음에는 보통 이렇게 작성합니다.

```tsx
function UserList({ users }: { users: User[] }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

데이터가 20개, 100개 정도라면 문제 없습니다.

하지만 10,000개라면 어떨까요?

```txt
10,000개의 li 생성
10,000개의 React element 비교
10,000개의 DOM 노드 유지
10,000개의 스타일 계산과 레이아웃 계산
스크롤할 때마다 많은 브라우저 작업 발생
```

사용자는 화면에서 한 번에 20개 정도만 보고 있는데, 브라우저는 보이지 않는 수천 개의 DOM까지 관리해야 합니다.

이때 사용하는 기법이 **가상화**입니다.

영어로는 `virtualization`, `virtual scrolling`, `windowing`이라고 부릅니다.

React 생태계에서는 예전에 `react-virtual`이라는 이름으로 많이 불렸고, 현재는 **TanStack Virtual**의 React 어댑터인 `@tanstack/react-virtual`을 주로 사용합니다.

이번 글에서는 `react-virtual`을 단순히 복사해서 쓰는 수준이 아니라, 왜 이런 구조로 작성하는지까지 이해해보겠습니다.

---

## react-virtual과 TanStack Virtual은 같은 걸까?

먼저 이름부터 정리해야 합니다.

예전에는 `react-virtual`이라는 이름이 익숙했습니다.

현재 공식 패키지는 보통 아래처럼 설치합니다.

```bash
npm install @tanstack/react-virtual
```

또는 yarn, pnpm을 사용한다면 이렇게 설치할 수 있습니다.

```bash
yarn add @tanstack/react-virtual
```

```bash
pnpm add @tanstack/react-virtual
```

TanStack Virtual은 React 전용 라이브러리가 아닙니다.

핵심 로직은 프레임워크에 독립적이고, React에서는 `@tanstack/react-virtual` 어댑터를 통해 사용합니다.

```tsx
import { useVirtualizer } from "@tanstack/react-virtual";
```

정리하면 이렇습니다.

| 표현 | 의미 |
| --- | --- |
| react-virtual | 예전부터 많이 부르던 이름, React에서 쓰는 가상화 라이브러리라는 의미로 사용됨 |
| TanStack Virtual | 현재 공식 라이브러리 이름 |
| @tanstack/react-virtual | React에서 사용하는 패키지 |
| useVirtualizer | React 컴포넌트에서 가상 리스트를 만드는 훅 |

그래서 실무에서는 "react-virtual 쓴다"라고 말해도, 실제 설치와 import는 `@tanstack/react-virtual`인 경우가 많습니다.

---

## 가상화의 핵심 아이디어

가상화의 핵심은 단순합니다.

```txt
전체 데이터를 모두 렌더링하지 않고,
현재 화면에 보이는 아이템과 그 주변 아이템만 렌더링한다.
```

예를 들어 데이터가 10,000개 있다고 해보겠습니다.

사용자의 화면에 실제로 보이는 아이템은 15개입니다.

그러면 브라우저에는 대략 15개에서 30개 정도만 렌더링합니다.

```txt
데이터 개수: 10,000개
실제 DOM 노드: 약 20개
스크롤 위치에 따라 DOM 노드가 바뀜
사용자는 전체 목록이 있는 것처럼 느낌
```

여기서 중요한 점은 **데이터를 줄이는 것이 아니라 DOM 렌더링 수를 줄이는 것**입니다.

```txt
데이터는 10,000개 그대로 있음
하지만 DOM에는 보이는 영역 근처의 아이템만 존재함
```

그래서 가상화는 다음 상황에 잘 맞습니다.

- 긴 테이블
- 긴 채팅 목록
- 로그 뷰어
- 검색 결과 목록
- 알림 목록
- 선택 가능한 대용량 옵션 목록
- 카드 그리드

반대로 아이템이 30개 정도라면 굳이 가상화를 쓰지 않아도 됩니다.

가상화는 구조가 조금 복잡해지고, 접근성이나 포커스 관리도 더 신경 써야 하기 때문입니다.

---

## TanStack Virtual의 렌더링 구조

가상 리스트는 보통 세 겹 구조로 만듭니다.

```txt
스크롤 컨테이너
└─ 전체 높이를 가진 내부 영역
   └─ 실제로 보이는 아이템들
```

React 코드로 보면 이런 모양입니다.

```tsx
<div className="scroll-container">
  <div className="total-size">
    <div className="virtual-row">Row 120</div>
    <div className="virtual-row">Row 121</div>
    <div className="virtual-row">Row 122</div>
  </div>
</div>
```

각 영역의 역할은 다릅니다.

| 영역 | 역할 |
| --- | --- |
| 스크롤 컨테이너 | 실제 스크롤이 발생하는 박스 |
| 전체 높이 영역 | 10,000개가 모두 있는 것처럼 스크롤바 길이를 만들어줌 |
| virtual item | 현재 보이는 아이템만 absolute 위치로 배치 |

핵심은 `전체 높이 영역`입니다.

실제 DOM 아이템은 20개뿐인데, 스크롤바는 10,000개가 있는 것처럼 보여야 합니다.

그래서 내부 영역에 전체 목록 높이를 부여합니다.

```tsx
height: `${rowVirtualizer.getTotalSize()}px`
```

그리고 실제 렌더링하는 아이템은 `position: absolute`와 `transform`으로 제 위치에 놓습니다.

```tsx
transform: `translateY(${virtualRow.start}px)`
```

즉, 사용자는 전체 리스트를 스크롤하는 것처럼 느끼지만, 실제로는 보이는 아이템만 계속 갈아끼우는 구조입니다.

---

## 가장 기본 예제: 고정 높이 리스트

먼저 모든 아이템의 높이가 같은 경우를 보겠습니다.

```tsx
import { useRef } from "react";
import { useVirtualizer } from "@tanstack/react-virtual";

type User = {
  id: string;
  name: string;
  email: string;
};

type UserListProps = {
  users: User[];
};

export function UserList({ users }: UserListProps) {
  const parentRef = useRef<HTMLDivElement | null>(null);

  const rowVirtualizer = useVirtualizer({
    count: users.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 56,
    overscan: 5,
  });

  return (
    <div ref={parentRef} className="user-list-scroll">
      <div
        className="user-list-inner"
        style={{
          height: `${rowVirtualizer.getTotalSize()}px`,
        }}
      >
        {rowVirtualizer.getVirtualItems().map((virtualRow) => {
          const user = users[virtualRow.index];

          return (
            <div
              key={virtualRow.key}
              className="user-list-row"
              style={{
                height: `${virtualRow.size}px`,
                transform: `translateY(${virtualRow.start}px)`,
              }}
            >
              <strong>{user.name}</strong>
              <span>{user.email}</span>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

CSS는 이런 형태가 됩니다.

```css
.user-list-scroll {
  height: 420px;
  overflow: auto;
  border: 1px solid #e5e7eb;
}

.user-list-inner {
  position: relative;
  width: 100%;
}

.user-list-row {
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  border-bottom: 1px solid #f1f5f9;
  box-sizing: border-box;
}
```

처음 보면 코드가 조금 낯설 수 있습니다.

하지만 핵심은 네 가지입니다.

```tsx
const rowVirtualizer = useVirtualizer({
  count: users.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 56,
  overscan: 5,
});
```

| 옵션 | 의미 |
| --- | --- |
| `count` | 전체 아이템 개수 |
| `getScrollElement` | 스크롤이 발생하는 DOM 요소 |
| `estimateSize` | 아이템 하나의 예상 높이 |
| `overscan` | 화면 밖 위아래로 미리 렌더링할 아이템 개수 |

그리고 렌더링할 때는 전체 데이터가 아니라 `getVirtualItems()`를 사용합니다.

```tsx
rowVirtualizer.getVirtualItems().map((virtualRow) => {
  const user = users[virtualRow.index];
});
```

`virtualRow.index`는 실제 데이터 배열에서 가져올 인덱스입니다.

즉, `users.map()`이 아니라 `virtualItems.map()`을 돌고, 그 안에서 원본 데이터에 접근합니다.

---

## virtualRow에는 무엇이 들어 있을까?

`getVirtualItems()`가 반환하는 값은 실제 데이터가 아닙니다.

가상 아이템의 위치 정보를 담은 객체입니다.

대표적으로 이런 값들이 있습니다.

| 값 | 의미 |
| --- | --- |
| `index` | 원본 데이터에서의 인덱스 |
| `key` | React key로 사용할 값 |
| `start` | 리스트 전체 기준 시작 위치 |
| `end` | 리스트 전체 기준 끝 위치 |
| `size` | 아이템 크기 |
| `lane` | 여러 lane을 사용할 때 배치된 lane |

그래서 아래 코드가 가능해집니다.

```tsx
const user = users[virtualRow.index];
```

그리고 이 아이템을 원래 있어야 할 위치로 옮깁니다.

```tsx
style={{
  height: `${virtualRow.size}px`,
  transform: `translateY(${virtualRow.start}px)`,
}}
```

여기서 `translateY`를 쓰는 이유는 실제 DOM 노드가 적기 때문입니다.

예를 들어 500번째 아이템이 화면에 보여야 한다면, 그 아이템을 500번째 위치까지 밀어줘야 합니다.

```txt
0번째부터 499번째까지 DOM을 실제로 만들지 않음
대신 500번째 아이템을 원래 위치만큼 translateY로 이동
```

이게 가상 리스트의 기본 원리입니다.

---

## estimateSize는 왜 필요할까?

가상화 라이브러리는 전체 아이템이 실제로 렌더링되기 전에 스크롤바 크기를 계산해야 합니다.

예를 들어 아이템이 10,000개이고, 각 아이템이 56px이라고 예상한다면 전체 높이는 이렇습니다.

```txt
10,000 * 56px = 560,000px
```

그래서 `estimateSize`가 필요합니다.

```tsx
estimateSize: () => 56
```

고정 높이 리스트라면 실제 높이와 같은 값을 넣으면 됩니다.

동적 높이 리스트라면 평균보다 약간 큰 값을 넣는 편이 안정적입니다.

공식 문서에서도 동적으로 측정하는 경우 가능한 범위 안에서 큰 쪽으로 추정하면 초기 위치 계산이 더 안정적이라고 설명합니다.

```tsx
estimateSize: () => 96
```

너무 작은 값을 넣으면 실제 측정 후 아이템 위치가 크게 밀리면서 스크롤 점프처럼 느껴질 수 있습니다.

너무 큰 값을 넣으면 처음 스크롤바가 실제보다 길게 보이다가 측정 후 줄어들 수 있습니다.

실무에서는 대략 이런 식으로 잡습니다.

| 아이템 형태 | estimateSize 예시 |
| --- | --- |
| 한 줄 목록 | 40 ~ 56 |
| 두 줄 카드형 목록 | 72 ~ 120 |
| 이미지가 있는 카드 | 160 이상 |
| 채팅 메시지 | 평균 메시지 높이보다 조금 크게 |

---

## overscan은 왜 필요할까?

가상화는 보이는 아이템만 렌더링합니다.

그런데 정말 화면 안에 딱 보이는 아이템만 렌더링하면 빠르게 스크롤할 때 빈 화면이 잠깐 보일 수 있습니다.

그래서 화면 밖 아이템을 조금 더 미리 렌더링합니다.

이게 `overscan`입니다.

```tsx
overscan: 5
```

의미는 대략 이렇습니다.

```txt
화면에 보이는 아이템
+ 위쪽 여분 5개
+ 아래쪽 여분 5개
```

`overscan`을 크게 하면 스크롤은 부드러워질 수 있지만 렌더링하는 DOM이 많아집니다.

작게 하면 DOM은 줄어들지만 빠른 스크롤에서 빈 공간이 보일 수 있습니다.

| 상황 | overscan |
| --- | --- |
| 단순 텍스트 목록 | 3 ~ 5 |
| 렌더링이 가벼운 행 | 5 ~ 10 |
| 카드가 무겁고 이미지가 많음 | 2 ~ 5 |
| 빠른 휠 스크롤이 중요한 화면 | 조금 크게 |

정답은 고정되어 있지 않습니다.

개발자 도구 Performance 탭과 실제 기기에서 스크롤 느낌을 보면서 조정하는 값입니다.

---

## key는 index보다 실제 id가 좋다

기본적으로 virtualizer는 index를 key처럼 사용할 수 있습니다.

하지만 데이터가 추가, 삭제, 정렬될 수 있다면 실제 id를 사용하는 편이 좋습니다.

```tsx
const rowVirtualizer = useVirtualizer({
  count: users.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 56,
  getItemKey: (index) => users[index].id,
});
```

그리고 렌더링에서는 `virtualRow.key`를 사용합니다.

```tsx
<div key={virtualRow.key}>
  {users[virtualRow.index].name}
</div>
```

왜 중요할까요?

index key를 사용하면 중간에 아이템이 추가되거나 삭제될 때 React가 다른 아이템을 같은 DOM으로 재사용할 수 있습니다.

특히 아래 상황에서 문제가 커집니다.

- 체크박스가 있는 리스트
- 입력창이 있는 리스트
- 펼침 상태가 있는 리스트
- 채팅처럼 앞쪽에 메시지를 추가하는 리스트
- 정렬과 필터가 자주 바뀌는 리스트

가상 리스트는 DOM을 재사용하는 구조라서 key 안정성이 더 중요합니다.

---

## 동적 높이 리스트 처리하기

실무 목록은 항상 같은 높이가 아닙니다.

예를 들어 댓글 목록을 생각해봅시다.

```txt
짧은 댓글: 48px
긴 댓글: 180px
이미지가 있는 댓글: 320px
```

이런 경우에는 아이템을 실제로 렌더링한 뒤 높이를 측정해야 합니다.

TanStack Virtual에서는 `measureElement`를 사용할 수 있습니다.

```tsx
export function CommentList({ comments }: { comments: Comment[] }) {
  const parentRef = useRef<HTMLDivElement | null>(null);

  const rowVirtualizer = useVirtualizer({
    count: comments.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 96,
    getItemKey: (index) => comments[index].id,
    overscan: 5,
  });

  return (
    <div ref={parentRef} className="comment-list-scroll">
      <div
        className="comment-list-inner"
        style={{
          height: `${rowVirtualizer.getTotalSize()}px`,
        }}
      >
        {rowVirtualizer.getVirtualItems().map((virtualRow) => {
          const comment = comments[virtualRow.index];

          return (
            <article
              key={virtualRow.key}
              ref={rowVirtualizer.measureElement}
              data-index={virtualRow.index}
              className="comment-row"
              style={{
                transform: `translateY(${virtualRow.start}px)`,
              }}
            >
              <h3>{comment.author}</h3>
              <p>{comment.body}</p>
            </article>
          );
        })}
      </div>
    </div>
  );
}
```

동적 높이에서 중요한 부분은 이것입니다.

```tsx
ref={rowVirtualizer.measureElement}
data-index={virtualRow.index}
```

`measureElement`는 실제 DOM 요소의 크기를 측정해서 virtualizer에게 알려줍니다.

`data-index`는 어떤 아이템의 크기인지 연결하는 데 사용됩니다.

CSS는 여전히 absolute 배치가 필요합니다.

```css
.comment-list-scroll {
  height: 520px;
  overflow: auto;
}

.comment-list-inner {
  position: relative;
  width: 100%;
}

.comment-row {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
  box-sizing: border-box;
}
```

동적 높이에서는 처음 추정값과 실제 측정값이 다를 수 있습니다.

그래서 스크롤 중 약간의 위치 보정이 발생할 수 있습니다.

이 현상 자체는 자연스럽습니다.

다만 `estimateSize`가 실제와 너무 다르면 보정 폭이 커져서 사용자에게 점프처럼 보일 수 있습니다.

---

## transform 방식과 top 방식

기본 예제에서는 아이템 위치를 `transform`으로 옮겼습니다.

```tsx
transform: `translateY(${virtualRow.start}px)`
```

다른 방식으로는 `top`을 직접 줄 수도 있습니다.

```tsx
top: `${virtualRow.start}px`
```

보통은 `transform` 방식이 스크롤 성능에 유리한 경우가 많습니다.

하지만 `transform`은 새로운 stacking context를 만들 수 있습니다.

그래서 아이템 내부에 `position: fixed`처럼 특수한 배치가 들어가거나, z-index 관계가 복잡한 UI라면 예상과 다르게 보일 수 있습니다.

실무에서는 대부분 `transform`으로 시작하고, 레이어나 고정 요소 문제가 생길 때 구조를 다시 봅니다.

---

## window 스크롤을 기준으로 가상화하기

지금까지는 특정 div 안에서 스크롤했습니다.

하지만 페이지 전체 스크롤을 기준으로 가상화하고 싶을 수도 있습니다.

이때는 `useWindowVirtualizer`를 사용할 수 있습니다.

```tsx
import { useWindowVirtualizer } from "@tanstack/react-virtual";

export function WindowPostList({ posts }: { posts: Post[] }) {
  const listRef = useRef<HTMLDivElement | null>(null);

  const virtualizer = useWindowVirtualizer({
    count: posts.length,
    estimateSize: () => 120,
    overscan: 5,
    scrollMargin: listRef.current?.offsetTop ?? 0,
  });

  return (
    <div ref={listRef}>
      <div
        className="post-list-inner"
        style={{
          height: `${virtualizer.getTotalSize()}px`,
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => {
          const post = posts[virtualRow.index];

          return (
            <article
              key={virtualRow.key}
              className="post-row"
              style={{
                height: `${virtualRow.size}px`,
                transform: `translateY(${
                  virtualRow.start - virtualizer.options.scrollMargin
                }px)`,
              }}
            >
              <h2>{post.title}</h2>
              <p>{post.description}</p>
            </article>
          );
        })}
      </div>
    </div>
  );
}
```

`useWindowVirtualizer`는 스크롤 컨테이너가 특정 div가 아니라 `window`입니다.

이때 리스트 위에 헤더나 다른 콘텐츠가 있다면 `scrollMargin`을 고려해야 합니다.

그렇지 않으면 아이템 위치가 실제 페이지 위치와 어긋날 수 있습니다.

일반적으로는 먼저 div 스크롤 방식인 `useVirtualizer`로 시작하고, 페이지 전체 스크롤과 자연스럽게 연결해야 할 때 `useWindowVirtualizer`를 검토하는 편이 좋습니다.

---

## 특정 아이템으로 이동하기

가상 리스트에서도 특정 인덱스로 이동할 수 있습니다.

```tsx
rowVirtualizer.scrollToIndex(500, {
  align: "center",
});
```

버튼과 연결하면 이렇게 쓸 수 있습니다.

```tsx
function UserListToolbar() {
  return (
    <button
      type="button"
      onClick={() => {
        rowVirtualizer.scrollToIndex(500, {
          align: "center",
        });
      }}
    >
      500번째 사용자로 이동
    </button>
  );
}
```

`align`에는 보통 이런 값을 사용합니다.

| 값 | 의미 |
| --- | --- |
| `start` | 아이템을 위쪽에 맞춤 |
| `center` | 아이템을 가운데에 맞춤 |
| `end` | 아이템을 아래쪽에 맞춤 |
| `auto` | 이미 보이면 유지하고, 안 보이면 적절히 이동 |

픽셀 위치로 이동하고 싶다면 `scrollToOffset`도 사용할 수 있습니다.

```tsx
rowVirtualizer.scrollToOffset(3000);
```

다만 동적 높이 리스트에서 아직 측정되지 않은 먼 위치로 이동하면 예상값을 기준으로 먼저 이동한 뒤, 측정이 진행되면서 위치가 보정될 수 있습니다.

이런 화면에서는 `estimateSize`를 최대한 현실적으로 잡는 것이 중요합니다.

---

## 무한 스크롤과 함께 사용하기

TanStack Virtual은 데이터 fetching 라이브러리가 아닙니다.

즉, 데이터를 불러오는 일은 직접 처리해야 합니다.

가상화는 "현재 어떤 아이템이 보이는지"를 알려주고, 우리는 마지막 아이템 근처에 도달했을 때 다음 페이지를 불러오면 됩니다.

```tsx
function InfiniteUserList({
  users,
  hasNextPage,
  fetchNextPage,
  isFetchingNextPage,
}: InfiniteUserListProps) {
  const parentRef = useRef<HTMLDivElement | null>(null);

  const rowVirtualizer = useVirtualizer({
    count: hasNextPage ? users.length + 1 : users.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 56,
    overscan: 5,
  });

  const virtualItems = rowVirtualizer.getVirtualItems();
  const lastItem = virtualItems[virtualItems.length - 1];

  useEffect(() => {
    if (!lastItem) return;

    const isLoaderRow = lastItem.index >= users.length - 1;

    if (isLoaderRow && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [
    lastItem?.index,
    users.length,
    hasNextPage,
    isFetchingNextPage,
    fetchNextPage,
  ]);

  return (
    <div ref={parentRef} className="user-list-scroll">
      <div
        className="user-list-inner"
        style={{
          height: `${rowVirtualizer.getTotalSize()}px`,
        }}
      >
        {virtualItems.map((virtualRow) => {
          const isLoaderRow = virtualRow.index > users.length - 1;
          const user = users[virtualRow.index];

          return (
            <div
              key={virtualRow.key}
              className="user-list-row"
              style={{
                height: `${virtualRow.size}px`,
                transform: `translateY(${virtualRow.start}px)`,
              }}
            >
              {isLoaderRow ? "더 불러오는 중..." : user.name}
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

여기서 `count`를 하나 늘린 이유는 마지막에 로더 행을 추가하기 위해서입니다.

```tsx
count: hasNextPage ? users.length + 1 : users.length
```

사용자가 아래로 내려가 로더 행 근처가 보이면 `fetchNextPage()`를 호출합니다.

실무에서는 같은 요청이 여러 번 나가지 않게 `isFetchingNextPage` 조건을 꼭 같이 둬야 합니다.

---

## 테이블에서 사용하기

가상화는 테이블에서도 자주 사용합니다.

다만 HTML table 구조는 일반 div보다 배치 제약이 많습니다.

아주 단순한 경우에는 `table`, `tbody`, `tr` 구조 안에서 virtual row를 배치할 수 있지만, sticky header, column resize, 동적 높이, 가로 스크롤이 섞이면 복잡해집니다.

이럴 때는 TanStack Table과 TanStack Virtual을 함께 사용하는 패턴이 많이 쓰입니다.

기본 아이디어는 같습니다.

```txt
전체 row 개수는 table row model 기준
보이는 row index만 virtualizer가 계산
렌더링은 virtual row에 해당하는 row만 수행
```

간단히 구조만 보면 이런 느낌입니다.

```tsx
const rows = table.getRowModel().rows;

const rowVirtualizer = useVirtualizer({
  count: rows.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 44,
  overscan: 10,
});

const virtualRows = rowVirtualizer.getVirtualItems();
```

그리고 `virtualRows`를 기준으로 `rows[virtualRow.index]`를 렌더링합니다.

테이블 가상화는 일반 리스트보다 고려할 것이 많습니다.

- header와 body의 column width 동기화
- sticky header
- 가로 스크롤
- row hover와 selection
- keyboard navigation
- 행 높이 측정

그래서 처음부터 테이블로 시작하기보다, 일반 리스트에서 `useVirtualizer`의 원리를 먼저 이해한 뒤 테이블에 적용하는 편이 좋습니다.

---

## 접근성에서 조심할 점

가상 리스트는 DOM에 모든 아이템이 존재하지 않습니다.

이 특성 때문에 접근성에서 몇 가지를 신경 써야 합니다.

첫째, 스크린 리더가 전체 목록을 DOM 기준으로만 이해하면 실제 총 개수를 알기 어려울 수 있습니다.

필요하다면 `aria-rowcount`, `aria-setsize`, `aria-posinset` 같은 속성을 고려할 수 있습니다.

```tsx
<div
  role="listitem"
  aria-posinset={virtualRow.index + 1}
  aria-setsize={users.length}
>
  {user.name}
</div>
```

둘째, 포커스 가능한 요소가 있는 리스트에서는 포커스가 사라지는 상황을 조심해야 합니다.

예를 들어 어떤 행 안의 버튼에 포커스가 있는데 스크롤로 그 행이 가상 영역 밖으로 나가면 DOM에서 제거될 수 있습니다.

이런 UI에서는 다음을 고려해야 합니다.

- 행 안에 복잡한 입력 폼을 넣지 않기
- 선택 상태는 DOM이 아니라 React 상태로 관리하기
- 키보드 이동 시 `scrollToIndex`로 해당 행을 먼저 보이게 만들기
- 포커스가 필요한 행은 overscan을 너무 작게 두지 않기

셋째, 검색이나 필터로 데이터가 바뀔 때 현재 스크롤 위치가 어색할 수 있습니다.

필터가 바뀌면 보통 맨 위로 보내는 것이 자연스럽습니다.

```tsx
useEffect(() => {
  rowVirtualizer.scrollToIndex(0);
}, [keyword]);
```

다만 검색어 입력마다 바로 맨 위로 보내면 사용자가 불편할 수 있으니, 실제 검색 결과가 확정되는 시점에 처리하는 것이 좋습니다.

---

## 실무에서 자주 만나는 문제

### 1. 내부 div에 position relative를 빼먹는다

아이템은 absolute로 배치됩니다.

그러려면 기준이 되는 부모가 필요합니다.

```css
.user-list-inner {
  position: relative;
}
```

이 값이 없으면 아이템 위치가 예상과 다르게 잡힐 수 있습니다.

### 2. 스크롤 컨테이너 높이가 없다

스크롤 컨테이너는 높이가 있어야 합니다.

```css
.user-list-scroll {
  height: 420px;
  overflow: auto;
}
```

높이가 없으면 스크롤 영역이 제대로 만들어지지 않습니다.

### 3. getTotalSize를 내부 영역에 적용하지 않는다

가상 리스트는 실제 DOM 아이템이 적습니다.

그래서 전체 스크롤 높이를 따로 만들어줘야 합니다.

```tsx
height: `${rowVirtualizer.getTotalSize()}px`
```

이 값이 없으면 스크롤바가 전체 목록 길이를 반영하지 못합니다.

### 4. 데이터 배열은 바뀌는데 key가 불안정하다

정렬, 필터, prepend가 있는 리스트에서 index key를 쓰면 상태가 꼬일 수 있습니다.

가능하면 `getItemKey`를 사용합니다.

```tsx
getItemKey: (index) => users[index].id
```

### 5. 동적 높이인데 estimateSize가 너무 작다

동적 높이 리스트에서 추정값이 너무 작으면 측정 후 보정이 크게 일어납니다.

처음에는 평균보다 조금 큰 값으로 시작하는 편이 좋습니다.

### 6. 개발 모드에서 성능을 판단한다

React 개발 모드는 렌더링 비용이 더 큽니다.

가상화 성능은 반드시 production build에서도 확인해야 합니다.

```bash
npm run build
npm run preview
```

Next.js라면 다음처럼 확인합니다.

```bash
npm run build
npm run start
```

---

## 언제 쓰고 언제 쓰지 말아야 할까?

가상화가 잘 어울리는 경우입니다.

- 목록 아이템이 수백 개 이상이다
- 각 아이템 렌더링 비용이 크다
- 스크롤할 때 버벅임이 느껴진다
- DOM 노드가 너무 많아진다
- 테이블, 로그, 채팅처럼 긴 목록이 핵심 UI다

반대로 이런 경우에는 굳이 쓰지 않아도 됩니다.

- 목록이 20 ~ 50개 정도다
- 페이지네이션으로 충분하다
- SEO가 중요한 콘텐츠 목록이다
- 브라우저 찾기 기능으로 전체 텍스트를 검색해야 한다
- 각 행에 입력 폼과 복잡한 포커스 흐름이 많다

특히 SEO가 중요한 블로그 글 목록이나 상품 목록이라면 신중해야 합니다.

가상화는 DOM에 일부 아이템만 존재하므로, 검색 엔진이나 브라우저 기본 기능과 충돌할 수 있습니다.

이런 경우에는 페이지네이션, 서버 페이지 분할, 더보기 버튼이 더 단순하고 안정적일 수 있습니다.

---

## react-window, react-virtuoso와 비교하면?

긴 리스트 가상화 라이브러리는 여러 개가 있습니다.

대표적으로 `react-window`, `react-virtuoso`, `@tanstack/react-virtual`이 있습니다.

| 라이브러리 | 특징 |
| --- | --- |
| `react-window` | 단순하고 오래 사용됨, API가 비교적 제한적 |
| `react-virtuoso` | 배터리 포함형에 가까움, 채팅/그룹/동적 높이 편의 기능이 많음 |
| `@tanstack/react-virtual` | headless 방식, 마크업과 스타일을 직접 제어하기 좋음 |

TanStack Virtual은 UI를 대신 만들어주지 않습니다.

대신 어떤 태그를 쓸지, 어떤 스타일을 줄지, 테이블과 어떻게 합칠지 직접 제어할 수 있습니다.

그래서 이미 디자인 시스템이 있거나, TanStack Table과 함께 쓰거나, 커스텀 레이아웃이 많은 프로젝트에 잘 맞습니다.

반대로 빠르게 완성된 채팅 목록을 만들고 싶다면 `react-virtuoso`가 더 편할 수도 있습니다.

---

## 전체 흐름 다시 보기

`@tanstack/react-virtual`을 이해할 때는 이 흐름을 기억하면 됩니다.

```txt
1. 스크롤 컨테이너를 ref로 잡는다.
2. useVirtualizer에 전체 개수와 예상 크기를 알려준다.
3. getTotalSize()로 전체 스크롤 높이를 만든다.
4. getVirtualItems()로 현재 렌더링할 아이템만 가져온다.
5. virtualRow.index로 실제 데이터를 찾는다.
6. virtualRow.start로 원래 위치에 배치한다.
7. 필요하면 overscan, getItemKey, measureElement를 조정한다.
```

코드로 압축하면 이 구조입니다.

```tsx
const rowVirtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 56,
  overscan: 5,
});

return (
  <div ref={parentRef} className="scroll">
    <div
      className="inner"
      style={{
        height: `${rowVirtualizer.getTotalSize()}px`,
      }}
    >
      {rowVirtualizer.getVirtualItems().map((virtualRow) => {
        const item = items[virtualRow.index];

        return (
          <div
            key={virtualRow.key}
            className="row"
            style={{
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {item.name}
          </div>
        );
      })}
    </div>
  </div>
);
```

---

## 마무리

`react-virtual`을 잘 쓰려면 먼저 가상화의 목적을 이해해야 합니다.

목적은 데이터를 줄이는 것이 아니라, **DOM에 실제로 렌더링되는 아이템 수를 줄이는 것**입니다.

TanStack Virtual은 이 작업을 headless 방식으로 도와줍니다.

라이브러리가 리스트 UI를 대신 만들어주는 것이 아니라, 현재 보여야 하는 아이템의 `index`, `start`, `size` 같은 계산 결과를 제공합니다.

개발자는 그 값을 이용해 직접 마크업과 스타일을 구성합니다.

처음에는 아래 네 가지만 제대로 잡으면 됩니다.

```txt
count
getScrollElement
estimateSize
getVirtualItems
```

그 다음 실무 요구사항에 따라 `overscan`, `getItemKey`, `measureElement`, `scrollToIndex`, `useWindowVirtualizer`를 하나씩 추가하면 됩니다.

가상화는 모든 리스트에 필요한 기술은 아닙니다.

하지만 긴 리스트가 실제 병목이 되는 화면에서는 체감 성능을 크게 바꿔주는 강력한 도구입니다.

---

## 참고 자료

- [TanStack Virtual 공식 문서 - React Virtual](https://tanstack.com/virtual/latest/docs/framework/react/react-virtual)
- [TanStack Virtual 공식 문서 - Virtualizer API](https://tanstack.com/virtual/latest/docs/api/virtualizer)
- [TanStack Virtual 공식 예제 - Fixed](https://tanstack.com/virtual/latest/docs/framework/react/examples/fixed)
- [TanStack Virtual 공식 문서 - Installation](https://tanstack.com/virtual/latest/docs/installation)

{% endraw %}
