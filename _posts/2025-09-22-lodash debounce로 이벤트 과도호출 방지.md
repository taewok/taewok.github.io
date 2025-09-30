---
title: "[JavaScript] lodash debounce로 이벤트 과도 호출 방지하기"
date: 2025-09-22 18:10:00 +0900
categories: [JavaScript, Performance]
tags: [javascript, lodash, debounce, event]
---

프론트엔드 개발을 하다 보면 검색창 입력, 윈도우 리사이즈 같은 이벤트가 `사용자 행동마다 너무 빠르게 반복`되는 경우가 많아요.  
이때마다 함수가 실행되면 서버 호출이 불필요하게 많이 발생하거나 성능에 문제가 생길 수 있습니다 😅

이럴 때 유용한 도구가 바로 lodash의 **debounce** 함수예요.  
이번 글에서는 `debounce`를 사용해서 이벤트 호출을 마지막 입력 시점에 한 번만 실행되도록 제어하는 방법을 정리해볼게요!

---

### 📦 lodash 설치

```bash
npm install lodash
# 또는
yarn add lodash
```

lodash는 debounce 외에도 throttle, cloneDeep, isEqual 등 다양한 유틸리티 함수를 제공합니다.

<br/>

<h3><b>⚡ debounce 기본 사용법</b></h3>

```jsx
import { debounce } from "lodash";

const search = (query) => {
  console.log("API 요청:", query);
};

const debouncedSearch = debounce(search, 1000); // 입력 후 1초 뒤 실행

document.getElementById("search").addEventListener("input", (e) => {
  debouncedSearch(e.target.value);
});
```

<ul> <li><code>debounce(func, wait)</code>: 입력이 멈춘 후 <code>wait</code> 밀리초가 지나야 실행</li> <li><code>search</code>: 원래 실행하려던 함수 (예: API 호출)</li> <li><code>debouncedSearch</code>: debounce로 감싼 함수</li> </ul>

위 코드에서는 사용자가 검색창에 입력할 때마다 API가 바로 호출되지 않고, 입력이 멈춘 뒤 1초 후에 한 번만 실행됩니다 🚀

<br/>

<h3><b>🛠️ 옵션 사용하기</b></h3>

lodash debounce는 실행 시점을 제어할 수 있는 옵션을 제공합니다.

```jsx
const debouncedSearch = debounce(search, 1000, {
  leading: true, // 입력 시작 시 실행 여부 (기본값 false)
  trailing: true, // 입력 끝난 후 실행 여부 (기본값 true)
});
```

- leading: true → 첫 입력 시 즉시 실행
- trailing: true → 입력이 끝난 뒤 실행
- 두 옵션을 조합해서 원하는 UX를 만들 수 있습니다.

<br/> 
<h3><b>🖼️ React에서 활용하기</b></h3>

React에서는 useCallback과 함께 debounce를 자주 사용합니다.

```jsx
import { useState, useCallback } from "react";
import { debounce } from "lodash";

export default function SearchBox() {
  const [query, setQuery] = useState("");

  const handleSearch = useCallback(
    debounce((value) => {
      console.log("검색 API 호출:", value);
    }, 500),
    []
  );

  return (
    <input
      type="text"
      value={query}
      placeholder="검색어 입력..."
      onChange={(e) => {
        setQuery(e.target.value);
        handleSearch(e.target.value);
      }}
      className="border rounded px-3 py-2"
    />
  );
}
```

이렇게 하면 사용자가 입력할 때마다 API가 과도하게 호출되는 걸 막고, 입력이 멈춘 뒤 0.5초 후에 한 번만 요청하도록 최적화할 수 있습니다 👍

<br/> 
<h3><b>📝 사용 후기</b></h3>

debounce를 쓰면서 가장 좋았던 점은, 사용자 입력 이벤트를 효율적으로 제어할 수 있다는 거였어요.
특히 검색창 자동완성, 입력 폼 검증, 윈도우 리사이즈 감지 같은 곳에서 효과적입니다.
throttle과 자주 비교되는데,

- debounce는 "마지막에 한 번 실행"
- throttle은 "주기적으로 실행"

이라는 차이를 이해하고 쓰면 상황에 맞게 쉽게 선택할 수 있습니다 🚀
