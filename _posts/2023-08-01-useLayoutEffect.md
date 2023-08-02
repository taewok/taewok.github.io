---
title: "[React] useLayoutEffect란?"
date: 2023-08-01T15:19:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b>useLayoutEffect란?</b>

<h3><blockquote>정의
</blockquote></h3>

useLayoutEffect는 React 컴포넌트에서 사이드 이펙트를 수행하는 데 사용되는 훅입니다.

- ※ <b>사이드 이펙트</b>: 함수나 메서드가 실행될 때 해당 함수나 메서드의 외부에 영향을 주는 모든 작업을 말합니다

<h3><blockquote>사용상황
</blockquote></h3>

- DOM 요소의 측정과 레이아웃 조정: 컴포넌트가 렌더링된 후에 DOM 요소의 크기나 위치를 측정해야 하는 경우에 유용합니다.
- 수동으로 DOM 조작: 특정 DOM 요소를 직접 조작하거나 스크롤 위치를 설정하거나 포커스를 설정하는 등의 작업을 수행할 수 있습니다.
- CSS 애니메이션 또는 트랜지션 설정: 렌더링 직후에 애니메이션이 시작되도록 할 수 있습니다.
- 외부 데이터 요청 후 DOM 조작: 데이터를 가져온 후에 해당 데이터를 기반으로 DOM을 조작해야 하는 경우 useLayoutEffect를 사용하여 데이터를 가져온 후 즉시 DOM을 업데이트할 수 있습니다.

## <b>useLayoutEffect 사용하기</b>

사용법은 useEffect와 유사합니다.

```js
console.log(document.body.clientHeight); // 0
useLayoutEffect(() => {
  const bodyElement = document.body;
  const height = bodyElement.clientHeight;
  console.log(height); // 컴포넌트가 로드된 후의 body 요소의 높이
}, [/* 의존성 배열 */]);
```

--- 

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
