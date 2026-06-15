---
title: "[TypeScript] Partial, Path, T 제대로 이해하고 활용하기"
date: 2026-06-04T10:30:00Z
categories: [typescript]
tags: [typescript, generic, partial, react-hook-form, type]
description: "TypeScript의 제네릭 T, Partial 같은 유틸리티 타입, React Hook Form에서 자주 만나는 Path<T>를 예제로 이해하고 실제 코드에서 활용하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 타입 문법이 갑자기 어려워지는 순간

TypeScript를 쓰다 보면 처음에는 `string`, `number`, `boolean` 정도만 알아도 꽤 많은 코드를 작성할 수 있습니다.

```ts
const name: string = "taewok";
const age: number = 20;
const isAdmin: boolean = false;
```

그런데 프로젝트가 조금 커지면 갑자기 이런 타입들이 등장합니다.

```ts
Partial<User>
Path<T>
T extends FieldValues
Record<string, string>
keyof T
```

처음 보면 기호도 많고, 읽는 순서도 헷갈립니다.

특히 `Partial`, `Path`, `T` 같은 타입은 그냥 외우면 금방 잊어버립니다. 그래서 이 글에서는 각각을 따로 암기하기보다, **타입을 재사용하고 제한하는 도구**라는 관점으로 정리해보겠습니다.

---

## 💡 먼저 T부터 이해하기

`T`는 TypeScript의 제네릭에서 자주 사용하는 이름입니다.

제네릭은 쉽게 말해 **타입을 나중에 받는 문법**입니다.

예를 들어 다음 함수가 있습니다.

```ts
function identity(value: string): string {
  return value;
}
```

이 함수는 문자열을 받아 문자열을 반환합니다.

그런데 숫자도 받고 싶고, boolean도 받고 싶다면 어떻게 해야 할까요?

```ts
function identityString(value: string): string {
  return value;
}

function identityNumber(value: number): number {
  return value;
}

function identityBoolean(value: boolean): boolean {
  return value;
}
```

이렇게 타입마다 함수를 만들면 너무 반복이 많습니다.

이때 제네릭을 사용할 수 있습니다.

```ts
function identity<T>(value: T): T {
  return value;
}
```

여기서 `T`는 고정된 타입이 아니라, 함수를 사용할 때 결정되는 타입입니다.

```ts
const name = identity("taewok");
const age = identity(20);
const isAdmin = identity(false);
```

TypeScript는 각각을 이렇게 추론합니다.

```ts
const name: string;
const age: number;
const isAdmin: boolean;
```

즉, `T`는 이런 의미로 볼 수 있습니다.

```txt
T는 아직 정해지지 않은 타입 자리입니다.
사용하는 순간 실제 타입으로 채워집니다.
```

---

## 📌 T라는 이름이 특별한 건 아닙니다

`T`는 TypeScript 예약어가 아닙니다.

관례적으로 많이 쓰는 이름일 뿐입니다.

```ts
function identity<Value>(value: Value): Value {
  return value;
}
```

이렇게 써도 똑같이 동작합니다.

다만 보통은 짧게 `T`를 많이 씁니다.

자주 보이는 이름은 다음과 같습니다.

| 이름 | 의미 |
| --- | --- |
| `T` | Type |
| `K` | Key |
| `V` | Value |
| `U` | 또 다른 타입 |

예를 들어 객체에서 특정 key의 값을 꺼내는 함수를 만들어보겠습니다.

```ts
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

처음 보면 어렵지만, 나눠서 보면 이렇게 읽을 수 있습니다.

```txt
T = 객체 타입
K = T 객체의 key 중 하나
T[K] = 그 key에 해당하는 value 타입
```

사용해보면 장점이 바로 보입니다.

```ts
const user = {
  id: 1,
  nickname: "taewok",
  isAdmin: false,
};

const nickname = getValue(user, "nickname");
```

`nickname`은 자동으로 `string` 타입이 됩니다.

```ts
const nickname: string;
```

잘못된 key를 넘기면 TypeScript가 막아줍니다.

```ts
getValue(user, "email"); // 타입 에러
```

`email`은 `user` 객체에 없는 key이기 때문입니다.

---

## 🧩 Partial<T> 이해하기

`Partial<T>`는 TypeScript가 기본으로 제공하는 유틸리티 타입입니다.

역할은 단순합니다.

```txt
T 타입의 모든 속성을 선택 사항으로 바꿉니다.
```

예를 들어 `User` 타입이 있다고 해볼게요.

```ts
interface User {
  id: number;
  nickname: string;
  email: string;
}
```

이 타입은 세 속성이 모두 필요합니다.

```ts
const user: User = {
  id: 1,
  nickname: "taewok",
  email: "taewok@example.com",
};
```

하나라도 빠지면 에러가 납니다.

```ts
const user: User = {
  id: 1,
  nickname: "taewok",
}; // email이 없어서 타입 에러
```

그런데 사용자 정보를 수정하는 API에서는 모든 값을 매번 보낼 필요가 없을 수 있습니다.

닉네임만 수정할 수도 있고, 이메일만 수정할 수도 있습니다.

```ts
const updateUser = (values: Partial<User>) => {
  // 변경된 값만 서버로 보냅니다.
};
```

이제 이런 호출이 가능합니다.

```ts
updateUser({
  nickname: "new-name",
});

updateUser({
  email: "new@example.com",
});
```

`Partial<User>`는 내부적으로 이런 타입처럼 동작합니다.

```ts
type PartialUser = {
  id?: number;
  nickname?: string;
  email?: string;
};
```

즉, 모든 속성에 `?`가 붙는다고 생각하면 됩니다.

---

## 🛠️ Partial은 언제 사용할까요?

`Partial<T>`는 주로 **일부 값만 다룰 때** 사용합니다.

대표적인 상황은 다음과 같습니다.

- 수정 API 요청 값
- 초기값 일부만 지정하는 옵션 객체
- 테스트에서 필요한 속성만 채운 mock 데이터
- 서버에서 내려온 필드별 에러 객체

예를 들어 설정 값을 업데이트하는 함수를 만들어보겠습니다.

```ts
interface AppSettings {
  theme: "light" | "dark";
  fontSize: number;
  isNotificationEnabled: boolean;
}

const updateSettings = (nextSettings: Partial<AppSettings>) => {
  // 일부 설정만 업데이트합니다.
};
```

이제 필요한 값만 넘길 수 있습니다.

```ts
updateSettings({
  theme: "dark",
});

updateSettings({
  fontSize: 16,
  isNotificationEnabled: false,
});
```

전체 타입은 유지하면서도, 입력은 유연하게 받을 수 있는 것이 `Partial`의 장점입니다.

---

## ⚠️ Partial을 사용할 때 주의할 점

`Partial<T>`는 모든 속성을 optional로 만듭니다.

그래서 값을 사용할 때는 `undefined` 가능성을 고려해야 합니다.

```ts
const updateUser = (values: Partial<User>) => {
  console.log(values.nickname.toUpperCase());
};
```

이 코드는 안전하지 않습니다.

`nickname`이 없을 수도 있기 때문입니다.

```ts
const updateUser = (values: Partial<User>) => {
  if (values.nickname) {
    console.log(values.nickname.toUpperCase());
  }
};
```

`Partial`은 입력을 유연하게 만들어주지만, 그만큼 값을 사용할 때는 존재 여부를 확인해야 합니다.

---

## 🧭 Path<T>는 TypeScript 기본 타입이 아닙니다

`Path<T>`는 TypeScript 기본 유틸리티 타입이 아닙니다.

React Hook Form을 사용하다 보면 자주 보게 되는 타입입니다.

```ts
import type { Path } from "react-hook-form";
```

`Path<T>`는 쉽게 말해 **객체 안의 필드 경로를 문자열로 안전하게 표현하는 타입**입니다.

예를 들어 이런 폼 타입이 있다고 해볼게요.

```ts
interface SignupFormValues {
  email: string;
  password: string;
  profile: {
    nickname: string;
    age: number;
  };
}
```

이 폼에서 가능한 필드 경로는 다음과 같습니다.

```txt
email
password
profile.nickname
profile.age
```

`Path<SignupFormValues>`는 이런 문자열만 허용하도록 도와줍니다.

```ts
const fieldName: Path<SignupFormValues> = "profile.nickname";
```

잘못된 경로는 타입 에러가 납니다.

```ts
const fieldName: Path<SignupFormValues> = "profile.name";
// 타입 에러
```

`profile.name`은 실제 타입에 없는 경로이기 때문입니다.

---

## 💡 Path<T>가 필요한 이유

폼에서는 필드 이름을 문자열로 다루는 경우가 많습니다.

```tsx
register("email");
setError("password", {
  message: "비밀번호를 입력해주세요.",
});
```

문제는 문자열은 오타에 약하다는 점입니다.

```tsx
register("emali");
```

`email`을 `emali`로 잘못 적어도 일반 문자열이면 TypeScript가 잡기 어렵습니다.

하지만 필드 이름을 `Path<T>`로 제한하면 실제 폼 타입에 존재하는 경로만 사용할 수 있습니다.

```ts
const setFieldError = <T extends FieldValues>(
  name: Path<T>,
  message: string,
) => {
  // ...
};
```

이렇게 하면 `name`에는 아무 문자열이나 들어올 수 없습니다.

`T` 타입 안에 실제로 존재하는 필드 경로만 들어올 수 있습니다.

---

## 🔗 T extends FieldValues 읽는 법

React Hook Form 코드에서 이런 타입을 자주 볼 수 있습니다.

```ts
const useFormServerError = <T extends FieldValues>() => {
  // ...
};
```

여기서 `T extends FieldValues`는 이렇게 읽을 수 있습니다.

```txt
T는 아무 타입이나 될 수 있는 것이 아니라,
FieldValues 조건을 만족하는 타입이어야 합니다.
```

즉, `T`의 범위를 제한하는 문법입니다.

일반 제네릭은 너무 자유롭습니다.

```ts
function printValue<T>(value: T) {
  console.log(value);
}
```

문자열도 되고, 숫자도 되고, 객체도 됩니다.

하지만 폼 값으로 사용할 타입이라면 최소한 객체 형태여야 합니다.

그래서 React Hook Form에서는 `FieldValues`를 기준으로 제한합니다.

```ts
<T extends FieldValues>
```

이렇게 하면 `Path<T>`, `setError`, `setFocus` 같은 React Hook Form 타입과 함께 사용할 수 있습니다.

---

## 🛠️ 실제 예시: 서버 에러를 폼 에러로 연결하기

프로젝트에서 서버 에러를 React Hook Form 에러로 연결할 때 이런 구조를 사용할 수 있습니다.

```ts
import { FieldValues, Path, useFormContext } from "react-hook-form";

export const useFormServerError = <T extends FieldValues>() => {
  const { setError, setFocus } = useFormContext<T>();

  const setFieldErrors = (fields?: Partial<Record<string, string>>) => {
    if (!fields) return false;

    const entries = Object.entries(fields);

    entries.forEach(([key, message]) => {
      setError(key as Path<T>, {
        type: "server",
        message,
      });
    });

    const firstErrorKey = entries[0]?.[0];

    if (firstErrorKey) {
      setFocus(firstErrorKey as Path<T>);
    }

    return true;
  };

  return { setFieldErrors };
};
```

이 코드에는 여러 타입 개념이 함께 들어 있습니다.

```ts
<T extends FieldValues>
```

폼 값 타입을 제네릭으로 받되, React Hook Form의 폼 값 타입이라는 조건을 붙입니다.

```ts
useFormContext<T>();
```

현재 폼이 어떤 값 구조를 가지는지 React Hook Form에 알려줍니다.

```ts
key as Path<T>
```

서버에서 받은 key는 런타임 문자열이기 때문에 TypeScript가 실제 폼 경로인지 알 수 없습니다. 그래서 React Hook Form의 필드 경로로 사용하겠다고 알려주는 타입 단언이 필요합니다.

```ts
Partial<Record<string, string>>
```

서버 에러 객체가 일부 필드에 대해서만 내려올 수 있음을 표현합니다.

---

## 📦 Record<string, string>도 같이 이해하기

방금 코드에 나온 `Record<string, string>`도 함께 보면 좋습니다.

```ts
Record<string, string>
```

이 타입은 다음과 비슷합니다.

```ts
type StringMap = {
  [key: string]: string;
};
```

즉, key도 문자열이고 value도 문자열인 객체입니다.

```ts
const errors: Record<string, string> = {
  email: "이미 사용 중인 이메일입니다.",
  password: "비밀번호가 너무 짧습니다.",
};
```

여기에 `Partial`을 붙이면 일부 key만 있을 수 있다는 의미를 더합니다.

```ts
Partial<Record<string, string>>
```

서버 에러는 모든 필드에 대해 내려오는 것이 아니라, 문제가 있는 필드만 내려오는 경우가 많습니다.

그래서 이런 타입이 자연스럽게 어울립니다.

---

## 🧠 한 줄로 다시 정리하기

각 타입을 짧게 정리하면 다음과 같습니다.

| 타입 | 의미 |
| --- | --- |
| `T` | 나중에 정해질 타입 자리 |
| `T extends SomeType` | T는 SomeType 조건을 만족해야 함 |
| `Partial<T>` | T의 모든 속성을 optional로 바꿈 |
| `Record<K, V>` | key 타입 K, value 타입 V인 객체 |
| `Path<T>` | T 안에 실제로 존재하는 필드 경로 문자열 |

처음에는 외워야 할 문법처럼 보이지만, 실제로는 타입을 더 유연하고 안전하게 재사용하기 위한 도구입니다.

---

## 🚧 자주 헷갈리는 부분

### 1. Partial은 타입을 느슨하게 만듭니다

`Partial<T>`를 쓰면 입력은 편해지지만, 값을 사용할 때는 `undefined` 가능성이 생깁니다.

그래서 업데이트 요청이나 옵션 객체처럼 일부 값만 받는 곳에 쓰는 것이 좋습니다.

반대로 반드시 모든 값이 있어야 하는 데이터에는 `Partial`을 남용하지 않는 편이 좋습니다.

### 2. T는 실제 타입이 아닙니다

`T`는 타입을 담는 변수에 가깝습니다.

```ts
function wrap<T>(value: T) {
  return { value };
}
```

이 함수에서 `T`는 호출할 때 결정됩니다.

```ts
wrap("hello"); // T는 string
wrap(123); // T는 number
```

### 3. Path<T>는 문자열을 더 안전하게 만들기 위한 타입입니다

폼 필드 이름은 결국 문자열입니다.

하지만 `Path<T>`를 사용하면 아무 문자열이나 넣는 대신, 실제 폼 타입에 존재하는 경로만 사용할 수 있습니다.

이 차이가 작은 것 같지만, 폼 필드가 많아질수록 오타를 줄이는 데 큰 도움이 됩니다.

---

## ✅ 정리

`Partial`, `Path`, `T`는 처음 보면 복잡해 보이지만 각각의 역할은 분명합니다.

`T`는 타입을 나중에 받기 위한 자리입니다.

`Partial<T>`는 어떤 타입의 일부 값만 받고 싶을 때 사용합니다.

`Path<T>`는 React Hook Form에서 폼 필드 경로를 안전하게 표현하기 위해 사용합니다.

이 셋을 함께 이해하면 이런 코드를 읽을 수 있게 됩니다.

```ts
export const useFormServerError = <T extends FieldValues>() => {
  const { setError } = useFormContext<T>();

  const setFieldError = (name: Path<T>, message: string) => {
    setError(name, {
      type: "server",
      message,
    });
  };

  return { setFieldError };
};
```

처음에는 낯설 수 있지만, 읽는 순서를 이렇게 잡으면 훨씬 편합니다.

```txt
T = 폼 값 타입
T extends FieldValues = FieldValues 조건을 만족하는 폼 값 타입
Path<T> = 그 폼 값 안에 존재하는 필드 경로
setError = 해당 필드에 에러를 넣는 함수
```

타입은 코드를 어렵게 만들기 위한 장치가 아니라, 실수를 줄이고 재사용 가능한 구조를 만들기 위한 도구입니다.

`Partial`, `Path`, `T`도 결국 이 목적을 위해 존재한다고 생각하면 훨씬 편하게 다룰 수 있습니다.
