---
title: "[Error] value prop without onChange handler 경고 해결하기"
date: 2023-06-26T18:18:00
categories: [error]
tags: [error, react, input, controlled-component]
description: "React input에서 value만 전달하고 onChange를 작성하지 않았을 때 발생하는 경고의 원인과 해결 방법을 정리했습니다."
custom_style: true
---

## 발생한 경고

React에서 input을 만들다가 다음 경고를 만날 때가 있습니다.

```txt
You provided a `value` prop to a form field without an `onChange` handler.
```

이 경고는 input에 `value`는 넣었는데, 값을 변경할 수 있는 `onChange`가 없을 때 발생합니다.

---

## 문제가 된 코드

```tsx
import { useState } from "react";

const Info = () => {
  const [id, setId] = useState("");

  return (
    <input
      value={id}
      placeholder="아이디를 입력해주세요"
    />
  );
};

export default Info;
```

`value={id}`를 넣는 순간 이 input은 React state가 값을 제어하는 controlled component가 됩니다.

그런데 `onChange`가 없기 때문에 사용자가 입력해도 state를 바꿀 방법이 없습니다.

그래서 React는 읽기 전용 input처럼 동작할 수 있다고 경고를 보여줍니다.

---

## 해결 방법 1. onChange 추가하기

사용자가 입력한 값을 state에 저장해야 한다면 `onChange`를 추가하면 됩니다.

```tsx
import { useState } from "react";

const Info = () => {
  const [id, setId] = useState("");

  return (
    <input
      value={id}
      onChange={(event) => setId(event.target.value)}
      placeholder="아이디를 입력해주세요"
    />
  );
};

export default Info;
```

이제 input 값이 바뀔 때마다 `setId`가 호출되고, state와 input 값이 함께 업데이트됩니다.

---

## 해결 방법 2. defaultValue 사용하기

값을 React state로 계속 관리할 필요가 없다면 `defaultValue`를 사용할 수 있습니다.

```tsx
const Info = () => {
  return (
    <input
      defaultValue=""
      placeholder="아이디를 입력해주세요"
    />
  );
};

export default Info;
```

`defaultValue`는 초기값만 지정합니다.

그 이후 입력값은 브라우저 DOM이 직접 관리합니다.

---

## value와 defaultValue 차이

| 속성 | 의미 |
| --- | --- |
| `value` | React state가 값을 제어함 |
| `defaultValue` | 처음 값만 지정하고 이후에는 DOM이 관리함 |

입력값을 검증하거나, 버튼 활성화 조건에 사용하거나, 서버로 보낼 데이터로 관리해야 한다면 `value`와 `onChange`를 함께 사용하는 것이 좋습니다.

단순 초기값만 필요하다면 `defaultValue`가 더 간단합니다.

---

## 마무리

이 경고는 React가 input의 제어 방식을 명확히 하라고 알려주는 신호입니다.

`value`를 사용한다면 `onChange`도 함께 작성해야 합니다.

반대로 입력값을 React state로 관리하지 않을 거라면 `defaultValue`를 사용하면 됩니다.
