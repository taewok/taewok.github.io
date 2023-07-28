---
title: "[React] react-query 사용하기"
date: 2023-07-28T16:19:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b>react-query 사용하기</b>

react-query는 React 애플리케이션에서 데이터 관리를 간편하게 해주는 라이브러리이며, 주로 서버로부터 데이터를 가져오거나 데이터를 업데이트하는 작업을 처리하는데 사용되고, 비동기 데이터 요청과 캐싱을 처리하며, 서버와의 상호작용을 간소화하여 데이터 관리를 용이하게 합니다.

<h3><blockquote>특징, 기능
</blockquote></h3>

- 데이터 캐싱: React Query는 데이터를 캐싱하여 여러 번의 요청에서 동일한 데이터를 다시 불러올 필요가 없도록 합니다. 이를 통해 애플리케이션의 성능을 향상시킵니다.

- 자동 재요청: 데이터가 변경되면 React Query는 자동으로 새로운 데이터를 요청하고 업데이트합니다. 이를 통해 데이터의 실시간 업데이트를 지원합니다.

- 비동기 데이터 요청: React Query는 비동기 API 호출을 처리하고 Promise, Axios, fetch 등과 같은 다양한 방법을 사용하여 데이터를 가져올 수 있습니다.

- 쿼리 키 기반 관리: 각 쿼리는 고유한 키로 관리되며, 이를 통해 쿼리에 대한 인터랙션과 갱신을 조작할 수 있습니다.

- 인터벌 리플레쉬: 일정 시간마다 데이터를 자동으로 새로고침하여 데이터를 최신 상태로 유지할 수 있습니다.

- 오류 핸들링: 데이터 요청 중 발생하는 오류를 쉽게 처리하고 관리합니다.

<h3><blockquote>설치
</blockquote></h3>

```js
//npm
npm install react-query

//yarn
yarn add react-query
```

<h3><blockquote>QueryClientProvider
</blockquote></h3>

React Query에서 제공되는 컴포넌트로 애플리케이션 내에서 QueryClient를 전역적으로 제공하고 관리하는 역할을 합니다.

- <b>QueryClient</b>: React Query에서 핵심적인 역할을 하는 클래스이며, 애플리케이션 전체의 쿼리 인스턴스와 캐시를 관리하고, 비동기 데이터 요청 및 쿼리 상태를 추적합니다.

```js
//index.js
import React from "react";
import { QueryClient, QueryClientProvider } from "react-query";
import App from "./App";

const queryClient = new QueryClient();

function App() {
  return (
    // QueryClientProvider로 둘러싼 모든 하위 컴포넌트에서 QueryClient를 사용할 수 있게 됨
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  );
}

export default App;
```

<h3><blockquote>react-query Hook
</blockquote></h3>

#### 💡 useQuery

비동기 데이터 요청을 처리하고 쿼리의 상태를 추적하는데 사용됩니다.

사용예시

```js
import { useQuery } from "react-query";

const { data: imgList, isLoading: imgLoading } = useQuery(["key이름"], 비동기통신 함수, {
  onSuccess: (res) => {},
  onError: (err) => {},
});
```

- <b>data</b>: 비동기 데이터 요청이 성공적으로 완료되었을 때 해당 데이터를 담고 있다
- <b>isLoading</b>: 비동기 데이터 요청이 진행 중인지를 나타내는 불리언(Boolean) 값
- <b>onSuccess</b>: 쿼리의 성공적인 데이터 가져오기가 완료되었을 때 실행할 함수를 정의
- <b>onError</b>: 쿼리 실행 중에 오류가 발생했을 때 실행되는 콜백 함수를 정의

다양한 객체와 옵션 객체들

```js
const {
  data, // 비동기 데이터를 담는 객체
  dataUpdatedAt, // 비동기 데이터가 최근에 업데이트된 시간 정보
  error, // 발생한 오류 객체
  errorUpdatedAt, // 오류가 최근에 업데이트된 시간 정보
  failureCount, // 데이터 요청에 실패한 횟수
  isError, // 데이터 요청이 오류 상태인지 여부
  isFetched, // 데이터 요청이 실행되었고 결과를 가져온 상태인지 여부
  isFetchedAfterMount, // 컴포넌트가 마운트된 후에 최초로 데이터를 가져온 상태인지 여부
  isFetching, // 데이터를 가져오는 중인지 여부
  isIdle, // 아직 데이터 요청이 실행되지 않았거나 에러가 없는 상태인지 여부
  isLoading, // 데이터를 로딩 중인지 여부
  isLoadingError, // 로딩 중에 오류가 발생한 상태인지 여부
  isPlaceholderData, // placeholderData가 사용되고 있는지 여부
  isPreviousData, // 이전 데이터를 사용하는지 여부
  isRefetchError, // refetch 중에 오류가 발생한 상태인지 여부
  isRefetching, // 데이터를 refetch 중인지 여부
  isStale, // 데이터가 캐시된 시간을 초과해서 Stale한 상태인지 여부
  isSuccess, // 데이터 요청이 성공적인 상태인지 여부
  refetch, // 데이터를 재요청하는 함수
  remove, // 쿼리의 데이터를 삭제하는 함수
  status, // 쿼리의 현재 상태를 나타내는 문자열 (예: 'loading', 'error', 'success' 등)
} = useQuery(queryKey, queryFn, {
  cacheTime, // 데이터의 캐시 유지 시간 (기본값: 5분)
  enabled, // 쿼리를 활성화 또는 비활성화하는 데 사용 (기본값: true)
  initialData, // 초기 데이터를 설정하는데 사용 (기본값: undefined)
  initialDataUpdatedAt, // 초기 데이터가 업데이트된 시간 정보
  isDataEqual, // 데이터가 이전과 동일한지를 비교하는 함수
  keepPreviousData, // 이전 데이터를 유지하는데 사용 (기본값: false)
  meta, // 쿼리의 추가 메타데이터를 저장하는 객체
  notifyOnChangeProps, // 쿼리 결과의 특정 속성 변경 시 재호출하도록 지정하는 속성 배열
  notifyOnChangePropsExclusions, // notifyOnChangeProps에서 제외할 속성 배열
  onError, // 쿼리 실행 중 오류 발생 시 실행되는 콜백 함수
  onSettled, // 쿼리 실행 종료 후 항상 실행되는 콜백 함수
  onSuccess, // 쿼리 실행 성공 시 실행되는 콜백 함수
  placeholderData, // 쿼리 데이터 로딩 중에 표시할 임시 데이터
  queryKeyHashFn, // 쿼리 키를 해싱하는 함수
  refetchInterval, // 자동으로 refetch를 실행하는 간격 (밀리초 단위)
  refetchIntervalInBackground, // 백그라운드에서 실행될 때의 refetch 간격 (밀리초 단위)
  refetchOnMount, // 컴포넌트가 마운트될 때 자동으로 refetch 실행 (기본값: true)
  refetchOnReconnect, // 네트워크 재연결 시 자동으로 refetch 실행 (기본값: true)
  refetchOnWindowFocus, // 윈도우 포커스 시 자동으로 refetch 실행 (기본값: true)
  retry, // 쿼리 실행 중 오류 발생 시 자동으로 재시도할 횟수 (기본값: 3)
  retryOnMount, // 컴포넌트가 마운트될 때 자동으로 retry 실행 (기본값: false)
  retryDelay, // 오류 발생 후 재시도할 때의 딜레이 시간 (밀리초 단위, 기본값: 0)
  select, // 쿼리 결과에서 특정 데이터를 선택하는 함수
  staleTime, // 쿼리 결과가 Stale 상태로 표시되는 시간 (밀리초 단위)
  structuralSharing, // 구조 공유(Structural Sharing) 옵션 (기본값: false)
  suspense, // Suspense를 사용할지 여부 (기본값: false)
  useErrorBoundary, // 오류 발생 시 ErrorBoundary를 사용할지 여부 (기본값: false)
});
```

#### 💡 useMutation

서버에 데이터를 변경하는데 사용됩니다. 주로 POST, PUT, DELETE 등의 HTTP 메서드를 사용하여 데이터를 생성, 수정, 삭제하는 작업을 처리할 때 유용하게 사용됩니다.

사용예시

```js
const { mutate: getUserInfo, isLoading: userInfoLoading } = useMutation(
  () => postUserId(id),
  {
    onSuccess: (res) => {},
    onError: (err) => {},
  }
);

useEffect(() => {
  getUserInfo();
}, []);
```

- <b>mutate</b>:  mutate 메서드를 호출하여 비동기 작업을 원하는 때에 호출할 수 있다.

---

## <b>마치며</b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>