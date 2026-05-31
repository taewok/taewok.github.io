---
title: "[React] 보이는 태그 개수만 계산해서 +N으로 표시하기"
date: 2026-05-29T02:20:00Z
categories: [frontend]
tags: [react, javascript, resizeobserver, requestanimationframe, ui]
description: "태그 목록에서 실제로 보이는 줄 안에 들어간 태그 개수를 DOM의 offsetTop으로 계산하고, 나머지 태그를 +N 형태로 표시하는 구현 방식을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 태그가 많아지면 카드 높이가 흔들려요

카드 UI를 만들다 보면 태그 목록을 자주 다루게 됩니다.

예를 들어 이런 형태예요.

```txt
#소꿉친구 #장난스러움 #츤데레 #학교 #청춘 #짝사랑 #일상
```

문제는 태그 개수가 많아질수록 카드 높이가 들쭉날쭉해진다는 점이었어요. 카드 목록에서는 높이가 어느 정도 일정해야 화면이 정돈되어 보이기 때문에, 보이는 태그만 보여주고 나머지는 `+5`처럼 표시하는 방식이 필요했습니다.

최종적으로 만들고 싶었던 UI는 이런 모습이었습니다.

```txt
#소꿉친구 #장난스러움 +5
```

여기서 중요한 점은 단순히 `tags.slice(0, 3)`처럼 고정 개수로 자르는 것이 아니라는 점이에요. 태그 길이나 카드 너비에 따라 한 줄에 들어가는 태그 개수가 달라질 수 있기 때문에, **실제로 화면에 보이는 줄 안에 들어간 태그 개수**를 계산해야 했습니다.

그래서 이번 기능은 배열의 개수만 보는 것이 아니라, 브라우저가 실제로 태그들을 어떻게 배치했는지를 기준으로 계산하는 방식으로 구현했습니다. 조금 낯설 수 있지만, 원리는 브라우저가 이미 계산한 위치 값을 읽어서 활용하는 쪽에 가깝습니다.

---

## 💡 핵심 아이디어

핵심은 각 태그 DOM의 `offsetTop`을 읽는 것입니다.

같은 줄에 있는 태그들은 부모 요소를 기준으로 위에서 떨어진 위치가 같기 때문에 `offsetTop` 값도 같아요. 반대로 다음 줄로 내려간 태그는 `offsetTop` 값이 달라집니다.

예를 들어 태그들이 이렇게 배치되었다고 해볼게요.

```txt
1번째 줄: #소꿉친구 #장난스러움 #츤데레
2번째 줄: #학교 #청춘
3번째 줄: #짝사랑 #일상
```

각 태그의 `offsetTop`은 대략 이렇게 나올 수 있습니다.

```ts
[0, 0, 0, 24, 24, 48, 48];
```

`maxLines`가 `1`이라면 첫 번째 줄의 `offsetTop`인 `0`만 허용하면 됩니다.

```ts
const visibleCount = 3;
const hiddenCount = tags.length - visibleCount;
```

그러면 화면에는 이렇게 표시할 수 있어요.

```txt
#소꿉친구 #장난스러움 #츤데레 +4
```

전체 흐름은 이렇게 정리할 수 있습니다.

1. 모든 태그를 일단 렌더링합니다.
2. 각 태그 DOM의 `offsetTop`을 확인합니다.
3. 같은 줄에 있는 태그들은 같은 `offsetTop` 값을 가진다고 판단합니다.
4. 허용할 줄 수만큼의 `offsetTop`만 남깁니다.
5. 그 줄 안에 들어간 태그 개수를 `visibleCount`로 저장합니다.
6. 전체 태그 개수에서 `visibleCount`를 빼서 `hiddenCount`를 만듭니다.

---

## 🛠️ useVisibleItemCount 훅 만들기

현재 프로젝트에서는 이 계산을 `useVisibleItemCount`라는 훅으로 분리했습니다.

```tsx
"use client";

import { useCallback, useLayoutEffect, useRef, useState } from "react";

interface UseVisibleItemCountOptions<T> {
  items: T[];
  maxLines?: number;
}

export const useVisibleItemCount = <T>({
  items,
  maxLines = 1,
}: UseVisibleItemCountOptions<T>) => {
  // 태그들이 들어있는 부모 요소입니다. 이 요소의 크기 변화를 감시합니다.
  const containerRef = useRef<HTMLDivElement | null>(null);

  // 각 태그 DOM을 index 기준으로 저장합니다.
  const itemRefs = useRef<(HTMLDivElement | null)[]>([]);

  // 실제로 화면에 보여줄 태그 개수입니다.
  const [visibleCount, setVisibleCount] = useState(items.length);

  const measure = useCallback(() => {
    // items 개수가 줄어들었을 때 이전 ref가 남지 않도록 잘라냅니다.
    itemRefs.current = itemRefs.current.slice(0, items.length);

    // 아직 렌더링되지 않은 null 값은 제외하고 실제 DOM만 모읍니다.
    const elements = itemRefs.current.filter(Boolean) as HTMLDivElement[];

    if (!elements.length) {
      setVisibleCount((prev) => (prev === items.length ? prev : items.length));
      return;
    }

    // 같은 줄에 있는 태그들은 offsetTop 값이 같습니다.
    // Set으로 중복을 제거하면 실제 줄 목록을 얻을 수 있습니다.
    const lineTops = Array.from(new Set(elements.map((el) => el.offsetTop)));

    // maxLines만큼의 줄만 화면에 보여주도록 허용합니다.
    const allowedTops = lineTops.slice(0, maxLines);

    // 허용된 줄에 포함된 태그 개수를 계산합니다.
    const nextVisibleCount = elements.filter((el) =>
      allowedTops.includes(el.offsetTop),
    ).length;

    // 값이 달라졌을 때만 상태를 업데이트해 불필요한 렌더링을 줄입니다.
    setVisibleCount((prev) =>
      prev === nextVisibleCount ? prev : nextVisibleCount,
    );
  }, [items.length, maxLines]);

  useLayoutEffect(() => {
    let animationFrameId: number | null = null;

    const scheduleMeasure = () => {
      // resize가 연속으로 발생하면 이전 측정 예약을 취소합니다.
      if (animationFrameId !== null) {
        cancelAnimationFrame(animationFrameId);
      }

      // 브라우저가 레이아웃을 정리한 다음 프레임에서 측정합니다.
      animationFrameId = requestAnimationFrame(measure);
    };

    // 최초 렌더링 이후 한 번 측정합니다.
    scheduleMeasure();

    // 컨테이너 크기가 바뀌면 태그 줄바꿈도 달라질 수 있으므로 다시 측정합니다.
    const observer = new ResizeObserver(scheduleMeasure);

    if (containerRef.current) {
      observer.observe(containerRef.current);
    }

    return () => {
      // 컴포넌트가 사라질 때 예약된 측정과 observer를 정리합니다.
      if (animationFrameId !== null) {
        cancelAnimationFrame(animationFrameId);
      }

      observer.disconnect();
    };
  }, [measure]);

  return {
    containerRef,
    itemRefs,
    visibleCount,
    // 전체 개수에서 보이는 개수를 빼면 +N으로 보여줄 개수가 됩니다.
    hiddenCount: Math.max(items.length - visibleCount, 0),
  };
};
```

처음 보면 조금 길어 보이지만, 역할을 하나씩 나눠보면 그렇게 어렵지 않습니다.

---

## 📌 containerRef: 크기 변화를 감시할 기준이에요

```tsx
const containerRef = useRef<HTMLDivElement | null>(null);
```

`containerRef`는 태그들이 들어있는 부모 요소를 가리킵니다.

부모 요소의 크기가 바뀌면 태그 배치도 함께 바뀔 수 있어요. 예를 들어 브라우저 크기가 줄어들면 한 줄에 들어가는 태그 수가 줄어듭니다.

그래서 이 부모 요소를 `ResizeObserver`로 감시합니다.

```tsx
const observer = new ResizeObserver(scheduleMeasure);

if (containerRef.current) {
  observer.observe(containerRef.current);
}
```

즉, `containerRef`는 **이 영역의 크기가 바뀌면 다시 계산해야 한다**는 기준점이라고 볼 수 있습니다.

---

## 📌 itemRefs: 각 태그 DOM을 저장해요

```tsx
const itemRefs = useRef<(HTMLDivElement | null)[]>([]);
```

`itemRefs`는 각각의 태그 DOM을 저장하는 배열입니다.

태그가 7개라면 내부적으로는 이런 식의 배열이 됩니다.

```ts
itemRefs.current = [
  첫번째태그Div,
  두번째태그Div,
  세번째태그Div,
  // ...
];
```

컴포넌트에서는 태그를 렌더링할 때 각 DOM을 배열에 저장해둡니다.

```tsx
ref={(el) => {
  itemRefs.current[index] = el;
}}
```

이렇게 해두어야 나중에 각 태그의 `offsetTop`을 읽을 수 있습니다.

```tsx
elements.map((el) => el.offsetTop);
```

---

## 📌 visibleCount: 실제로 보여줄 태그 개수예요

```tsx
const [visibleCount, setVisibleCount] = useState(items.length);
```

`visibleCount`는 실제로 화면에 보여줄 태그 개수입니다.

초기값은 `items.length`로 잡았습니다. 아직 측정하기 전에는 일단 전부 보이는 것으로 가정하는 셈이에요. 측정이 끝나면 실제로 몇 개까지 보여야 하는지 계산해서 업데이트합니다.

```tsx
setVisibleCount((prev) =>
  prev === nextVisibleCount ? prev : nextVisibleCount,
);
```

여기서 이전 값과 새 값이 같으면 상태를 바꾸지 않도록 했습니다. 같은 값으로 계속 `setState`가 발생하면 불필요한 렌더링이 생길 수 있기 때문이에요.

---

## 🔍 measure 함수 자세히 보기

이 훅의 핵심은 `measure` 함수입니다.

```tsx
const measure = useCallback(() => {
  itemRefs.current = itemRefs.current.slice(0, items.length);

  const elements = itemRefs.current.filter(Boolean) as HTMLDivElement[];

  if (!elements.length) {
    setVisibleCount((prev) => (prev === items.length ? prev : items.length));
    return;
  }

  const lineTops = Array.from(new Set(elements.map((el) => el.offsetTop)));
  const allowedTops = lineTops.slice(0, maxLines);

  const nextVisibleCount = elements.filter((el) =>
    allowedTops.includes(el.offsetTop),
  ).length;

  setVisibleCount((prev) =>
    prev === nextVisibleCount ? prev : nextVisibleCount,
  );
}, [items.length, maxLines]);
```

먼저 ref 배열을 현재 아이템 개수만큼 잘라줍니다.

```tsx
itemRefs.current = itemRefs.current.slice(0, items.length);
```

태그 개수가 줄어들었을 때 이전 ref가 남아 있으면 잘못된 계산을 할 수 있기 때문입니다.

그다음 실제 DOM이 있는 요소만 모읍니다.

```tsx
const elements = itemRefs.current.filter(Boolean) as HTMLDivElement[];
```

그리고 각 태그의 `offsetTop`을 읽습니다.

```tsx
elements.map((el) => el.offsetTop);
```

`offsetTop`은 부모를 기준으로 요소가 위에서 얼마나 떨어져 있는지를 의미합니다. 같은 줄에 있는 태그들은 같은 `offsetTop` 값을 갖습니다.

그래서 `Set`을 사용해 중복을 제거하면 줄 목록을 얻을 수 있어요.

```tsx
const lineTops = Array.from(new Set(elements.map((el) => el.offsetTop)));
```

예를 들어 결과가 이렇게 나왔다고 해볼게요.

```ts
[0, 0, 0, 24, 24, 48];
```

중복 제거 후에는 이렇게 됩니다.

```ts
[0, 24, 48];
```

이건 총 3줄이라는 뜻입니다.

이제 `maxLines`만큼 허용할 줄을 고릅니다.

```tsx
const allowedTops = lineTops.slice(0, maxLines);
```

`maxLines`가 `1`이면 첫 번째 줄만 허용합니다.

```ts
const allowedTops = [0];
```

그다음 각 태그의 `offsetTop`이 허용된 줄에 포함되는지 확인합니다.

```tsx
const nextVisibleCount = elements.filter((el) =>
  allowedTops.includes(el.offsetTop),
).length;
```

이 값이 최종적으로 보여줄 태그 개수가 됩니다.

---

## 🔄 ResizeObserver를 사용하는 이유

카드 너비가 바뀌면 한 줄에 들어가는 태그 개수도 바뀝니다.

넓은 화면에서는 이렇게 들어갈 수 있어요.

```txt
#소꿉친구 #장난스러움 #츤데레
```

하지만 좁은 화면에서는 이렇게 줄바꿈될 수 있습니다.

```txt
#소꿉친구 #장난스러움
#츤데레
```

따라서 처음 한 번만 계산하면 부족합니다. 컨테이너 크기가 바뀔 때마다 다시 계산해야 해요.

그래서 `ResizeObserver`를 사용합니다.

```tsx
const observer = new ResizeObserver(scheduleMeasure);
observer.observe(containerRef.current);
```

`ResizeObserver`는 관찰 중인 요소의 크기가 바뀌면 콜백을 실행합니다. 덕분에 반응형 카드에서도 보이는 태그 개수와 `+N` 값이 다시 계산됩니다.

---

## ⏱️ requestAnimationFrame을 사용하는 이유

`ResizeObserver` 콜백 안에서 바로 DOM을 측정하고 state를 바꾸면 렌더링 타이밍과 맞지 않을 수 있습니다.

그래서 측정을 바로 실행하지 않고 `requestAnimationFrame`으로 다음 프레임에 예약했습니다.

```tsx
const scheduleMeasure = () => {
  if (animationFrameId !== null) {
    cancelAnimationFrame(animationFrameId);
  }

  animationFrameId = requestAnimationFrame(measure);
};
```

이렇게 하면 두 가지 장점이 있습니다.

1. 브라우저가 레이아웃을 정리한 뒤에 측정할 수 있습니다.
2. 짧은 시간 안에 여러 번 크기 변화가 발생해도 마지막 측정만 실행할 수 있습니다.

이전 예약을 취소하고 새 측정만 남기는 방식입니다.

```tsx
cancelAnimationFrame(animationFrameId);
```

---

## 🧱 OverflowTagList 컴포넌트

훅은 계산만 담당합니다. 실제 UI는 `OverflowTagList`에서 만들었습니다.

```tsx
"use client";

import { useVisibleItemCount } from "@/hooks/useVisibleItemCount";

interface OverflowTagListProps {
  tags: string[];
  maxLines?: number;
}

const OverflowTagList = ({ tags, maxLines = 1 }: OverflowTagListProps) => {
  // 훅은 계산에 필요한 ref와 보이는 개수, 숨겨진 개수를 반환합니다.
  const { containerRef, itemRefs, visibleCount, hiddenCount } =
    useVisibleItemCount({
      items: tags,
      maxLines,
    });

  return (
    <div className="w-60 p-3 bg-zinc-900 rounded-xl flex items-start gap-2">
      <div className="flex-1 flex flex-col gap-2 min-w-0">
        <div className="text-white text-sm leading-5">장난꾸러기 소꿉친구</div>

        <div className="flex">
          {/* 이 컨테이너의 너비가 바뀌면 태그 배치를 다시 계산합니다. */}
          <div
            ref={containerRef}
            className="flex flex-wrap gap-1 h-5 overflow-hidden"
          >
            {tags.map((tag, index) => {
              // visibleCount보다 앞에 있는 태그만 실제로 보이게 합니다.
              const isVisible = index < visibleCount;

              return (
                <div
                  key={`${tag}-${index}`}
                  ref={(el) => {
                    // 각 태그 DOM을 저장해두어 offsetTop을 읽을 수 있게 합니다.
                    itemRefs.current[index] = el;
                  }}
                  className={`
                    px-1.5
                    py-0.5
                    bg-zinc-800
                    rounded-md
                    flex
                    items-center
                    gap-0.5
                    shrink-0
                    ${isVisible ? "" : "invisible pointer-events-none"}
                  `}
                >
                  <span className="text-xs text-zinc-200">#</span>
                  <span className="text-xs text-zinc-200 whitespace-nowrap">
                    {tag}
                  </span>
                </div>
              );
            })}
          </div>

          {/* 숨겨진 태그가 있을 때만 +N 배지를 보여줍니다. */}
          {hiddenCount > 0 && (
            <div className="px-1.5 py-0.5 bg-zinc-800 rounded-md flex items-center shrink-0">
              <span className="text-[10px] text-zinc-300">+{hiddenCount}</span>
            </div>
          )}
        </div>
      </div>

      <div className="text-zinc-500 text-sm">✔</div>
    </div>
  );
};

export default OverflowTagList;
```

`useVisibleItemCount`는 계산 결과만 반환하고, 컴포넌트는 그 결과를 바탕으로 태그와 `+N` UI를 렌더링합니다.

이렇게 나누어두면 같은 훅을 태그뿐 아니라 카테고리, 필터 칩, 키워드 목록 같은 UI에도 재사용할 수 있습니다.

---

## 🙈 왜 안 보이는 태그를 완전히 제거하지 않을까요?

여기서 가장 헷갈릴 수 있는 부분이 있습니다.

```tsx
const isVisible = index < visibleCount;
```

보이지 않는 태그라면 아예 렌더링하지 않으면 되는 것 아닐까요?

```tsx
tags.slice(0, visibleCount).map(...);
```

하지만 이 방식은 이번 기능과 잘 맞지 않았습니다.

우리는 **전체 태그가 실제로 배치되었을 때 몇 개가 첫 줄에 들어가는지**를 알아야 합니다. 그런데 보이지 않는 태그를 렌더링하지 않으면 브라우저가 전체 태그 배치를 계산할 수 없습니다.

그래서 모든 태그를 렌더링하되, 보이지 않는 태그는 `invisible`로 숨겼습니다.

```tsx
${isVisible ? "" : "invisible pointer-events-none"}
```

`invisible`은 화면에는 보이지 않지만 레이아웃 공간은 유지합니다. 이 점이 중요합니다.

---

## ⚠️ absolute를 쓰면 안 되는 이유

처음에는 숨겨진 태그에 이런 스타일을 줄 수도 있습니다.

```txt
invisible absolute pointer-events-none
```

하지만 `absolute`를 주면 해당 요소는 일반 레이아웃 흐름에서 빠집니다.

그러면 다음과 같은 문제가 생길 수 있어요.

1. 처음에는 모든 태그가 배치됩니다.
2. 측정 후 일부 태그를 숨깁니다.
3. 숨겨진 태그가 `absolute` 때문에 레이아웃에서 빠집니다.
4. 컨테이너 크기나 태그 줄 배치가 다시 바뀝니다.
5. `ResizeObserver`가 다시 실행됩니다.
6. 다시 측정하고 state를 바꿉니다.
7. 렌더링이 반복될 수 있습니다.

즉, 무한 렌더링에 가까운 상황이 생길 수 있습니다.

그래서 숨겨진 태그도 크기는 그대로 유지해야 합니다. 보이는 태그와 숨겨진 태그가 같은 크기 관련 스타일을 가져야 측정 결과가 안정적이에요.

```tsx
className={`
  px-1.5
  py-0.5
  bg-zinc-800
  rounded-md
  flex
  items-center
  gap-0.5
  shrink-0
  ${isVisible ? "" : "invisible pointer-events-none"}
`}
```

---

## ➕ hiddenCount 계산하기

훅에서는 `hiddenCount`도 같이 반환합니다.

```tsx
hiddenCount: Math.max(items.length - visibleCount, 0);
```

전체 태그 개수에서 보이는 태그 개수를 뺀 값입니다.

```txt
전체 태그: 7개
보이는 태그: 2개
hiddenCount = 7 - 2 = 5
```

그래서 UI에서는 이렇게 표시합니다.

```tsx
{
  hiddenCount > 0 && <div>+{hiddenCount}</div>;
}
```

숨겨진 태그가 없으면 `+N` 표시도 하지 않습니다.

---

## ✅ 이 방식의 장점

이 방식의 가장 큰 장점은 태그 길이와 컨테이너 너비에 맞게 결과가 달라진다는 점입니다.
태그가 짧으면 더 많이 보여줄 수 있습니다.

```txt
#AI #일상 #로맨스 +2
```

태그가 길면 적게 보여줍니다.

```txt
#장난꾸러기소꿉친구 +5
```

즉, 단순히 앞에서 3개만 보여주는 방식보다 실제 화면에 더 잘 맞는 결과를 만들 수 있습니다.

---

## 🚧 주의할 점

다만 이 방식은 DOM 측정을 사용합니다.

각 태그의 `offsetTop`을 읽고, 컨테이너 크기 변화를 관찰합니다. 따라서 카드가 아주 많이 렌더링되는 무한스크롤 목록에 모든 카드마다 붙이는 것은 조심해야 합니다.

예를 들어 검색 결과 카드가 100개 있고, 각 카드마다 이 훅이 실행된다면 `ResizeObserver`와 DOM 측정도 100개 단위로 늘어납니다.

그래서 현재 기준으로는 이런 영역에 사용하는 것이 좋습니다.

- 카드가 적은 영역
- 대표 카드
- 상세 페이지
- 정확한 `+N` 표시가 중요한 UI

반대로 무한스크롤 카드 목록 전체에는 단순한 방식이 더 나을 수 있습니다.

```tsx
const visibleTags = tags.slice(0, 3);
const hiddenCount = tags.length - visibleTags.length;
```

즉, 목록에서는 고정 개수로 자르고 자세히 보는 화면에서만 정확한 측정을 사용하는 식입니다.

---

## 🏁 정리

이번 기능은 실제로 보이는 태그 개수를 계산해 나머지를 `+N`으로 표시하는 기능입니다.

핵심은 다음과 같습니다.

- 태그들을 모두 렌더링합니다.
- 각 태그 DOM을 `itemRefs`에 저장합니다.
- 각 태그의 `offsetTop`을 읽습니다.
- 같은 `offsetTop`을 가진 태그는 같은 줄에 있다고 판단합니다.
- `maxLines`만큼 허용할 줄을 정합니다.
- 허용된 줄에 포함된 태그 개수를 `visibleCount`로 저장합니다.
- 나머지는 `hiddenCount`로 계산해서 `+N`으로 표시합니다.
- 숨겨진 태그는 `absolute`로 제거하지 않고 `invisible`로만 숨깁니다.
- 컨테이너 크기 변화는 `ResizeObserver`로 감지합니다.
- 측정은 `requestAnimationFrame`으로 예약해 렌더링 타이밍을 안정화합니다.

처음 보면 복잡해 보이지만, 결국 브라우저가 이미 계산한 태그의 실제 위치를 읽어서 활용하는 방식입니다.

직접 줄바꿈 계산 로직을 만들기보다 DOM의 `offsetTop`을 이용해 **이 태그가 몇 번째 줄에 있는지**를 판단하는 것이 핵심이었습니다.
