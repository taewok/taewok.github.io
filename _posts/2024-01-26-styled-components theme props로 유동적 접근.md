---
title: "[React] Styled-Components theme 객체 속성 prop을 통해 접근"
date: 2024-01-28T16:40:000
categories: [React, JavaScript]
tags: [react, javascript] #소문자만 가능
---

---

<p>프로젝트 진행중 버튼 컴포넌트를 따로 만들어 재사용성을 높이는 중 props을 통해 theme객체에 속성을 유동적으로 바꾸는 방법을 알아내었다.</p>

<h3><blockquote>Button.tsx
</blockquote></h3>

```tsx
import React, { ReactNode } from 'react';
import styled from 'styled-components';

interface ButtonProps {
  children?: ReactNode;
  font: string;
  color: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  color,
  font,
}) => {
  return (
    <ColoredBtn
      style={style}
      $font={font}
      $color={color}
    >
      {children}
    </ColoredBtn>
  );
};

const ColoredBtn = styled.button<{
  $font: string;
  $color: string;
  $backgroundColor: string;
}>`
  padding: 10px 30px;
  width: 100%;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  /* font 변경 부분 */
  // 대괄호 표기법을 사용하여 $font prop에 해당하는 속성명에 접근
  font: ${(props) =>
    ({ theme }) =>
      theme.fontSizes[props.$font]};

  /* color 변경 부분 */
  // if문 사용으로 $color prop에 해당 문자열이 들어있는지 판단 각 theme 속성명 구분 대괄호 표기법으로 $color prop에 해당하는 속성명에 접근
  color: ${(props) => {
    if (props.$color.includes('Grey'))
      return ({ theme }) => theme.grey[props.$color];
    else if (props.$color.includes('Orange'))
      return ({ theme }) => theme.mainColor[props.$color];
    else if (props.$color === 'black') return 'black';
    else return 'white';
  }};
`;

export default ColoredButton;

```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>