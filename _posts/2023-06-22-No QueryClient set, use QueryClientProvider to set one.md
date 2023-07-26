---
title: "[Error] No QueryClient set, use QueryClientProvider to set one"
date: 2023-06-22T12:40:000
categories: [Error]
tags: [error] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray" class="h2">No QueryClient set, use QueryClientProvider to set one 발생</b>

아래 이미지와 같은 error 코드가 8줄 주르륵 발생했다

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/4087e19b-d638-4a16-ab51-6daff42d1b08"/>

처음 검색 결과에는 QueryClientProviderd안에 하나의 컴포넌트만 있지를 않아서 발생하기도 한다는데 나는 App 컴포너트 하나이기 떄문에 이 이유는 아니었고

```jsx
import { useQuery } from "react-query";

<QueryClientProvider client={client}>
  <App />
</QueryClientProvider>;
```

react-query가 V4가 나오며 Tanstack Query로 이름이 바뀌어서 'npm i @tanstack/react-query' 설치를 했었는데 아직도 import를 'react-query'로 했기 때문에 발생한 오류였다. 그래서 여기저기 잘못된 import를 '@tanstack/react-query' 변경해주었더니 Error가 해결되었다.

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
