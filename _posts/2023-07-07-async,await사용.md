---
title: "[JavaScript] async,await 사용하기"
date: 2023-07-07T16:18:000
categories: [JavaScript]
tags: [javaScript] #소문자만 가능
---

<style type="text/css">
    .center{
        text-align:center;
        font-size:1.5rem;
    }
    blockquote{
        font-size:1.2rem;
    }
    blockquote>h3{
        margin:0;
    }
</style>

---

## <b style="border-bottom:2px solid gray" class="h2">async & await이란?</b>

자바스크립트는 비동기 처리가 기반인 언어이다.<br/>
async와 await는 JavaScript에서 비동기 프로그래밍을 할 때 사용되는 키워드이다. 이 키워드를 사용하면 비동기 작업을 동기적으로 처리할 수 있어 코드의 가독성과 유지보수성을 높일 수 있습니다.


<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">async</blockquote></h3>

async 키워드는 함수를 비동기 함수로 정의할 때 사용됩니다. <br/>
이 함수는 항상 Promise 객체를 반환하며, 함수 내부에서 비동기 작업을 수행할 수 있다. <br/>

```js
const fn = async () => {
  return 1;
};

const result = fn();
console.log(result); //Promise{1}
```

실제 결과를 얻기 위해서는 Promise의 메서드인 then이나 await를 사용하여 비동기 작업의 완료를 기다려야 한다.

<h3><blockquote style="color:black; padding: 0.5rem 1rem; border-left: 5px solid #5cc55b;">await</blockquote></h3>

await 키워드는 Promise 객체가 처리될 때까지 함수의 실행을 일시 중지할 때 사용되며.
await을 사용하려면 async함수 안에서 사용해야한다.

```js
const f = async () => {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("완료!"), 1000);
  });

  let result = await promise; // 프라미스가 이행될 때까지 기다림 (*)

  console.log(result); //완료!
  //await이 없었다면 Promise {<pending>}을 반환
};

f();
```

## <b style="border-bottom:2px solid gray" class="h2">async & await 기본 사용법</b>

```js
const Name = async () => {
 const res = await axios.get("https://toOnline.com/api");
 # 비동기 작업 완료 후 처리할 로직
}

```

먼저 함수의 앞에 async 라는 예약어를 붙입니다. 그러고 통신을 하는 비동기 처리 코드 앞에 await를 붙입니다. 여기서 주의 할 점은 비동기 처리 메서드가 꼭 프로미스 객체를 반환해야한다.

일반적으로 await의 대상이 되는 비동기 처리 코드는 Axios 등 프로미스를 반환하는 API 호출 함수이다.

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>