---
title: "[React] useMemo와 useCallback 완벽 이해하기: 언제 쓰고 언제 쓰지 말아야 할까"
date: 2026-09-19T02:30:00Z
categories: [frontend]
tags: [react, hooks, usememo, usecallback, memoization, performance]
description: "React의 useMemo와 useCallback이 각각 무엇을 캐싱하는지, React.memo와 어떤 관계가 있는지, dependency 배열과 stale closure, 과한 최적화 실수까지 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: useMemo와 useCallback은 성능 버튼이 아니에요

React를 공부하다 보면 `useMemo`와 `useCallback`을 자주 만나게 됩니다.

처음에는 이런 식으로 이해하기 쉽습니다.

```txt
useMemo를 쓰면 성능이 좋아진다
useCallback을 쓰면 함수가 새로 안 만들어진다
렌더링 최적화하려면 일단 감싸면 된다
```

하지만 실제로는 조금 다릅니다.

`useMemo`와 `useCallback`은 무조건 성능을 올려주는 버튼이 아닙니다.

오히려 필요 없는 곳에 많이 쓰면 코드가 복잡해지고, dependency 배열을 잘못 관리해서 버그가 생길 수 있습니다.

먼저 결론부터 말하면 이렇습니다.

```txt
useMemo
→ 계산 결과를 기억한다

useCallback
→ 함수 자체를 기억한다

React.memo
→ props가 같으면 컴포넌트 리렌더링을 건너뛴다
```

이 세 가지는 따로 떨어진 기능처럼 보이지만, 실제로는 함께 이해해야 합니다.

이번 글에서는 `useMemo`와 `useCallback`을 언제 쓰고, 언제 쓰지 않는 게 좋은지까지 정리해보겠습니다.

---

## React는 리렌더링될 때 함수 컴포넌트를 다시 실행한다

먼저 React의 렌더링 방식을 이해해야 합니다.

함수 컴포넌트는 리렌더링될 때 함수 본문이 다시 실행됩니다.

```tsx
function ProductList({ products, keyword }: ProductListProps) {
  const filteredProducts = products.filter((product) =>
    product.name.includes(keyword),
  );

  return (
    <ul>
      {filteredProducts.map((product) => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

`ProductList`가 리렌더링되면 아래 코드도 다시 실행됩니다.

```tsx
const filteredProducts = products.filter((product) =>
  product.name.includes(keyword),
);
```

대부분은 문제가 아닙니다.

React 컴포넌트가 다시 실행되는 것 자체는 자연스러운 일입니다.

문제는 이 계산이 아주 비싸거나, 이 계산으로 만들어진 값이 자식 컴포넌트의 불필요한 리렌더링을 유발할 때입니다.

이때 `useMemo`나 `useCallback`을 고려할 수 있습니다.

---

## useMemo는 계산 결과를 기억한다

`useMemo`는 계산 결과를 캐싱합니다.

```tsx
const cachedValue = useMemo(() => {
  return calculateValue();
}, [dependencies]);
```

React 공식 문서에서는 `useMemo`를 "리렌더링 사이에서 계산 결과를 캐싱하는 Hook"으로 설명합니다.

예를 들어 상품 목록을 필터링한다고 해보겠습니다.

```tsx
import { useMemo } from "react";

function ProductList({ products, keyword }: ProductListProps) {
  const filteredProducts = useMemo(() => {
    return products.filter((product) => product.name.includes(keyword));
  }, [products, keyword]);

  return (
    <ul>
      {filteredProducts.map((product) => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

이제 `products`와 `keyword`가 이전 렌더링과 같다면 React는 필터링을 다시 하지 않고, 이전에 계산한 `filteredProducts`를 재사용합니다.

흐름으로 보면 이렇습니다.

```txt
첫 렌더링
→ products.filter 실행
→ 결과를 저장

다음 렌더링
→ products와 keyword가 이전과 같은지 비교
→ 같으면 저장된 결과 재사용
→ 다르면 다시 계산
```

핵심은 이것입니다.

```txt
useMemo는 렌더링을 막는 것이 아니라
렌더링 중 특정 계산을 다시 하지 않게 도와준다.
```

---

## useMemo를 쓰기 좋은 경우

`useMemo`는 아무 계산에나 붙이는 Hook이 아닙니다.

다음 상황에서 의미가 있습니다.

### 1. 계산이 실제로 비싼 경우

예를 들어 큰 배열을 필터링하거나 정렬하거나 그룹핑하는 경우입니다.

```tsx
const groupedOrders = useMemo(() => {
  return orders.reduce<Record<string, Order[]>>((acc, order) => {
    const key = order.status;

    if (!acc[key]) {
      acc[key] = [];
    }

    acc[key].push(order);

    return acc;
  }, {});
}, [orders]);
```

데이터가 적다면 굳이 필요 없습니다.

하지만 `orders`가 수천 개이고, 부모 상태 변경 때문에 이 컴포넌트가 자주 리렌더링된다면 효과가 있을 수 있습니다.

### 2. memo로 감싼 자식에게 객체나 배열을 넘기는 경우

`React.memo`는 props가 이전과 같으면 자식 컴포넌트 리렌더링을 건너뜁니다.

그런데 객체나 배열은 매번 새로 만들면 값이 같아 보여도 참조가 다릅니다.

```tsx
const visibleProducts = products.filter((product) => product.visible);

return <ProductTable products={visibleProducts} />;
```

`filter`는 항상 새 배열을 만듭니다.

그래서 `ProductTable`을 `memo`로 감싸도 `products` prop이 매번 달라진 것처럼 보일 수 있습니다.

```tsx
const ProductTable = memo(function ProductTable({
  products,
}: ProductTableProps) {
  return (
    <table>
      {/* ... */}
    </table>
  );
});
```

이때 `useMemo`가 의미 있어집니다.

```tsx
const visibleProducts = useMemo(() => {
  return products.filter((product) => product.visible);
}, [products]);

return <ProductTable products={visibleProducts} />;
```

이제 `products`가 바뀌지 않았다면 `visibleProducts`도 같은 배열 참조를 유지합니다.

그러면 `memo`로 감싼 `ProductTable`이 리렌더링을 건너뛸 수 있습니다.

### 3. 다른 Hook의 dependency로 쓰는 객체를 안정화해야 하는 경우

아래 코드를 봅시다.

```tsx
function ChatRoom({ roomId }: { roomId: string }) {
  const options = {
    serverUrl: "https://example.com",
    roomId,
  };

  useEffect(() => {
    const connection = createConnection(options);

    connection.connect();

    return () => connection.disconnect();
  }, [options]);

  return <div>채팅방</div>;
}
```

`options`는 렌더링마다 새 객체입니다.

그래서 `roomId`가 바뀌지 않아도 `useEffect`가 다시 실행될 수 있습니다.

이때 `useMemo`로 객체 참조를 안정화할 수 있습니다.

```tsx
const options = useMemo(() => {
  return {
    serverUrl: "https://example.com",
    roomId,
  };
}, [roomId]);

useEffect(() => {
  const connection = createConnection(options);

  connection.connect();

  return () => connection.disconnect();
}, [options]);
```

다만 더 단순한 방법도 있습니다.

객체를 Effect 안으로 옮기면 `useMemo`가 필요 없어집니다.

```tsx
useEffect(() => {
  const options = {
    serverUrl: "https://example.com",
    roomId,
  };

  const connection = createConnection(options);

  connection.connect();

  return () => connection.disconnect();
}, [roomId]);
```

가능하다면 이런 구조가 더 읽기 쉽습니다.

`useMemo`를 추가하기 전에 먼저 구조를 단순하게 만들 수 있는지 보는 것이 좋습니다.

---

## useMemo를 쓰지 않아도 되는 경우

다음 상황에서는 `useMemo`가 거의 의미 없습니다.

### 1. 계산이 가벼운 경우

```tsx
const fullName = `${firstName} ${lastName}`;
```

이런 값까지 `useMemo`로 감쌀 필요는 없습니다.

```tsx
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`;
}, [firstName, lastName]);
```

오히려 코드만 길어집니다.

### 2. primitive 값을 단순히 만드는 경우

문자열, 숫자, boolean 같은 primitive 값은 비교가 단순합니다.

```tsx
const isEmpty = items.length === 0;
```

이 정도는 그냥 계산하는 편이 낫습니다.

### 3. dependency가 매번 바뀌는 경우

`useMemo`는 dependency가 같을 때만 이전 값을 재사용합니다.

그런데 dependency가 매번 새로 만들어진다면 캐싱 효과가 없습니다.

```tsx
const filterOptions = {
  keyword,
  sort,
};

const filteredItems = useMemo(() => {
  return filterItems(items, filterOptions);
}, [items, filterOptions]);
```

`filterOptions`는 렌더링마다 새 객체입니다.

그래서 `useMemo`도 매번 다시 실행됩니다.

이 경우에는 dependency를 더 직접적으로 쓰는 편이 좋습니다.

```tsx
const filteredItems = useMemo(() => {
  return filterItems(items, {
    keyword,
    sort,
  });
}, [items, keyword, sort]);
```

---

## useCallback은 함수 자체를 기억한다

`useCallback`은 함수 정의를 캐싱합니다.

```tsx
const cachedFn = useCallback(() => {
  doSomething();
}, [dependencies]);
```

`useMemo`가 계산 결과를 기억한다면, `useCallback`은 함수 자체를 기억합니다.

아래 두 코드는 개념적으로 비슷합니다.

```tsx
const handleClick = useCallback(() => {
  console.log("click");
}, []);
```

```tsx
const handleClick = useMemo(() => {
  return () => {
    console.log("click");
  };
}, []);
```

즉, `useCallback(fn, deps)`는 `useMemo(() => fn, deps)`와 비슷하게 이해할 수 있습니다.

다만 함수에는 `useCallback`을 쓰는 편이 의도가 더 명확합니다.

---

## 함수는 렌더링마다 새로 만들어진다

컴포넌트 안에서 함수를 만들면 렌더링마다 새 함수가 만들어집니다.

```tsx
function ProductPage({ productId }: { productId: string }) {
  function handleSubmit() {
    buyProduct(productId);
  }

  return <OrderForm onSubmit={handleSubmit} />;
}
```

`ProductPage`가 리렌더링될 때마다 `handleSubmit`은 새 함수입니다.

이 자체는 보통 문제가 아닙니다.

함수가 새로 만들어진다고 해서 항상 성능 문제가 생기는 것은 아닙니다.

문제가 되는 상황은 이 함수가 `memo`로 감싼 자식 컴포넌트의 props로 내려갈 때입니다.

```tsx
const OrderForm = memo(function OrderForm({ onSubmit }: OrderFormProps) {
  return <button onClick={onSubmit}>주문하기</button>;
});
```

`OrderForm`은 `memo`로 감싸져 있습니다.

하지만 부모가 매번 새 `handleSubmit` 함수를 넘기면 `onSubmit` prop이 매번 바뀐 것처럼 보입니다.

그러면 `memo`가 효과를 내기 어렵습니다.

이때 `useCallback`을 사용할 수 있습니다.

```tsx
function ProductPage({ productId }: { productId: string }) {
  const handleSubmit = useCallback(() => {
    buyProduct(productId);
  }, [productId]);

  return <OrderForm onSubmit={handleSubmit} />;
}
```

이제 `productId`가 바뀌지 않는 한 `handleSubmit`은 같은 함수 참조를 유지합니다.

---

## useCallback을 쓰기 좋은 경우

### 1. memo로 감싼 자식에게 함수를 넘길 때

가장 대표적인 경우입니다.

```tsx
const SearchResultItem = memo(function SearchResultItem({
  item,
  onSelect,
}: SearchResultItemProps) {
  return (
    <button type="button" onClick={() => onSelect(item.id)}>
      {item.title}
    </button>
  );
});
```

부모에서 `onSelect`를 매번 새로 만들면 memo 효과가 줄어듭니다.

```tsx
function SearchResults({ items }: SearchResultsProps) {
  const handleSelect = useCallback((id: string) => {
    console.log(id);
  }, []);

  return (
    <div>
      {items.map((item) => (
        <SearchResultItem
          key={item.id}
          item={item}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
}
```

여기서는 `SearchResultItem`이 많고, 각 아이템 렌더링 비용이 크다면 `useCallback`이 의미 있을 수 있습니다.

### 2. 함수를 다른 Hook의 dependency로 사용할 때

함수를 `useEffect` 안에서 사용해야 하는 경우가 있습니다.

```tsx
function SearchBox({ keyword }: { keyword: string }) {
  const fetchResults = useCallback(() => {
    return fetch(`/api/search?q=${keyword}`);
  }, [keyword]);

  useEffect(() => {
    fetchResults();
  }, [fetchResults]);

  return <div>검색</div>;
}
```

`fetchResults`가 `keyword`가 바뀔 때만 새로 만들어지므로, Effect도 필요한 때만 다시 실행됩니다.

하지만 이 경우도 더 단순하게 만들 수 있습니다.

```tsx
useEffect(() => {
  function fetchResults() {
    return fetch(`/api/search?q=${keyword}`);
  }

  fetchResults();
}, [keyword]);
```

함수가 Effect 안에서만 쓰인다면 안으로 옮기는 편이 더 좋을 때가 많습니다.

### 3. 커스텀 Hook이 함수를 반환할 때

커스텀 Hook에서 함수를 반환한다면 `useCallback`으로 감싸는 것이 좋습니다.

```tsx
function useModal() {
  const [isOpen, setIsOpen] = useState(false);

  const open = useCallback(() => {
    setIsOpen(true);
  }, []);

  const close = useCallback(() => {
    setIsOpen(false);
  }, []);

  return {
    isOpen,
    open,
    close,
  };
}
```

이렇게 하면 이 Hook을 사용하는 컴포넌트가 `open`, `close`를 dependency로 사용하거나 memoized child에 넘길 때 더 안정적입니다.

---

## useCallback을 쓰지 않아도 되는 경우

### 1. 일반 DOM 이벤트 핸들러

아래 코드는 보통 괜찮습니다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button type="button" onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

버튼 하나에 넘기는 이벤트 핸들러를 꼭 `useCallback`으로 감쌀 필요는 없습니다.

```tsx
const handleClick = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

이렇게 바꿔도 `count`가 바뀔 때마다 함수는 새로 만들어집니다.

그리고 자식 컴포넌트 최적화와 연결된 것도 아닙니다.

이런 경우에는 그냥 인라인 함수가 더 읽기 쉽습니다.

### 2. 자식 컴포넌트가 memo로 감싸져 있지 않은 경우

```tsx
function Parent() {
  const handleClick = useCallback(() => {
    console.log("click");
  }, []);

  return <Child onClick={handleClick} />;
}
```

`Child`가 `memo`로 감싸져 있지 않다면 부모가 리렌더링될 때 자식도 리렌더링됩니다.

즉, 함수 참조를 안정화해도 자식 리렌더링을 막지 못합니다.

이런 경우 `useCallback`만 추가해도 성능 최적화가 되는 것은 아닙니다.

### 3. dependency가 자주 바뀌는 경우

```tsx
const handleChange = useCallback(() => {
  submit(formValues);
}, [formValues]);
```

`formValues`가 입력마다 새로 바뀐다면 `handleChange`도 입력마다 새로 만들어집니다.

이 경우 `useCallback`의 캐싱 효과는 제한적입니다.

---

## useMemo와 useCallback 차이 정리

두 Hook의 차이는 한 줄로 정리할 수 있습니다.

```txt
useMemo는 값을 캐싱하고,
useCallback은 함수를 캐싱한다.
```

표로 보면 더 분명합니다.

| 구분 | useMemo | useCallback |
| --- | --- | --- |
| 기억하는 것 | 계산 결과 | 함수 참조 |
| 반환값 | 어떤 값이든 가능 | 함수 |
| 주 사용처 | 비싼 계산 결과 재사용, 객체/배열 참조 안정화 | memoized child에 넘기는 콜백 안정화 |
| 예시 | 필터링된 배열, 정렬된 배열, 옵션 객체 | onSubmit, onSelect, onClose |

예를 들어 이런 계산은 `useMemo`입니다.

```tsx
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);
```

이런 함수는 `useCallback`입니다.

```tsx
const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);
```

---

## React.memo와 같이 이해해야 한다

`useMemo`와 `useCallback`을 제대로 이해하려면 `React.memo`를 같이 봐야 합니다.

```tsx
import { memo } from "react";

const ProductTable = memo(function ProductTable({
  products,
  onSelect,
}: ProductTableProps) {
  return (
    <table>
      {/* ... */}
    </table>
  );
});
```

`memo`로 감싼 컴포넌트는 props가 이전과 같으면 리렌더링을 건너뛸 수 있습니다.

하지만 객체, 배열, 함수는 렌더링마다 새로 만들면 매번 다른 값으로 취급됩니다.

```tsx
<ProductTable
  products={products.filter((product) => product.visible)}
  onSelect={(id) => setSelectedId(id)}
/>
```

위 코드는 렌더링마다 새 배열과 새 함수를 만듭니다.

그래서 `ProductTable`이 `memo`로 감싸져 있어도 props가 계속 바뀐 것처럼 보입니다.

이때 `useMemo`와 `useCallback`이 의미 있어집니다.

```tsx
const visibleProducts = useMemo(() => {
  return products.filter((product) => product.visible);
}, [products]);

const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);

return (
  <ProductTable
    products={visibleProducts}
    onSelect={handleSelect}
  />
);
```

정리하면 이렇습니다.

```txt
React.memo
→ 자식 컴포넌트가 props가 같으면 리렌더링을 건너뛸 수 있게 함

useMemo
→ 자식에게 넘기는 객체/배열 값을 같게 유지할 수 있음

useCallback
→ 자식에게 넘기는 함수 참조를 같게 유지할 수 있음
```

셋 중 하나만 이해하면 부족합니다.

대부분의 렌더링 최적화는 이 셋의 관계를 이해할 때 명확해집니다.

---

## dependency 배열을 정확히 이해하기

`useMemo`와 `useCallback`의 두 번째 인자는 dependency 배열입니다.

```tsx
const value = useMemo(() => {
  return calculate(a, b);
}, [a, b]);
```

```tsx
const handleClick = useCallback(() => {
  submit(userId);
}, [userId]);
```

dependency 배열에는 내부에서 사용하는 reactive value를 넣어야 합니다.

여기서 reactive value는 보통 다음을 말합니다.

- props
- state
- 컴포넌트 내부에서 선언한 변수
- 컴포넌트 내부에서 선언한 함수

dependency를 빼먹으면 오래된 값을 참조하는 문제가 생길 수 있습니다.

```tsx
function UserButton({ userId }: { userId: string }) {
  const handleClick = useCallback(() => {
    console.log(userId);
  }, []);

  return <button onClick={handleClick}>확인</button>;
}
```

이 코드는 `userId`를 사용하지만 dependency 배열에는 없습니다.

처음 렌더링의 `userId`만 기억하고, 이후 `userId`가 바뀌어도 오래된 값을 출력할 수 있습니다.

이런 문제를 stale closure라고 부릅니다.

올바른 코드는 이렇습니다.

```tsx
const handleClick = useCallback(() => {
  console.log(userId);
}, [userId]);
```

dependency 배열은 성능을 위해 마음대로 줄이는 곳이 아닙니다.

정확성을 위해 필요한 값을 모두 넣어야 합니다.

---

## stale closure 피하기

아래 코드를 봅시다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const increase = useCallback(() => {
    setCount(count + 1);
  }, []);

  return <button onClick={increase}>{count}</button>;
}
```

`increase`는 `count`를 사용하지만 dependency 배열은 비어 있습니다.

그래서 `increase`는 처음 렌더링의 `count`를 기억합니다.

이 문제를 해결하는 방법은 두 가지입니다.

첫 번째는 dependency에 `count`를 넣는 것입니다.

```tsx
const increase = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

두 번째는 updater function을 사용하는 것입니다.

```tsx
const increase = useCallback(() => {
  setCount((prevCount) => prevCount + 1);
}, []);
```

이 방식은 현재 state를 직접 읽지 않고 React에게 이전 값을 기반으로 업데이트하라고 전달합니다.

그래서 dependency에서 `count`를 제거할 수 있습니다.

이 패턴은 특히 콜백을 안정적으로 유지하고 싶을 때 유용합니다.

---

## "함수가 새로 만들어지면 무조건 나쁜가?"

아닙니다.

함수가 새로 만들어지는 것 자체는 대부분 큰 문제가 아닙니다.

```tsx
<button onClick={() => setIsOpen(true)}>
  열기
</button>
```

이 정도 코드는 괜찮습니다.

문제가 되는 것은 보통 아래 조건이 함께 있을 때입니다.

```txt
1. 부모가 자주 리렌더링된다
2. 자식 컴포넌트가 무겁다
3. 자식을 memo로 감싸서 리렌더링을 건너뛰고 싶다
4. 그런데 함수 prop이 매번 새로 만들어져 memo가 깨진다
```

이 조건이 없다면 `useCallback`을 붙여도 체감 효과가 없을 가능성이 큽니다.

---

## "useMemo를 쓰면 첫 렌더링도 빨라질까?"

아닙니다.

`useMemo`는 첫 렌더링에서 계산을 실행합니다.

```tsx
const result = useMemo(() => {
  return expensiveCalculation(items);
}, [items]);
```

첫 렌더링에서는 `expensiveCalculation`이 실행됩니다.

`useMemo`가 도와주는 것은 이후 리렌더링에서 dependency가 같을 때 계산을 건너뛰는 것입니다.

```txt
첫 렌더링
→ 계산 실행

다음 렌더링
→ dependency가 같으면 계산 생략
```

즉, `useMemo`는 초기 로딩을 빠르게 만드는 도구가 아닙니다.

업데이트 렌더링에서 불필요한 계산을 줄이는 도구입니다.

---

## Strict Mode에서 두 번 실행되는 것처럼 보이는 이유

개발 환경에서 React Strict Mode를 사용하면 `useMemo`의 계산 함수가 두 번 실행되는 것처럼 보일 수 있습니다.

```tsx
const value = useMemo(() => {
  console.log("calculate");

  return expensiveCalculation(items);
}, [items]);
```

콘솔이 두 번 찍힌다고 해서 `useMemo`가 고장난 것은 아닙니다.

React는 개발 환경에서 순수하지 않은 계산을 찾기 위해 일부 함수를 의도적으로 한 번 더 호출할 수 있습니다.

프로덕션 빌드에서는 이 동작이 그대로 적용되지 않습니다.

그래서 성능 측정은 개발 모드가 아니라 production build 기준으로 확인하는 것이 좋습니다.

---

## 성능 최적화 전에 측정하기

`useMemo`와 `useCallback`은 "느릴 것 같아서" 넣는 도구가 아닙니다.

먼저 실제로 느린지 확인해야 합니다.

간단한 계산은 `console.time`으로도 확인할 수 있습니다.

```tsx
console.time("filter products");

const filteredProducts = products.filter((product) =>
  product.name.includes(keyword),
);

console.timeEnd("filter products");
```

React 컴포넌트 렌더링 비용은 React DevTools Profiler로 보는 편이 좋습니다.

확인할 질문은 이렇습니다.

```txt
어떤 컴포넌트가 자주 리렌더링되는가?
그 리렌더링이 실제로 느린가?
props가 같아도 다시 렌더링되는가?
객체, 배열, 함수 prop 때문에 memo가 깨지는가?
```

이 질문에 답한 뒤에 `useMemo`, `useCallback`, `memo`를 적용하는 것이 좋습니다.

---

## 실무 판단 기준

아래 기준으로 판단하면 헷갈림이 줄어듭니다.

### useMemo를 고려할 때

```txt
이 계산이 실제로 비싼가?
dependency가 자주 바뀌지 않는가?
계산 결과가 객체나 배열이고 memoized child에 전달되는가?
이 값이 다른 Hook의 dependency로 사용되는가?
```

전부 아니라면 보통 필요 없습니다.

### useCallback을 고려할 때

```txt
이 함수가 memoized child의 prop으로 내려가는가?
이 함수가 다른 Hook의 dependency로 쓰이는가?
커스텀 Hook에서 외부로 반환되는 함수인가?
dependency가 안정적인가?
```

전부 아니라면 보통 필요 없습니다.

---

## 흔한 실수 정리

### 1. 모든 함수를 useCallback으로 감싼다

```tsx
const handleClick = useCallback(() => {
  setOpen(true);
}, []);
```

단순 버튼 이벤트라면 꼭 필요하지 않습니다.

### 2. dependency를 일부러 비운다

```tsx
const handleSubmit = useCallback(() => {
  submit(form);
}, []);
```

`form`을 사용한다면 dependency에 넣어야 합니다.

```tsx
const handleSubmit = useCallback(() => {
  submit(form);
}, [form]);
```

dependency를 줄이는 것은 최적화가 아니라 버그의 시작일 수 있습니다.

### 3. memo 없이 useCallback만 쓴다

```tsx
const handleSelect = useCallback((id: string) => {
  setSelectedId(id);
}, []);

return <ItemList onSelect={handleSelect} />;
```

`ItemList`가 `memo`로 감싸져 있지 않거나, 자체 렌더링 비용이 작다면 의미가 거의 없을 수 있습니다.

### 4. 매번 새 객체를 dependency로 넣는다

```tsx
const options = {
  page,
  size,
};

const result = useMemo(() => {
  return calculate(options);
}, [options]);
```

`options`가 매번 새 객체라면 `useMemo`는 매번 다시 실행됩니다.

### 5. useMemo에 부수 효과를 넣는다

```tsx
const value = useMemo(() => {
  localStorage.setItem("keyword", keyword);

  return keyword.trim();
}, [keyword]);
```

`useMemo`는 계산 결과를 만들기 위한 곳입니다.

부수 효과는 `useEffect`에서 처리해야 합니다.

---

## 예제로 보는 최적화 전후

먼저 최적화 전 코드입니다.

```tsx
function Dashboard({ users, selectedRole, theme }: DashboardProps) {
  const visibleUsers = users.filter((user) => user.role === selectedRole);

  const handleSelect = (id: string) => {
    console.log(id);
  };

  return (
    <section className={theme}>
      <UserTable users={visibleUsers} onSelect={handleSelect} />
    </section>
  );
}
```

만약 `UserTable`이 무거운 컴포넌트라면 `theme`만 바뀌어도 다시 렌더링될 수 있습니다.

```tsx
const UserTable = memo(function UserTable({
  users,
  onSelect,
}: UserTableProps) {
  return (
    <table>
      {/* 무거운 테이블 렌더링 */}
    </table>
  );
});
```

이때 부모에서 배열과 함수를 안정화할 수 있습니다.

```tsx
function Dashboard({ users, selectedRole, theme }: DashboardProps) {
  const visibleUsers = useMemo(() => {
    return users.filter((user) => user.role === selectedRole);
  }, [users, selectedRole]);

  const handleSelect = useCallback((id: string) => {
    console.log(id);
  }, []);

  return (
    <section className={theme}>
      <UserTable users={visibleUsers} onSelect={handleSelect} />
    </section>
  );
}
```

이제 `theme`만 바뀌는 경우 `visibleUsers`와 `handleSelect`는 이전과 같은 참조를 유지합니다.

그러면 `UserTable`은 props가 바뀌지 않았다고 판단해 리렌더링을 건너뛸 수 있습니다.

이 예제에서 중요한 것은 `useMemo`, `useCallback`, `memo`가 함께 있다는 점입니다.

`useMemo`와 `useCallback`만 넣고 자식을 `memo`로 감싸지 않으면 효과가 제한적입니다.

---

## React Compiler와 수동 메모이제이션

React 공식 문서에서는 React Compiler가 값과 함수를 자동으로 메모이제이션해 수동 `useMemo`, `useCallback` 필요를 줄일 수 있다고 설명합니다.

하지만 모든 프로젝트가 React Compiler를 바로 사용하는 것은 아닙니다.

또 실제 실무 코드에서는 여전히 다음을 이해해야 합니다.

```txt
왜 객체/배열/함수 참조가 바뀌는가?
왜 memo가 깨지는가?
dependency 배열은 왜 필요한가?
어떤 최적화가 실제로 의미 있는가?
```

즉, Compiler가 있더라도 `useMemo`와 `useCallback`의 개념은 React 렌더링을 이해하는 데 중요합니다.

---

## 마무리

`useMemo`와 `useCallback`은 React 성능 최적화에서 자주 보이는 Hook입니다.

하지만 핵심은 단순합니다.

```txt
useMemo
→ 계산 결과를 기억한다

useCallback
→ 함수 참조를 기억한다

React.memo
→ props가 같으면 컴포넌트 리렌더링을 건너뛴다
```

무조건 많이 쓴다고 좋은 것이 아닙니다.

먼저 컴포넌트가 실제로 느린지 측정하고, 어떤 값이나 함수 때문에 memoization이 깨지는지 확인한 뒤 적용하는 것이 좋습니다.

실무에서 가장 좋은 기준은 이것입니다.

```txt
코드가 useMemo/useCallback 없이도 올바르게 동작하는가?
실제로 느린 지점이 확인되었는가?
dependency가 안정적인가?
memoized child나 Hook dependency와 연결되어 있는가?
```

이 질문에 답할 수 있다면 `useMemo`와 `useCallback`은 더 이상 헷갈리는 Hook이 아니라, 필요한 곳에 정확히 쓰는 도구가 됩니다.

---

## 참고 자료

- [React 공식 문서 - useMemo](https://react.dev/reference/react/useMemo)
- [React 공식 문서 - useCallback](https://react.dev/reference/react/useCallback)
- [React 공식 문서 - memo](https://react.dev/reference/react/memo)

{% endraw %}
