---
title: "[Error] styled-components props error 해결"
date: 2024-01-28T18:00:000
categories: [Error]
tags: [error] #소문자만 가능
---

---

<p>프로젝트 진행중 styled-components 컴포넌트에 prop을 전달하는 과정에서 다음과 같은 error가 발생했다.</p>

![image](https://github.com/taewok/taewok.github.io/assets/88264006/d1037059-c09a-45a4-9276-54c8606d5851)

<p>첫번재 error문구 같은 경우 스타일드 컴포넌트에서 전달된 스타일 속성 중에서 DOM에 직접 적용되지 않는 속성을 감지했을 때 발생된 것 입니다.</p>

<p>두번재 error문구 같은 경우는 React에서 DOM 요소로 전달되는 속성 중에서 backgroundColor라는 속성을 React가 인식하지 못하고 있다는 것을 나타냅니다</p>

<p>두 에러가 발생하는 이유는 공통적인 원인은
styled-components 컴포넌트에 전달하는 prop이 React의 DOM 요소까지 전달되어 DOM 요소 속성으로 처리되기 떄문입니다.</p>

<h3><blockquote>해결방법
</blockquote></h3>

<p>prop명 앞에 $을 붙이는겁니다. 이걸 transient props 방식 중 하나라고 하는데요. 일반적으로 렌더링된 결과에 직접 영향을 주지 않는 속성을 말합니다 그렇다는 것은 실제 HTML 요소로 전달되지 않는다는거죠</p>

```js
const MyComponent = styled.div`
  color: ${({ $backgroundColor }) => $backgroundColor || 'black'};
`;

// 사용 예시
<MyComponent $backgroundColor="red">Hello World</MyComponent>
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>