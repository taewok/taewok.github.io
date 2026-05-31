---
title: "[CSS] Tailwind CSS에서 Container Queries 사용하기"
date: 2026-05-01T11:10:27Z
categories: [frontend]
tags: [tailwind-css, container-queries, responsive-design, css]
description: "뷰포트가 아니라 부모 컨테이너의 크기에 따라 스타일을 바꾸는 Container Queries의 개념과 Tailwind CSS 사용법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 화면 크기보다 컴포넌트 크기가 더 중요할 때

반응형 UI를 만들 때 가장 익숙한 도구는 미디어 쿼리입니다.

```css
@media (min-width: 768px) {
  /* tablet 이상 스타일 */
}
```

미디어 쿼리는 브라우저의 뷰포트 너비를 기준으로 스타일을 바꿉니다. 대부분의 레이아웃에서는 충분히 유용합니다.

하지만 컴포넌트 기반 UI를 만들다 보면 이런 상황을 만납니다.

같은 카드 컴포넌트가 메인 영역에서는 넓게 보이고, 사이드바나 모달 안에서는 좁게 보일 수 있습니다. 이때 브라우저 너비는 같지만 카드가 실제로 사용할 수 있는 공간은 전혀 다릅니다.

이런 경우에는 뷰포트가 아니라 **부모 컨테이너의 크기**를 기준으로 반응해야 합니다. 이때 사용하는 기능이 `Container Queries`입니다.

---

## 💡 Container Queries란?

Container Queries는 특정 요소를 컨테이너로 지정하고, 그 컨테이너의 크기에 따라 자식 요소의 스타일을 바꾸는 CSS 기능입니다.

기존 미디어 쿼리가 화면 전체를 기준으로 했다면, 컨테이너 쿼리는 컴포넌트가 놓인 박스의 크기를 기준으로 합니다.

```css
.card-wrapper {
  container-type: inline-size;
}

@container (min-width: 480px) {
  .card {
    flex-direction: row;
  }
}
```

이렇게 하면 브라우저 너비가 아니라 `.card-wrapper`의 너비가 `480px` 이상일 때 카드 스타일이 바뀝니다.

---

## 🛠️ Tailwind CSS에서는 어떻게 쓸까요?

Tailwind CSS에서는 부모 요소에 `@container` 클래스를 붙여 컨테이너로 지정할 수 있습니다.

그다음 자식 요소에서는 `@md:`, `@lg:` 같은 컨테이너 기준 접두사를 사용합니다.

```tsx
const CharacterCard = () => {
  return (
    <div className="w-full @container rounded-xl border p-4">
      <div className="flex flex-col gap-4 @md:flex-row">
        <div className="h-32 w-full shrink-0 rounded-lg bg-blue-500 @md:w-32" />

        <div className="min-w-0 flex-1">
          <h3 className="text-lg font-bold @md:text-2xl">
            컨테이너 쿼리 적용 제목
          </h3>

          <p className="mt-2 hidden text-sm text-gray-500 @lg:block">
            이 설명은 카드가 놓인 컨테이너가 충분히 넓을 때만 보입니다.
          </p>
        </div>
      </div>
    </div>
  );
};
```

여기서 중요한 구조는 다음과 같습니다.

```tsx
<div className="@container">
  <div className="@md:flex-row">...</div>
</div>
```

`@container`는 크기 변화를 감시할 부모에게 붙입니다. 실제로 스타일이 바뀌어야 하는 요소에는 `@md:` 같은 접두사를 붙입니다.

---

## 📌 미디어 쿼리와 무엇이 다를까요?

미디어 쿼리는 브라우저 화면 전체를 기준으로 합니다.

```tsx
<div className="flex flex-col md:flex-row" />
```

위 코드는 뷰포트가 `md` 이상이면 가로 배치가 됩니다.

반면 컨테이너 쿼리는 부모 컨테이너의 크기를 기준으로 합니다.

```tsx
<div className="@container">
  <div className="flex flex-col @md:flex-row" />
</div>
```

이 코드는 화면이 아무리 넓어도, 부모 컨테이너가 좁으면 세로 배치를 유지합니다.

그래서 재사용 컴포넌트에 특히 잘 맞습니다. 같은 컴포넌트가 어디에 들어가든 자신이 실제로 가진 공간에 맞춰 스타일을 바꿀 수 있기 때문입니다.

---

## 📊 비교 정리

| 항목 | 미디어 쿼리 | 컨테이너 쿼리 |
| --- | --- | --- |
| 기준 | 브라우저 뷰포트 | 부모 컨테이너 |
| Tailwind 접두사 | `md:`, `lg:` | `@md:`, `@lg:` |
| 장점 | 전체 레이아웃 제어에 좋음 | 재사용 컴포넌트에 좋음 |
| 예시 | 페이지 레이아웃 변경 | 카드 내부 배치 변경 |

둘 중 하나만 써야 하는 것은 아닙니다.

페이지 전체 구조는 미디어 쿼리로 잡고, 카드나 위젯처럼 재사용되는 컴포넌트 내부는 컨테이너 쿼리로 다루면 좋습니다.

---

## ⚠️ 자주 헷갈리는 부분

가장 흔한 실수는 `@container`를 자식 요소에 붙이는 것입니다.

```tsx
<div className="flex flex-col @container @md:flex-row" />
```

이렇게 작성하면 의도한 대로 동작하지 않을 수 있습니다.

컨테이너 쿼리는 기준이 되는 부모와, 그 기준에 따라 변하는 자식을 나누어 생각해야 합니다.

```tsx
<div className="@container">
  <div className="flex flex-col @md:flex-row" />
</div>
```

또 여러 컨테이너가 중첩되어 있다면 이름을 붙여 명확하게 지정할 수도 있습니다.

```tsx
<div className="@container/card">
  <div className="@md/card:flex-row" />
</div>
```

---

## ✅ 마무리

Container Queries는 컴포넌트가 실제로 놓인 공간에 맞춰 반응형 스타일을 적용할 수 있게 해줍니다.

미디어 쿼리가 여전히 페이지 전체 반응형 레이아웃에 유용하다면, 컨테이너 쿼리는 카드, 위젯, 사이드 패널처럼 재사용되는 컴포넌트에 잘 맞습니다.

Tailwind CSS에서는 `@container`와 `@md:` 같은 접두사로 비교적 간단하게 사용할 수 있으니, 컴포넌트가 배치되는 위치에 따라 스타일이 어색해지는 경우에 적용해볼 만합니다.
