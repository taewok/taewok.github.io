---
title: "[TypeScropt] 타입스크립트 styled-components props 사용하기"
date: 2023-03-16T17:01:000
categories: [TypeScropt]
tags: [typeScropt] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">styled-components 설치</b>
```js
npm i styled-components
```

***

## <b style="border-bottom:2px solid gray">타입스크립트일 경우 추가 설치</b>
```js
npm i -D @types/styled-components
```

***

## <b style="border-bottom:2px solid gray">props 전달</b>
- props로 animation 실행 딜레이를 다르게 준다.

```tsx
const Answer = () => {
  return (
    <Input delay={100}/>
    <Input delay={300}/>
    <Input delay={500}/>
  );
};
export default Answer;
```

***

1. <b>단일 props 사용시</b>
```tsx
const Input = styled.input<{ delay: number }>`
    animation-delay: ${(props) => props.delay}ms;
`;
```

2. <b>다수 props 사용시</b>
```tsx
interface Container{
  delay:number;
  check:boolean;
}
const Input = styled.input<Container>`
    ${(props)=>props.check&&css`
        animation-delay: ${(props) => props.delay}ms;
    `}
`;
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>
<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
