---
title: "[React] Zod와 React Hook Form으로 폼 검증 구조화하기"
date: 2026-05-29T00:30:00Z
categories: [frontend]
tags: [react, react-hook-form, zod, validation, typescript]
description: "React Hook Form으로 관리하던 복잡한 폼에 Zod 스키마 검증을 연결해 검증 규칙을 분리하고, 타입 안정성과 에러 처리 흐름을 개선한 경험을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 폼 상태 관리 다음에는 검증 구조가 복잡해져요

이전 글에서는 `React Hook Form`을 사용해 회원가입 폼과 캐릭터 생성 폼의 상태를 관리한 경험을 정리했습니다.

React Hook Form은 입력값, 에러, 동적 배열, `isDirty` 같은 폼 상태를 하나의 흐름으로 다룰 수 있게 해줍니다. 하지만 폼이 커질수록 또 다른 문제가 생겨요. 바로 **검증 규칙이 컴포넌트 안에 흩어지기 쉽다**는 점입니다.

처음에는 `register` 안에 `required`, `maxLength`, `pattern` 같은 규칙을 직접 작성해도 크게 불편하지 않습니다.

```tsx
<SmartInput
  {...register("nickname", {
    required: "닉네임을 입력해주세요.",
    maxLength: {
      value: 20,
      message: "최대 20자까지 가능합니다.",
    },
    pattern: {
      value: NICKNAME_REGEX,
      message: "특수문자는 사용할 수 없습니다.",
    },
  })}
/>
```

하지만 필드가 많아지고, 비밀번호 확인처럼 다른 필드와 비교해야 하는 검증이 생기고, 배열 데이터까지 들어오면 검증 로직이 점점 UI 컴포넌트와 섞이기 시작합니다.

이 문제를 해결하기 위해 프로젝트에서는 `zod`를 함께 사용했습니다. 핵심은 **폼 상태 관리는 React Hook Form이 담당하고, 값이 올바른지 판단하는 규칙은 Zod 스키마에 맡기는 구조**입니다.

---

## 💡 왜 Zod를 함께 사용했을까요?

React Hook Form만으로도 기본적인 검증은 충분히 가능합니다. 그럼에도 Zod를 함께 사용한 이유는 검증 규칙을 더 명확한 단위로 분리하고 싶었기 때문이에요.

Zod를 사용하면 폼 값의 모양과 검증 조건을 하나의 스키마로 표현할 수 있습니다.

```ts
const signupSchema = z
  .object({
    nickname: z
      .string()
      .min(1, "닉네임을 입력해주세요.")
      .max(20, "최대 20자까지 가능합니다.")
      .regex(NICKNAME_REGEX, "특수문자는 사용할 수 없습니다."),
    email: z
      .string()
      .min(1, "이메일을 입력해주세요.")
      .email("올바른 이메일 형식이 아닙니다."),
    password: z.string().min(8, "비밀번호는 8자 이상 입력해주세요."),
    passwordCheck: z.string().min(1, "비밀번호 확인을 입력해주세요."),
    signupToken: z.string(),
    isPrivacyAgreed: z.boolean(),
    isTermsAgreed: z.boolean(),
  })
  .refine((data) => data.password === data.passwordCheck, {
    path: ["passwordCheck"],
    message: "비밀번호가 일치하지 않습니다.",
  });
```

이렇게 작성하면 어떤 필드가 어떤 규칙을 갖는지 컴포넌트 바깥에서 한눈에 확인할 수 있습니다. 특히 `password`와 `passwordCheck`처럼 두 값을 비교해야 하는 검증은 `register` 옵션에 넣기보다 스키마에서 처리하는 편이 훨씬 읽기 쉬웠습니다.

---

## 📘 Zod 기본 사용법 익히기

React Hook Form과 연결하기 전에 Zod 자체의 기본 사용법을 먼저 정리해볼게요.

Zod는 TypeScript 환경에서 사용할 수 있는 스키마 검증 라이브러리입니다. 쉽게 말하면 **어떤 데이터가 어떤 형태여야 하는지 코드로 정의하고, 실제 값이 그 규칙을 만족하는지 검사하는 도구**입니다.

### 1. 설치하기

Zod와 React Hook Form을 함께 사용할 때는 `zod`와 `@hookform/resolvers`가 필요합니다.

```bash
npm install zod @hookform/resolvers
```

`zod`는 스키마를 정의하고 검증하는 역할을 하고, `@hookform/resolvers`는 Zod 스키마를 React Hook Form과 연결해주는 역할을 합니다.

### 2. 기본 스키마 만들기

Zod는 `z.string()`, `z.number()`, `z.boolean()`처럼 타입을 기준으로 스키마를 만듭니다.

```ts
import { z } from "zod";

const userSchema = z.object({
  name: z.string(),
  age: z.number(),
  isAdmin: z.boolean(),
});
```

위 스키마는 다음과 같은 데이터를 기대합니다.

```ts
const user = {
  name: "taewok",
  age: 20,
  isAdmin: false,
};
```

`name`은 문자열이어야 하고, `age`는 숫자여야 하며, `isAdmin`은 boolean이어야 합니다. 만약 `age`에 문자열 `"20"`이 들어오면 검증에 실패합니다.

### 3. 문자열 검증 추가하기

폼에서는 단순히 문자열인지 확인하는 것보다, 비어 있는지, 길이가 적절한지, 특정 형식인지 확인하는 경우가 많습니다.

```ts
const signupSchema = z.object({
  nickname: z
    .string()
    .min(1, "닉네임을 입력해주세요.")
    .max(20, "닉네임은 최대 20자까지 가능합니다."),
  email: z
    .string()
    .min(1, "이메일을 입력해주세요.")
    .email("올바른 이메일 형식이 아닙니다."),
  password: z.string().min(8, "비밀번호는 8자 이상 입력해주세요."),
});
```

자주 사용하는 메서드는 다음과 같습니다.

- `min`: 최소 길이 또는 최소값을 검사합니다.
- `max`: 최대 길이 또는 최대값을 검사합니다.
- `email`: 이메일 형식인지 검사합니다.
- `regex`: 정규식 조건을 만족하는지 검사합니다.
- `url`: URL 형식인지 검사합니다.

각 메서드의 두 번째 인자나 `message` 옵션으로 에러 메시지를 지정할 수 있습니다. 이 메시지는 React Hook Form과 연결했을 때 `errors.email?.message` 같은 형태로 화면에 표시할 수 있어요.

### 4. parse로 검증하기

스키마를 만들었다면 `parse`로 실제 데이터를 검증할 수 있습니다.

```ts
const result = signupSchema.parse({
  nickname: "taewok",
  email: "taewok@example.com",
  password: "12345678",
});
```

검증에 성공하면 `parse`는 검증된 데이터를 그대로 반환합니다. 반대로 실패하면 에러를 throw합니다.

```ts
signupSchema.parse({
  nickname: "",
  email: "wrong-email",
  password: "123",
});
```

위 코드는 닉네임, 이메일 형식, 비밀번호 길이 조건을 만족하지 못하므로 `ZodError`가 발생합니다. 그래서 일반적인 폼 처리에서는 `parse`를 직접 쓰기보다 다음에 나오는 `safeParse`나 React Hook Form의 `zodResolver`를 더 자주 사용합니다.

### 5. safeParse로 안전하게 검증하기

`safeParse`는 검증에 실패해도 에러를 throw하지 않고, 성공 여부를 객체로 반환합니다.

```ts
const result = signupSchema.safeParse({
  nickname: "",
  email: "wrong-email",
  password: "123",
});

if (!result.success) {
  console.log(result.error.issues);
} else {
  console.log(result.data);
}
```

`safeParse`의 결과는 `success` 값으로 구분할 수 있습니다.

- `success: true`: 검증 성공, `data`에 검증된 값이 들어 있습니다.
- `success: false`: 검증 실패, `error.issues`에 에러 목록이 들어 있습니다.

폼 라이브러리 없이 직접 입력값을 검증해야 한다면 `safeParse`가 다루기 편합니다. React Hook Form과 함께 사용할 때는 이 과정을 `zodResolver`가 대신 처리해줍니다.

### 6. TypeScript 타입 추론하기

Zod의 큰 장점 중 하나는 스키마에서 TypeScript 타입을 추론할 수 있다는 점입니다.

```ts
type SignupFormValues = z.infer<typeof signupSchema>;
```

이렇게 하면 `signupSchema`의 구조를 기반으로 `SignupFormValues` 타입이 자동으로 만들어집니다.

```ts
type SignupFormValues = {
  nickname: string;
  email: string;
  password: string;
};
```

스키마와 타입을 따로 작성하면 둘 중 하나를 수정했을 때 다른 하나를 깜빡하기 쉽습니다. `z.infer`를 사용하면 스키마가 곧 타입의 기준이 되기 때문에 폼 데이터 타입을 더 안전하게 관리할 수 있습니다.

### 7. optional, nullable, array, object 사용하기

실제 폼에는 선택값, null 값, 배열, 중첩 객체가 자주 등장합니다.

```ts
const characterSchema = z.object({
  name: z.string().min(1, "캐릭터 이름을 입력해주세요."),
  description: z.string().optional(),
  profileImage: z.string().nullable(),
  tags: z.array(z.string()),
  asset: z
    .array(
      z.object({
        assetName: z.string().min(1, "에셋 이름을 입력해주세요."),
        assetImage: z.string().min(1, "에셋 이미지를 등록해주세요."),
      }),
    )
    .optional(),
});
```

각 메서드의 의미는 다음과 같습니다.

- `optional`: 값이 없어도 허용합니다. 즉 `undefined`를 허용합니다.
- `nullable`: `null` 값을 허용합니다.
- `array`: 배열을 정의합니다.
- `object`: 객체 구조를 정의합니다.

이런 문법을 사용하면 캐릭터 생성 폼처럼 중첩된 데이터도 스키마로 표현할 수 있습니다.

---

## 🔗 React Hook Form과 연결하기

Zod를 React Hook Form과 함께 사용할 때는 `@hookform/resolvers/zod`에서 제공하는 `zodResolver`를 사용합니다.

```tsx
import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";
import { z } from "zod";

type SignupFormValues = z.infer<typeof signupSchema>;

const methods = useForm<SignupFormValues>({
  resolver: zodResolver(signupSchema),
  mode: "onChange",
  reValidateMode: "onChange",
  defaultValues: {
    nickname: "",
    email: "",
    password: "",
    passwordCheck: "",
    signupToken: "",
    isPrivacyAgreed: false,
    isTermsAgreed: false,
  },
});
```

여기서 중요한 부분은 `resolver: zodResolver(signupSchema)`입니다.

React Hook Form은 폼 이벤트와 상태를 관리하고, 실제 검증은 Zod 스키마에 위임합니다. 사용자가 값을 입력하면 React Hook Form이 현재 값을 수집하고, `zodResolver`가 해당 값을 스키마 기준으로 검증한 뒤 에러를 React Hook Form의 `formState.errors`에 넣어줍니다.

덕분에 기존에 사용하던 에러 표시 방식은 그대로 유지할 수 있습니다.

```tsx
const {
  register,
  formState: { errors },
} = useFormContext<SignupFormValues>();

<SmartInput
  {...register("email")}
  label="이메일"
  error={errors.email?.message}
/>
```

UI 컴포넌트 입장에서는 검증이 `register` 옵션에서 온 것인지, Zod 스키마에서 온 것인지 알 필요가 없습니다. 여전히 `errors.email?.message`만 표시하면 됩니다.

---

## ✅ RHF와 Zod를 함께 사용할 때의 장점

React Hook Form과 Zod를 연결하면 두 도구의 역할이 자연스럽게 나뉩니다.

React Hook Form은 다음을 담당합니다.

- 입력값 등록과 추적
- `dirty`, `touched`, `valid`, `errors` 같은 폼 상태 관리
- `useFieldArray`를 통한 동적 배열 관리
- `FormProvider`, `useFormContext`를 통한 폼 메서드 공유
- `setValue`, `getValues`, `reset`, `setError` 같은 명령형 제어

Zod는 다음을 담당합니다.

- 폼 데이터의 구조 정의
- 필드별 유효성 검증
- 여러 필드를 함께 비교하는 검증
- 중첩 객체와 배열 데이터 검증
- TypeScript 타입 추론

즉, **React Hook Form은 폼의 흐름을 관리하고, Zod는 데이터의 규칙을 관리합니다.** 이 구분이 생기면 폼이 커져도 코드가 훨씬 덜 엉킵니다.

특히 장점이 크게 느껴진 부분은 타입 추론이었습니다.

```ts
type SignupFormValues = z.infer<typeof signupSchema>;
```

스키마에서 타입을 추론하면 검증 규칙과 TypeScript 타입을 따로 관리하지 않아도 됩니다. 필드를 추가하거나 이름을 바꿨을 때 스키마와 타입이 어긋나는 문제를 줄일 수 있습니다.

---

## 🧩 캐릭터 생성 폼처럼 복잡한 구조 검증하기

캐릭터 생성 폼은 단순한 문자열 필드만 있는 구조가 아니었습니다. 에셋 배열, 시나리오 배열, 태그 배열처럼 중첩된 데이터가 포함되어 있었어요.

이런 구조는 Zod의 `array`, `object`를 사용해 표현할 수 있습니다.

```ts
const scenarioSchema = z.object({
  name: z.string().min(1, "시나리오 이름을 입력해주세요."),
  contents: z.array(
    z.object({
      role: z.enum(["user", "assistant"]),
      message: z.string().min(1, "메시지를 입력해주세요."),
    }),
  ),
});

const assetSchema = z.object({
  assetFile: z.instanceof(File).nullable().optional(),
  assetName: z.string().min(1, "에셋 이름을 입력해주세요."),
  assetImage: z.string().min(1, "에셋 이미지를 등록해주세요."),
  assetSituation: z.string().optional(),
});

const characterCreateSchema = z.object({
  representativeImage: z.string().min(1, "대표 이미지를 등록해주세요."),
  title: z.string().min(1, "제목을 입력해주세요."),
  name: z.string().min(1, "캐릭터 이름을 입력해주세요."),
  characterIntroduce: z.string().min(1, "캐릭터 소개를 입력해주세요."),
  characterDetailSetting: z.string().min(1, "상세 설정을 입력해주세요."),
  asset: z.array(assetSchema).optional(),
  scenarios: z
    .array(scenarioSchema)
    .min(1, "시나리오는 최소 1개 이상 필요합니다."),
  tagIds: z.array(
    z.object({
      id: z.number(),
      label: z.string(),
    }),
  ),
  isPublic: z.boolean(),
  characterDescription: z.string(),
  tendency: z.string(),
  category: z.string(),
});
```

이렇게 스키마를 만들어두면 `useFieldArray`로 추가한 `asset`, `scenarios` 값도 제출 시점에 같은 기준으로 검증할 수 있습니다.

React Hook Form만 사용할 때는 배열 내부 필드의 검증 규칙을 각 컴포넌트마다 넣기 쉽습니다. 반면 Zod를 함께 사용하면 **배열 내부 항목이 어떤 구조여야 하는지 스키마 한 곳에서 관리**할 수 있습니다.

---

## 🧪 refine으로 필드 간 검증 처리하기

폼 검증에서 자주 까다로워지는 부분은 한 필드만 보고 판단할 수 없는 조건입니다.

회원가입 폼의 비밀번호 확인이 대표적입니다. `passwordCheck` 필드는 자기 값만으로는 올바른지 판단할 수 없고, `password` 값과 비교해야 합니다.

이럴 때는 `refine`을 사용했습니다.

```ts
const signupSchema = z
  .object({
    password: z.string().min(8, "비밀번호는 8자 이상 입력해주세요."),
    passwordCheck: z.string().min(1, "비밀번호 확인을 입력해주세요."),
  })
  .refine((data) => data.password === data.passwordCheck, {
    path: ["passwordCheck"],
    message: "비밀번호가 일치하지 않습니다.",
  });
```

`path`를 `["passwordCheck"]`로 지정하면 에러가 `passwordCheck` 필드에 연결됩니다. React Hook Form에서는 이 에러를 기존처럼 `errors.passwordCheck?.message`로 표시하면 됩니다.

```tsx
<SmartInput
  {...register("passwordCheck")}
  label="비밀번호 확인"
  error={errors.passwordCheck?.message}
/>
```

검증은 Zod가 처리하지만, 에러를 보여주는 흐름은 React Hook Form과 자연스럽게 이어집니다.

---

## 📌 register 옵션을 줄이고 컴포넌트를 가볍게 만들기

Zod를 사용하기 전에는 필드 컴포넌트 안에 검증 규칙이 함께 들어가는 경우가 많았습니다.

```tsx
<SmartInput
  {...register("email", {
    required: "이메일을 입력해주세요.",
    pattern: {
      value: EMAIL_REGEX,
      message: "올바른 이메일 형식이 아닙니다.",
    },
  })}
  error={errors.email?.message}
/>
```

Zod를 연결한 뒤에는 컴포넌트가 훨씬 단순해집니다.

```tsx
<SmartInput
  {...register("email")}
  label="이메일"
  error={errors.email?.message}
/>
```

검증 규칙이 컴포넌트에서 빠지면서 필드 컴포넌트는 UI와 입력 연결에만 집중할 수 있습니다. 규칙을 수정해야 할 때도 여러 컴포넌트를 찾아다니지 않고 스키마만 수정하면 됩니다.

이 점은 React Hook Form의 `FormProvider`, `useFormContext` 구조와도 잘 맞았습니다. 하위 필드 컴포넌트는 `register`와 `errors`만 사용하고, 폼 전체의 검증 정책은 상위 스키마에서 관리할 수 있기 때문입니다.

---

## 🚨 서버 에러와 Zod 에러를 함께 다루기

Zod는 클라이언트에서 확인할 수 있는 규칙을 검증합니다. 하지만 이미 사용 중인 이메일, 중복된 닉네임처럼 서버에서만 알 수 있는 에러도 있습니다.

이 경우에는 기존 React Hook Form 흐름처럼 `setError`를 사용했습니다.

```tsx
authRegister(data, {
  onError: (error) => {
    setFieldErrors(error.fields);
  },
});
```

Zod 에러와 서버 에러가 모두 React Hook Form의 `errors` 객체로 모이기 때문에 UI는 복잡해지지 않습니다.

```tsx
<SmartInput
  {...register("nickname")}
  label="닉네임"
  error={errors.nickname?.message}
/>
```

클라이언트 검증 실패든 서버 검증 실패든 필드 컴포넌트는 동일하게 에러 메시지를 보여주면 됩니다. 이 일관성이 React Hook Form과 Zod를 함께 사용할 때 꽤 큰 장점이었습니다.

---

## ⚠️ 적용하면서 주의했던 점

Zod를 붙인다고 모든 검증이 자동으로 좋아지는 것은 아니었습니다. 몇 가지 주의할 점도 있었습니다.

### 1. 기본값과 스키마 타입을 맞춰야 합니다

React Hook Form의 `defaultValues`와 Zod 스키마의 타입이 어긋나면 예상하지 못한 검증 에러가 발생할 수 있습니다.

예를 들어 스키마에서는 `tagIds`를 배열로 기대하는데 기본값이 `undefined`라면 초기 상태부터 다루기 불편해집니다.

```tsx
defaultValues: {
  tagIds: [],
  scenarios: [],
  isPublic: false,
}
```

배열은 빈 배열, 체크박스는 `false`, 문자열은 빈 문자열처럼 폼에서 사용하는 초기값을 스키마와 맞춰두는 것이 좋았습니다.

### 2. 파일 검증은 환경을 고려해야 합니다

파일 업로드를 검증할 때 `z.instanceof(File)`을 사용할 수 있지만, 서버 렌더링 환경에서는 `File` 객체가 없을 수 있습니다.

클라이언트 전용 폼인지, 서버에서도 스키마가 평가되는지에 따라 `z.custom<File>()`이나 `unknown` 기반 검증을 고려해야 합니다.

```ts
const fileSchema = z
  .custom<File>((value) => value instanceof File, "파일을 선택해주세요.")
  .optional();
```

Next.js 환경에서는 이런 브라우저 전용 객체 검증을 특히 조심해야 했습니다.

### 3. 모든 검증을 Zod에 넣을 필요는 없습니다

비밀번호 길이, 이메일 형식, 배열 최소 개수처럼 데이터 규칙에 가까운 것은 Zod에 두는 편이 좋았습니다.

반면 닉네임 중복 확인처럼 API 호출이 필요한 검증은 `useWatch`, `debounce`, 서버 에러 처리와 함께 다루는 편이 자연스러웠습니다. Zod는 동기적인 데이터 구조 검증에 집중시키고, 비동기 검증은 React Hook Form 흐름 안에서 별도로 처리했습니다.

---

## ✅ 정리

React Hook Form만으로도 폼 상태 관리는 충분히 강력합니다. 하지만 폼이 커질수록 검증 규칙까지 컴포넌트 안에서 함께 관리하면 코드가 쉽게 복잡해집니다.

Zod를 함께 사용하면서 가장 크게 느낀 장점은 역할 분리였습니다.

- React Hook Form은 폼 상태와 입력 흐름을 관리합니다.
- Zod는 폼 데이터의 구조와 검증 규칙을 관리합니다.
- `zodResolver`는 두 도구를 연결해 Zod 에러를 React Hook Form의 `errors` 흐름으로 넘겨줍니다.
- `z.infer`를 사용하면 스키마에서 TypeScript 타입을 추론할 수 있습니다.
- 중첩 객체와 배열 데이터도 스키마 한 곳에서 검증할 수 있습니다.
- 서버 에러는 `setError`로 같은 에러 표시 흐름에 통합할 수 있습니다.

결국 이 프로젝트에서 Zod를 사용한 이유는 단순히 검증 코드를 줄이기 위해서가 아니었습니다. **폼의 상태 관리와 검증 규칙을 분리하면서도, 사용자에게 보여주는 에러 흐름은 하나로 유지하기 위해서**였습니다.

React Hook Form이 폼을 움직이는 엔진이라면, Zod는 그 폼이 어떤 데이터만 통과시킬지 정하는 규칙표에 가까웠습니다. 두 도구를 함께 사용하면서 복잡한 폼도 훨씬 예측 가능한 구조로 관리할 수 있었습니다.
