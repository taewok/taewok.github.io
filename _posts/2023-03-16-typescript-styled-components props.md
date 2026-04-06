---
title: "[TypeScript] styled-components에서 Props 타입 지정하기"
date: 2023-03-16T17:01:000
categories: [typescript]
tags: [typescript] #소문자만 가능
---

리액트에서 스타일을 입힐 때 자주 사용하는 **styled-components**를 타입스크립트와 함께 사용하면, Props를 통해 전달되는 값에 타입을 엄격하게 지정하여 런타임 에러를 방지할 수 있습니다.

오늘은 설치 방법부터 단일/다중 Props 타입 지정 방식까지 정리해 보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. 환경 설정</b>

타입스크립트 프로젝트에서 styled-components를 사용하려면 라이브러리 본체와 함께 타입 정의 파일(`@types`)을 별도로 설치해야 합니다.

```bash
# 라이브러리 설치
npm i styled-components

# 타입 정의 파일 설치 (개발 의존성)
npm i -D @types/styled-components
```

---

## <b style="border-bottom:2px solid gray" class="h2">2. Props 전달하기</b>

컴포넌트에서 스타일 컴포넌트로 동적인 값을 전달하는 상황입니다. 예를 들어, 애니메이션 지연 시간(`delay`)을 각각 다르게 주고 싶을 때 다음과 같이 작성합니다.

```tsx
const Answer = () => {
  return (
    <>
      <Input delay={100} />
      <Input delay={300} />
      <Input delay={500} />
    </>
  );
};

export default Answer;
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. 스타일 컴포넌트에서 타입 정의하기</b>

스타일 정의부에서 Generic(`<>`)을 사용하여 Props의 타입을 선언합니다.

### ① 단일 Props 사용 시

전달할 속성이 하나라면 태그 옆에 바로 타입을 지정할 수 있습니다.

```tsx
const Input = styled.input<{ delay: number }>`
  animation-delay: ${(props) => props.delay}ms;
`;
```

### ② 다수 Props 사용 시 (Interface 권장)

전달해야 할 속성이 많아지면 `interface`를 별도로 선언하여 관리하는 것이 가독성 면에서 훨씬 유리합니다. `css` 블록을 활용하면 조건부 스타일링도 깔끔하게 처리할 수 있습니다.

```tsx
import styled, { css } from "styled-components";

// Props 타입 정의
interface InputProps {
  delay: number;
  check: boolean;
}

const Input = styled.input<InputProps>`
  width: 100px;
  height: 40px;

  // check가 true일 때만 animation-delay 적용
  ${(props) =>
    props.check &&
    css`
      animation-delay: ${props.delay}ms;
    `}
`;
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>타입스크립트와 styled-components를 조합하면 어떤 Props가 필요한지 코드를 작성하는 단계에서 바로 알 수 있어 개발 생산성이 크게 향상됩니다. 특히 <code>interface</code>를 활용해 복잡한 스타일 로직을 안전하게 관리해 보세요!</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
