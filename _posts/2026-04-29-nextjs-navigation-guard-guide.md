---
title: "[Next.js] 페이지 이동을 막는 방법: next-navigation-guard 도입기 및 적용"
date: 2026-04-29T23:42:00Z
categories: [nextjs, frontend]
tags: [nextjs, navigation-guard, react, web-publishing]
description: "Next.js App Router 환경에서 뒤로가기나 페이지 이탈을 막아야 할 때, next-navigation-guard 라이브러리를 활용하여 사용자 경험을 개선한 실무 해결 과정을 공유합니다."
custom_style: true
---

## 🚦 페이지 이탈 방지, 왜 이렇게 힘들까?

Next.js App Router를 사용하면서 가장 당혹스러웠던 순간은 **'작성 중인 폼이 있는데 사용자가 실수로 뒤로가기를 누를 때'**였습니다. 이전 Pages Router에서는 `router.events.on('routeChangeStart', ...)`라는 이벤트 리스너를 통해 비교적 쉽게 제어할 수 있었지만, App Router는 브라우저의 네이티브 히스토리를 깊게 관여하면서 이 기능을 기본적으로 제공하지 않게 되었죠.

단순히 `window.onbeforeunload`를 쓰면 되지 않냐고 생각할 수 있지만, 이는 **브라우저 새로고침이나 탭 닫기**만 막아줄 뿐, Next.js 내부의 `Link` 컴포넌트나 `router.back()`을 통한 **'소프트 내비게이션'**은 전혀 막지 못한다는 문제가 있었습니다.

### 16버전(최신)'으로 오면서 더 힘들어진 이유

🔒 브라우저 보안 및 표준 정책 강화

Next.js 16은 최신 브라우저 환경에 최적화되어 있습니다. 최근 Chrome 등 주요 브라우저는 사용자 경험을 해치는 **'페이지 이탈 차단'**을 엄격하게 제한하고 있습니다.

Next.js가 점점 웹 표준을 따르다 보니 개발자가 커스텀하게 내비게이션을 막는 기능을 넣기 점점 꺼려지는 구조가 되었습니다.

⚡️ "이미 출발한 기차를 멈출 순 없습니다" : Prefetching의 역설

Next.js 16의 App Router는 사용자보다 한발 앞서 움직입니다. 사용자가 링크에 마우스만 올려도(Hover), 혹은 링크가 화면에 보이기만 해도 다음 페이지의 데이터를 미리 가져오는 **'Link Prefetching'**이 정교하게 작동하기 때문이죠.

클릭하는 순간 화면이 즉시 바뀌도록 모든 준비를 마친 상태인데, 여기서 개발자가 "잠깐, 정말 나갈 거야?"라고 묻는 자바스크립트 가드(Guard)를 끼워 넣는 것은 이미 최고 속도로 달리는 기차의 급브레이크를 밟는 것과 같습니다.
Next.js가 추구하는 사용자 경험을 프레임워크 스스로 보호하려다 보니, 내비게이션 가드 같은 인위적인 차단 로직은 최적화 엔진과 계속해서 충돌할 수밖에 없는 것 같습니다.

---

## 💡 라이브러리 설명: App Router와 Navigation Guard

Next.js App Router 환경에서 페이지 전환을 가로채기 위해서는 프레임워크의 내부 상태를 감시하거나, 히스토리 스택을 강제로 제어하는 로직이 필요합니다.

`next-navigation-guard`는 이러한 복잡한 내부 로직을 추상화하여, 특정 조건(`shouldBlock`)이 `true`일 때 사용자에게 확인 창을 띄우거나 이동을 취소할 수 있게 도와주는 라이브러리입니다.

---

## 🛠️ 실무 적용 코드: 폼 작성 이탈 방지

실제 프로젝트에서 사용자가 게시글을 작성하다가 이탈하는 것을 막기 위해 구현한 코드입니다.

```jsx
"use client";

import { useState } from "react";
import { useNavigationGuard } from "next-navigation-guard";
import { useRouter } from "next/navigation";

export default function PostEditor() {
  const router = useRouter();
  const [content, setContent] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);

  // 가드가 가로챈 '목적지 주소' 저장
  const [pendingPath, setPendingPath] = (useState < string) | (null > null);

  // 차단 조건: 내용이 있고, 제출 중이 아닐 때
  const isDirty = content.length > 0 && !isSubmitting;

  useNavigationGuard({
    enabled: isDirty,
    confirm: (info) => {
      setPendingPath(info.to); // 가고자 했던 주소 기억
      return false; // 일단 이동 차단! (모달은 이 상태에서 띄움)
    },
  });

  const handleLeave = () => {
    if (pendingPath) {
      setIsSubmitting(true); // 가드 해제
      router.push(pendingPath); // 중단됐던 이동 재개
    }
  };

  return (
    <div className="p-6">
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="내용을 입력하면 가드가 활성화됩니다."
        className="w-full border p-2"
      />

      {/* pendingPath가 있다는 것은 이동이 차단되었다는 뜻 (모달 역할) */}
      {pendingPath && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
          <div className="bg-white p-6 rounded shadow-lg">
            <p>저장되지 않은 변경사항이 있습니다. 나갈까요?</p>
            <button onClick={() => setPendingPath(null)}>취소</button>
            <button onClick={handleLeave} className="text-red-500 ml-4">
              나가기
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### 실행 결과

- 이탈 시도 감지: 텍스트 입력 후 다른 페이지로 이동하거나 뒤로 가기를 클릭하면, 브라우저 기본 창 대신 커스텀 경고 모달이 즉시 노출됩니다.
- 이동 제어: 모달에서 취소를 누르면 현재 페이지에 머물고, 나가기를 누르면 `pendingPath`에 저장해둔 목적지로 강제 이동합니다.
- 예외 처리: 저장하기 버튼 클릭 시에는 `isSubmitting` 상태가 `true`가 되어, 가드 조건(`isDirty`)을 통과하므로 경고 없이 부드럽게 다음 단계로 넘어갑니다.

---

## 🚀 실전 포인트

### 언제 써야 하는가?

- 결제 및 예약: 결제 정보 입력 중 실수로 이탈하여 데이터가 날아가는 것을 방지할 때.
- 긴 글 작성: 블로그 에디터나 설문조사 등 입력값이 많은 폼에서 유저의 노력을 보호할 때.
- 관리자 설정: 시스템 설정을 변경한 후 저장 버튼을 누르지 않고 메뉴를 이동할 때.

### 실수하기 쉬운 부분

- **브라우저 기본 팝업**: `confirm()` 대신 커스텀 모달을 쓰려면 `NavigationGuard`의 비동기 처리를 지원하는 옵션을 잘 살펴야 합니다.

### 🛠️ 실제 개발 중 겪은 에로사항: "침묵하는 document.referrer"

이번 프로젝트를 진행하며 가장 당혹스러웠던 순간은, 당연히 될 줄 알았던 `document.referrer`가 콘솔에서 계속 빈 값("")만 뱉어낼 때였습니다. 내비게이션 가드를 세우려면 유저가 **'어디서 왔는지'**를 아는 것이 핵심인데, 이 데이터가 침묵하니 모든 로직이 꼬이기 시작했습니다.

### 🕵️‍♂️ 왜 리퍼러(Referrer)는 비어 있었을까?

조사 끝에 알아낸 이유는 `Next.js App Router`의 구조와 브라우저 보안 정책의 '합작품'이었습니다.

`Soft Navigation`의 함정: `Next.js`의 `Link`나 `router.push`는 페이지 전체를 새로고침하지 않고 자바스크립트가 주소만 갈아 끼우는 '부드러운 이동'입니다. 브라우저 입장에서는 새로운 HTTP 요청을 보낸 게 아니므로, 리퍼러를 굳이 업데이트하지 않는 경우가 많았습니다.

강화된 개인정보 보호: 최신 브라우저들은 보안을 위해 리퍼러 전송 정책(`Referrer-Policy`)을 매우 까다롭게 관리합니다. 특히 보안이 강화된 HTTPS 환경에서는 상세 주소를 아예 삭제해버리는 일이 흔합니다.

### 🛡️ 구원투수의 등장: "역시 라이브러리(next-navigation-guard)!!!"

원래 브라우저의 기본 기능인 window.history.state만 봐서는 우리 사이트 내에서 이전 기록이 얼마나 쌓였는지, 즉 '우리가 통제할 수 있는 히스토리'인지 알 방법이 없었습니다.

하지만 next-navigation-guard 라이브러리를 도입하자 놀라운 변화가 생겼습니다. 바로 `__next_navigation_guard_stack_index`라는 고마운 값이 추가된 것이죠!

이 값의 정체는? 우리 Next.js 프로젝트 내에서 페이지 히스토리가 얼마나 쌓였는지 알려주는 '내부 번호표'입니다.

왜 중요한가? 이 인덱스 값이 0보다 크다면, 유저가 외부(구글, 네이버 등)에서 뚝 떨어진 게 아니라 우리 서비스 안에서 이동 중이라는 확실한 증거가 됩니다.

결국, 브라우저가 알려주지 않는 유저의 이동 맥락을 이 라이브러리가 '인덱스'라는 명확한 숫자로 시각화해 준 셈입니다. 덕분에 우리는 "뒤로 갈 내부 페이지가 있을 때만 가드를 작동시킨다"는 정교한 로직을 짤 수 있게 되었습니다.

---

## 📊 방식 비교: 내비게이션 제어 전략

| 항목        | `onbeforeunload`       | `next-navigation-guard`           |
| :---------- | :--------------------- | :-------------------------------- |
| 타겟        | 브라우저 종료/새로고침 | SPA 내부 페이지 이동 (Link, back) |
| UX/UI       | 브라우저 기본 UI 강제  | 커스텀 확인 로직 가능             |
| 구현 난이도 | 매우 쉬움              | 중간 (Hook 설정 필요)             |
| 적용 범위   | 외부 사이트 이동 포함  | Next.js 앱 내부 경로만 해당       |

---

## ✅ 마무리하며

Next.js App Router는 강력하지만, 페이지 이동 제어 같은 세밀한 UX 구현에서는 아직 개발자의 손길이 많이 필요합니다. `next-navigation-guard`는 이런 간극을 메워주는 훌륭한 도구였습니다.

**핵심 요약:**

1. App Router에서 `Link` 이동을 막으려면 전용 라이브러리 활용이 정신 건강에 이롭다.
2. `when` 조건문에 `isSubmitting` 같은 예외 처리를 반드시 포함하자.
3. 완벽한 방어를 위해서는 브라우저 네이티브 이벤트(`beforeunload`)와 병행하라.
