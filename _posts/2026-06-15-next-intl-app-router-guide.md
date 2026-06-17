---
title: "[Next.js] next-intl로 다국어 처리 제대로 이해하고 적용하기"
date: 2026-06-15T13:00:00Z
categories: [frontend]
tags: [nextjs, next-intl, i18n, app-router, locale]
description: "Next.js App Router에서 next-intl을 사용해 다국어 처리 구조를 잡는 방법을 설치부터 routing, proxy, request config, Server/Client Component 사용법까지 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 다국어 처리는 번역 파일만 만든다고 끝나지 않아요

Next.js에서 다국어 처리를 하려고 하면 처음에는 이렇게 생각하기 쉽습니다.

```txt
언어별 JSON 파일만 만들면 되는 거 아닐까?
```

물론 번역 파일은 중요합니다. 하지만 실제 프로젝트에서는 그보다 더 많은 결정이 필요합니다.

- URL을 `/ko/about`, `/en/about`처럼 언어별로 나눌지
- 현재 locale을 어디서 결정할지
- 서버 컴포넌트와 클라이언트 컴포넌트에서 번역을 어떻게 사용할지
- 링크 이동 시 locale을 어떻게 유지할지
- 메시지를 클라이언트에 얼마나 내려보낼지

`next-intl`은 이런 고민을 App Router 구조에 맞춰 비교적 자연스럽게 풀 수 있게 도와주는 라이브러리입니다.

이번 글에서는 `next-intl`을 단순 설치 수준이 아니라, 실제로 구조를 이해하고 적용할 수 있도록 정리해보겠습니다.

이 글은 `Next.js App Router` 기준으로 설명합니다.

---

## 💡 next-intl이 하는 일

`next-intl`은 Next.js에서 국제화 기능을 다루기 위한 라이브러리입니다.

공식 문서 기준으로 `next-intl`은 다음 영역을 담당합니다.

- 번역 메시지 관리
- locale 기반 라우팅
- 날짜, 숫자 같은 locale 포맷 처리
- Server Components / Client Components 환경에 맞는 번역 API 제공

즉, 단순히 `t('hello')`를 찍는 도구라기보다, **Next.js에서 locale이 어떻게 흘러가야 하는지**를 함께 다루는 라이브러리라고 보는 편이 좋습니다.

---

## 📦 1. 설치하기

공식 문서의 시작점은 아주 단순합니다.

```bash
npm install next-intl
```

그다음 프로젝트에 `next-intl` 플러그인을 연결합니다.

```ts
import {NextConfig} from 'next';
import createNextIntlPlugin from 'next-intl/plugin';

const nextConfig: NextConfig = {};

const withNextIntl = createNextIntlPlugin();

export default withNextIntl(nextConfig);
```

이 설정은 `i18n/request.ts` 파일과 `next-intl`을 연결해주는 역할을 합니다.

---

## 🗂️ 2. 가장 기본적인 파일 구조

App Router에서 locale 기반 라우팅까지 함께 쓰는 구조는 보통 이렇게 잡습니다.

```txt
src
├─ app
│  └─ [locale]
│     ├─ layout.tsx
│     ├─ page.tsx
│     └─ ...
├─ i18n
│  ├─ navigation.ts
│  ├─ request.ts
│  └─ routing.ts
└─ proxy.ts

messages
├─ en.json
└─ ko.json
```

각 파일의 역할은 이렇게 보면 됩니다.

| 파일 | 역할 |
| --- | --- |
| `messages/*.json` | 언어별 번역 메시지 |
| `src/i18n/routing.ts` | 지원 locale, 기본 locale, 라우팅 규칙 |
| `src/proxy.ts` | 들어온 요청에서 locale을 판단하고 라우팅 연결 |
| `src/i18n/navigation.ts` | locale-aware `Link`, `useRouter` 같은 래퍼 |
| `src/i18n/request.ts` | 현재 요청에서 locale과 messages를 준비 |
| `app/[locale]/*` | locale 세그먼트를 실제 페이지 구조에 반영 |

이 구조를 먼저 머릿속에 잡아두면 이후 설정이 훨씬 덜 헷갈립니다.

---

## 🌍 3. 메시지 파일 만들기

가장 먼저 번역 메시지를 준비합니다.

`messages/en.json`

```json
{
  "HomePage": {
    "title": "Hello world!",
    "description": "This is an internationalized app."
  },
  "Navigation": {
    "home": "Home",
    "about": "About"
  }
}
```

`messages/ko.json`

```json
{
  "HomePage": {
    "title": "안녕하세요!",
    "description": "다국어 처리가 적용된 앱입니다."
  },
  "Navigation": {
    "home": "홈",
    "about": "소개"
  }
}
```

여기서 중요한 점은 메시지를 **중첩 구조로 관리하는 것**입니다.

```txt
HomePage.title
HomePage.description
Navigation.home
Navigation.about
```

이 구조를 사용하면 컴포넌트에서 namespace를 기준으로 번역을 꺼내기 편합니다.

---

## 🧭 4. routing.ts로 지원 언어 정하기

locale 기반 라우팅을 쓰려면 먼저 어떤 locale을 지원하는지 정해야 합니다.

`src/i18n/routing.ts`

```ts
import {defineRouting} from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['ko', 'en'],
  defaultLocale: 'ko'
});
```

이 파일은 프로젝트의 다국어 라우팅 기준점입니다.

```txt
지원 locale = ko, en
기본 locale = ko
```

이제 `next-intl`은 이 설정을 기준으로 경로와 navigation을 처리할 수 있습니다.

---

## 🚦 5. proxy.ts로 locale 라우팅 연결하기

다음은 `proxy.ts`입니다.

`next-intl` 공식 문서 기준으로 Next.js 16부터는 `middleware.ts` 대신 `proxy.ts`를 사용합니다.

`src/proxy.ts`

```ts
import createMiddleware from 'next-intl/middleware';
import {routing} from './i18n/routing';

export default createMiddleware(routing);

export const config = {
  matcher: '/((?!api|trpc|_next|_vercel|.*\\..*).*)'
};
```

이 파일은 들어오는 요청을 보고 locale 기반 라우팅을 적용하는 역할을 합니다.

예를 들어 사용자가 `/about`로 들어왔을 때, 설정에 따라 `/ko/about` 또는 `/en/about` 같은 경로로 연결되도록 도와줍니다.

`matcher`는 어떤 경로에 이 로직을 적용할지 정합니다.

```txt
적용 제외:
api
trpc
_next
_vercel
정적 파일 경로
```

이 matcher를 그대로 두는 경우가 많지만, 점이 포함된 사용자 경로 같은 특수한 케이스가 있다면 문서 예시처럼 별도 matcher를 추가할 수 있습니다.

---

## 🔗 6. navigation.ts로 locale-aware 링크 만들기

`next-intl`을 사용할 때 가장 편한 부분 중 하나가 locale을 고려한 navigation 래퍼를 만들 수 있다는 점입니다.

`src/i18n/navigation.ts`

```ts
import {createNavigation} from 'next-intl/navigation';
import {routing} from './routing';

export const {Link, redirect, usePathname, useRouter, getPathname} =
  createNavigation(routing);
```

이제 컴포넌트에서 `next/link` 대신 이 `Link`를 사용할 수 있습니다.

```tsx
import {Link} from '@/i18n/navigation';

export default function Navigation() {
  return (
    <nav>
      <Link href="/">홈</Link>
      <Link href="/about">소개</Link>
    </nav>
  );
}
```

현재 locale이 `ko`라면 `/about`은 내부적으로 `/ko/about`에 맞게 처리됩니다.

이 구조가 좋은 이유는 locale prefix를 우리가 매번 수동으로 붙이지 않아도 된다는 점입니다.

---

## 🧠 7. request.ts는 왜 필요한가요?

`request.ts`는 현재 요청 기준으로 어떤 locale과 메시지를 사용할지 정하는 파일입니다.

`src/i18n/request.ts`

```ts
import {getRequestConfig} from 'next-intl/server';
import {hasLocale} from 'next-intl';
import {routing} from './routing';

export default getRequestConfig(async ({requestLocale}) => {
  const requested = await requestLocale;

  const locale = hasLocale(routing.locales, requested)
    ? requested
    : routing.defaultLocale;

  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default
  };
});
```

이 파일을 읽을 때 중요한 흐름은 이렇습니다.

```txt
요청에서 locale 후보를 받는다
↓
지원하는 locale인지 검사한다
↓
아니면 defaultLocale로 fallback 한다
↓
해당 locale의 messages JSON을 불러온다
```

즉, 이 파일은 `next-intl`이 현재 요청에서 사용할 번역 데이터를 최종 확정하는 곳입니다.

---

## 🧱 8. [locale] 세그먼트로 App Router 구조 만들기

locale 기반 라우팅을 쓰려면 App Router 구조에도 locale 세그먼트를 반영해야 합니다.

```txt
src/app/[locale]/layout.tsx
src/app/[locale]/page.tsx
src/app/[locale]/about/page.tsx
```

`app/[locale]/layout.tsx`

```tsx
import {NextIntlClientProvider, hasLocale} from 'next-intl';
import {notFound} from 'next/navigation';
import {routing} from '@/i18n/routing';

type Props = {
  children: React.ReactNode;
  params: Promise<{locale: string}>;
};

export default async function LocaleLayout({children, params}: Props) {
  const {locale} = await params;

  if (!hasLocale(routing.locales, locale)) {
    notFound();
  }

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider>{children}</NextIntlClientProvider>
      </body>
    </html>
  );
}
```

여기서 중요한 점은 두 가지입니다.

1. `params.locale`이 실제 지원 locale인지 검사합니다.
2. `NextIntlClientProvider`로 클라이언트 컴포넌트에서도 번역 컨텍스트를 사용할 수 있게 만듭니다.

locale 검증 없이 그냥 렌더링하면 `/abc` 같은 잘못된 경로도 처리되어버릴 수 있으니 `notFound()` 처리가 중요합니다.

---

## ✍️ 9. 번역 사용하기: useTranslations

일반적인 컴포넌트에서는 `useTranslations`를 사용합니다.

```tsx
import {useTranslations} from 'next-intl';

export default function HomePage() {
  const t = useTranslations('HomePage');

  return (
    <main>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </main>
  );
}
```

`useTranslations('HomePage')`는 `HomePage` namespace를 기준으로 번역을 꺼냅니다.

```txt
t('title')
→ HomePage.title

t('description')
→ HomePage.description
```

이 방식의 장점은 컴포넌트가 필요한 번역 범위를 작게 유지할 수 있다는 점입니다.

---

## 🧵 10. async 서버 컴포넌트에서는 getTranslations

여기서 많은 분들이 한 번 헷갈립니다.

`async` 서버 컴포넌트에서는 훅을 직접 사용할 수 없기 때문에 `getTranslations`를 사용해야 합니다.

```tsx
import {getTranslations} from 'next-intl/server';

export default async function ProfilePage() {
  const user = await fetchUser();
  const t = await getTranslations('ProfilePage');

  return (
    <section>
      <h1>{t('title', {username: user.name})}</h1>
    </section>
  );
}
```

정리하면 이렇게 기억하면 편합니다.

```txt
일반 컴포넌트
→ useTranslations

async 서버 컴포넌트
→ getTranslations
```

공식 문서에서도 이 구분을 명확하게 안내하고 있습니다.

---

## 🖥️ 11. Server Component와 Client Component에서 어떻게 나눌까요?

`next-intl`의 큰 장점 중 하나는 번역을 서버 쪽에 더 많이 남겨둘 수 있다는 점입니다.

공식 문서도 이 방향을 권장합니다.

예를 들어 인터랙션이 필요한 컴포넌트가 있다고 해보겠습니다.

```tsx
import {useTranslations} from 'next-intl';
import Expandable from './Expandable';

export default function FAQEntry() {
  const t = useTranslations('FAQEntry');

  return (
    <Expandable title={t('title')}>
      {t('description')}
    </Expandable>
  );
}
```

여기서 중요한 아이디어는 이겁니다.

```txt
번역은 서버 컴포넌트에서 처리
↓
번역된 문자열만 클라이언트 컴포넌트로 전달
```

즉, 클라이언트에서 꼭 `useTranslations`를 바로 써야 하는 것은 아닙니다. 오히려 가능한 경우에는 서버에서 번역한 뒤, 버튼 텍스트나 라벨을 props로 내려주는 쪽이 더 깔끔하고 성능에도 유리합니다.

---

## ⚙️ 12. 클라이언트에 모든 메시지를 넘겨야 할까요?

`NextIntlClientProvider`는 기본적으로 클라이언트에서 번역을 사용할 수 있게 해줍니다.

하지만 모든 메시지를 무조건 클라이언트에 넘겨야 하는 것은 아닙니다.

공식 문서에서는 이런 방식도 보여줍니다.

```tsx
<NextIntlClientProvider messages={null}>
  {children}
</NextIntlClientProvider>
```

이렇게 하면 클라이언트에 메시지를 넘기지 않습니다.

대신 필요한 영역에서만 별도의 `NextIntlClientProvider`를 두거나, 서버에서 번역한 문자열을 props로 내려주는 방식으로 최적화할 수 있습니다.

처음에는 전체 메시지를 내려도 충분한 경우가 많지만, 앱이 커지고 번역 파일이 커지면 이 전략이 꽤 중요해집니다.

---

## 🛣️ 13. locale switcher는 어떻게 만들까요?

locale 전환은 보통 `Link`나 `useRouter`를 통해 구현합니다.

가장 단순한 예시는 `Link`의 `locale` prop을 사용하는 방식입니다.

```tsx
import {Link} from '@/i18n/navigation';

export default function LocaleSwitcher() {
  return (
    <div>
      <Link href="/" locale="ko">한국어</Link>
      <Link href="/" locale="en">English</Link>
    </div>
  );
}
```

공식 문서에서 설명하는 중요한 포인트는 이것입니다.

`locale` prop을 준 `Link`는 locale prefix를 명시적으로 포함하는 링크를 만들 수 있습니다. 이는 locale cookie 업데이트와 hydration 이전 동작까지 고려한 설계입니다.

실무에서는 현재 경로를 유지한 채 locale만 바꾸고 싶을 때가 많으니, `usePathname`, `useRouter`, `getPathname`을 함께 사용하는 패턴도 자주 쓰입니다.

---

## 🧾 14. metadata도 locale을 고려해야 합니다

페이지 제목이나 description 같은 metadata도 다국어 처리 대상입니다.

공식 문서에서는 `params.locale`을 `getTranslations`에 직접 넘기는 예시를 제공합니다.

```tsx
import {getTranslations} from 'next-intl/server';

export async function generateMetadata({params}: {params: Promise<{locale: string}>}) {
  const {locale} = await params;
  const t = await getTranslations({locale, namespace: 'Metadata'});

  return {
    title: t('title')
  };
}
```

이 부분을 놓치면 화면 본문은 번역되는데 `<title>`은 한 언어로 고정되는 일이 생길 수 있습니다.

다국어 SEO까지 생각한다면 metadata 번역은 꼭 챙기는 편이 좋습니다.

---

## 🧊 15. static rendering을 하려면 setRequestLocale을 알아야 합니다

locale 기반 라우팅을 사용할 때 `next-intl`은 서버 컴포넌트에서 locale 정보를 읽기 위해 내부적으로 request header를 활용할 수 있습니다.

그런데 이 방식은 route를 동적으로 만들 수 있습니다.

정적 렌더링을 유지하고 싶다면 공식 문서에서 안내하는 `setRequestLocale`을 함께 알아두는 것이 좋습니다.

예시는 이런 식입니다.

```tsx
import {setRequestLocale} from 'next-intl/server';
import {hasLocale} from 'next-intl';
import {notFound} from 'next/navigation';
import {routing} from '@/i18n/routing';

export default async function LocaleLayout({
  children,
  params
}: {
  children: React.ReactNode;
  params: Promise<{locale: string}>;
}) {
  const {locale} = await params;

  if (!hasLocale(routing.locales, locale)) {
    notFound();
  }

  setRequestLocale(locale);

  return <>{children}</>;
}
```

공식 문서 기준으로 기억할 점은 세 가지입니다.

1. `locale`은 먼저 검증해야 합니다.
2. 정적 렌더링을 원하는 layout과 page마다 호출해야 할 수 있습니다.
3. `useTranslations`, `getMessages` 같은 `next-intl` API를 호출하기 전에 실행해야 합니다.

처음에는 이 부분이 조금 낯설 수 있지만, locale 기반 라우팅과 static rendering을 함께 가져가려면 중요한 포인트입니다.

---

## 🚧 자주 헷갈리는 부분

### 1. `useTranslations`를 아무 데서나 쓰면 안 됩니다

`async` 서버 컴포넌트에서는 훅을 직접 사용할 수 없습니다.

이 경우는 `getTranslations`를 써야 합니다.

### 2. `next/link` 대신 locale-aware `Link`를 쓰는 편이 안전합니다

수동으로 locale prefix를 붙이는 방식은 금방 중복과 실수를 만들기 쉽습니다.

```txt
가능하면
@/i18n/navigation 의 Link, useRouter, redirect 사용
```

### 3. 메시지 구조는 처음부터 namespace를 잘 나누는 편이 좋습니다

처음부터 flat key를 마구 만들기보다 페이지나 기능 단위로 나누는 편이 유지보수에 유리합니다.

```json
{
  "Auth": {
    "SignIn": {
      "title": "로그인"
    }
  }
}
```

### 4. 클라이언트에 번역을 넘기는 범위를 생각해야 합니다

처음에는 편하게 전체 메시지를 넘겨도 되지만, 규모가 커지면 서버 번역 후 props 전달 패턴이 더 좋아질 수 있습니다.

---

## ✅ 정리

`next-intl`을 잘 쓰려면 단순히 `t('hello')`를 찍는 방법보다, locale이 프로젝트 안에서 어떻게 흐르는지를 이해하는 것이 중요합니다.

흐름을 다시 정리하면 이렇습니다.

1. `routing.ts`에서 지원 locale과 기본 locale을 정합니다.
2. `proxy.ts`에서 요청을 locale 라우팅과 연결합니다.
3. `request.ts`에서 현재 locale과 messages를 준비합니다.
4. `app/[locale]` 구조로 App Router에 locale 세그먼트를 반영합니다.
5. 일반 컴포넌트에서는 `useTranslations`, async 서버 컴포넌트에서는 `getTranslations`를 사용합니다.
6. navigation은 `createNavigation`으로 만든 locale-aware API를 사용합니다.
7. 필요하면 `setRequestLocale`로 static rendering까지 챙깁니다.

이 구조를 이해하고 나면 `next-intl`은 단순한 번역 라이브러리보다, **Next.js App Router에서 다국어 앱의 뼈대를 잡아주는 도구**처럼 느껴질 겁니다.
