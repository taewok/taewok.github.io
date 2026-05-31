---
title: "[React] SweetAlert2로 알림 모달 사용하기"
date: 2023-05-21T21:28:00
categories: [react]
tags: [react, sweetalert2, library, modal, alert]
description: "React 프로젝트에서 SweetAlert2를 설치하고 alert, confirm, prompt 형태로 사용하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

브라우저 기본 `alert()`와 `confirm()`은 간단하지만 디자인을 바꾸기 어렵고, 서비스 분위기와도 잘 맞지 않을 때가 많습니다.

이럴 때 SweetAlert2를 사용하면 조금 더 보기 좋은 알림 모달을 빠르게 만들 수 있습니다.

이번 글에서는 React 프로젝트에서 SweetAlert2를 설치하고 기본 알림, 확인 모달, 입력 모달을 사용하는 방법을 정리해볼게요.

---

## 설치하기

먼저 라이브러리를 설치합니다.

```bash
npm install sweetalert2
```

설치한 뒤 사용할 파일에서 import합니다.

```tsx
import Swal from "sweetalert2";
```

---

## 기본 알림 띄우기

가장 기본적인 사용법은 `Swal.fire()`를 호출하는 것입니다.

```tsx
const handleAlert = () => {
  Swal.fire({
    title: "저장 완료",
    text: "데이터가 정상적으로 저장되었습니다.",
    icon: "success",
  });
};
```

`icon`에는 `success`, `error`, `warning`, `info`, `question` 등을 사용할 수 있습니다.

---

## 확인 모달 만들기

삭제나 로그아웃처럼 사용자의 확인이 필요한 작업에는 확인 모달을 사용할 수 있습니다.

```tsx
const handleDelete = async () => {
  const result = await Swal.fire({
    title: "정말 삭제하시겠어요?",
    text: "삭제한 데이터는 복구할 수 없습니다.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "삭제",
    cancelButtonText: "취소",
  });

  if (result.isConfirmed) {
    await deleteItem();

    Swal.fire({
      title: "삭제 완료",
      icon: "success",
    });
  }
};
```

`showCancelButton: true`를 주면 취소 버튼이 함께 표시됩니다.

사용자가 확인 버튼을 누르면 `result.isConfirmed`가 `true`가 됩니다.

---

## 입력 모달 만들기

간단한 값을 입력받아야 할 때는 `input` 옵션을 사용할 수 있습니다.

```tsx
const handleNicknameChange = async () => {
  const result = await Swal.fire({
    title: "닉네임 변경",
    input: "text",
    inputLabel: "새 닉네임을 입력해주세요",
    inputPlaceholder: "닉네임",
    showCancelButton: true,
    confirmButtonText: "변경",
    cancelButtonText: "취소",
  });

  if (result.value) {
    console.log("입력한 닉네임:", result.value);
  }
};
```

입력값은 `result.value`에서 확인할 수 있습니다.

---

## 버튼 문구와 색상 변경하기

SweetAlert2는 버튼 문구와 색상도 쉽게 바꿀 수 있습니다.

```tsx
Swal.fire({
  title: "로그아웃하시겠어요?",
  icon: "question",
  showCancelButton: true,
  confirmButtonText: "로그아웃",
  cancelButtonText: "머무르기",
  confirmButtonColor: "#2563eb",
  cancelButtonColor: "#6b7280",
});
```

서비스의 디자인 톤에 맞춰 색상을 조정하면 기본 브라우저 알림보다 훨씬 자연스럽게 보입니다.

---

## 자주 확인하는 응답 값

| 값 | 의미 |
| --- | --- |
| `isConfirmed` | 확인 버튼을 눌렀는지 |
| `isDenied` | 거절 버튼을 눌렀는지 |
| `isDismissed` | 취소, 바깥 클릭, ESC 등으로 닫혔는지 |
| `value` | 입력 모달에서 사용자가 입력한 값 |

confirm 모달에서는 `isConfirmed`를 가장 자주 확인합니다.

prompt 형태에서는 `value`를 확인하면 됩니다.

---

## React에서 사용할 때 주의할 점

SweetAlert2는 React 컴포넌트처럼 JSX로 렌더링되는 방식이 아니라, 라이브러리가 직접 모달을 띄우는 방식입니다.

그래서 React state와 완전히 같은 흐름으로 움직인다고 생각하면 헷갈릴 수 있습니다.

API 호출이나 상태 변경은 `await Swal.fire()` 이후 결과값을 확인한 다음 처리하는 편이 좋습니다.

---

## 마무리

SweetAlert2를 사용하면 기본 브라우저 알림보다 보기 좋고 유연한 알림 모달을 빠르게 만들 수 있습니다.

단순 알림은 `Swal.fire()`, 확인이 필요한 작업은 `showCancelButton`, 입력이 필요한 작업은 `input` 옵션을 사용하면 됩니다.

작은 프로젝트에서는 간단한 모달 대체제로도 충분히 유용하게 사용할 수 있습니다.
