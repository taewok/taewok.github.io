---
title: "window.matchMedia로 반응형 사이드바 완벽 구현하기 (resize 이벤트 지옥 탈출)"
date: 2026-04-17T13:48:00Z
categories: [frontend]
tags: [javascript, matchmedia, responsive-design, sidebar-optimization]
description: "창 크기가 변할 때마다 실행되는 resize 이벤트 대신, 성능과 정확도를 모두 잡은 window.matchMedia 활용법을 공유합니다."
custom_style: true
---

---

## 📝 도입: 왜 resize 이벤트가 불편했을까?

사이드바가 특정 화면 너비(예: `1024px`) 이하로 내려갈 때 자동으로 접히는 기능을 구현하면서 처음에는 당연하게 `window.addEventListener('resize', ...)`를 떠올렸습니다. 하지만 구현 후 몇 가지 찝찝한 점이 남더군요.

1.  불필요한 호출: 1px만 변해도 함수가 미친 듯이 실행됩니다. `debounce`를 걸어도 결국 '계산'이 필요하죠.
2.  가독성: `window.innerWidth < 1024` 같은 조건문이 코드 곳곳에 흩어지기 시작합니다.
3.  일관성: CSS 미디어 쿼리는 `1024px` 기준인데, JS는 스크롤바 너비 포함 여부에 따라 결과가 미묘하게 달라지는 버그가 발생하곤 했습니다.

이때 발견한 해결책이 바로 **window.matchMedia**입니다. CSS 미디어 쿼리의 메커니즘을 그대로 자바스크립트로 가져올 수 있어 성능과 일관성을 동시에 잡을 수 있었습니다.

---

## 💡 window.matchMedia란?

*window.matchMedia*는 주어진 미디어 쿼리 문자열이 현재 문서와 일치하는지 확인하는 메서드입니다. 가장 큰 장점은 **"상태가 변할 때만"** 이벤트를 발생시킬 수 있다는 점입니다.

---

## 💻 코드 예시: 반응형 사이드바 로직

실제 프로젝트에서 사이드바의 `isOpen` 상태를 제어하기 위해 작성한 코드입니다.

```jsx
  const [isFolded, setIsFolded] = useState(false);

  // 화면 너비에 따른 자동 폴딩 로직 추가
  useEffect(() => {
    // 1024px 미만인지 체크하는 매치 미디어
    const mql = window.matchMedia("(max-width: 1023px)");

    const handleSceneChange = (e: MediaQueryListEvent | MediaQueryList) => {
      // 1024px 미만이면 fold(true), 이상이면 fold 해제(false)
      setIsFolded(e.matches);
    };

    // 초기 실행
    handleSceneChange(mql);

    // 이벤트 리스너 등록
    mql.addEventListener("change", handleSceneChange);
    return () => mql.removeEventListener("change", handleSceneChange);
  }, []);
```

matchMedia는 한마디로 `자바스크립트용 미디어 쿼리(Media Query)`입니다.

CSS에서 @media (max-width: 1024px) { ... }처럼 화면 크기에 따라 스타일을 바꿨다면, 자바스크립트에서는 window.matchMedia()를 사용해 화면 크기나 상태 변화에 따라 특정 로직(함수)을 실행할 수 있게 해줍니다.

---

## 🚀 실무 포인트

### 언제 써야 할까?

단순히 요소의 너비를 실시간으로 계산해야 한다면 `ResizeObserver`가 맞지만, **"특정 브레이크포인트"**를 기점으로 레이아웃 모드(모바일/데스크탑)를 전환해야 한다면 \***\*matchMedia\*\***가 정답입니다.

### 실수하기 쉬운 부분 (Legacy 지원)

오래된 브라우저나 일부 환경에서는 `.addEventListener('change', ...)` 대신 `.addListener(...)`를 사용해야 할 수도 있습니다. 최신 브라우저 위주라면 `change`가 표준입니다.

### 실제 개발 경험 (Performance)

`resize` 이벤트에 `console.log`를 찍어보면 창을 드래그하는 동안 수백 번 찍히지만, `matchMedia`는 브레이크포인트를 넘나들 때 딱 **한 번씩만** 찍힙니다. 브라우저의 메인 스레드 부담을 획기적으로 줄이면서 사이드바 애니메이션 꼬임 현상을 해결했습니다.

---

## 📊 비교/정리

| 항목      | window.innerWidth + resize               | window.matchMedia                              |
| :-------- | :--------------------------------------- | :--------------------------------------------- |
| 성능      | 매우 빈번한 이벤트 호출 (최적화 필수)    | 브레이크포인트 통과 시에만 발생 (가벼움)       |
| 정확도    | 스크롤바 포함 여부에 따라 오차 발생 가능 | CSS 미디어 쿼리와 100% 일치                    |
| 복잡도    | `debounce` 등 추가 로직 필요             | 직관적인 `matches` 불리언 값 제공              |
| 권장 용도 | 실시간 수치 계산 (차트 리사이징 등)      | **반응형 UI 레이아웃 전환 (사이드바, 네비바)** |

---

## 🏁 마치며

사이드바를 접고 펼치는 단순한 기능이지만, **"어떤 이벤트를 구독하느냐"**에 따라 코드의 질이 달라진다는 것을 깨달았습니다. 이제 `resize` 이벤트 안에서 `if (window.innerWidth < 768)`를 남발하지 말고, 미디어 쿼리를 자바스크립트의 영역으로 끌어와 보세요. 훨씬 우아한 코드가 완성될 겁니다!
