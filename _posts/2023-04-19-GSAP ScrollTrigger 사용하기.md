---
title: "[JavaScript] GSAP ScrollTrigger 기본 사용법"
date: 2023-04-19T14:12:00
categories: [javascript]
tags: [javascript, gsap, scrolltrigger, animation]
description: "GSAP ScrollTrigger의 기본 설정과 trigger, start, end, markers, toggleActions, scrub 사용법을 정리했습니다."
custom_style: true
---

## ScrollTrigger란?

ScrollTrigger는 GSAP에서 제공하는 플러그인입니다.

이름 그대로 스크롤 위치에 따라 애니메이션을 실행할 수 있게 도와줍니다.

예를 들어 사용자가 특정 섹션까지 스크롤했을 때 요소가 나타나게 하거나, 스크롤 진행 정도에 맞춰 이미지가 움직이게 만들 수 있습니다.

이번 글에서는 ScrollTrigger를 사용할 때 자주 만나는 기본 옵션들을 정리해볼게요.

---

## 설치와 기본 설정

먼저 GSAP을 설치합니다.

```bash
npm install gsap
```

그리고 ScrollTrigger를 함께 가져온 뒤 등록합니다.

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);
```

`registerPlugin`을 하지 않으면 ScrollTrigger 옵션이 제대로 동작하지 않을 수 있습니다.

---

## 기본 사용 예시

```js
gsap.to(".box", {
  x: 300,
  duration: 1,
  scrollTrigger: {
    trigger: ".box",
    start: "top center",
    end: "bottom top",
    markers: true,
  },
});
```

이 코드는 `.box` 요소가 특정 스크롤 위치에 도달했을 때 오른쪽으로 이동하는 애니메이션을 실행합니다.

`scrollTrigger` 객체 안에 스크롤 기준과 동작 옵션을 작성합니다.

---

## trigger

`trigger`는 애니메이션을 실행할 기준 요소입니다.

```js
gsap.to(".box", {
  x: 300,
  scrollTrigger: {
    trigger: ".section",
  },
});
```

위 코드에서는 `.section`이 스크롤 기준이 됩니다.

애니메이션 대상은 `.box`이고, 스크롤을 감지하는 기준은 `.section`인 구조입니다.

대상 요소와 trigger가 항상 같을 필요는 없습니다.

---

## start와 end

`start`와 `end`는 애니메이션이 언제 시작하고 끝날지 정하는 옵션입니다.

```js
scrollTrigger: {
  trigger: ".box",
  start: "top center",
  end: "bottom top",
}
```

`start: "top center"`는 trigger 요소의 top이 viewport의 center에 닿을 때 시작한다는 뜻입니다.

`end: "bottom top"`은 trigger 요소의 bottom이 viewport의 top에 닿을 때 끝난다는 뜻입니다.

처음에는 조금 낯설 수 있지만, 다음처럼 나누어 보면 이해하기 쉽습니다.

```txt
"trigger의 위치 viewport의 위치"
```

예를 들어 `"top center"`는 trigger의 위쪽이 화면 가운데에 닿는 순간을 의미합니다.

---

## markers

`markers`는 ScrollTrigger의 시작 지점과 끝 지점을 화면에 표시해주는 디버깅 옵션입니다.

```js
scrollTrigger: {
  trigger: ".box",
  start: "top center",
  end: "bottom top",
  markers: true,
}
```

처음 ScrollTrigger를 사용할 때는 `markers: true`를 켜두는 것이 좋습니다.

화면에 start와 end 위치가 표시되기 때문에, 내가 생각한 시점과 실제 동작 시점이 같은지 확인하기 쉽습니다.

작업이 끝난 뒤에는 `markers`를 제거하거나 `false`로 바꿔주면 됩니다.

---

## toggleActions

`toggleActions`는 스크롤이 특정 지점을 지나갈 때 애니메이션을 어떻게 제어할지 정하는 옵션입니다.

```js
scrollTrigger: {
  trigger: ".box",
  start: "top center",
  end: "bottom top",
  toggleActions: "play pause resume reverse",
}
```

문자열은 네 개의 동작으로 이루어져 있습니다.

```txt
onEnter onLeave onEnterBack onLeaveBack
```

각 위치의 의미는 다음과 같습니다.

| 위치 | 의미 |
| --- | --- |
| `onEnter` | start 지점에 들어올 때 |
| `onLeave` | end 지점을 벗어날 때 |
| `onEnterBack` | 아래에서 위로 스크롤하며 end 지점에 다시 들어올 때 |
| `onLeaveBack` | 아래에서 위로 스크롤하며 start 지점을 벗어날 때 |

사용할 수 있는 대표 동작은 다음과 같습니다.

| 동작 | 설명 |
| --- | --- |
| `play` | 재생 |
| `pause` | 일시 정지 |
| `resume` | 다시 재생 |
| `reverse` | 반대로 재생 |
| `restart` | 처음부터 다시 재생 |
| `reset` | 처음 상태로 되돌림 |
| `complete` | 끝 상태로 이동 |
| `none` | 아무 동작도 하지 않음 |

단순히 한 번 재생만 하고 싶다면 아래처럼 사용할 수 있습니다.

```js
toggleActions: "play none none none";
```

---

## scrub

`scrub`은 스크롤 진행 정도와 애니메이션 진행 정도를 연결해주는 옵션입니다.

```js
gsap.to(".box", {
  x: 500,
  scrollTrigger: {
    trigger: ".box",
    start: "top center",
    end: "bottom top",
    scrub: true,
  },
});
```

`scrub: true`를 사용하면 스크롤을 내리는 만큼 애니메이션이 진행되고, 다시 올리면 애니메이션도 되돌아갑니다.

스크롤과 함께 자연스럽게 움직이는 인터랙션을 만들 때 많이 사용합니다.

숫자를 넣을 수도 있습니다.

```js
scrub: 1;
```

이렇게 쓰면 스크롤을 따라가되 약간 부드럽게 지연되며 움직입니다.

---

## 자주 사용하는 패턴

섹션이 화면에 들어올 때 카드가 위로 올라오며 나타나는 예시입니다.

```js
gsap.from(".card", {
  y: 50,
  opacity: 0,
  duration: 0.8,
  stagger: 0.1,
  scrollTrigger: {
    trigger: ".card-list",
    start: "top 70%",
    toggleActions: "play none none none",
  },
});
```

`stagger`를 사용하면 여러 카드가 조금씩 시간 차이를 두고 등장합니다.

이런 패턴은 포트폴리오, 랜딩 페이지, 소개 섹션에서 자주 사용됩니다.

---

## 마무리

ScrollTrigger를 사용할 때는 먼저 `trigger`, `start`, `end`를 이해하는 것이 중요합니다.

처음에는 `markers: true`를 켜두고 기준점이 어디에 잡히는지 눈으로 확인하면 훨씬 쉽게 감을 잡을 수 있습니다.

스크롤 위치에서 한 번 실행되는 애니메이션은 `toggleActions`로 제어하고, 스크롤 진행과 애니메이션을 연결하고 싶다면 `scrub`을 사용하면 됩니다.

기본 옵션만 익혀도 스크롤 기반 인터랙션을 꽤 다양하게 만들 수 있습니다.
