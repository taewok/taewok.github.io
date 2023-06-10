---
title: "[React] JWT token에 담긴 내용 읽기"
date: 2023-06-03T17:38:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">jwt-decode 사용상황</b>

로그인이 정상적으로 성공했을때 백엔드에서 jwt token 즉 <b style="color:#ff526f">accessToken</b>을 넘겨주는데<br/> 그안에 user에 nickname이나 권한 알려주고 싶은 정보를 담은 부분을 <b style="color:#ff526f">payload</b>라고 한다.<br/> 하지만 암호화 해서 보내주기 때문에 알아볼 수 있게 해독하는 것을 <b style="color:#ff526f">decoding</b>이라고 하며 그걸 위해 jwt-decode 라이브러리를 사용한다.

---

## <b style="border-bottom:2px solid gray">jwt-decode 설치</b>

```js
npm install jwt-decode
```

---

## <b style="border-bottom:2px solid gray">jwt-decode 사용</b>

1. 우선은 jwt-decode를 import 해준다
2. import한 jwt_decode 함수에 매개변수로 <b style="color:#ff526f">accessToken</b>을 넣어준다.

```js
import jwt_decode from "jwt-decode";

const loginBtnOnClick = async (email, pw) => {
  const res = await api.post(`/api/login `, {
    email: email,
    password: pw,
  });

  //건네 받은 accessToken 추출
  const token = res.headers.authorization.split(" ")[1];
  //decoding 시작
  const jwt = jwt_decode(token);
  console.log(jwt);
  //data: 
  //   email: "email@email.com";
  //   memberId: 1;
  //   name: "고객님";
  //   username: "test1";
};
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
