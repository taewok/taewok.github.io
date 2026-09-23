---
title: "[React] useSyncExternalStore 이해하기: 외부 store를 안전하게 구독하는 방법"
date: 2026-09-22T00:30:00Z
categories: [frontend]
tags: [react, hooks, useSyncExternalStore, external-store, state-management, ssr]
description: "React의 useSyncExternalStore가 왜 필요한지, subscribe, getSnapshot, getServerSnapshot의 역할과 외부 store, 브라우저 API, SSR에서 안전하게 사용하는 방법을 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: React 바깥의 상태를 어떻게 읽어야 할까?

React 컴포넌트에서 상태를 다룰 때는 보통 `useState`, `useReducer`, `useContext`를 사용합니다.

```tsx
const [count, setCount] = useState(0);
```

이 상태들은 React가 직접 관리합니다.

하지만 모든 상태가 React 안에 있는 것은 아닙니다.

예를 들어 이런 값들은 React 바깥에 있습니다.

```txt
window.navigator.onLine
window.innerWidth
localStorage
BroadcastChannel
WebSocket으로 받은 외부 데이터
Redux, Zustand 같은 외부 store
직접 만든 전역 store
```

이런 값을 React 컴포넌트에서 읽고, 값이 바뀔 때 다시 렌더링하고 싶다면 어떻게 해야 할까요?

예전에는 보통 `useEffect`와 `useState`를 조합했습니다.

```tsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return isOnline;
}
```

이 방식도 동작합니다.

하지만 React 18 이후 concurrent rendering까지 고려하면, React 바깥의 store를 읽고 구독하는 일은 더 조심해야 합니다.

이때 React가 제공하는 공식 API가 `useSyncExternalStore`입니다.

---

## useSyncExternalStore는 무엇인가?

React 공식 문서에서는 `useSyncExternalStore`를 외부 store를 구독할 수 있게 해주는 Hook이라고 설명합니다.

기본 형태는 이렇습니다.

```tsx
const snapshot = useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot,
);
```

각 인자의 역할은 다음과 같습니다.

| 인자 | 역할 |
| --- | --- |
| `subscribe` | 외부 store 변경을 구독하고, 변경 시 React에 알려주는 함수 |
| `getSnapshot` | 현재 외부 store 값을 읽어오는 함수 |
| `getServerSnapshot` | 서버 렌더링과 hydration에서 사용할 초기 값을 읽는 함수 |

간단히 말하면 이렇습니다.

```txt
subscribe
→ 값이 바뀌면 알려줘

getSnapshot
→ 지금 값이 뭐야?

getServerSnapshot
→ 서버에서는 초기 값을 뭐라고 볼까?
```

`useSyncExternalStore`는 이 세 가지를 이용해서 외부 store와 React 렌더링을 안전하게 연결합니다.

---

## 외부 store란 무엇일까?

여기서 말하는 external store는 꼭 Redux나 Zustand 같은 라이브러리만 의미하지 않습니다.

React가 직접 소유하지 않는 상태라면 외부 store로 볼 수 있습니다.

예를 들어 브라우저의 온라인 상태도 외부 store입니다.

```tsx
navigator.onLine
```

브라우저 창 크기도 외부 store입니다.

```tsx
window.innerWidth
```

직접 만든 전역 객체도 외부 store입니다.

```tsx
const counterStore = {
  count: 0,
  listeners: new Set<() => void>(),
};
```

핵심은 이 질문입니다.

```txt
이 값이 React state인가?
아니면 React 바깥에서 독립적으로 바뀔 수 있는 값인가?
```

React 바깥에서 바뀔 수 있고, React 컴포넌트가 그 변경을 구독해야 한다면 `useSyncExternalStore`를 고려할 수 있습니다.

---

## 가장 간단한 예: 온라인 상태 구독하기

React 공식 문서에서도 온라인 상태 예제를 보여줍니다.

먼저 현재 값을 읽는 함수가 필요합니다.

```tsx
function getSnapshot() {
  return navigator.onLine;
}
```

그리고 값이 바뀔 때 React에게 알려주는 구독 함수가 필요합니다.

```tsx
function subscribe(callback: () => void) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);

  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}
```

이제 Hook으로 만들 수 있습니다.

```tsx
import { useSyncExternalStore } from "react";

function subscribe(callback: () => void) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);

  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}

export function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot);
}
```

사용하는 쪽은 단순합니다.

```tsx
function NetworkStatus() {
  const isOnline = useOnlineStatus();

  return (
    <p>
      {isOnline ? "온라인 상태입니다." : "오프라인 상태입니다."}
    </p>
  );
}
```

흐름은 이렇습니다.

```txt
컴포넌트 렌더링
→ getSnapshot으로 현재 navigator.onLine 읽기
→ subscribe로 online/offline 이벤트 구독
→ 브라우저 온라인 상태 변경
→ callback 호출
→ React가 getSnapshot 다시 호출
→ 이전 snapshot과 다르면 리렌더링
```

---

## useEffect와 무엇이 다를까?

같은 기능을 `useEffect`와 `useState`로도 만들 수 있습니다.

```tsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function updateOnlineStatus() {
      setIsOnline(navigator.onLine);
    }

    window.addEventListener("online", updateOnlineStatus);
    window.addEventListener("offline", updateOnlineStatus);

    return () => {
      window.removeEventListener("online", updateOnlineStatus);
      window.removeEventListener("offline", updateOnlineStatus);
    };
  }, []);

  return isOnline;
}
```

단순한 UI에서는 이 코드도 동작합니다.

하지만 `useEffect`는 렌더링이 끝난 뒤 실행됩니다.

즉, 첫 렌더링 시점과 구독이 연결되는 시점 사이에 외부 store 값이 바뀔 수 있습니다.

그리고 React 18의 concurrent rendering에서는 렌더링 중 외부 store가 바뀌는 상황까지 더 안전하게 다뤄야 합니다.

`useSyncExternalStore`는 이런 외부 store 구독을 위해 만들어진 공식 Hook입니다.

정리하면 이렇습니다.

```txt
useEffect + useState
→ 렌더링 이후 구독을 연결하고 state로 복사함

useSyncExternalStore
→ 렌더링 과정에서 snapshot을 읽고 외부 store 변경과 React 렌더링을 안전하게 연결함
```

외부 store를 React와 연결하는 라이브러리를 만든다면 `useSyncExternalStore`를 사용하는 것이 권장됩니다.

---

## getSnapshot은 매우 중요하다

`getSnapshot`은 현재 외부 store의 값을 반환합니다.

```tsx
function getSnapshot() {
  return store.getState();
}
```

React는 store가 변경되었다는 알림을 받으면 `getSnapshot`을 다시 호출합니다.

그리고 이전 snapshot과 새 snapshot을 `Object.is`로 비교합니다.

```txt
이전 snapshot과 같음
→ 리렌더링하지 않음

이전 snapshot과 다름
→ 컴포넌트 리렌더링
```

그래서 `getSnapshot`은 같은 상태에서는 같은 값을 반환해야 합니다.

나쁜 예시입니다.

```tsx
function getSnapshot() {
  return {
    isOnline: navigator.onLine,
  };
}
```

이 함수는 호출할 때마다 새 객체를 반환합니다.

값은 같아도 객체 참조가 매번 다릅니다.

```txt
{ isOnline: true } !== { isOnline: true }
```

그러면 React는 snapshot이 계속 바뀐다고 판단할 수 있습니다.

공식 문서에서도 `getSnapshot` 결과는 캐시되어야 한다고 경고합니다.

primitive 값은 괜찮습니다.

```tsx
function getSnapshot() {
  return navigator.onLine;
}
```

객체를 반환해야 한다면 store 쪽에서 변경이 없을 때 같은 객체를 재사용해야 합니다.

```tsx
let snapshot = {
  count: 0,
};

function getSnapshot() {
  return snapshot;
}
```

store가 변경될 때만 새 snapshot을 만들어야 합니다.

```tsx
function setCount(nextCount: number) {
  snapshot = {
    count: nextCount,
  };

  emitChange();
}
```

핵심은 이것입니다.

```txt
getSnapshot은 "지금 상태"를 반환하되,
상태가 변하지 않았다면 같은 값을 반환해야 한다.
```

---

## subscribe는 무엇을 해야 할까?

`subscribe`는 callback을 받아 store 변경 시 호출해야 합니다.

```tsx
function subscribe(callback: () => void) {
  store.subscribe(callback);

  return () => {
    store.unsubscribe(callback);
  };
}
```

React는 이 callback이 호출되면 `getSnapshot`을 다시 실행합니다.

그래서 callback에 직접 새 값을 넣어줄 필요는 없습니다.

나쁜 예시입니다.

```tsx
function subscribe(callback: (nextValue: number) => void) {
  store.subscribe((nextValue) => {
    callback(nextValue);
  });
}
```

`useSyncExternalStore`의 callback은 값을 받는 함수가 아닙니다.

그저 "store가 바뀌었으니 다시 snapshot을 확인해봐"라고 알려주는 신호입니다.

좋은 예시는 이렇습니다.

```tsx
function subscribe(callback: () => void) {
  return store.subscribe(callback);
}
```

그리고 반드시 unsubscribe 함수를 반환해야 합니다.

```tsx
return () => {
  listeners.delete(callback);
};
```

컴포넌트가 언마운트될 때 구독을 정리하지 않으면 메모리 누수나 불필요한 업데이트가 생길 수 있습니다.

---

## 직접 작은 store 만들기

간단한 counter store를 직접 만들어보겠습니다.

```tsx
type Listener = () => void;

let count = 0;
const listeners = new Set<Listener>();

export const counterStore = {
  getSnapshot() {
    return count;
  },

  subscribe(listener: Listener) {
    listeners.add(listener);

    return () => {
      listeners.delete(listener);
    };
  },

  increment() {
    count += 1;

    listeners.forEach((listener) => listener());
  },

  decrement() {
    count -= 1;

    listeners.forEach((listener) => listener());
  },
};
```

이 store는 React를 모릅니다.

그냥 값을 가지고 있고, 값이 바뀌면 listeners를 호출합니다.

이제 React Hook으로 연결합니다.

```tsx
import { useSyncExternalStore } from "react";
import { counterStore } from "./counterStore";

export function useCounterStore() {
  return useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getSnapshot,
  );
}
```

컴포넌트에서는 이렇게 사용합니다.

```tsx
function Counter() {
  const count = useCounterStore();

  return (
    <section>
      <p>{count}</p>

      <button type="button" onClick={counterStore.decrement}>
        감소
      </button>

      <button type="button" onClick={counterStore.increment}>
        증가
      </button>
    </section>
  );
}
```

이제 `counterStore.increment()`가 호출되면 listeners가 실행되고, React는 `getSnapshot`을 다시 호출한 뒤 값이 바뀌었으면 컴포넌트를 리렌더링합니다.

---

## selector를 붙이면 필요한 값만 구독할 수 있다

store가 커지면 전체 store를 snapshot으로 반환하는 것은 부담스러울 수 있습니다.

예를 들어 store가 이런 구조라고 해보겠습니다.

```tsx
type AppState = {
  user: {
    id: string;
    name: string;
  };
  theme: "light" | "dark";
  notificationCount: number;
};
```

어떤 컴포넌트는 `theme`만 필요합니다.

어떤 컴포넌트는 `notificationCount`만 필요합니다.

이때 selector 패턴을 사용할 수 있습니다.

```tsx
function useAppStore<T>(selector: (state: AppState) => T) {
  return useSyncExternalStore(
    appStore.subscribe,
    () => selector(appStore.getSnapshot()),
  );
}
```

사용 예시는 이렇습니다.

```tsx
function ThemeButton() {
  const theme = useAppStore((state) => state.theme);

  return <button>{theme}</button>;
}
```

다만 이 방식에도 주의점이 있습니다.

selector가 매번 새 객체를 만들면 snapshot 안정성이 깨질 수 있습니다.

```tsx
const userView = useAppStore((state) => ({
  name: state.user.name,
  theme: state.theme,
}));
```

이 selector는 호출할 때마다 새 객체를 반환합니다.

그러면 상태가 실제로 바뀌지 않았어도 React가 다른 snapshot으로 볼 수 있습니다.

이런 경우에는 primitive 값을 고르거나, store/library 수준에서 equality 비교를 지원하는 구조가 필요합니다.

그래서 실무에서는 직접 복잡한 selector store를 만들기보다 Zustand, Redux 같은 검증된 라이브러리를 사용하는 경우가 많습니다.

`useSyncExternalStore`는 이런 라이브러리들이 React와 안전하게 연결되는 기초 API라고 이해하면 좋습니다.

---

## localStorage와 함께 사용할 때

`localStorage` 값도 React 바깥의 값입니다.

다만 `localStorage`는 값을 바꿨다고 자동으로 현재 탭의 React 컴포넌트에 알려주지 않습니다.

그래서 직접 이벤트를 발생시키는 store를 만들 수 있습니다.

```tsx
type Listener = () => void;

const listeners = new Set<Listener>();
const STORAGE_KEY = "theme";

function emitChange() {
  listeners.forEach((listener) => listener());
}

export const themeStore = {
  getSnapshot() {
    return localStorage.getItem(STORAGE_KEY) ?? "light";
  },

  subscribe(listener: Listener) {
    listeners.add(listener);

    window.addEventListener("storage", listener);

    return () => {
      listeners.delete(listener);
      window.removeEventListener("storage", listener);
    };
  },

  setTheme(theme: string) {
    localStorage.setItem(STORAGE_KEY, theme);

    emitChange();
  },
};
```

Hook은 이렇게 만들 수 있습니다.

```tsx
export function useTheme() {
  return useSyncExternalStore(
    themeStore.subscribe,
    themeStore.getSnapshot,
  );
}
```

여기서 `storage` 이벤트만으로는 같은 탭에서 직접 `localStorage.setItem`을 호출한 변경을 잡기 어렵습니다.

그래서 `setTheme` 안에서 직접 `emitChange()`를 호출했습니다.

이처럼 브라우저 API를 외부 store처럼 연결하려면 "값을 읽는 함수"와 "변경을 알리는 함수"를 직접 설계해야 합니다.

---

## media query 구독하기

다크 모드나 반응형 UI에서는 media query 상태를 구독하고 싶을 수 있습니다.

```tsx
function subscribe(callback: () => void) {
  const mediaQueryList = window.matchMedia("(prefers-color-scheme: dark)");

  mediaQueryList.addEventListener("change", callback);

  return () => {
    mediaQueryList.removeEventListener("change", callback);
  };
}

function getSnapshot() {
  return window.matchMedia("(prefers-color-scheme: dark)").matches;
}

export function usePrefersDarkMode() {
  return useSyncExternalStore(subscribe, getSnapshot);
}
```

다만 이 코드는 `subscribe`와 `getSnapshot`에서 각각 `window.matchMedia`를 호출합니다.

더 안정적으로는 같은 query 문자열을 기준으로 helper를 만들 수 있습니다.

```tsx
function createMediaQueryStore(query: string) {
  return {
    subscribe(callback: () => void) {
      const mediaQueryList = window.matchMedia(query);

      mediaQueryList.addEventListener("change", callback);

      return () => {
        mediaQueryList.removeEventListener("change", callback);
      };
    },

    getSnapshot() {
      return window.matchMedia(query).matches;
    },
  };
}

const darkModeStore = createMediaQueryStore(
  "(prefers-color-scheme: dark)",
);

export function usePrefersDarkMode() {
  return useSyncExternalStore(
    darkModeStore.subscribe,
    darkModeStore.getSnapshot,
  );
}
```

이렇게 하면 React 상태 없이도 브라우저 media query를 구독할 수 있습니다.

---

## SSR에서는 getServerSnapshot이 필요할 수 있다

서버 렌더링을 사용하는 환경에서는 세 번째 인자인 `getServerSnapshot`을 고려해야 합니다.

```tsx
const snapshot = useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot,
);
```

`getServerSnapshot`은 서버에서 렌더링할 때 사용됩니다.

그리고 서버에서 만든 HTML을 클라이언트가 hydration할 때도 사용됩니다.

중요한 점은 서버와 클라이언트의 초기 snapshot이 같아야 한다는 것입니다.

예를 들어 온라인 상태는 서버에서 정확히 알 수 없습니다.

이럴 때 서버에서는 기본값을 정할 수 있습니다.

```tsx
function getServerSnapshot() {
  return true;
}

export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot,
  );
}
```

하지만 서버에서는 `true`로 렌더링했는데 클라이언트 hydration 시점에 다른 값이 나오면 UI가 달라질 수 있습니다.

그래서 SSR에서는 아래 질문을 해야 합니다.

```txt
서버에서도 이 값을 의미 있게 알 수 있는가?
서버와 클라이언트의 초기 값이 같은가?
다르면 hydration mismatch가 생기지 않는가?
```

서버에서 알 수 없는 브라우저 전용 값이라면, 초기 값을 보수적으로 잡거나 클라이언트에서만 렌더링하는 구조를 고려해야 합니다.

---

## getServerSnapshot을 생략하면 어떻게 될까?

브라우저에서만 렌더링하는 앱이라면 세 번째 인자를 생략해도 됩니다.

```tsx
const value = useSyncExternalStore(subscribe, getSnapshot);
```

하지만 서버 렌더링 중에 이 Hook을 사용한다면 `getServerSnapshot`이 필요할 수 있습니다.

React 공식 문서에 따르면 `getServerSnapshot`은 서버 렌더링과 hydration 중 초기 snapshot을 제공하는 역할을 합니다.

서버에서 사용할 수 없는 값이라면 아래처럼 기본값을 제공할 수 있습니다.

```tsx
function getServerSnapshot() {
  return "light";
}
```

다만 이 기본값이 실제 클라이언트 값과 달라도 괜찮은지 UI 관점에서 확인해야 합니다.

예를 들어 테마 값이 달라서 서버에서는 light, 클라이언트에서는 dark로 바뀐다면 화면이 깜빡일 수 있습니다.

이런 경우에는 서버에서 쿠키를 읽어 초기 테마를 맞추거나, HTML에 초기 값을 주입해서 클라이언트가 같은 값을 읽도록 설계할 수 있습니다.

---

## useSyncExternalStore가 필요한 경우

이 Hook은 매일 쓰는 Hook은 아닙니다.

일반적인 컴포넌트 상태라면 `useState`가 더 적합합니다.

```tsx
const [isOpen, setIsOpen] = useState(false);
```

부모-자식 사이에서 상태를 공유하려면 props나 Context가 더 단순합니다.

```tsx
const ThemeContext = createContext("light");
```

`useSyncExternalStore`가 필요한 경우는 조금 더 특수합니다.

- React 바깥의 store를 구독해야 할 때
- 외부 상태 관리 라이브러리를 만들 때
- 브라우저 API 상태를 React와 연결해야 할 때
- 여러 컴포넌트가 같은 외부 store를 안전하게 읽어야 할 때
- concurrent rendering에서도 tearing 없이 일관된 snapshot을 읽고 싶을 때
- SSR과 hydration에서 외부 값의 초기 snapshot을 통제해야 할 때

즉, "React 안의 상태"를 만들기 위한 Hook이라기보다, "React 바깥의 상태를 React에 연결하기 위한 Hook"입니다.

---

## tearing이란 무엇일까?

`useSyncExternalStore`를 설명할 때 tearing이라는 말이 자주 나옵니다.

tearing은 같은 렌더링 흐름 안에서 서로 다른 컴포넌트가 같은 외부 store의 서로 다른 값을 읽는 문제를 말합니다.

예를 들어 외부 store 값이 `A`에서 `B`로 바뀌는 중이라고 해보겠습니다.

```txt
ComponentA는 A를 읽음
store가 B로 바뀜
ComponentB는 B를 읽음
같은 화면 안에 A와 B가 섞여 보임
```

React 상태만 사용하면 React가 렌더링 일관성을 관리합니다.

하지만 React 바깥의 store를 직접 읽으면 React가 그 변경 타이밍을 완전히 통제하기 어렵습니다.

`useSyncExternalStore`는 외부 store snapshot을 React 렌더링과 동기화해서 이런 문제를 줄이기 위해 제공된 API입니다.

그래서 이 Hook은 단순히 이벤트 구독을 편하게 해주는 Hook이 아닙니다.

React 18 이후의 렌더링 모델에서 외부 store를 안전하게 읽기 위한 공식 통로에 가깝습니다.

---

## useEffect로 충분한 경우도 있다

그렇다고 모든 이벤트 구독을 `useSyncExternalStore`로 바꿔야 하는 것은 아닙니다.

예를 들어 특정 컴포넌트 안에서만 쓰는 DOM 이벤트 처리라면 `useEffect`가 자연스럽습니다.

```tsx
useEffect(() => {
  function handleKeyDown(event: KeyboardEvent) {
    if (event.key === "Escape") {
      close();
    }
  }

  window.addEventListener("keydown", handleKeyDown);

  return () => {
    window.removeEventListener("keydown", handleKeyDown);
  };
}, [close]);
```

이 코드는 외부 store 값을 구독하는 것이 아니라 이벤트에 반응해서 어떤 동작을 수행합니다.

이런 경우에는 `useEffect`가 맞습니다.

반면 이런 경우라면 `useSyncExternalStore`가 더 적합합니다.

```txt
외부 값이 있고
컴포넌트가 그 값을 렌더링에 사용하고
값이 바뀌면 컴포넌트가 다시 렌더링되어야 한다
```

정리하면 이렇습니다.

| 상황 | 적합한 도구 |
| --- | --- |
| 이벤트가 발생하면 명령형 동작 실행 | `useEffect` |
| React 내부 상태 관리 | `useState`, `useReducer` |
| 여러 컴포넌트에 값 전달 | props, Context |
| React 바깥 store 값을 읽고 구독 | `useSyncExternalStore` |

---

## 흔한 실수 1. getSnapshot에서 매번 새 객체 반환하기

가장 흔한 실수입니다.

```tsx
function getSnapshot() {
  return {
    count: store.count,
  };
}
```

이렇게 하면 store가 변하지 않아도 `getSnapshot`이 매번 새 객체를 반환합니다.

React는 snapshot이 계속 달라진다고 판단할 수 있습니다.

가능하면 primitive 값을 반환하거나, 변경이 있을 때만 새 객체를 만들어야 합니다.

```tsx
function getSnapshot() {
  return store.count;
}
```

---

## 흔한 실수 2. subscribe를 컴포넌트 안에서 매번 새로 만들기

아래 코드는 렌더링마다 새 `subscribe` 함수가 만들어집니다.

```tsx
function Component() {
  const value = useSyncExternalStore(
    (callback) => store.subscribe(callback),
    () => store.getSnapshot(),
  );

  return <p>{value}</p>;
}
```

물론 작은 예제에서는 동작할 수 있습니다.

하지만 React는 `subscribe` 함수가 바뀌면 다시 구독해야 할 수 있습니다.

가능하면 컴포넌트 바깥에 정의하는 편이 좋습니다.

```tsx
function subscribe(callback: () => void) {
  return store.subscribe(callback);
}

function getSnapshot() {
  return store.getSnapshot();
}

function Component() {
  const value = useSyncExternalStore(subscribe, getSnapshot);

  return <p>{value}</p>;
}
```

또는 커스텀 Hook 안에서 안정적인 store 메서드를 그대로 전달합니다.

```tsx
function useCounterStore() {
  return useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getSnapshot,
  );
}
```

---

## 흔한 실수 3. unsubscribe를 반환하지 않기

나쁜 예시입니다.

```tsx
function subscribe(callback: () => void) {
  listeners.add(callback);
}
```

이렇게 하면 컴포넌트가 언마운트되어도 listener가 남습니다.

좋은 예시는 cleanup 함수를 반환하는 것입니다.

```tsx
function subscribe(callback: () => void) {
  listeners.add(callback);

  return () => {
    listeners.delete(callback);
  };
}
```

`useEffect`에서 cleanup을 반환하는 것처럼, `subscribe`도 구독 해제 함수를 반환해야 합니다.

---

## 흔한 실수 4. React state 대체제로 오해하기

`useSyncExternalStore`는 `useState`의 고급 버전이 아닙니다.

아래처럼 단순 모달 상태를 위해 외부 store를 만들 필요는 없습니다.

```tsx
const [isOpen, setIsOpen] = useState(false);
```

폼 입력 상태도 마찬가지입니다.

```tsx
const [name, setName] = useState("");
```

이런 상태는 React 내부 상태가 더 단순하고 자연스럽습니다.

`useSyncExternalStore`는 React 바깥의 값을 구독해야 할 때 쓰는 도구입니다.

---

## Zustand나 Redux와의 관계

Zustand, Redux 같은 상태 관리 라이브러리를 사용하면 보통 직접 `useSyncExternalStore`를 호출하지 않습니다.

라이브러리의 Hook을 사용합니다.

```tsx
const count = useCounterStore((state) => state.count);
```

또는 Redux라면 이렇게 사용합니다.

```tsx
const count = useSelector((state) => state.counter.count);
```

이런 라이브러리 내부에서 React와 외부 store를 연결하기 위해 `useSyncExternalStore` 계열 API를 활용할 수 있습니다.

즉, 일반 앱 개발자는 이 Hook을 매일 직접 쓰지 않을 수 있습니다.

하지만 이 Hook을 이해하면 상태 관리 라이브러리가 React와 어떻게 연결되는지 더 잘 이해할 수 있습니다.

```txt
store가 변경됨
→ 구독 callback 호출
→ React가 snapshot 확인
→ selector 결과가 바뀐 컴포넌트만 리렌더링
```

이 흐름을 알면 전역 상태 라이브러리를 사용할 때도 리렌더링과 selector 설계를 더 잘 판단할 수 있습니다.

---

## 실무 판단 기준

`useSyncExternalStore`를 직접 써야 할지 고민된다면 아래 질문을 해보면 됩니다.

```txt
이 값은 React state인가, React 바깥의 값인가?
컴포넌트가 이 값을 렌더링에 직접 사용하는가?
값이 바뀔 때 컴포넌트가 다시 렌더링되어야 하는가?
여러 컴포넌트가 같은 외부 값을 구독하는가?
SSR에서 초기 값을 맞춰야 하는가?
```

대부분 "예"라면 `useSyncExternalStore`를 고려할 수 있습니다.

반대로 아래에 가깝다면 다른 도구가 더 맞습니다.

```txt
컴포넌트 내부에서만 쓰는 상태
→ useState

이벤트에 반응해 명령형 작업만 실행
→ useEffect

부모에서 자식으로 값 전달
→ props

앱 전체 설정값 공유
→ Context

복잡한 전역 상태 관리
→ Zustand, Redux 등 라이브러리
```

---

## 전체 흐름 다시 보기

`useSyncExternalStore`는 세 부분으로 이해하면 됩니다.

```tsx
const value = useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot,
);
```

첫 번째는 구독입니다.

```tsx
function subscribe(callback: () => void) {
  store.subscribe(callback);

  return () => {
    store.unsubscribe(callback);
  };
}
```

두 번째는 현재 값 읽기입니다.

```tsx
function getSnapshot() {
  return store.getState();
}
```

세 번째는 서버 초기 값입니다.

```tsx
function getServerSnapshot() {
  return initialServerState;
}
```

흐름은 이렇습니다.

```txt
렌더링 중 getSnapshot 호출
→ 현재 외부 store 값 읽기
→ subscribe로 변경 구독
→ store 변경 시 callback 호출
→ React가 getSnapshot 재호출
→ snapshot이 바뀌었으면 리렌더링
```

이 패턴을 이해하면 브라우저 API, 직접 만든 store, 상태 관리 라이브러리가 React와 연결되는 방식이 훨씬 선명해집니다.

---

## 마무리

`useSyncExternalStore`는 React 바깥의 store를 React 컴포넌트와 안전하게 연결하기 위한 Hook입니다.

일반적인 컴포넌트 상태를 관리하려고 매일 쓰는 Hook은 아닙니다.

하지만 다음 상황에서는 중요한 도구가 됩니다.

```txt
외부 store를 구독해야 할 때
브라우저 API 값을 React 렌더링에 연결할 때
직접 상태 관리 store를 만들 때
React 18 이후 concurrent rendering에서 일관된 snapshot을 보장하고 싶을 때
SSR과 hydration에서 외부 값의 초기 상태를 맞춰야 할 때
```

핵심은 세 가지입니다.

```txt
subscribe
→ 변경을 알려준다

getSnapshot
→ 현재 값을 읽는다

getServerSnapshot
→ 서버와 hydration의 초기 값을 맞춘다
```

특히 `getSnapshot`은 상태가 바뀌지 않았다면 같은 값을 반환해야 합니다.

매번 새 객체를 반환하거나, unsubscribe를 빼먹거나, 단순 React state까지 외부 store로 만들면 오히려 복잡해집니다.

`useSyncExternalStore`는 자주 쓰는 Hook은 아니지만, React와 외부 상태가 만나는 지점을 이해하는 데 아주 좋은 API입니다.

---

## 참고 자료

- [React 공식 문서 - useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- [React 18 릴리즈 글](https://react.dev/blog/2022/03/29/react-v18)
- [React RFC - useSyncExternalStore](https://github.com/reactjs/rfcs/blob/main/text/0214-use-sync-external-store.md)

{% endraw %}
