---
title: "[Error] You provided a value prop to a form field without an onChange handler"
date: 2023-06-26T18:18:000
categories: [Error]
tags: [error] #소문자만 가능
---

<style type="text/css">
    .center{
        text-align:center;
        font-size:1.5rem;
    }
    blockquote{
        font-size:1.1rem;
    }
</style>

---

## <b style="border-bottom:2px solid gray" class="h2">Warning: You provided a value prop to a form field without an onChange handler. 발생</b>

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/0559ffa5-d06a-450b-ad2b-485cf6452dc2"/>

## <b style="border-bottom:2px solid gray" class="h2">해결</b>

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
기존 코드
</blockquote>

```tsx
import React, { useEffect, useState } from "react";

const Info = () => {
  const [id, setId] = useState<string>("");

  return (
    <div>
      <input value={id} placeholder="아이디를 입력해주세요" />
    </div>
  );
};

export default Info;
```

<blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">
수정 코드
</blockquote>

```tsx
import React, { useEffect, useState } from "react";

const Info = () => {
  const [id, setId] = useState<string>("");

  return (
    <div>
      <input defaultValue={id} placeholder="아이디를 입력해주세요" />
    </div>
  );
};

export default Info;
```

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>