---
title: "[Error] typeScript에서 ~~ is not a function"
date: 2023-06-24T18:13:000
categories: [Error]
tags: [error] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray" class="h2"> ~~ is not a function 발생</b>

부모 컴포넌트에서 생성한 useState를 자식 컴포넌트로 넘기려는데 state는 잘 전달됬지만 setState에서 아래와 같은 error가 발생했다....

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/060d7248-1ee8-479a-bb35-106f3c763d26"/>

평소 같았다면 문제 없었을 코드인데.... 달라진건 이번엔 TypeScript를 쓴다는 것

---

## <b style="border-bottom:2px solid gray" class="h2"> ~~ is not a function 해결</b>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
기존 코드
</blockquote>

```tsx
const StarClick = (count: any, setCount: any) => {
  const starOnChange = (e: any) => {
    const num = e.target.defaultValue;
    setCount(num);
  };

  return (
    <div>
      <input value={count} onChange={(e) => starOnChange(e)} />
    </div>
  );
};

export default StarClick;
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
해결한 코드
</blockquote>

이렇게 interface를 생성해 type 정의를 해주니 해결되었다.

```tsx
interface childProps {
  count: number;
  setCount: React.Dispatch<React.SetStateAction<number>>;
}

const StarClick = ({ count, setCount }: childProps) => {
  const starOnChange = (e: any) => {
    const num = e.target.defaultValue;
    setCount(num);
  };

  return (
    <div>
      <input value={count} onChange={(e) => starOnChange(e)} />
    </div>
  );
};

export default StarClick;
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
