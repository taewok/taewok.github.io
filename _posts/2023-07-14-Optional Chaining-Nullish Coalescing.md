---
title: "[JavaScript] 안전한 코딩을 위한 필수 문법: ?. (Optional Chaining) & ?? (Nullish Coalescing)"
date: 2023-07-14T17:50:000
categories: [javascript]
tags: [javascript] #소문자만 가능
---

프론트엔드 개발을 하다 보면 API로 데이터를 받아오기 전, 값이 `null`이거나 `undefined`여서 화면이 깨지는 상황을 자주 마주하게 됩니다. 오늘은 방어 코드를 획기적으로 줄여주는 현대 자바스크립트의 핵심 문법 두 가지를 알아보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. ?. (Optional Chaining) 연산자</b>

`?.` 연산자는 좌항의 객체가 `null` 또는 `undefined`라면 에러를 발생시키는 대신 **undefined**를 즉시 반환합니다.

### 기본 문법

```javascript
const user = {
  name: "kim",
};

console.log(user?.age); // undefined (에러 발생 X)
```

### 실무 활용 (React/TypeScript)

데이터 통신으로 리스트를 받아올 때, 데이터가 아직 도착하지 않은 시점(null/undefined)에서의 런타임 에러를 방지합니다.

```tsx
// list가 없으면 에러 발생
{
  list.data.map((item) => <ListItem key={item.id} />);
}

// list가 있을 때만 map 실행
{
  list?.data.map((item) => <ListItem key={item.id} />);
}
```

---

## <b style="border-bottom:2px solid gray" class="h2">2. ?? (Nullish Coalescing) 연산자</b>

`??` 연산자는 왼쪽 피연산자가 **null** 또는 **undefined**일 때만 오른쪽 값을 반환합니다.

### 기본 문법

```javascript
const response = undefined;
console.log(response ?? "로딩 중..."); // "로딩 중..."
```

### 💡 OR(`||`) 연산자와의 결정적 차이

이 부분이 가장 중요합니다. `||` 연산자는 **Falsy** 값(`0`, `""`, `false`)을 모두 오른쪽 값으로 대체해버리는 문제가 있습니다.

```javascript
const count = 0;

// OR 연산자: 0을 유효한 값으로 처리하지 못함
console.log(count || 10); // 10

// Nullish 연산자: null/undefined만 체크하므로 0을 유지함
console.log(count ?? 10); // 0
```

따라서 **0이나 빈 문자열("")이 유효한 데이터**인 경우에는 반드시 `??`를 사용해야 합니다.

---

## 3. 요약 및 조합해서 사용하기

이 두 문법을 조합하면 데이터 바인딩 시 매우 강력한 힘을 발휘합니다.

```javascript
// 사용자 이름이 없으면 '익명'을 표시 (방어 코드 최적화)
const userName = apiResponse?.user?.name ?? "익명";
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>코드의 가독성을 높이고 런타임 에러를 줄여주는 <code>?.</code>와 <code>??</code>는 이제 선택이 아닌 필수입니다. 특히 API 연동이 잦은 프론트엔드 환경에서 적극적으로 활용하기 좋습니다.</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
