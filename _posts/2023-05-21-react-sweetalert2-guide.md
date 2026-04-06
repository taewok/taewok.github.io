---
title: "[React] SweetAlert2 완벽 활용하기"
date: 2023-05-21T21:28:00
categories: [react]
tags: [react, sweetalert2, library, modal, alert]
description: "기본 브라우저 alert은 이제 그만! React 프로젝트의 사용자 경험을 높여주는 SweetAlert2 설치부터 커스텀 활용법까지 실제 개발 경험을 담아 정리했습니다."
custom_style: true
---

## 🚀 도입: 브라우저 기본 Alert, 그대로 쓰실 건가요?

사용자에게 중요한 알림을 띄울 때 브라우저 기본 `alert()`이나 `confirm()`을 쓰면 디자인이 일관되지 않고 제어도 까다롭습니다. 저 역시 프로젝트를 진행하며 "서비스 브랜딩에 맞는 깔끔한 모달이 필요하다"는 고민 끝에 **SweetAlert2**를 도입하게 되었습니다. 단순한 메시지 전달을 넘어, 실전에서 마주한 다양한 처리 방식과 팁을 공유합니다.

---

## 📦 SweetAlert2 설치

먼저 라이브러리를 설치해야 합니다. React 환경에서 터미널을 열고 아래 명령어를 입력하세요.

```jsx
npm install sweetalert2
```

---

## 💡 주요 사용법 및 해결 과정

### 1. 기본 Alert (Simple Notification)

가장 간단한 형태로, 작업 성공이나 경고를 알릴 때 사용합니다.

```jsx
import Swal from "sweetalert2";

const handleBasicAlert = () => {
  Swal.fire({
    title: "성공!",
    text: "데이터가 안전하게 저장되었습니다.",
    icon: "success",
  });
};
```

### 2. Confirm(확인) 모달: 사용자 의사 묻기

로그아웃이나 삭제처럼 '실행 전 확인'이 필요한 경우에 사용합니다. `then` 메서드를 통해 비동기 처리가 가능하다는 점이 큰 장점입니다.

```jsx
const handleConfirm = () => {
  Swal.fire({
    title: "정말 삭제하시겠습니까?",
    text: "삭제 후에는 복구가 불가능합니다!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: "#d33",
    confirmButtonText: "삭제",
    cancelButtonText: "취소",
  }).then((result) => {
    if (result.isConfirmed) {
      // 삭제 API 호출 로직 등을 여기에 작성
      Swal.fire("삭제 완료!", "파일이 삭제되었습니다.", "success");
    }
  });
};
```

### 3. Prompt Alert: 데이터 입력받기

별도의 폼 페이지를 만들기 부담스러울 때, 간단한 텍스트 입력을 받는 용도로 활용했습니다.

```jsx
const handlePrompt = () => {
  Swal.fire({
    title: "닉네임 수정",
    input: "text",
    inputLabel: "새로운 닉네임을 입력하세요",
    inputPlaceholder: "닉네임 입력...",
  }).then((res) => {
    if (res.value) {
      console.log("입력된 값:", res.value);
    }
  });
};
```

---

## 🛠️ 실무 포인트: 이것만은 꼭!

<div style="background-color: #f8f9fa; border-left: 5px solid #2196F3; padding: 15px; margin-bottom: 20px; color: #91512c;">
    <strong>✅ 언제 써야 할까?</strong><br/>
    단순 알림뿐만 아니라, 로딩 상태 표시(swal.showLoading)나 복잡한 입력 폼이 없는 간단한 데이터 수정 창이 필요할 때 생산성을 극대화할 수 있습니다.
</div>

<div style="background-color: #fff3cd; border-left: 5px solid #ffc107; padding: 15px; margin-bottom: 20px; color: #91512c;">
    <strong>⚠️ 실수하기 쉬운 부분</strong><br/>
    React 환경에서 <code>Swal.fire</code>는 컴포넌트 생명주기와 상관없이 DOM에 직접 접근하여 렌더링됩니다. 따라서 <strong>상태값(State)이 즉시 반영되지 않을 수 있으므로</strong>, <code>then</code> 구문 안에서 후처리 로직을 정확히 작성해야 합니다.
</div>

<div style="background-color: #fce4ec; border-left: 5px solid #e91e63; padding: 15px; color: #91512c;">
    <strong>🔥 실제 개발 경험</strong><br/>
    전역 스타일링이 꼬여서 모달이 뒤로 숨는 현상을 겪었습니다. 이럴 땐 <code>customClass</code> 속성을 사용하여 <code>z-index</code>를 명시적으로 조절하거나 테마 파일을 별도로 임포트하여 해결했습니다.
</div>

---

## 📊 버튼 응답 값 정리

|   속성명    | 설명                                                      |
| :---------: | :-------------------------------------------------------- |
| isConfirmed | 사용자가 '확인' 또는 'OK' 버튼을 클릭함                   |
|  isDenied   | 사용자가 '거부' 버튼을 클릭함                             |
| isDismissed | 사용자가 '취소'를 누르거나, 배경을 클릭하거나, ESC를 누름 |
|    value    | Prompt 모달에서 사용자가 입력한 값                        |

---

## 🏁 마무리하며

SweetAlert2는 단순한 라이브러리를 넘어, 유저와 시스템 사이의 상호작용을 풍성하게 만들어주는 도구입니다. 특히 수많은 커스텀 옵션(timer, toast 모드 등)을 지원하니, 공식 문서를 참고하여 프로젝트에 최적화된 모달을 구현해 보세요!

<p>혹시 사용 중 잘 안되는 부분이나 궁금한 점이 있다면 댓글로 남겨주세요. 피드백은 언제나 환영입니다! 😊</p>
