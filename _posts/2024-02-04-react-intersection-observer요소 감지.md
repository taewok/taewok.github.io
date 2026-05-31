---
title: "[React] react-intersection-observer로 요소 감지하기"
date: 2024-02-04T19:30:00
categories: [react]
tags: [react, intersection-observer, infinite-scroll]
description: "react-intersection-observer의 useInView를 사용해 요소가 화면에 들어왔는지 감지하고 무한 스크롤에 활용하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

무한 스크롤이나 등장 애니메이션을 만들 때는 특정 요소가 화면에 들어왔는지 감지해야 합니다.

브라우저에는 Intersection Observer API가 있지만, React에서 더 편하게 쓰고 싶다면 `react-intersection-observer` 라이브러리를 사용할 수 있습니다.

이번 글에서는 `useInView`를 사용해 요소 감지와 무한 스크롤 트리거를 만드는 방법을 정리해볼게요.

---

## 설치하기

```bash
npm install react-intersection-observer
```

---

## 기본 사용법

```tsx
import { useInView } from "react-intersection-observer";

const MyComponent = () => {
  const { ref, inView } = useInView();

  return (
    <div ref={ref}>
      {inView ? "요소가 화면에 보입니다." : "아직 보이지 않습니다."}
    </div>
  );
};

export default MyComponent;
```

`ref`는 감지할 요소에 연결합니다.

`inView`는 해당 요소가 화면에 들어왔는지를 알려주는 boolean 값입니다.

---

## 주요 옵션

```tsx
const { ref, inView } = useInView({
  threshold: 0.5,
  rootMargin: "0px",
  triggerOnce: true,
});
```

자주 사용하는 옵션은 다음과 같습니다.

| 옵션 | 설명 |
| --- | --- |
| `threshold` | 요소가 얼마나 보여야 감지할지 설정 |
| `rootMargin` | 감지 영역의 여백을 설정 |
| `triggerOnce` | 한 번만 감지할지 설정 |
| `skip` | 감지를 건너뛸지 설정 |

`threshold: 0.5`는 요소의 50% 이상이 보일 때 `inView`가 true가 된다는 뜻입니다.

---

## 무한 스크롤에 활용하기

목록 맨 아래에 감지용 요소를 두고, 그 요소가 화면에 들어오면 다음 데이터를 불러올 수 있습니다.

```tsx
import { useEffect } from "react";
import { useInView } from "react-intersection-observer";

const ProductList = ({ fetchNextPage, hasNextPage, isFetchingNextPage }) => {
  const { ref, inView } = useInView({
    threshold: 0.2,
  });

  useEffect(() => {
    if (!inView) return;
    if (!hasNextPage) return;
    if (isFetchingNextPage) return;

    fetchNextPage();
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <>
      {/* 상품 목록 */}
      <div ref={ref} />
    </>
  );
};
```

`inView`가 true가 되었을 때 다음 페이지를 요청합니다.

중복 요청을 막기 위해 `hasNextPage`와 `isFetchingNextPage`도 함께 확인하는 것이 좋습니다.

---

## rootMargin으로 미리 불러오기

사용자가 맨 아래까지 도달하기 전에 미리 데이터를 불러오고 싶다면 `rootMargin`을 사용할 수 있습니다.

```tsx
const { ref, inView } = useInView({
  rootMargin: "200px",
});
```

이렇게 하면 요소가 실제 화면에 들어오기 전, 200px 정도 가까워졌을 때 미리 감지할 수 있습니다.

---

## 마무리

`react-intersection-observer`를 사용하면 요소가 화면에 들어왔는지 쉽게 감지할 수 있습니다.

`ref`를 감지할 요소에 연결하고, `inView` 값으로 로직을 실행하면 됩니다.

무한 스크롤, lazy loading, 등장 애니메이션처럼 화면 진입 여부가 필요한 기능에 활용하기 좋습니다.
