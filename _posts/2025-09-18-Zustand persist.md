---
title: "[React] Zustand persist로 새로고침 후에도 상태 유지하기"
date: 2025-09-18 14:20:00 +0900
categories: [react, state-management]
tags: [react, zustand, state-management, persist]
description: "Zustand persist middleware를 사용해 전역 상태를 localStorage나 sessionStorage에 저장하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

Zustand로 전역 상태를 관리하다 보면 브라우저를 새로고침했을 때 상태가 초기화되는 문제를 만날 수 있습니다.

예를 들어 로그인 토큰, 다크 모드 설정, 언어 설정처럼 새로고침 후에도 유지되어야 하는 값이 있습니다.

이럴 때는 Zustand의 `persist` middleware를 사용할 수 있습니다.

---

## persist란?

`persist`는 Zustand store의 상태를 브라우저 저장소에 저장해주는 middleware입니다.

기본적으로 localStorage를 사용하며, 필요하면 sessionStorage로 바꿀 수도 있습니다.

Zustand 패키지 안에 포함되어 있으므로 별도 라이브러리를 설치할 필요는 없습니다.

---

## 기본 사용법

```tsx
// store/useAuthStore.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AuthState {
  token: string | null;
  setToken: (token: string) => void;
  clearToken: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      setToken: (token) => set({ token }),
      clearToken: () => set({ token: null }),
    }),
    {
      name: "auth-storage",
    },
  ),
);
```

`name`은 localStorage에 저장될 key입니다.

이제 `token` 값은 새로고침 후에도 복원됩니다.

---

## 컴포넌트에서 사용하기

```tsx
import { useAuthStore } from "@/store/useAuthStore";

const LoginButton = () => {
  const token = useAuthStore((state) => state.token);
  const setToken = useAuthStore((state) => state.setToken);
  const clearToken = useAuthStore((state) => state.clearToken);

  return (
    <div>
      <p>현재 토큰: {token ?? "없음"}</p>
      <button onClick={() => setToken("my-secret-token")}>로그인</button>
      <button onClick={clearToken}>로그아웃</button>
    </div>
  );
};

export default LoginButton;
```

로그인 버튼을 눌러 token을 저장한 뒤 새로고침해도 값이 유지됩니다.

---

## sessionStorage 사용하기

기본 저장소는 localStorage입니다.

브라우저 탭이나 세션이 끝나면 사라지는 상태로 관리하고 싶다면 sessionStorage를 사용할 수 있습니다.

```tsx
import { createJSONStorage, persist } from "zustand/middleware";

persist(
  (set) => ({
    token: null,
    setToken: (token) => set({ token }),
  }),
  {
    name: "auth-storage",
    storage: createJSONStorage(() => sessionStorage),
  },
);
```

localStorage는 브라우저를 닫았다 열어도 남고, sessionStorage는 탭을 닫으면 사라집니다.

---

## 저장할 상태를 제한하기

store의 모든 값을 저장하고 싶지 않을 때는 `partialize`를 사용할 수 있습니다.

```tsx
persist(
  (set) => ({
    token: null,
    user: null,
    temporaryText: "",
  }),
  {
    name: "auth-storage",
    partialize: (state) => ({
      token: state.token,
      user: state.user,
    }),
  },
);
```

이렇게 하면 `temporaryText`는 저장하지 않고, `token`과 `user`만 저장합니다.

---

## 마무리

Zustand의 `persist`를 사용하면 전역 상태를 새로고침 후에도 유지할 수 있습니다.

로그인 토큰, 테마 설정, 언어 설정처럼 사용자가 다시 방문해도 유지되어야 하는 값에 잘 어울립니다.

다만 모든 상태를 저장하기보다, 정말 유지가 필요한 값만 선택해서 저장하는 것이 좋습니다.
