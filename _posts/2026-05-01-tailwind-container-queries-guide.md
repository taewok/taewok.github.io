---
title: "[Css] 반응형 웹의 패러다임 전환: CSS Container Queries(@container) 실전 활용법"
date: 2026-05-01T11:10:27Z
categories: [frontend]
tags: [tailwind-css, container-queries, responsive-design, css]
description: "미디어 쿼리의 한계를 넘어서는 컨테이너 쿼리(@container)의 개념과 Tailwind CSS에서의 실전 적용 사례를 개발일지 형식으로 정리했습니다."
custom_style: true
---

# 🚀 반응형 웹의 패러다임 전환: @container 완벽 가이드

최근 대시보드 UI를 개발하면서 **"특정 컴포넌트의 넓이에 따라 레이아웃이 유연하게 변해야 한다"**는 요구사항을 맞닥뜨렸습니다. 기존의 미디어 쿼리(`@media`)만으로는 해결하기 어려운 복잡한 구조였죠. 이때 구원투수가 되어준 Tailwind CSS의 `@container` 도입 과정과 실전 인사이트를 공유합니다.

---

## 🧐 왜 미디어 쿼리로는 부족했을까? (문제 인식)

기존의 미디어 쿼리는 **브라우저의 뷰포트(Viewport)** 넓이를 기준으로 작동합니다. 하지만 컴포넌트 중심의 현대 개발 환경에서는 다음과 같은 고민이 생깁니다.

**"사이드바가 있을 때와 없을 때, 같은 '카드 컴포넌트'라도 가용 공간이 다른데 왜 똑같이 반응해야 하지?"**

이런 브라우저 크기가 아닌, 부모 요소의 크기에 따라 스타일을 변경하고 싶을 때 사용하는 것이 바로 `Container Queries(@container)`입니다.

---

## 🛠️ 개념 설명: CSS와 Tailwind CSS의 연결고리

표준 CSS의 **Container Queries**는 특정 요소를 '컨테이너'로 지정하고, 그 자식 요소들이 컨테이너의 크기를 감시하도록 만듭니다.

### 1. CSS 표준에서의 역할

기존 CSS에서는 `container-type: inline-size`를 설정하여 부모를 기준점으로 만듭니다. Tailwind CSS에서는 이를 `@container` 클래스로 간단히 구현합니다.

### 2. Tailwind CSS에서의 매칭

| Tailwind 클래스              | CSS 속성                       | 비고                        |
| :--------------------------- | :----------------------------- | :-------------------------- |
| `@container`                 | `container-type: inline-size;` | 부모 요소를 컨테이너로 선언 |
| `@container-normal`          | `container-type: normal;`      | 기본값 (크기 감시 안 함)    |
| `@md:`, `@lg:` (Container용) | `@container (min-width: ...)`  | 컨테이너 크기별 접두사      |

---

## 💻 실전 코드 예시: 카드 컴포넌트 최적화

사이드바 유무에 따라 크기가 변하는 가변형 컨테이너 안에서, **카드 내 텍스트와 이미지 배치를 자동으로 조절**하는 로직을 구현했습니다.

```jsx
/* 1. 부모를 컨테이너로 선언 (@container) */
<div className="w-full @container border border-border-main p-4 rounded-xl bg-bg-darker">
  {/* 2. 컨테이너 크기에 따라 내부 요소 스타일 변경 */}
  <div className="flex flex-col @md:flex-row gap-4">
    {/* 이미지 영역 */}
    <div className="w-full @md:w-32 h-32 bg-blue-500 rounded-lg shrink-0"></div>

    <div className="flex-1">
      <h3 className="text-lg font-bold @md:text-2xl text-font-1">
        컨테이너 쿼리 적용 제목
      </h3>
      <p className="text-sm text-font-2 @lg:block hidden mt-2">
        이 텍스트는 컨테이너가 'lg' 사이즈(보통 32rem 이상)일 때만 보입니다.
        뷰포트가 아닌 실제 카드가 놓인 박스의 크기에 반응합니다!
      </p>
    </div>
  </div>
</div>
```

### 💡 실행 결과 포인트

- 부모 요소가 좁은 곳(예: 사이드바 내부)에 배치되면 `flex-col`로 보입니다.
- 부모 요소가 넓은 곳(예: 메인 콘텐츠 영역)에 배치되면 자동으로 `@md:flex-row`가 적용되어 가로로 배열됩니다.
- **사용자는 브라우저 리사이징 없이도 배치된 위치에 맞는 최적의 UI를 보게 됩니다.**

---

## ⚡ 실전 포인트: 언제, 어떻게 써야 할까?

### ✅ 언제 써야 하는가?

1.  재사용 컴포넌트 제작 시: 카드가 메인 화면, 사이드바, 모달 등 어디에 배치될지 모를 때 필수입니다.
2.  대시보드 UI: 위젯의 크기가 사용자에 의해 조절되거나 그리드 레이아웃이 복잡할 때 유용합니다.
3.  CMS 환경: 에디터 내부에 들어가는 컴포넌트처럼 주변 환경 예측이 어려울 때 사용합니다.

### ⚠️ 실수하기 쉬운 부분

- 이름 중복 주의: 여러 컨테이너가 중첩될 경우 `@container/{name}` 형식을 사용하여 특정 부모를 지칭해야 헷갈리지 않습니다. ex) @container/main이라면 @md/main:w-full

### 🛠️ 개발하며 겪은 삽질기

처음에 `@container` 클래스를 **자식 요소**에 직접 넣고 왜 안 변하냐며 헤맸던 기억이 있습니다. 반드시 **변화를 감지할 부모 요소**에 `@container`를 선언하고, **실제 변해야 하는 자식**에 `@md:` 등의 프리픽스를 붙여야 한다는 점을 잊지 마세요!

---

## 📊 비교: @media vs @container

| 비교 항목     | 미디어 쿼리 (@media)                   | 컨테이너 쿼리 (@container)             |
| :------------ | :------------------------------------- | :------------------------------------- |
| 기준점        | 브라우저 창 (Viewport) 전체 넓이       | 가장 가까운 부모 컨테이너 넓이         |
| 재사용성      | 배치 위치에 따라 스타일이 깨질 수 있음 | 어디에 배치해도 알아서 적응함 (독립성) |
| 복잡도        | 전역 레이아웃 설계에 유리              | 개별 컴포넌트 단위 설계에 유리         |
| Tailwind 사용 | `md:`, `lg:`, `xl:`                    | `@md:`, `@lg:`, `@xl:`                 |

---

## 🏁 마치며

`@container`의 등장은 컴포넌트 기반 개발 방식에서 "진정한 독립성"을 부여해 준 혁신적인 도구입니다. 이제 "이 컴포넌트 사이드바에 넣으면 깨지나요?"라는 질문에 당당하게 **"아니요, 알아서 반응합니다"**라고 답할 수 있게 되었습니다.

아직 미디어 쿼리에만 의존하고 있다면, 오늘 바로 Tailwind CSS 설정에 `@container`를 추가해 보세요. 퍼블리싱의 질이 달라질 것입니다! 🚀

---
