---
title: "[React] hello-pangea/dnd로 드래그 앤 드롭 리스트 만들기"
date: 2026-06-21T10:00:00Z
categories: [frontend]
tags: [react, dnd, drag-and-drop, typescript, frontend]
description: "hello-pangea/dnd의 핵심 개념부터 리스트 재정렬, 드래그 핸들, 접근성, 주의점까지 한 번에 이해할 수 있게 정리했습니다."
custom_style: true
---

## 드래그 앤 드롭, 생각보다 금방 복잡해진다

리스트 순서를 바꾸는 기능은 처음엔 단순해 보여요.

그런데 막상 구현하려고 하면 생각보다 챙길 게 많습니다.

- 마우스로 끌기
- 터치로 끌기
- 키보드로도 조작하기
- 드롭했을 때 배열 순서 바꾸기
- 드래그 중 UI 상태 보여주기
- 스크롤 컨테이너 안에서도 자연스럽게 동작하기

이럴 때 `@hello-pangea/dnd`를 쓰면 드래그 앤 드롭의 기본 골격을 꽤 안정적으로 잡을 수 있어요.

이번 글에서는 이 라이브러리를 처음 보는 사람도 흐름을 따라가면서 바로 써볼 수 있도록 정리해보려고 합니다.

## 이 라이브러리는 어떤 역할을 할까

`@hello-pangea/dnd`는 React에서 리스트 기반 드래그 앤 드롭을 쉽게 만들 수 있게 도와주는 라이브러리예요.

공식 README를 보면 핵심 방향이 꽤 분명합니다.

- 리스트를 자연스럽게 재정렬할 수 있다
- 키보드와 스크린 리더를 포함한 접근성 경험을 제공한다
- 마우스, 터치, 키보드 입력을 모두 다룬다
- 별도의 wrapper DOM을 많이 만들지 않아 기존 레이아웃에 얹기 쉽다

즉, 이 라이브러리는 "아무 UI나 막 끌어다 놓는 도구"라기보다, **리스트 재정렬**을 안정적으로 처리하는 데 초점이 맞춰져 있어요.

그래서 카드 정렬, 장바구니 순서 변경, 태그 재배치, 시나리오 순서 조정 같은 화면에 잘 어울립니다.

## 가장 먼저 기억할 세 가지

이 라이브러리를 이해할 때는 딱 세 개만 먼저 잡으면 좋아요.

- `DragDropContext`
- `Droppable`
- `Draggable`

이 세 개가 드래그 앤 드롭의 뼈대를 만듭니다.

```tsx
import {
  DragDropContext,
  Droppable,
  Draggable,
  DropResult,
} from "@hello-pangea/dnd";
```

## 각 컴포넌트가 맡는 역할

### DragDropContext

드래그 앤 드롭이 일어나는 전체 영역을 감싸는 부모예요.

여기서 가장 중요한 건 `onDragEnd`입니다.

드래그를 끝냈을 때 어떤 일이 일어나야 하는지 여기서 결정해요.

```tsx
<DragDropContext onDragEnd={handleDragEnd}>
  ...
</DragDropContext>
```

### Droppable

드롭 가능한 영역이에요.

보통 리스트 하나를 감싸는 박스라고 생각하면 편합니다.

```tsx
<Droppable droppableId="character-list">
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.droppableProps}
      className={snapshot.isDraggingOver ? "bg-zinc-100" : ""}
    >
      ...
      {provided.placeholder}
    </div>
  )}
</Droppable>
```

여기서 중요한 건 `provided.innerRef`, `provided.droppableProps`, `provided.placeholder`예요.

### Draggable

실제로 끌 수 있는 개별 항목이에요.

```tsx
<Draggable draggableId={item.id} index={index}>
  {(provided, snapshot) => (
    <div
      ref={provided.innerRef}
      {...provided.draggableProps}
      {...provided.dragHandleProps}
      className={snapshot.isDragging ? "opacity-80" : ""}
    >
      {item.title}
    </div>
  )}
</Draggable>
```

`Draggable`는 각 아이템의 움직임과 상태를 맡고, `snapshot.isDragging`으로 현재 드래그 중인지도 확인할 수 있어요.

## 기본 예제 전체 흐름

아래는 리스트를 하나 재정렬하는 가장 기본적인 예시예요.

```tsx
"use client";

import { useState } from "react";
import {
  DragDropContext,
  Droppable,
  Draggable,
  DropResult,
} from "@hello-pangea/dnd";

type Item = {
  id: string;
  title: string;
};

const initialItems: Item[] = [
  { id: "1", title: "첫 번째 카드" },
  { id: "2", title: "두 번째 카드" },
  { id: "3", title: "세 번째 카드" },
];

export default function SortableList() {
  const [items, setItems] = useState(initialItems);

  const handleDragEnd = (result: DropResult) => {
    const { destination, source } = result;

    // 드롭된 위치가 없으면 그대로 종료
    if (!destination) return;

    // 같은 자리에서 놓았으면 순서를 바꿀 필요가 없음
    if (
      destination.droppableId === source.droppableId &&
      destination.index === source.index
    ) {
      return;
    }

    setItems((prev) => {
      const next = [...prev];
      const [movedItem] = next.splice(source.index, 1);
      next.splice(destination.index, 0, movedItem);
      return next;
    });
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <Droppable droppableId="items">
        {(provided, snapshot) => (
          <div
            ref={provided.innerRef}
            {...provided.droppableProps}
            className={`space-y-2 rounded-xl border p-4 ${
              snapshot.isDraggingOver ? "border-blue-400 bg-blue-50" : ""
            }`}
          >
            {items.map((item, index) => (
              <Draggable key={item.id} draggableId={item.id} index={index}>
                {(provided, snapshot) => (
                  <div
                    ref={provided.innerRef}
                    {...provided.draggableProps}
                    {...provided.dragHandleProps}
                    className={`rounded-lg border bg-white p-3 shadow-sm ${
                      snapshot.isDragging ? "shadow-md" : ""
                    }`}
                  >
                    {item.title}
                  </div>
                )}
              </Draggable>
            ))}
            {provided.placeholder}
          </div>
        )}
      </Droppable>
    </DragDropContext>
  );
}
```

이 코드의 핵심은 `onDragEnd`에서 배열 순서를 다시 만드는 부분이에요.

## onDragEnd는 왜 중요할까

드래그 UI는 화면상 움직임만으로 끝나면 안 돼요.

실제로 순서가 바뀐 결과를 상태에 저장해야 다음 렌더에서 그 순서가 유지됩니다.

`onDragEnd`는 그 마무리 지점이에요.

```tsx
const handleDragEnd = (result: DropResult) => {
  const { destination, source } = result;

  if (!destination) return;

  if (destination.index === source.index) return;

  setItems((prev) => {
    const next = [...prev];
    const [movedItem] = next.splice(source.index, 1);
    next.splice(destination.index, 0, movedItem);
    return next;
  });
};
```

여기서 자주 보는 필드가 두 개예요.

- `source`: 드래그를 시작한 위치
- `destination`: 드롭한 위치

그리고 `destination`이 없을 수도 있어요.

예를 들면 바깥으로 버렸거나, 허용되지 않은 영역에 놓은 경우예요. 그래서 항상 먼저 확인해야 합니다.

## draggableId와 index는 어떻게 써야 할까

여기서 실수하기 쉬운 부분이 하나 있어요.

`index`는 현재 배열 순서를 따라가야 하고, `draggableId`는 안정적으로 유지되는 고유값이어야 해요.

```tsx
<Draggable draggableId={item.id} index={index}>
```

왜 `index`를 id로 쓰면 안 되느냐면, 순서가 바뀔 때마다 id도 흔들리기 쉬워서 드래그 상태가 꼬일 수 있기 때문이에요.

그래서 보통 서버에서 받은 고유 id나 UUID 같은 값을 씁니다.

## provided와 snapshot은 무엇일까

처음 보면 `provided`와 `snapshot`이 조금 낯설 수 있어요.

둘 다 render prop으로 내려오는 값인데, 역할이 다릅니다.

### provided

DOM에 꼭 붙여야 하는 props와 ref를 담고 있어요.

- `provided.innerRef`
- `provided.droppableProps`
- `provided.draggableProps`
- `provided.dragHandleProps`

이걸 빠뜨리면 드래그가 제대로 안 되거나, 아예 동작하지 않을 수 있어요.

### snapshot

현재 상태를 알려주는 값이에요.

예를 들면:

- `snapshot.isDragging`
- `snapshot.isDraggingOver`

이 값으로 스타일을 바꾸면 사용자가 지금 어떤 상태인지 훨씬 잘 느낄 수 있어요.

## 드래그 핸들을 따로 분리할 수도 있다

항목 전체를 잡고 끄는 대신, 특정 아이콘이나 영역만 드래그 핸들로 만들 수도 있어요.

```tsx
<Draggable draggableId={item.id} index={index}>
  {(provided) => (
    <div ref={provided.innerRef} {...provided.draggableProps}>
      <button
        type="button"
        {...provided.dragHandleProps}
        aria-label="드래그 핸들"
      >
        ☰
      </button>
      <span>{item.title}</span>
    </div>
  )}
</Draggable>
```

이 패턴은 편집 UI에서 꽤 자주 써요.

전체 카드가 드래그되면 버튼 클릭이나 입력 같은 인터랙션이 살짝 불편해질 수 있는데, 핸들을 따로 두면 그런 충돌을 줄일 수 있습니다.

## 여러 리스트 간 이동도 가능하다

`hello-pangea/dnd`는 한 리스트 안에서만 순서를 바꾸는 데 그치지 않고, 리스트 간 이동도 할 수 있어요.

예를 들어:

- 할 일
- 진행 중
- 완료

같은 칸반 보드 구조가 대표적입니다.

이 경우 `source.droppableId`와 `destination.droppableId`를 비교해서

- 같은 리스트면 재정렬
- 다른 리스트면 이동

으로 나눠 처리하면 됩니다.

이런 구조는 처음엔 조금 길어져도, 상태를 잘 나누면 꽤 깔끔하게 유지돼요.

## 접근성은 왜 장점일까

공식 README에서도 접근성을 중요한 장점으로 강조해요.

이건 생각보다 꽤 큰 포인트예요.

드래그 앤 드롭은 시각적으로만 보면 멋져도, 키보드나 스크린 리더에서 허술하면 실제 서비스에 넣기 어렵거든요.

이 라이브러리는 기본적으로:

- 마우스
- 터치
- 키보드

조작을 지원하고, 스크린 리더 경험도 고려돼 있어요.

그래서 단순 데모를 넘어서 실서비스 UI에 넣기 좋은 편입니다.

## 자주 만나는 주의점

### 1. placeholder를 빼먹지 않기

`Droppable` 안에는 `provided.placeholder`를 넣어야 해요.

이게 있어야 드래그 중인 아이템이 빠졌을 때 리스트 높이가 자연스럽게 유지됩니다.

### 2. ref와 props를 정확히 연결하기

`provided.innerRef`는 실제 DOM 노드에 연결해야 해요.

커스텀 컴포넌트를 쓴다면 `forwardRef`가 필요할 수 있습니다.

### 3. HTML 구조가 너무 복잡하면 레이아웃을 다시 보기

이 라이브러리는 리스트 구조와 잘 맞아요.

반대로 중첩된 인터랙티브 요소가 너무 많으면 드래그 핸들과 클릭 이벤트가 부딪힐 수 있습니다.

### 4. SSR 환경에서는 클라이언트 렌더링을 의식하기

Next.js처럼 서버 렌더링이 있는 환경에서는 이 컴포넌트가 브라우저에서 동작해야 하므로, 클라이언트 컴포넌트로 다루는 편이 안전해요.

### 5. `html` 문서 구조와 기본 스타일도 중요하기

README에서도 `html5 doctype`을 쓰는 걸 권장합니다.

이런 라이브러리는 브라우저의 레이아웃 계산에 기대는 부분이 있어서, 문서 기본 설정이 엉키면 예상 밖 동작이 나올 수 있어요.

## 실무에서는 어떻게 쓰면 좋을까

이 라이브러리는 다음 같은 화면에 특히 잘 맞아요.

- 리스트 순서 변경
- 카드 정렬
- 태그 재배치
- 시나리오 순서 조정
- 칸반 보드
- 일정 우선순위 정렬

반대로 아주 자유로운 캔버스형 편집기라면 다른 접근이 더 나을 수도 있어요.

즉, 이건 "무엇이든 막 끌어다 놓기"보다 "순서가 있는 항목을 안정적으로 다루기"에 더 어울립니다.

## 정리

`@hello-pangea/dnd`를 이해할 때 가장 중요한 건 복잡한 API를 외우는 게 아니라, 흐름을 잡는 거예요.

1. `DragDropContext`로 전체 드래그 영역을 감싼다
2. `Droppable`으로 드롭 가능한 리스트를 만든다
3. `Draggable`로 실제 아이템을 렌더링한다
4. `onDragEnd`에서 배열 순서를 다시 정리한다
5. `provided`와 `snapshot`으로 ref, 스타일, 상태를 연결한다

이 흐름만 익히면 기본적인 리스트 재정렬은 충분히 구현할 수 있어요.

그리고 그다음부터는:

- 드래그 핸들 분리
- 여러 리스트 이동
- 접근성 세부 조정
- 스크롤 컨테이너 대응

같은 확장도 자연스럽게 붙습니다.

드래그 앤 드롭은 처음엔 복잡해 보여도, 구조를 한 번 이해하면 꽤 예쁘게 정리되는 기능이에요. `hello-pangea/dnd`는 그 구조를 잡아주는 데 꽤 든든한 도구였습니다.

## 참고

- [hello-pangea/dnd GitHub README](https://github.com/hello-pangea/dnd)
