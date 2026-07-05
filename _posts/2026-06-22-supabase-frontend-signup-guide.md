---
title: "[Supabase] 프론트엔드 개발자를 위한 Supabase 회원가입 구현하기"
date: 2026-06-22T10:00:00Z
categories: [frontend]
tags: [supabase, auth, react, frontend, signup]
description: "프론트엔드 개발자 관점에서 Supabase가 어떤 역할을 하는지 이해하고, 이메일과 비밀번호 기반 회원가입 기능까지 구현하는 과정을 정리했습니다."
custom_style: true
---

## 들어가며: 백엔드 없이 회원가입을 만들 수 있을까?

프론트엔드 프로젝트를 만들다 보면 로그인과 회원가입은 거의 항상 필요해요.

처음에는 화면만 만들면 될 것 같지만, 실제로는 생각보다 많은 백엔드 기능이 필요합니다.

- 사용자 정보를 저장할 데이터베이스
- 이메일과 비밀번호를 검증하는 인증 시스템
- 로그인 세션 관리
- 이메일 인증
- 사용자별 데이터 접근 권한
- 비밀번호 재설정

직접 만들 수도 있지만, 작은 프로젝트나 빠르게 검증해야 하는 서비스에서는 이 모든 걸 처음부터 구현하는 게 꽤 부담스럽습니다.

이럴 때 Supabase를 사용하면 프론트엔드에서도 백엔드 기능을 비교적 빠르게 붙일 수 있어요.

이번 글에서는 Supabase를 처음 접하는 프론트엔드 개발자 입장에서, **이메일과 비밀번호 기반 회원가입 기능**을 목표로 하나씩 정리해보겠습니다.

---

## Supabase는 무엇일까?

Supabase는 백엔드 기능을 서비스 형태로 제공하는 플랫폼이에요.

프론트엔드 개발자가 자주 필요로 하는 기능을 이미 준비해두고, SDK를 통해 쉽게 사용할 수 있게 해줍니다.

대표적으로 이런 기능이 있어요.

- PostgreSQL 데이터베이스
- 이메일, 비밀번호, 소셜 로그인 같은 인증 기능
- 파일 업로드를 위한 Storage
- 실시간 데이터 구독
- Edge Functions
- Row Level Security 기반 권한 제어

프론트엔드 입장에서 가장 크게 체감되는 점은, 직접 서버 API를 만들지 않아도 SDK로 인증과 데이터 요청을 시작할 수 있다는 점이에요.

예를 들어 회원가입은 아래처럼 호출할 수 있습니다.

```ts
const { data, error } = await supabase.auth.signUp({
  email,
  password,
});
```

이 한 번의 호출로 Supabase Auth에 사용자를 만들고, 설정에 따라 이메일 인증 메일도 보낼 수 있어요.

---

## 프론트엔드 개발자가 Supabase를 이해할 때 중요한 것

Supabase를 처음 보면 "그럼 백엔드가 아예 필요 없는 건가?"라는 생각이 들 수 있어요.

간단한 기능은 Supabase만으로도 충분히 만들 수 있습니다. 하지만 더 정확히 말하면 Supabase는 백엔드가 사라지는 도구라기보다, **반복적인 백엔드 기능을 미리 제공해주는 도구**에 가까워요.

프론트엔드는 Supabase SDK를 통해 다음 흐름을 다룹니다.

```txt
React 화면
↓
Supabase Client
↓
Supabase Auth / Database / Storage
```

회원가입을 예로 들면:

```txt
사용자가 이메일과 비밀번호 입력
↓
supabase.auth.signUp 호출
↓
Supabase Auth에 사용자 생성
↓
이메일 인증 또는 세션 반환
↓
화면에서 성공/실패 처리
```

즉, 프론트엔드는 여전히 폼 상태, 로딩, 에러 메시지, 성공 후 화면 전환 같은 UI 흐름을 잘 관리해야 합니다.

---

## 프로젝트 준비하기

먼저 Supabase 프로젝트를 만들고, 프로젝트 URL과 anon key를 확인해야 해요.

Supabase 대시보드에서 보통 다음 값을 확인할 수 있습니다.

- Project URL
- anon public key

프론트엔드에서는 이 두 값을 사용해 Supabase Client를 만듭니다.

설치는 다음처럼 합니다.

```bash
npm install @supabase/supabase-js
```

Next.js 프로젝트에서는 브라우저에서 접근해야 하는 환경변수에 `NEXT_PUBLIC_` prefix를 붙입니다.

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

여기서 중요한 점이 있어요.

`anon key`는 브라우저에 노출되는 걸 전제로 사용하는 공개 키입니다. 대신 실제 데이터 접근 권한은 Supabase의 RLS 정책으로 제어해야 해요.

반대로 `service_role key`는 절대 프론트엔드에 넣으면 안 됩니다.

---

## Supabase Client 만들기

이제 Supabase Client를 따로 분리해둡니다.

Next.js 기준이라면 이렇게 만들 수 있어요.

```ts
// src/lib/supabase.ts
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

이제 앱 어디에서든 `supabase`를 import해서 Auth나 Database 기능을 사용할 수 있어요.

---

## 회원가입 API 호출하기

Supabase에서 이메일과 비밀번호 기반 회원가입은 `auth.signUp`으로 처리합니다.

가장 기본적인 코드는 이렇게 생겼어요.

```ts
const { data, error } = await supabase.auth.signUp({
  email: "user@example.com",
  password: "password1234",
});

if (error) {
  console.error(error.message);
}

console.log(data.user);
```

`data`에는 생성된 사용자 정보나 세션 정보가 들어오고, 실패하면 `error`가 내려옵니다.

이 방식이 프론트엔드에서 좋은 이유는 응답 구조가 명확하다는 점이에요.

```txt
성공 → data 확인
실패 → error.message로 UI 표시
```

그래서 try/catch만으로 모든 흐름을 감싸기보다, Supabase가 반환하는 `error`를 먼저 확인하는 패턴을 자주 사용합니다.

---

## React 회원가입 폼 만들기

이제 실제 React 컴포넌트로 연결해볼게요.

```tsx
import { FormEvent, useState } from "react";
import { supabase } from "@/lib/supabase";

export default function SignupForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [message, setMessage] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    setMessage("");
    setIsSubmitting(true);

    const { data, error } = await supabase.auth.signUp({
      email,
      password,
    });

    setIsSubmitting(false);

    if (error) {
      setMessage(error.message);
      return;
    }

    if (data.user) {
      setMessage("회원가입이 완료되었습니다. 이메일 인증을 확인해주세요.");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        이메일
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
      </label>

      <label>
        비밀번호
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
          minLength={8}
        />
      </label>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "가입 중..." : "회원가입"}
      </button>

      {message && <p>{message}</p>}
    </form>
  );
}
```

이 예시는 일부러 단순하게 작성했어요.

실제 프로젝트에서는 React Hook Form이나 Zod를 함께 사용해서 입력값 검증을 더 깔끔하게 분리할 수 있습니다.

---

## 닉네임 같은 추가 정보도 같이 보낼 수 있을까?

회원가입 화면에는 이메일과 비밀번호만 있는 경우보다 닉네임, 이름, 프로필 이미지 같은 값이 같이 있는 경우가 많아요.

Supabase Auth의 `signUp`은 `options.data`로 사용자 metadata를 함께 보낼 수 있습니다.

```ts
const { data, error } = await supabase.auth.signUp({
  email,
  password,
  options: {
    data: {
      nickname,
    },
  },
});
```

이 값은 Auth user의 metadata로 저장됩니다.

다만 서비스에서 자주 조회하거나 검색해야 하는 값이라면 별도 `profiles` 테이블을 두는 편이 더 좋아요.

예를 들면 이런 구조입니다.

```txt
auth.users
└─ 인증 시스템이 관리하는 사용자

profiles
└─ 서비스에서 사용하는 사용자 프로필
```

Auth는 로그인과 세션을 맡기고, 서비스 프로필은 `profiles` 테이블에서 관리하는 식이에요.

---

## 이메일 인증 흐름 이해하기

Supabase 프로젝트 설정에 따라 회원가입 직후 동작이 달라질 수 있어요.

이메일 인증이 켜져 있다면 보통 이런 흐름이 됩니다.

```txt
회원가입 요청
↓
Supabase가 사용자 생성
↓
인증 메일 발송
↓
사용자가 이메일 링크 클릭
↓
로그인 가능한 상태가 됨
```

이때 프론트엔드에서는 바로 "로그인 완료"로 처리하기보다, "이메일을 확인해주세요" 같은 안내를 보여주는 게 자연스럽습니다.

반대로 이메일 인증을 끄면 회원가입 직후 바로 세션이 생길 수 있어요.

그래서 `signUp` 이후에는 항상 `data.session`이 있는지 확인하고, 프로젝트 설정에 맞춰 후속 처리를 나누는 게 좋습니다.

```ts
const { data, error } = await supabase.auth.signUp({
  email,
  password,
});

if (error) {
  setMessage(error.message);
  return;
}

if (data.session) {
  setMessage("회원가입과 로그인이 완료되었습니다.");
  return;
}

setMessage("회원가입이 완료되었습니다. 이메일 인증을 확인해주세요.");
```

---

## 회원가입 후 profiles 테이블에 저장하기

조금 더 실제 서비스에 가깝게 만들고 싶다면, 회원가입 후 `profiles` 테이블에 닉네임을 저장할 수 있어요.

```ts
const { data, error } = await supabase.auth.signUp({
  email,
  password,
});

if (error) {
  setMessage(error.message);
  return;
}

if (!data.user) {
  setMessage("사용자 정보를 확인할 수 없습니다.");
  return;
}

const { error: profileError } = await supabase.from("profiles").insert({
  id: data.user.id,
  nickname,
});

if (profileError) {
  setMessage(profileError.message);
  return;
}

setMessage("회원가입이 완료되었습니다.");
```

여기서 `profiles.id`는 보통 `auth.users.id`와 연결합니다.

다만 이 방식은 프로젝트 정책에 따라 달라질 수 있어요. 어떤 팀은 프론트엔드에서 insert를 호출하고, 어떤 팀은 데이터베이스 trigger로 `auth.users` 생성 시 자동으로 profile을 만들기도 합니다.

중요한 건 사용자 인증 정보와 서비스 프로필 정보를 구분해서 생각하는 거예요.

---

## RLS를 꼭 이해해야 하는 이유

Supabase를 프론트엔드에서 직접 호출할 때 가장 중요한 개념은 RLS입니다.

RLS는 Row Level Security의 줄임말이에요.

쉽게 말하면 "이 사용자가 이 row를 읽거나 쓸 수 있는가?"를 데이터베이스 정책으로 제어하는 기능입니다.

프론트엔드에서 anon key를 사용해도 괜찮은 이유는, 최종 권한 판단을 RLS가 맡기 때문이에요.

예를 들어 profiles 테이블에서 사용자가 자기 프로필만 수정할 수 있게 하려면 이런 식의 정책이 필요합니다.

```sql
create policy "Users can update own profile"
on profiles
for update
using (auth.uid() = id);
```

이 정책이 있으면 로그인한 사용자의 id와 profile row의 id가 같을 때만 수정할 수 있어요.

Supabase를 사용할 때는 SDK 코드만 보는 것보다, 테이블의 RLS 정책까지 함께 확인하는 습관이 중요합니다.

---

## 에러 메시지를 그대로 보여줘도 될까?

처음에는 `error.message`를 바로 화면에 보여줘도 개발 중에는 충분히 편합니다.

하지만 실제 서비스에서는 메시지를 조금 다듬는 게 좋아요.

```ts
if (error) {
  if (error.message.includes("Password")) {
    setMessage("비밀번호 조건을 확인해주세요.");
    return;
  }

  setMessage("회원가입에 실패했습니다. 잠시 후 다시 시도해주세요.");
}
```

특히 이메일이 이미 가입되어 있는지 여부를 너무 자세히 노출하면 보안상 좋지 않을 수 있어요.

Supabase도 사용자 존재 여부를 숨기기 위해 의도적으로 모호한 에러를 반환할 수 있습니다. 그래서 UI에서는 사용자가 다음 행동을 할 수 있을 정도로만 안내하는 편이 안전합니다.

---

## React Hook Form과 같이 쓰면 더 깔끔하다

폼이 단순하면 `useState`만으로도 충분해요.

하지만 회원가입 폼이 커지면 React Hook Form과 같이 쓰는 편이 더 편합니다.

예를 들어:

- 이메일 형식 검증
- 비밀번호 최소 길이 검증
- 비밀번호 확인 필드
- 약관 동의 여부
- 서버 에러를 필드 에러로 표시

이런 흐름을 폼 라이브러리와 함께 관리하면 UI 코드가 훨씬 정돈됩니다.

```tsx
const onSubmit = async (values: SignupFormValues) => {
  const { error } = await supabase.auth.signUp({
    email: values.email,
    password: values.password,
    options: {
      data: {
        nickname: values.nickname,
      },
    },
  });

  if (error) {
    setError("email", {
      type: "server",
      message: "회원가입에 실패했습니다.",
    });
  }
};
```

Supabase는 인증 요청을 맡고, React Hook Form은 입력 상태와 검증 UI를 맡는 식으로 역할을 나누면 좋아요.

---

## 프론트엔드에서 조심해야 할 보안 포인트

Supabase를 프론트엔드에서 사용할 때는 아래만큼은 꼭 기억해야 합니다.

- `anon key`는 브라우저에서 사용해도 되는 공개 키다
- `service_role key`는 절대 브라우저에 노출하면 안 된다
- 테이블에는 필요한 RLS 정책을 설정해야 한다
- 사용자 입력값은 프론트엔드와 데이터베이스 양쪽에서 검증하는 편이 좋다
- 에러 메시지는 사용자에게 필요한 만큼만 보여준다
- 민감한 로직은 Edge Function이나 서버 환경에서 처리한다

특히 `service_role key`는 RLS를 우회할 수 있는 강한 권한을 갖기 때문에, 프론트엔드 코드나 `NEXT_PUBLIC_` 환경변수에 넣으면 안 됩니다.

---

## 정리

Supabase를 처음 배울 때는 모든 기능을 한 번에 보려고 하면 조금 벅찰 수 있어요.

프론트엔드 개발자라면 먼저 회원가입 같은 작은 목표부터 잡는 게 좋습니다.

이번 흐름을 다시 정리하면 이렇습니다.

1. Supabase 프로젝트를 만든다
2. Project URL과 anon key를 환경변수에 저장한다
3. `createClient`로 Supabase Client를 만든다
4. 회원가입 폼에서 `supabase.auth.signUp`을 호출한다
5. `data`와 `error`를 기준으로 성공과 실패 UI를 나눈다
6. 이메일 인증 설정에 따라 안내 문구나 화면 전환을 처리한다
7. 서비스 프로필 정보는 `profiles` 테이블로 분리해서 관리한다
8. RLS 정책으로 데이터 접근 권한을 제어한다

Supabase의 장점은 인증, 데이터베이스, 스토리지를 하나의 흐름으로 빠르게 붙일 수 있다는 점이에요.

하지만 프론트엔드에서 직접 백엔드 리소스에 접근하는 만큼, RLS와 공개 가능한 키의 범위를 이해하는 게 정말 중요합니다.

회원가입 기능을 하나 완성해보면 Supabase가 어떤 역할을 해주는지 감이 훨씬 잘 잡혀요. 그다음 로그인, 로그아웃, 프로필 수정, 파일 업로드로 확장하면 Supabase를 꽤 자연스럽게 다룰 수 있게 됩니다.

## 참고

- [Supabase JavaScript Client 초기화 문서](https://supabase.com/docs/reference/javascript/initializing)
- [Supabase JavaScript signUp 문서](https://supabase.com/docs/reference/javascript/auth-signup)
- [Supabase Auth 개요 문서](https://supabase.com/docs/guides/auth)
