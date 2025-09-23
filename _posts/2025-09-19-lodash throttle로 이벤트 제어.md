---
title: "[JavaScript] lodash throttle로 이벤트 호출 횟수 제한하기"
date: 2025-09-19 17:45:00 +0900
categories: [JavaScript, Performance]
tags: [javascript, lodash, throttle, event]
---

프론트엔드 개발을 하다 보면 스크롤 이벤트, 윈도우 리사이즈 이벤트처럼 `짧은 시간에 너무 많이 발생하는 이벤트`를 다뤄야 하는 경우가 많은데요.
이때 이벤트 핸들러가 매번 실행되면 성능에 큰 문제가 생길 수 있습니다 😅

이럴 때 유용한 도구가 바로 lodash의 **throttle** 함수예요.  
이번 글에서는 **throttle**을 사용해서 이벤트 호출 횟수를 일정 시간 안에서 제한하는 방법을 정리해볼게요!

---

### 📦 lodash 설치

```bash
npm install lodash
# 또는
yarn add lodash
```

lodash는 JavaScript에서 자주 쓰이는 유틸리티 라이브러리로, debounce, cloneDeep, isEqual 등 다양한 기능이 들어있습니다.

<br/> 
<h3><b>⚡ throttle 기본 사용법</b></h3>

```jsx
import { throttle } from "lodash";

const log = () => {
  console.log("scroll event!", new Date().toLocaleTimeString());
};

const throttledLog = throttle(log, 2000); // 2초에 한 번만 실행

window.addEventListener("scroll", throttledLog);
```

<ul> <li><code>throttle(func, wait)</code>: 지정한 <code>wait</code> 밀리초 동안 함수 실행을 제한</li> 
<li><code>log</code>: 원래 실행하려던 함수</li> <li><code>throttledLog</code>: throttle로 감싼 새로운 함수</li> </ul>

위 코드에서는 스크롤 이벤트가 수십 번 발생하더라도 2초에 한 번만 log 함수가 실행됩니다 🚀

<br/> 
<h3><b>🛠️ 옵션 사용하기</b></h3>

lodash throttle은 실행 시점을 제어할 수 있는 옵션을 제공합니다.

```jsx
const throttledLog = throttle(log, 2000, {
  leading: true, // 처음 호출 시 실행 여부 (기본값 true)
  trailing: false, // 마지막 호출 후 실행 여부 (기본값 true)
});
```

- leading: true → 이벤트가 시작되자마자 실행
- trailing: true → 마지막 이벤트가 끝난 후 실행
- 필요에 따라 true/false를 조합해서 원하는 타이밍으로 제어할 수 있습니다.

<br/> 
<h3><b>🖼️ React에서 활용하기</b></h3>

React 컴포넌트에서 throttle을 쓸 때는 useCallback과 함께 쓰는 게 좋아요.

```jsx
import { useEffect, useCallback } from "react";
import { throttle } from "lodash";

export default function ScrollTracker() {
  const handleScroll = useCallback(
    throttle(() => {
      console.log("스크롤 위치:", window.scrollY);
    }, 1000),
    []
  );

  useEffect(() => {
    window.addEventListener("scroll", handleScroll);
    return () => window.removeEventListener("scroll", handleScroll);
  }, [handleScroll]);

  return <div style={{ height: "200vh" }}>스크롤 테스트 🚀</div>;
}
```

이렇게 하면 스크롤할 때마다 로그가 찍히지 않고, 1초에 한 번만 실행되어 성능을 최적화할 수 있습니다 👍

<br/> 
<h3><b>📝 사용 후기</b></h3>

throttle을 쓰면서 가장 좋았던 점은, 필요 이상으로 이벤트 핸들러가 실행되는 걸 막아 성능 저하를 예방할 수 있다는 거였어요.
특히 스크롤 기반 애니메이션, 무한 스크롤, 윈도우 리사이즈 감지 같은 곳에서 효과적입니다.
debounce와 헷갈리기 쉬운데, throttle은 "주기적으로 실행", debounce는 "마지막에만 실행"이라는 차이를 잘 이해해 두면 유용하게 쓸 수 있습니다.
