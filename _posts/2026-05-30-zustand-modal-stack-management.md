---
title: "[React] Zustand 기반 모달 스택 관리 최적화하기"
date: 2026-05-30T00:30:00Z
categories: [frontend]
tags: [react, zustand, modal, typescript, state-management]
description: "Zustand로 여러 모달을 배열 스택으로 관리하고, ModalTypeMap과 GlobalModalProps를 활용해 중첩 모달과 타입 안전한 openModal 구조를 만든 과정을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 모달이 하나일 때와 여러 개일 때는 달라요

이번 프로젝트에서는 여러 모달을 전역에서 관리하기 위해 `Zustand` 기반의 모달 시스템을 사용하고 있습니다.

처음에는 단일 모달을 여닫는 정도의 구조였습니다. 예를 들어 로그인 모달을 열고, 닫기 버튼을 누르면 닫히는 단순한 방식이었어요.

하지만 서비스가 커지면서 모달을 하나만 관리하는 구조로는 부족한 상황이 생겼습니다. 대표적인 예가 **로그인 모달 위에서 비밀번호 찾기 모달을 여는 경우**입니다.

```txt
로그인 모달
└─ 비밀번호 찾기 모달
```

이때 로그인 모달을 닫고 비밀번호 찾기 모달을 여는 방식도 가능하지만, 사용자 흐름상 기존 모달 위에 새 모달이 쌓이는 편이 더 자연스러웠습니다.

그래서 모달을 단일 상태가 아니라 **배열 스택**으로 관리하는 구조로 개선했습니다. 이렇게 해두면 새 모달을 열 때 기존 모달을 억지로 닫지 않아도 되고, 닫을 때도 가장 위에 있는 모달만 자연스럽게 제거할 수 있습니다.

이번 글의 핵심은 다음 세 가지입니다.

- 모달을 배열 스택으로 관리합니다.
- 모달 타입과 props를 타입으로 매핑합니다.
- 모든 모달이 공유하는 props를 `GlobalModalProps`로 분리합니다.

---

## 💡 왜 스택 구조가 필요할까요?

모달을 하나만 관리한다면 상태는 단순합니다.

```ts
{
  type: "LOGIN",
  props: {}
}
```

하지만 중첩 모달이 필요해지는 순간 이 구조는 한계가 생깁니다.

로그인 모달이 열린 상태에서 비밀번호 찾기 모달을 열면 기존 모달을 어떻게 처리해야 할까요?

1. 로그인 모달을 닫고 비밀번호 찾기 모달만 보여줍니다.
2. 로그인 모달을 유지하고 그 위에 비밀번호 찾기 모달을 쌓습니다.

이번 프로젝트에서는 두 번째 방식이 더 적합했습니다. 사용자가 비밀번호 찾기를 닫으면 다시 로그인 모달로 돌아올 수 있어야 했기 때문이에요.

그래서 모달 상태를 배열로 관리했습니다.

```ts
modals: [
  { type: "LOGIN", props: { triggerRef } },
  { type: "FIND_PASSWORD", props: {} },
];
```

배열의 앞쪽은 먼저 열린 모달이고, 뒤쪽은 나중에 열린 모달입니다. 즉, 마지막에 추가된 모달이 가장 위에 있는 모달이 됩니다.

---

## 🧱 모달 타입과 props 매핑하기

먼저 모달 타입별로 어떤 props가 필요한지 정의합니다.

```ts
export type ModalTypeMap = {
  LOGIN: LoginModalProps;
  PROFILE_EDIT: ProfileEditModalProps;
  FIND_PASSWORD: FindPasswordModalProps;
};
```

이 구조의 장점은 모달 타입과 props 관계를 한 곳에서 볼 수 있다는 점입니다.

예를 들어 `LOGIN` 모달은 `LoginModalProps`를 받고, `FIND_PASSWORD` 모달은 `FindPasswordModalProps`를 받습니다. 나중에 새로운 모달을 추가할 때도 `ModalTypeMap`에 타입을 등록하면 됩니다.

이제 `ModalTypeMap`을 기반으로 실제 스토어에 저장할 모달 인스턴스 타입을 만들어볼게요.

```ts
type ModalInstanceUnion = {
  [K in keyof ModalTypeMap]: {
    type: K;
    props: Omit<ModalTypeMap[K], "onClose">;
  };
}[keyof ModalTypeMap];
```

처음 보면 조금 어렵지만, 의미는 단순합니다.

`ModalTypeMap`에 있는 모든 모달 타입을 순회하면서 다음과 같은 union 타입을 만드는 것입니다.

```ts
type ModalInstanceUnion =
  | {
      type: "LOGIN";
      props: Omit<LoginModalProps, "onClose">;
    }
  | {
      type: "PROFILE_EDIT";
      props: Omit<ProfileEditModalProps, "onClose">;
    }
  | {
      type: "FIND_PASSWORD";
      props: Omit<FindPasswordModalProps, "onClose">;
    };
```

여기서 `onClose`를 제외하는 이유는 `onClose`가 모달을 여는 쪽에서 넘겨야 하는 값이 아니기 때문입니다.

모달을 닫는 동작은 `ModalManager`가 Zustand store의 `closeModal`을 연결해서 공통으로 주입합니다. 따라서 `openModal`을 호출하는 쪽에서는 모달별로 필요한 props만 넘기면 됩니다.

---

## 🛠️ Zustand store에서 스택 관리하기

스토어는 `modals` 배열과 `openModal`, `closeModal` 액션을 가집니다.

```ts
interface ModalStore {
  modals: ModalInstanceUnion[];
  openModal: <T extends keyof ModalTypeMap>(
    type: T,
    props: Omit<ModalTypeMap[T], "onClose" | "stackIndex">,
  ) => void;
  closeModal: () => void;
  clearModals: () => void;
}
```

`openModal`은 새 모달을 배열 끝에 추가합니다.

```ts
openModal: (type, props) =>
  set((state) => ({
    modals: [...state.modals, { type, props } as ModalInstanceUnion],
  }));
```

배열 끝에 추가한다는 것은 스택의 가장 위에 새 모달을 올린다는 뜻입니다.

```txt
기존: [LOGIN]
추가: [LOGIN, FIND_PASSWORD]
```

반대로 `closeModal`은 마지막 모달만 닫습니다.

```ts
closeModal: () =>
  set((state) => ({
    modals: state.modals.slice(0, -1),
  }));
```

`slice(0, -1)`은 배열의 마지막 요소를 제외한 새 배열을 만듭니다.

```txt
닫기 전: [LOGIN, FIND_PASSWORD]
닫기 후: [LOGIN]
```

이 구조 덕분에 중첩 모달이 자연스럽게 동작합니다. 로그인 모달 위에서 비밀번호 찾기 모달을 열고, 비밀번호 찾기 모달만 닫으면 다시 로그인 모달이 남습니다.

---

## 🧭 ModalManager에서 모달 렌더링하기

`ModalManager`는 store의 `modals` 배열을 순회하면서 실제 모달 컴포넌트를 렌더링합니다.

```tsx
{
  modals.map((modal, index) => {
    const ModalComponent = MODAL_COMPONENTS[modal.type];

    return (
      <ModalComponent
        key={`${modal.type}-${index}`}
        {...modal.props}
        onClose={closeModal}
        stackIndex={index}
      />
    );
  });
}
```

여기서 중요한 값은 `stackIndex`입니다.

중첩 모달이 가능해지면 단순히 여러 모달을 렌더링하는 것만으로는 부족합니다. 나중에 열린 모달이 시각적으로도 더 위에 있어야 하기 때문입니다.

그래서 렌더링 순서의 `index`를 `stackIndex`로 넘겨줍니다.

```txt
LOGIN         -> stackIndex: 0
FIND_PASSWORD -> stackIndex: 1
```

이 값은 `ModalLayout`에서 `z-index`를 계산하는 데 사용됩니다.

---

## 🎚️ ModalLayout에서 z-index 계산하기

`ModalLayout`은 `stackIndex`를 받아 overlay와 modal content의 `z-index`를 계산합니다.

```tsx
const overlayZIndex = 100 + stackIndex * 2;
const modalZIndex = overlayZIndex + 1;
```

그리고 각각에 적용합니다.

```tsx
<div
  className="fixed inset-0 bg-black/50"
  style={{ zIndex: overlayZIndex }}
/>

<div
  role="dialog"
  aria-modal="true"
  style={{ zIndex: modalZIndex }}
>
  {children}
</div>
```

이렇게 하면 모달은 다음처럼 쌓입니다.

```txt
첫 번째 모달 overlay: 100
첫 번째 모달 content: 101

두 번째 모달 overlay: 102
두 번째 모달 content: 103

세 번째 모달 overlay: 104
세 번째 모달 content: 105
```

좋은 점은 명확합니다.

- 나중에 열린 모달이 항상 위에 뜹니다.
- overlay도 각 모달과 함께 올바른 계층을 가집니다.
- 중첩 모달을 열 때 기존 모달을 억지로 닫지 않아도 됩니다.
- z-index 값을 모달마다 직접 하드코딩하지 않아도 됩니다.

---

## 🧩 공통 props를 GlobalModalProps로 분리하기

이번 개선에서 가장 중요했던 부분은 타입 정의였습니다.

처음에는 각 모달 props가 이런 식으로 따로 `onClose`를 선언하고 있었습니다.

```ts
export interface LoginModalProps {
  onClose: () => void;
  triggerRef: React.RefObject<HTMLElement | null> | undefined;
}

export interface ProfileEditModalProps {
  onClose: () => void;
}

export interface StorageModalProps {
  onClose: () => void;
}
```

문제는 `onClose`가 모든 모달에 공통으로 필요한 값인데도 각 타입마다 반복되고 있었다는 점입니다.

여기에 중첩 모달을 위한 `stackIndex`까지 추가되면 반복은 더 심해집니다.

그래서 공통 props를 분리했습니다.

```ts
export interface GlobalModalProps {
  onClose: () => void;
  stackIndex?: number;
}
```

추가 props가 있는 모달은 `GlobalModalProps`를 상속합니다.

```ts
export interface LoginModalProps extends GlobalModalProps {
  triggerRef: React.RefObject<HTMLElement | null> | undefined;
}

export interface FollowModalProps extends GlobalModalProps {
  activeTab: "followers" | "following";
}

export interface PersonaAddModalProps extends GlobalModalProps {
  isEditMode?: boolean;
  personaId?: string;
}
```

추가 props가 없는 모달은 빈 interface 대신 type alias를 사용했습니다.

```ts
export type ProfileEditModalProps = GlobalModalProps;
export type StorageModalProps = GlobalModalProps;
export type FindPasswordModalProps = GlobalModalProps;
```

빈 interface를 쓰지 않은 이유는 ESLint 규칙 때문입니다.

```ts
export interface ProfileEditModalProps extends GlobalModalProps {}
```

이런 코드는 타입상 문제는 없지만, `@typescript-eslint/no-empty-object-type` 규칙에 걸릴 수 있습니다.

빈 interface는 상위 타입과 완전히 동일하기 때문에 굳이 새 interface로 선언할 이유가 없습니다. 추가 필드가 없다면 type alias가 더 명확합니다.

---

## ✅ 이렇게 타입을 정의하면 좋은 점

### 1. 공통 props 변경이 쉬워집니다

모든 모달에 `stackIndex`, `onClose`, 나중에는 `modalId` 같은 값이 필요해질 수 있습니다.

이때 `GlobalModalProps`만 수정하면 됩니다.

```ts
export interface GlobalModalProps {
  onClose: () => void;
  stackIndex?: number;
  modalId?: string;
}
```

각 모달 props에 일일이 `modalId`를 추가할 필요가 없습니다.

### 2. 모달별 props가 명확해집니다

```ts
export interface FollowModalProps extends GlobalModalProps {
  activeTab: "followers" | "following";
}
```

이 타입을 보면 `FollowModal`이 공통 모달 props 외에 실제로 필요한 값은 `activeTab`뿐이라는 것을 바로 알 수 있습니다.

공통 props와 모달별 props가 섞여 있지 않기 때문에 컴포넌트가 어떤 값을 요구하는지 더 쉽게 파악할 수 있습니다.

### 3. openModal 호출이 안전해집니다

`ModalTypeMap`이 있기 때문에 모달 타입에 맞는 props만 넘길 수 있습니다.

```ts
openModal("FOLLOW", { activeTab: "followers" });
```

반대로 잘못된 props를 넘기면 TypeScript가 잡아줍니다.

```ts
openModal("FOLLOW", { userId: 1 }); // 타입 에러
```

`FOLLOW` 모달은 `activeTab`을 요구하는데 `userId`를 넘겼기 때문에 타입 에러가 발생합니다.

### 4. 컴포넌트 구현과 store 구조가 느슨하게 연결됩니다

모달 컴포넌트는 자신의 props를 받으면 됩니다.

```tsx
const FindPasswordModal = ({ onClose, stackIndex }: FindPasswordModalProps) => {
  return (
    <ModalLayout onClose={onClose} stackIndex={stackIndex} hasBackground>
      ...
    </ModalLayout>
  );
};
```

store는 모달이 어떤 컴포넌트인지 자세히 알 필요가 없습니다. `ModalManager`가 `MODAL_COMPONENTS` registry를 보고 렌더링하면 됩니다.

```ts
export const MODAL_COMPONENTS: {
  [K in keyof ModalTypeMap]: ComponentType<ModalTypeMap[K]>;
} = {
  ADD_LANGUAGE: AddLanguageModal,
  CHATTING_START: ChattingStartModal,
  FIND_PASSWORD: FindPasswordModal,
  FOLLOW: FollowModal,
  LOGIN: LoginModal,
  PERSONA_ADD: PersonaAddModal,
  PROFILE_EDIT: ProfileEditModal,
  STORAGE: StorageModal,
  TAG_ADD: TagAddModal,
  TAG_SUGGESTIONS: TagSuggestionsModal,
  USER_NOTE: UserNoteModal,
  PERSONA: PersonaModal,
};
```

이 구조에서는 새로운 모달을 추가할 때도 흐름이 일정합니다.

1. 모달 컴포넌트를 만듭니다.
2. 모달 props 타입을 만듭니다.
3. `ModalTypeMap`에 등록합니다.
4. `MODAL_COMPONENTS`에 컴포넌트를 연결합니다.

---

## 🔄 최종 흐름으로 다시 보기

로그인 모달이 열린 상태에서 비밀번호 찾기 모달을 여는 상황을 다시 볼게요.

```ts
openModal("LOGIN", { triggerRef });
openModal("FIND_PASSWORD", {});
```

Zustand store에는 모달이 배열로 쌓입니다.

```ts
modals: [
  { type: "LOGIN", props: { triggerRef } },
  { type: "FIND_PASSWORD", props: {} },
];
```

`ModalManager`가 순서대로 렌더링합니다.

```tsx
<LoginModal onClose={closeModal} stackIndex={0} triggerRef={triggerRef} />
<FindPasswordModal onClose={closeModal} stackIndex={1} />
```

각 모달은 `ModalLayout`에 `stackIndex`를 넘깁니다.

```tsx
<ModalLayout onClose={onClose} hasBackground stackIndex={stackIndex}>
  ...
</ModalLayout>
```

결과적으로 로그인 모달 위에 비밀번호 찾기 모달이 안정적으로 쌓입니다.

비밀번호 찾기 모달에서 닫기 버튼을 누르면 `closeModal`이 실행되고, 배열의 마지막 모달만 제거됩니다.

```txt
[LOGIN, FIND_PASSWORD] -> [LOGIN]
```

사용자는 다시 로그인 모달을 볼 수 있습니다.

---

## 🚧 주의할 점

### 1. closeModal은 마지막 모달만 닫는다는 것을 명확히 해야 합니다

스택 구조에서는 `closeModal`이 항상 마지막 모달만 닫습니다.

이름만 보면 특정 모달을 닫는 함수처럼 오해할 수 있습니다. 팀에서 함께 사용하는 구조라면 `closeTopModal`처럼 더 명확한 이름을 사용하는 것도 좋습니다.

```ts
closeTopModal: () =>
  set((state) => ({
    modals: state.modals.slice(0, -1),
  }));
```

### 2. 너무 깊은 중첩은 UX를 해칠 수 있습니다

기술적으로는 모달을 계속 쌓을 수 있지만, 사용자가 여러 겹의 모달을 이해하기는 어렵습니다.

스택 구조를 만들더라도 실제 서비스에서는 2단계 정도까지만 허용하거나, 특정 상황에서는 기존 모달을 닫고 새 모달을 여는 정책을 정해두는 것이 좋습니다.

---

## 🏁 마무리

이번 개선의 핵심은 모달을 잘 띄우는 것보다 **모달을 타입으로 잘 설명하는 것**에 가까웠습니다.

모달은 프로젝트가 커질수록 종류가 늘어나고, props도 제각각 달라집니다. 이때 `ModalTypeMap`과 `GlobalModalProps`를 함께 사용하면 모달 시스템을 훨씬 관리하기 쉬워집니다.

특히 중첩 모달을 지원해야 한다면 `stackIndex` 같은 공통 props가 필요해집니다. 이 값을 각 모달 타입에 반복해서 넣기보다 `GlobalModalProps`로 모으는 것이 더 안전하고 확장성도 좋습니다.

결과적으로 이번 구조는 다음 장점을 가집니다.

- 중첩 모달 지원
- 타입 안전한 `openModal`
- 공통 props의 중앙 관리
- 모달별 props의 명확한 분리
- z-index 충돌 방지
- 새로운 모달 추가 시 일관된 패턴 유지

Zustand는 상태를 단순하게 관리할 수 있게 해주는 도구지만, TypeScript와 함께 구조를 잘 잡으면 UI 시스템의 복잡도도 꽤 깔끔하게 다룰 수 있습니다.
