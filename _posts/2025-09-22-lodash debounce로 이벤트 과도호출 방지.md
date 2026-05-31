---
title: "[JavaScript] lodash debounce로 이벤트 과도 호출 방지하기"
date: 2025-09-22 18:10:00 +0900
categories: [javascript, performance]
tags: [javascript, lodash, debounce, event]
description: "lodash debounce를 사용해 검색 입력이나 resize 이벤트처럼 반복 호출되는 함수를 마지막 시점에만 실행하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

검색창에 글자를 입력할 때마다 API 요청을 보내면 서버 요청이 너무 많이 발생할 수 있습니다.

사용자가 `react`를 입력하는 동안 `r`, `re`, `rea`, `reac`, `react`마다 요청이 나가면 비효율적입니다.

이럴 때 사용할 수 있는 기법이 `debounce`입니다.

---

## debounce란?

debounce는 이벤트가 연속해서 발생할 때, 마지막 이벤트 이후 일정 시간이 지난 뒤 함수가 한 번만 실행되도록 하는 기법입니다.

예를 들어 입력이 멈춘 뒤 500ms가 지나면 검색 API를 호출하는 방식입니다.

검색 입력, 자동 저장, 유효성 검사에 자주 사용합니다.

---

## lodash 설치하기

```bash
npm install lodash
```

yarn을 사용한다면 다음 명령어로 설치할 수 있습니다.

```bash
yarn add lodash
```

---

## 기본 사용법

```js
import { debounce } from "lodash";

const search = (query) => {
  console.log("검색 API 호출:", query);
};

const debouncedSearch = debounce(search, 500);

debouncedSearch("r");
debouncedSearch("re");
debouncedSearch("rea");
debouncedSearch("react");
```

위 코드는 마지막 호출인 `"react"`만 500ms 뒤에 실행됩니다.

---

## 옵션 사용하기

```js
const debouncedSearch = debounce(search, 500, {
  leading: false,
  trailing: true,
});
```

`leading`은 첫 호출 시 바로 실행할지 정합니다.

`trailing`은 마지막 호출 이후 실행할지 정합니다.

기본적으로 debounce는 마지막 호출 이후 실행되는 `trailing` 방식에 가깝습니다.

---

## React에서 사용하기

```tsx
import { useEffect, useMemo, useState } from "react";
import { debounce } from "lodash";

const SearchBox = () => {
  const [query, setQuery] = useState("");

  const debouncedSearch = useMemo(
    () =>
      debounce((value: string) => {
        console.log("검색 API 호출:", value);
      }, 500),
    [],
  );

  useEffect(() => {
    return () => {
      debouncedSearch.cancel();
    };
  }, [debouncedSearch]);

  return (
    <input
      value={query}
      placeholder="검색어를 입력해주세요"
      onChange={(event) => {
        const nextValue = event.target.value;
        setQuery(nextValue);
        debouncedSearch(nextValue);
      }}
    />
  );
};

export default SearchBox;
```

컴포넌트가 사라질 때 `cancel()`을 호출하면 예약된 debounce 실행을 정리할 수 있습니다.

---

## throttle과의 차이

| 구분 | debounce | throttle |
| --- | --- | --- |
| 실행 시점 | 마지막 이벤트 이후 | 일정 시간마다 |
| 대표 사용처 | 검색 입력, 자동 저장 | 스크롤, 리사이즈 |
| 목적 | 마지막 상태만 반영 | 지속적으로 상태 반영 |

입력이 끝난 뒤 한 번만 실행해야 한다면 debounce가 좋습니다.

이벤트가 계속 발생하는 동안 주기적으로 실행해야 한다면 throttle이 좋습니다.

---

## 마무리

debounce는 연속으로 발생하는 이벤트를 마지막 시점에 한 번만 처리하게 해줍니다.

검색창 API 호출, 입력 검증, 자동 저장처럼 사용자의 동작이 멈춘 뒤 실행되어야 하는 작업에 잘 어울립니다.

React에서 사용할 때는 `useMemo`로 debounce 함수를 유지하고, cleanup에서 `cancel()`을 호출하는 방식이 안전합니다.
