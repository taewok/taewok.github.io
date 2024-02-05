---
title: "[React] react-intersection-observer 라이브러리 사용"
date: 2024-02-04T19:30:000
categories: [React]
tags: [React] #소문자만 가능
---

---

<p>프로젝트 진행중 무한스크롤 기능이 필요해서 구글링중 react-intersection-observer라는 라이브러리를 발견하여</p>

## <b>react-intersection-observer란?</b>

<p>React 애플리케이션에서 Intersection Observer API를 간편하게 사용할 수 있도록 도와주는 라이브러리로 웹 페이지에서 요소가 뷰포트에 들어오거나 나가는 등의 상태 변화를 비동기적으로 감지할 수 있는 기능을 제공하는 브라우저 API입니다.</p>

<h3><blockquote>설치
</blockquote></h3>

```js
npm install react-intersection-observer
```

<br/>

<h3><blockquote>Code
</blockquote></h3>

```jsx
import React from 'react';
import { useInView } from 'react-intersection-observer';

const MyComponent = () => {
  // useInView 훅을 사용하여 뷰포트에 들어온지 여부를 감지합니다.
  const [ref, inView] = useInView();

  return (
    <div ref={ref}>
      {inView ? '이 요소가 뷰포트에 들어왔습니다!' : '이 요소가 아직 뷰포트에 들어오지 않았습니다.'}
    </div>
  );
};

export default MyComponent;
```

- <strong>ref</strong>: 관찰할 대상 요소에 적용할 React ref로 이 ref를 요소에 연결함으로써 해당 요소의 변화를 감지한다.

- <strong>inView</strong>: 현재 요소의 상태를 나타내는 불리언(boolean) 값으로 만약 요소가 뷰포트에 들어오면 true가 되고, 뷰포트를 벗어나면 false가 된다.

<br/>

#### 💡 useInView 속성

- <strong>threshold (기본값: 0)</strong>: 요소가 뷰포트에 어느 정도 보이는지를 나타내는 값. 0은 일부만 보이면 트리거, 1은 완전히 보일 때 트리거됩니다.

- <strong>root (기본값: null)</strong>: 요소의 가시성을 판단할 때 사용되는 루트 엘리먼트. 기본값인 null은 브라우저의 뷰포트를 의미.

- <strong>rootMargin (기본값: '0px')</strong>: 루트 엘리먼트와 요소 간의 여백을 지정. 여러 단위로 지정 가능하며, CSS와 유사한 형식을 따름.

- <strong>triggerOnce (기본값: false)</strong>: true로 설정하면 요소가 한 번만 뷰포트에 들어왔을 때만 트리거되고, 그 이후에는 더 이상 감지하지 않음.

- <strong>skip (기본값: false)</strong>: true로 설정하면 Intersection Observer를 사용하지 않고 항상 inView 값이 true로 설정됨.

```js
  const [ref, inView] = useInView({
    threshold: 0.5,     // 50% 이상이 뷰포트에 들어왔을 때 트리거
    root: document.getElementById('customRoot'),  // 특정한 루트 엘리먼트로 지정
    rootMargin: '20px', // 루트 엘리먼트와 요소 간에 20px의 여백 사용
    triggerOnce: true,   // 한 번만 트리거하고 더 이상 감지하지 않음
  });
```

<p>다음과 같은 내용을 토대로 요소가 뷰포트에 보일때마다 상품 리스트를 추가로 불러와 무한스크롤을 구현하는 코드를 짜보았다.</p>

```jsx
const MyComponent = () => {
  const [ref, inView] = useInView();

  // 요소가 뷰포트에 보일 때 마다 실행될 함수
  const callback = () => {
    // 상품 리스트 가져오는 함수
    getProductList();
  };


  useEffect(() => {
    // 요소가 뷰포트에 감지될 때만 실행 가능하도록 if문 사용
    if (inView) {
      callback();
    }
  // 의존성 배열에 inView를 넣음으로써 요소가 뷰포트에 보일때 안보일때마다 실행
  }, [inView]);

  return <div ref={ref}></div>;
};

export default MyComponent;
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>