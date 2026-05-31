---
title: "[Next.js] 안전한 뒤로가기 구현하기: router.back()만으로 부족할 때"
date: 2026-04-19T13:38:55Z
categories: [frontend]
tags: [nextjs, react, javascript, navigation, seo]
description: "외부 유입과 내부 이동을 구분해 router.back()과 fallback 이동을 안전하게 처리하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 뒤로가기는 생각보다 까다로워요

상세 페이지나 결제 페이지를 만들다 보면 `이전으로` 버튼이 필요할 때가 많습니다.

처음에는 단순하게 `router.back()`을 사용하면 충분해 보입니다.

```tsx
router.back();
```

하지만 실제 서비스에서는 몇 가지 애매한 상황이 생깁니다.

- 검색이나 광고를 통해 상세 페이지로 바로 들어온 사용자는 돌아갈 내부 페이지가 없습니다.
- 외부 사이트에서 들어온 사용자를 다시 외부 사이트로 내보낼 수 있습니다.
- 브라우저 히스토리만 보고 판단하면 서비스 내부 이동인지 알기 어렵습니다.

그래서 뒤로가기 버튼을 만들 때는 **정말 뒤로 가도 되는 상황인지**를 먼저 확인하고, 그렇지 않으면 안전한 경로로 이동시키는 fallback 처리가 필요합니다.

---

## 💡 기본 아이디어

안전한 뒤로가기를 만들기 위해 두 가지를 함께 확인했습니다.

1. 브라우저에 이전 히스토리가 있는지
2. 이전 페이지가 현재 서비스 내부 페이지인지

이때 사용할 수 있는 값이 `window.history.length`와 `document.referrer`입니다.

```tsx
const hasHistory = window.history.length > 1;
const isInternalNavigation = document.referrer.includes(window.location.host);
```

`hasHistory`는 브라우저 세션 히스토리에 이전 기록이 있는지 확인하고, `isInternalNavigation`은 이전 페이지가 현재 서비스 도메인인지 확인합니다.

둘 다 만족할 때만 `router.back()`을 실행하고, 그렇지 않으면 미리 정한 경로로 이동시키면 됩니다.

---

## 🛠️ 구현 코드

Next.js에서 사용할 수 있는 형태로 정리하면 다음과 같습니다.

```tsx
import { useRouter } from "next/navigation";

export const useSmartBack = (fallbackPath = "/") => {
  const router = useRouter();

  const goBack = () => {
    const hasHistory = window.history.length > 1;
    const isInternalNavigation = document.referrer.includes(
      window.location.host,
    );

    if (hasHistory && isInternalNavigation) {
      router.back();
      return;
    }

    router.push(fallbackPath);
  };

  return { goBack };
};
```

사용하는 쪽에서는 이렇게 호출할 수 있습니다.

```tsx
const { goBack } = useSmartBack("/characters");

return (
  <button type="button" onClick={goBack}>
    이전으로
  </button>
);
```

이렇게 하면 내부 목록 페이지에서 상세 페이지로 들어온 사용자는 자연스럽게 이전 페이지로 돌아가고, 외부에서 바로 들어온 사용자는 `/characters` 같은 안전한 경로로 이동합니다.

---

## 📌 window.history.length

`window.history.length`는 현재 브라우저 탭의 세션 히스토리 개수를 알려줍니다.

```tsx
const hasHistory = window.history.length > 1;
```

새 탭에서 현재 페이지를 처음 열었다면 보통 `1`입니다. 그래서 `1`보다 크면 이전 기록이 있다고 볼 수 있습니다.

다만 이것만으로는 부족합니다. 이전 기록이 있더라도 그 기록이 우리 서비스 내부 페이지인지, 검색 엔진이나 메신저 같은 외부 페이지인지는 알 수 없기 때문입니다.

---

## 📌 document.referrer

`document.referrer`는 현재 페이지로 오기 전 페이지의 URL을 담고 있습니다.

```tsx
const isInternalNavigation = document.referrer.includes(window.location.host);
```

이 값에 현재 서비스의 host가 포함되어 있다면 내부 페이지에서 이동해온 것으로 판단할 수 있습니다.

다만 `document.referrer`도 항상 완벽한 값은 아닙니다. 직접 URL을 입력하거나, 브라우저 보안 정책에 따라 referrer가 비어 있을 수 있습니다.

그래서 `history.length`와 `referrer`를 함께 확인하는 편이 더 안전합니다.

---

## 📊 방식 비교

| 방식 | 장점 | 아쉬운 점 |
| --- | --- | --- |
| `router.back()`만 사용 | 구현이 가장 간단함 | 외부 페이지로 돌아갈 수 있음 |
| `history.length`만 확인 | 이전 기록 유무를 알 수 있음 | 내부 이동인지 알 수 없음 |
| `referrer`까지 확인 | 내부 이동 여부를 함께 판단 가능 | referrer가 비어 있는 경우를 고려해야 함 |

완벽한 정답이라기보다는, 사용자를 서비스 밖으로 갑자기 내보내지 않기 위한 안전장치에 가깝습니다.

---

## ⚠️ 주의할 점

이 방식은 브라우저 환경에서만 동작합니다. `window`와 `document`를 사용하기 때문에 서버 컴포넌트나 서버 렌더링 중에는 직접 실행하면 안 됩니다.

Next.js App Router에서는 클라이언트 컴포넌트 안에서 버튼 클릭 이벤트로 실행하는 방식이 가장 자연스럽습니다.

또 `document.referrer`는 정책에 따라 비어 있을 수 있으므로, referrer가 없을 때 이동할 fallback 경로를 반드시 준비해두는 것이 좋습니다.

---

## ✅ 마무리

뒤로가기 버튼은 작아 보이지만, 사용자의 이동 흐름을 크게 좌우합니다.

`router.back()`만 사용하면 간단하지만, 외부 유입 사용자를 예상하지 못한 곳으로 보내버릴 수 있습니다.

`window.history.length`로 이전 기록이 있는지 확인하고, `document.referrer`로 내부 이동인지 한 번 더 확인하면 조금 더 안전한 뒤로가기 경험을 만들 수 있습니다.
