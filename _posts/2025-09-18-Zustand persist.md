---
title: "[React] Zustand 전역 상태 새로고침 시 초기화 문제 해결하기"
date: 2025-09-19 14:20:00 +0900
categories: [React, State-Management]
tags: [react, zustand, state-management, persist]
---

리액트에서 Zustand로 전역 상태 관리를 하다 보면, 브라우저를 새로고침할 때 상태가 전부 초기화되는 문제가 있어요 😅  
예를 들어 로그인 토큰, 다크 모드 여부 같은 값은 새로고침해도 유지돼야 하잖아요?  
저도 프로젝트를 하면서 이 이슈를 마주했고, **persist** 미들웨어를 사용해서 해결했습니다.

이번 글에서는 `Zustand 전역 상태를 로컬스토리지(LocalStorage)에 저장해서 새로고침 후에도 유지하는 방법`을 정리해볼게요!

---

### 💾 persist 미들웨어 설치

Zustand 자체에는 상태 영속화 기능이 없어요. 대신 공식적으로 제공되는 **persist** 미들웨어를 사용하면 됩니다.

- 영속화란? 브라우저 저장소 같은 지속적인 공간에 보관해서 새로고침이나 앱 재실행에도 유지되는 것

```bash
npm install zustand
# 이미 zustand가 있다면 따로 설치 필요 없음
```

zustand 패키지 안에 **persist**가 포함돼 있어요. 따로 다른 라이브러리를 설치할 필요는 없습니다.

<br/> 
<h3><b>🗂️ persist 적용한 스토어 만들기</b></h3>

```jsx
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
      name: "auth-storage", // 로컬스토리지 key
    }
  )
);
```

<ul> 
<li><code>persist</code>: 상태를 브라우저 저장소(LocalStorage/SessionStorage)에 저장해주는 미들웨어</li>
 <li><code>name</code>: 스토리지가 저장될 key 이름</li> 
 <li><code>token</code>: 로그인 토큰 같은 전역 상태</li> 
 </ul>

<br/> 
<h3><b>⚡ 컴포넌트에서 사용하기</b></h3>

```jsx
// components/LoginButton.tsx
import { useAuthStore } from "@/store/useAuthStore";

export default function LoginButton() {
  const { token, setToken, clearToken } = useAuthStore();

  return (
    <div className="flex flex-col gap-3 p-4">
      <h2 className="text-lg font-bold">현재 토큰: {token ?? "없음"}</h2>
      <button
        onClick={() => setToken("my-secret-token")}
        className="rounded-lg bg-blue-500 px-4 py-2 text-white"
      >
        로그인
      </button>
      <button
        onClick={clearToken}
        className="rounded-lg bg-red-500 px-4 py-2 text-white"
      >
        로그아웃
      </button>
    </div>
  );
}
```

이제 로그인 버튼을 눌러 토큰을 저장하고, 브라우저를 새로고침해도 상태가 유지되는 걸 확인할 수 있습니다 🙌

<br/> 
<h3><b>🔑 세션 스토리지 사용하기</b></h3>

**persist**는 기본적으로 LocalStorage를 쓰지만, SessionStorage로도 바꿀 수 있어요.

```jsx
persist(
  (set) => ({ ... }),
  {
    name: "auth-storage",
    storage: () => sessionStorage,
  }
);
```

이렇게 하면 브라우저를 완전히 닫았다가 다시 열면 상태가 사라지고, 새로고침까지만 유지됩니다.

<br/> 
<h3><b>📝 사용 후기</b></h3>

- Zustand의 **persist** 미들웨어 덕분에 로그인 상태나 사용자 설정 같은 데이터를 새로고침해도 잃어버리지 않을 수 있었어요.<br/>
- Redux-persist 같은 무거운 라이브러리를 쓰지 않아도 되고, 코드도 간단해서 프론트엔드 개발 생산성이 훨씬 좋아졌습니다<br/>
- 특히 다크 모드, 언어 설정, 토큰 관리 같은 경우에 강력하게 추천합니다.
