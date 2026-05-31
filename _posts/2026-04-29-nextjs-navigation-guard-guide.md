---
title: "[Next.js] next-navigation-guard로 페이지 이탈 방지하기"
date: 2026-04-29T23:42:00Z
categories: [nextjs, frontend]
tags: [nextjs, navigation-guard, react, web-publishing]
description: "Next.js App Router 환경에서 작성 중인 폼의 페이지 이탈을 막기 위해 next-navigation-guard를 적용한 과정을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 작성 중인 폼을 그냥 떠나면 안 될 때

게시글 작성, 캐릭터 생성, 결제 정보 입력처럼 시간이 오래 걸리는 화면에서는 사용자가 실수로 페이지를 떠나는 상황을 막아야 합니다.

특히 작성 중인 내용이 저장되지 않았는데 뒤로가기를 누르거나 다른 링크를 클릭하면, 사용자는 입력한 내용을 잃어버릴 수 있습니다.

예전 Next.js Pages Router에서는 `router.events`를 활용해 페이지 이동을 감지하는 방식이 자주 쓰였습니다. 하지만 App Router에서는 이 흐름이 달라졌고, 내부 페이지 이동을 직접 가로채는 일이 생각보다 까다로워졌습니다.

브라우저의 `beforeunload` 이벤트도 사용할 수 있지만, 이 방식은 새로고침이나 탭 닫기에는 대응할 수 있어도 Next.js의 `Link`나 `router.push` 같은 소프트 내비게이션을 다루기에는 부족합니다.

그래서 이번 프로젝트에서는 `next-navigation-guard`를 사용해 페이지 이탈 방지 로직을 구성했습니다.

---

## 💡 next-navigation-guard를 사용한 이유

`next-navigation-guard`는 Next.js App Router에서 페이지 이동을 감지하고, 특정 조건일 때 이동을 막을 수 있게 도와주는 라이브러리입니다.

예를 들어 폼이 수정된 상태라면 이동을 막고, 사용자에게 확인 모달을 보여줄 수 있습니다.

핵심 흐름은 다음과 같습니다.

1. 폼이 수정되었는지 확인합니다.
2. 사용자가 다른 페이지로 이동하려고 합니다.
3. `useNavigationGuard`가 이동을 가로챕니다.
4. 이동하려던 경로를 저장하고 모달을 엽니다.
5. 사용자가 확인하면 저장해둔 경로로 이동합니다.

이 구조를 사용하면 브라우저 기본 confirm 창이 아니라, 서비스 디자인에 맞춘 커스텀 모달을 보여줄 수 있습니다.

---

## 🛠️ 기본 구현 코드

아래 예시는 작성 중인 내용이 있을 때 페이지 이동을 막고, 확인 모달을 보여주는 코드입니다.

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { useNavigationGuard } from "next-navigation-guard";

export default function PostEditor() {
  const router = useRouter();

  const [content, setContent] = useState("");
  const [pendingPath, setPendingPath] = useState<string | null>(null);
  const [isLeaving, setIsLeaving] = useState(false);

  const isDirty = content.trim().length > 0;

  useNavigationGuard({
    enabled: isDirty && !isLeaving,
    confirm: (info) => {
      setPendingPath(info.to);
      return false;
    },
  });

  const handleCancelLeave = () => {
    setPendingPath(null);
  };

  const handleConfirmLeave = () => {
    if (!pendingPath) return;

    setIsLeaving(true);
    router.push(pendingPath);
  };

  return (
    <div className="p-6">
      <textarea
        value={content}
        onChange={(event) => setContent(event.target.value)}
        placeholder="내용을 입력하면 페이지 이동 가드가 활성화됩니다."
        className="w-full rounded border p-3"
      />

      {pendingPath && (
        <div className="fixed inset-0 flex items-center justify-center bg-black/50">
          <div className="rounded bg-white p-6 shadow-lg">
            <p>저장되지 않은 변경사항이 있습니다. 나가시겠어요?</p>

            <div className="mt-4 flex justify-end gap-2">
              <button type="button" onClick={handleCancelLeave}>
                취소
              </button>
              <button
                type="button"
                onClick={handleConfirmLeave}
                className="text-red-500"
              >
                나가기
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

이 예제에서 가장 중요한 값은 `pendingPath`입니다.

사용자가 이동하려던 경로를 저장해두었다가, 모달에서 `나가기`를 누르면 그 경로로 다시 이동합니다.

---

## 📌 enabled는 언제 true가 되어야 할까요?

`enabled`는 페이지 이동을 막을지 결정하는 조건입니다.

```tsx
enabled: isDirty && !isLeaving;
```

폼이 수정된 상태라면 `isDirty`가 `true`가 됩니다. 하지만 사용자가 모달에서 나가기를 확정한 뒤에는 더 이상 이동을 막으면 안 됩니다.

그래서 `isLeaving` 같은 예외 상태를 함께 둡니다.

```tsx
setIsLeaving(true);
router.push(pendingPath);
```

이 처리가 없으면 사용자가 나가기를 눌렀는데도 guard가 다시 이동을 막아버릴 수 있습니다.

---

## 📌 confirm에서 false를 반환하는 이유

`confirm` 함수에서 `false`를 반환하면 현재 이동이 취소됩니다.

```tsx
confirm: (info) => {
  setPendingPath(info.to);
  return false;
};
```

여기서는 이동을 즉시 허용하지 않고, 사용자가 확인 모달에서 선택할 시간을 줍니다.

`info.to`에는 사용자가 이동하려던 경로가 들어 있으므로, 나중에 다시 이동을 이어갈 수 있습니다.

---

## 🧭 beforeunload와의 차이

`beforeunload`는 브라우저 탭을 닫거나 새로고침할 때 유용합니다.

하지만 Next.js 내부 링크 이동은 페이지 전체를 새로고침하지 않는 경우가 많습니다. 이때는 `beforeunload`가 기대처럼 동작하지 않을 수 있습니다.

비교하면 다음과 같습니다.

| 항목 | beforeunload | next-navigation-guard |
| --- | --- | --- |
| 새로고침/탭 닫기 | 대응 가능 | 주 목적은 아님 |
| App Router 내부 이동 | 부족할 수 있음 | 대응 가능 |
| UI 커스터마이징 | 브라우저 기본 UI | 커스텀 모달 가능 |
| 사용 목적 | 페이지 자체를 떠날 때 | 앱 내부 경로 이동을 막을 때 |

실무에서는 두 가지를 함께 사용할 수도 있습니다. 내부 이동은 `next-navigation-guard`, 새로고침이나 탭 닫기는 `beforeunload`로 보완하는 방식입니다.

---

## ⚠️ 실제 적용할 때 주의할 점

페이지 이동을 막는 기능은 사용자에게 꼭 필요한 상황에서만 사용하는 것이 좋습니다.

예를 들어 단순 필터 선택이나 탭 전환까지 전부 막으면 오히려 답답한 경험이 됩니다. 반대로 긴 폼 작성이나 결제 정보 입력처럼 사용자의 노력이 날아갈 수 있는 화면에서는 매우 유용합니다.

또 저장이 완료된 뒤에는 `reset`이나 `isDirty` 초기화를 통해 guard가 꺼지도록 만들어야 합니다. 저장했는데도 계속 이탈 경고가 뜨면 사용자는 혼란스러울 수 있어요.

---

## ✅ 마무리

Next.js App Router에서 페이지 이탈 방지는 생각보다 단순하지 않습니다.

`beforeunload`만으로는 내부 라우팅을 충분히 다루기 어렵고, 직접 히스토리를 제어하려고 하면 예외 케이스가 많아집니다.

`next-navigation-guard`를 사용하면 작성 중인 폼의 이탈 방지처럼 명확한 UX 요구사항을 비교적 안정적으로 구현할 수 있습니다.

핵심은 이동을 무조건 막는 것이 아니라, 사용자의 입력이 사라질 수 있는 순간에만 부드럽게 확인 과정을 넣는 것입니다.
