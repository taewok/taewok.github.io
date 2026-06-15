---
title: "[JavaScript] requestAnimationFrame 이해하기: 왜 setTimeout 대신 사용할까?"
date: 2026-06-01T10:00:00Z
categories: [javascript]
tags: [javascript, browser, animation, performance, rendering]
description: "requestAnimationFrame과 cancelAnimationFrame의 동작 원리부터 브라우저 렌더링 파이프라인, DOM 측정과 애니메이션 활용 사례까지 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 단순히 setTimeout을 쓰면 안 될까요?

태그 Overflow UI를 구현하던 중이었습니다.

부모 영역의 크기에 따라 태그가 줄바꿈되는지 계산하기 위해 각 태그의 `offsetTop`을 측정해야 했습니다. 그런데 이상하게도 어떤 순간에는 측정값이 예상과 다르게 나왔어요.

처음에는 이렇게 바로 측정했습니다.

```tsx
useLayoutEffect(() => {
  measure();
}, []);
```

분명 렌더링이 끝난 것 같은데도, 태그가 실제로 배치된 결과와 측정값이 어긋나는 경우가 있었습니다.

결국 해결 방법은 다음 한 줄이었습니다.

```tsx
requestAnimationFrame(measure);
```

그런데 여기서 궁금해졌습니다.

```txt
왜 requestAnimationFrame을 사용하면 더 안정적으로 측정될까?
그냥 setTimeout으로 조금 늦게 실행하면 안 될까?
```

이번 글에서는 `requestAnimationFrame`을 단순히 "애니메이션에 쓰는 함수"로만 보지 않고, 브라우저 렌더링 흐름 안에서 왜 필요한지 차근차근 정리해보겠습니다.

---

## 💡 requestAnimationFrame이란?

`requestAnimationFrame`은 브라우저에게 **다음 화면을 그리기 직전에 특정 함수를 실행해달라고 요청하는 API**입니다.

```tsx
requestAnimationFrame(() => {
  console.log("다음 프레임 직전에 실행됩니다.");
});
```

여기서 중요한 점은 `requestAnimationFrame`이 단순한 타이머가 아니라는 것입니다.

`setTimeout`은 지정한 시간이 지난 뒤 콜백을 실행합니다. 반면 `requestAnimationFrame`은 브라우저의 렌더링 주기에 맞춰 콜백을 실행합니다.

즉, 기준이 다릅니다.

```txt
setTimeout
→ 시간 기준

requestAnimationFrame
→ 브라우저 렌더링 프레임 기준
```

이 차이 때문에 DOM 측정, 애니메이션, 스크롤 위치 보정 같은 작업에서 `requestAnimationFrame`이 더 잘 맞는 경우가 많습니다.

---

## 🖥️ 브라우저는 어떻게 화면을 그릴까요?

브라우저는 화면을 그릴 때 대략 다음 과정을 반복합니다.

```txt
JavaScript 실행
↓
Style 계산
↓
Layout 계산
↓
Paint
↓
Composite
↓
화면 출력
```

이 과정을 보통 **렌더링 파이프라인(Rendering Pipeline)** 이라고 부릅니다.

예를 들어 다음 코드를 실행한다고 해볼게요.

```tsx
box.style.width = "300px";
```

브라우저는 단순히 값만 바꾸고 끝내지 않습니다. 요소의 크기가 바뀌었으니 레이아웃을 다시 계산해야 하고, 바뀐 결과를 화면에 다시 그려야 합니다.

```txt
JavaScript 실행
↓
Style 계산
↓
Layout 재계산
↓
Paint
↓
화면 반영
```

우리가 DOM의 크기나 위치를 읽을 때 중요한 부분은 `Layout`입니다. 브라우저가 레이아웃 계산을 마친 뒤에 읽어야 실제 화면 배치와 맞는 값을 얻을 수 있습니다.

---

## ⚠️ setTimeout의 한계

처음에는 이런 방식으로 해결하고 싶을 수 있습니다.

```tsx
setTimeout(() => {
  measure();
}, 16);
```

`16ms`는 60fps 기준으로 한 프레임에 가까운 시간입니다. 그래서 얼핏 보면 다음 프레임쯤에 실행될 것처럼 느껴집니다.

하지만 브라우저 입장에서 `setTimeout`은 그저 다음 의미일 뿐입니다.

```txt
최소 16ms 정도 지난 뒤 실행해줘
```

중요한 것은 `setTimeout`이 **브라우저가 화면을 그릴 준비를 마쳤는지**를 보장하지 않는다는 점입니다.

```txt
setTimeout
≠
렌더링 완료
```

브라우저가 바쁘거나, 다른 JavaScript 작업이 길거나, 레이아웃 계산이 아직 정리되지 않은 상황이라면 `setTimeout` 콜백이 기대한 타이밍과 다르게 실행될 수 있습니다.

그래서 DOM 측정이나 애니메이션처럼 렌더링 타이밍과 맞아야 하는 작업에는 `setTimeout`보다 `requestAnimationFrame`이 더 적합합니다.

---

## 🚀 requestAnimationFrame은 무엇이 다를까요?

`requestAnimationFrame`은 브라우저가 다음 프레임을 그리기 전에 콜백을 실행합니다.

```tsx
requestAnimationFrame(() => {
  measure();
});
```

흐름으로 보면 대략 이런 느낌입니다.

```txt
JavaScript 실행
↓
Style 계산
↓
Layout 계산
↓
requestAnimationFrame 실행
↓
Paint
↓
화면 출력
```

정확히 말하면 `requestAnimationFrame` 콜백은 다음 repaint 전에 실행됩니다. 그래서 브라우저 렌더링 주기와 맞춰 DOM 측정이나 애니메이션 값을 계산하기 좋습니다.

쉽게 기억하면 이렇게 볼 수 있습니다.

```txt
setTimeout
→ 시간이 지나면 실행

requestAnimationFrame
→ 다음 화면을 그리기 전에 실행
```

---

## 📏 DOM 측정에서 자주 사용하는 이유

태그 Overflow UI를 예로 들어보겠습니다.

태그가 실제로 몇 줄에 배치되었는지 계산하려면 각 태그의 `offsetTop`을 읽어야 합니다.

```tsx
const measure = () => {
  console.log(element.offsetTop);
};
```

처음에는 렌더링 직후 바로 측정하고 싶어집니다.

```tsx
useLayoutEffect(() => {
  measure();
}, []);
```

하지만 UI에 따라서는 DOM이 업데이트된 직후에도 브라우저의 레이아웃 계산과 우리가 측정하려는 시점이 미묘하게 어긋날 수 있습니다.

이럴 때 다음 프레임으로 측정을 미루면 더 안정적인 값을 얻을 수 있습니다.

```tsx
useLayoutEffect(() => {
  const frameId = requestAnimationFrame(() => {
    measure();
  });

  return () => {
    cancelAnimationFrame(frameId);
  };
}, []);
```

여기서 cleanup에서 `cancelAnimationFrame`을 호출하는 것도 중요합니다. 컴포넌트가 언마운트되었는데 예약된 측정이 나중에 실행되면 불필요한 작업이 될 수 있기 때문입니다.

---

## 🎞️ 애니메이션에서 사용하는 이유

`requestAnimationFrame`이 가장 많이 알려진 분야는 애니메이션입니다.

```tsx
let x = 0;

function animate() {
  x += 5;

  box.style.transform = `translateX(${x}px)`;

  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

이 코드는 매 프레임마다 `x` 값을 조금씩 증가시키고, 요소의 위치를 이동시킵니다.

브라우저가 보통 60Hz 환경에서 동작한다면 1초에 약 60번 화면을 갱신합니다.

```txt
1초 = 약 60프레임
1프레임 = 약 16.67ms
```

`requestAnimationFrame`은 이 화면 갱신 주기에 맞춰 실행되기 때문에 애니메이션이 더 자연스럽게 보입니다.

---

## 📉 setInterval로 애니메이션을 만들면 어떨까요?

물론 `setInterval`로도 애니메이션을 만들 수 있습니다.

```tsx
setInterval(() => {
  move();
}, 16);
```

하지만 이 방식에는 문제가 있습니다.

`setInterval`은 브라우저의 렌더링 상태와 상관없이 지정한 시간 간격으로 콜백을 실행하려고 합니다.

브라우저가 바쁘거나 탭이 비활성화된 상황에서는 이 간격이 정확히 지켜지지 않을 수 있습니다. 또 화면을 그릴 준비가 되지 않았는데 애니메이션 계산만 계속 실행될 수도 있습니다.

반면 `requestAnimationFrame`은 브라우저의 렌더링 주기에 맞춰 실행됩니다.

```txt
setInterval
→ 정해진 시간마다 실행 시도

requestAnimationFrame
→ 브라우저가 다음 프레임을 그릴 때 실행
```

그래서 화면에 실제로 반영되는 애니메이션에는 `requestAnimationFrame`이 더 잘 맞습니다.

---

## 🔋 성능 측면의 장점

`requestAnimationFrame`은 성능 측면에서도 장점이 있습니다.

브라우저는 현재 탭이 백그라운드로 이동하면 `requestAnimationFrame` 실행 빈도를 줄이거나 멈춥니다.

```txt
현재 탭
→ 프레임에 맞춰 실행

백그라운드 탭
→ 실행 빈도 감소 또는 중단
```

사용자가 보고 있지 않은 탭에서 애니메이션을 계속 계산할 필요가 없기 때문입니다.

이 덕분에 불필요한 CPU 사용과 배터리 소모를 줄일 수 있습니다.

---

## 🛑 cancelAnimationFrame으로 예약 취소하기

`requestAnimationFrame`은 콜백 실행을 예약합니다. 예약한 콜백을 취소하려면 `cancelAnimationFrame`을 사용합니다.

```tsx
const frameId = requestAnimationFrame(() => {
  measure();
});

cancelAnimationFrame(frameId);
```

React에서는 보통 effect cleanup에서 함께 사용합니다.

```tsx
useEffect(() => {
  const frameId = requestAnimationFrame(() => {
    measure();
  });

  return () => {
    cancelAnimationFrame(frameId);
  };
}, []);
```

이렇게 하면 다음 흐름이 됩니다.

```txt
컴포넌트 마운트
↓
측정 예약
↓
컴포넌트 언마운트
↓
예약 취소
```

특히 DOM을 참조하는 작업이라면 cleanup을 해두는 편이 안전합니다.

---

## 🛠️ 실무에서 자주 만나는 조합

### ResizeObserver와 함께 사용하기

`ResizeObserver`는 요소의 크기 변화를 감지하는 API입니다.

반응형 UI를 만들 때 컨테이너 너비가 바뀌면 DOM을 다시 측정해야 하는 경우가 많습니다. 이때 `ResizeObserver` 콜백에서 바로 측정하기보다 `requestAnimationFrame`으로 한 프레임 미루면 더 안정적으로 처리할 수 있습니다.

```tsx
const observer = new ResizeObserver(() => {
  requestAnimationFrame(measure);
});
```

이 조합은 다음과 같은 UI에서 자주 사용됩니다.

- 태그 Overflow 계산
- 반응형 레이아웃 측정
- 자동 높이 계산
- 차트나 캔버스 크기 재계산

태그 Overflow 글에서 사용했던 방식도 이 흐름에 가깝습니다.

```tsx
const observer = new ResizeObserver(scheduleMeasure);

const scheduleMeasure = () => {
  if (animationFrameId !== null) {
    cancelAnimationFrame(animationFrameId);
  }

  animationFrameId = requestAnimationFrame(measure);
};
```

크기 변화가 연속으로 발생하면 이전 예약을 취소하고 마지막 측정만 남기는 방식입니다.

### Motion / Framer Motion 같은 애니메이션 라이브러리

Framer Motion 같은 애니메이션 라이브러리도 내부적으로는 브라우저 프레임에 맞춰 값을 갱신합니다.

우리가 사용하는 코드는 단순해 보이지만,

```tsx
<motion.div animate={{ x: 100 }} />
```

내부에서는 프레임마다 값을 계산하고 화면에 반영하는 작업이 일어납니다.

직접 `requestAnimationFrame`을 매번 작성하지 않아도 되는 이유는 이런 복잡한 프레임 관리를 라이브러리가 대신 처리해주기 때문입니다.

---

## 📊 setTimeout vs requestAnimationFrame

두 API의 차이를 표로 정리하면 다음과 같습니다.

| 항목 | setTimeout | requestAnimationFrame |
| :--- | :--- | :--- |
| 실행 기준 | 시간 | 브라우저 렌더링 프레임 |
| 렌더링 동기화 | 직접 보장하지 않음 | 렌더링 주기에 맞춰 실행 |
| DOM 측정 | 타이밍이 어긋날 수 있음 | 더 안정적으로 측정 가능 |
| 애니메이션 | 부드럽지 않을 수 있음 | 프레임에 맞춰 부드럽게 실행 |
| 백그라운드 최적화 | 직접 관리 필요 | 브라우저가 실행 빈도를 줄임 |
| 대표 용도 | 지연 실행, 타이머 | 애니메이션, DOM 측정, 스크롤 보정 |

---

## ✅ 언제 사용하면 좋을까?

`requestAnimationFrame`은 다음 상황에서 특히 유용합니다.

- DOM 크기나 위치를 측정해야 할 때
- 스크롤 위치를 렌더링 타이밍에 맞춰 보정해야 할 때
- 부드러운 애니메이션을 직접 구현해야 할 때
- `ResizeObserver` 이후 레이아웃 측정을 안정적으로 하고 싶을 때
- 짧은 시간에 여러 번 발생하는 측정을 한 프레임 단위로 정리하고 싶을 때

반대로 단순히 몇 초 뒤에 실행해야 하는 작업이라면 `setTimeout`이 더 적합합니다.

```tsx
setTimeout(() => {
  console.log("3초 뒤 실행");
}, 3000);
```

즉, 기준은 이렇게 나눌 수 있습니다.

```txt
시간이 중요하다
→ setTimeout

렌더링 타이밍이 중요하다
→ requestAnimationFrame
```

---

## 🏁 마치며

처음에는 `requestAnimationFrame`을 단순히 애니메이션용 함수 정도로만 알고 있었습니다.

하지만 실제로 사용해보니 애니메이션뿐만 아니라 DOM 측정, `ResizeObserver` 연동, 스크롤 보정처럼 브라우저 렌더링과 맞물린 작업에서 자주 필요했습니다.

이제는 다음처럼 기억하면 좋을 것 같습니다.

```txt
setTimeout
→ 일정 시간이 지나면 실행

requestAnimationFrame
→ 다음 화면을 그리기 직전에 실행
```

DOM 크기를 측정하거나 애니메이션을 구현해야 한다면, 단순히 시간을 늦추는 방식보다 브라우저의 렌더링 흐름에 맞춰 실행하는 방식을 먼저 떠올려보면 좋겠습니다.
