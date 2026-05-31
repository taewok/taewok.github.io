---
title: "[JavaScript] lodash throttle로 이벤트 호출 횟수 제한하기"
date: 2025-09-19 17:45:00 +0900
categories: [javascript, performance]
tags: [javascript, lodash, throttle, event]
description: "lodash throttle을 사용해 scroll, resize처럼 자주 발생하는 이벤트의 실행 횟수를 제한하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

프론트엔드 개발을 하다 보면 scroll, resize, mousemove처럼 짧은 시간 안에 아주 많이 발생하는 이벤트를 다룰 때가 있습니다.

이때 이벤트가 발생할 때마다 무거운 로직을 실행하면 성능 문제가 생길 수 있습니다.

이런 상황에서 사용할 수 있는 도구가 `throttle`입니다.

---

## throttle이란?

throttle은 일정 시간 동안 함수가 너무 자주 실행되지 않도록 제한하는 기법입니다.

예를 들어 1초에 한 번만 실행되도록 설정하면, 이벤트가 수십 번 발생해도 함수는 최대 1초에 한 번만 실행됩니다.

스크롤 위치 추적이나 브라우저 크기 감지처럼 주기적으로 실행되면 충분한 작업에 잘 어울립니다.

---

## lodash 설치하기

```bash
npm install lodash
```

또는 yarn을 사용한다면 다음 명령어로 설치할 수 있습니다.

```bash
yarn add lodash
```

---

## 기본 사용법

```js
import { throttle } from "lodash";

const logScroll = () => {
  console.log("scroll event", window.scrollY);
};

const throttledLogScroll = throttle(logScroll, 1000);

window.addEventListener("scroll", throttledLogScroll);
```

위 코드는 스크롤 이벤트가 계속 발생해도 `logScroll`이 1초에 한 번 정도만 실행되게 만듭니다.

---

## 옵션 사용하기

lodash throttle은 실행 시점을 제어할 수 있는 옵션을 제공합니다.

```js
const throttledLog = throttle(logScroll, 1000, {
  leading: true,
  trailing: true,
});
```

`leading`은 첫 호출 시 바로 실행할지 정합니다.

`trailing`은 마지막 호출 이후 한 번 더 실행할지 정합니다.

기본값은 둘 다 `true`입니다.

---

## React에서 사용하기

React에서는 이벤트 리스너 정리를 함께 해주는 것이 중요합니다.

```tsx
import { useEffect, useMemo } from "react";
import { throttle } from "lodash";

const ScrollTracker = () => {
  const handleScroll = useMemo(
    () =>
      throttle(() => {
        console.log("스크롤 위치:", window.scrollY);
      }, 1000),
    [],
  );

  useEffect(() => {
    window.addEventListener("scroll", handleScroll);

    return () => {
      window.removeEventListener("scroll", handleScroll);
      handleScroll.cancel();
    };
  }, [handleScroll]);

  return <div style={{ height: "200vh" }}>스크롤 테스트</div>;
};

export default ScrollTracker;
```

cleanup에서 이벤트 리스너를 제거하고 `cancel()`도 호출하면 예약된 trailing 실행까지 정리할 수 있습니다.

---

## debounce와의 차이

throttle과 debounce는 자주 비교됩니다.

| 구분 | throttle | debounce |
| --- | --- | --- |
| 실행 방식 | 일정 시간마다 실행 | 이벤트가 멈춘 뒤 실행 |
| 어울리는 상황 | scroll, resize, mousemove | 검색 입력, 자동 저장, 유효성 검사 |

스크롤 중 계속 위치를 확인해야 한다면 throttle이 어울립니다.

입력이 끝난 뒤 한 번만 실행하고 싶다면 debounce가 어울립니다.

---

## 마무리

throttle은 너무 자주 발생하는 이벤트의 실행 횟수를 일정 시간 단위로 제한해줍니다.

스크롤, 리사이즈, 마우스 이동처럼 연속적으로 발생하는 이벤트에 사용하면 성능 부담을 줄일 수 있습니다.

React에서 사용할 때는 이벤트 제거와 `cancel()` 호출까지 함께 정리해두면 더 안전합니다.
