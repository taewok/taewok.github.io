---
title: "[HTML/JS] 브라우저 저장소 정리: localStorage vs sessionStorage 차이점"
date: 2023-03-02T15:31:00
categories: [hTML, javascript]
tags: [html, javascript, web-storage, localstorage, sessionstorage]
description: "브라우저에 데이터를 저장하는 Web Storage API인 localStorage와 sessionStorage의 개념, 사용법(CRUD), 그리고 두 저장소의 결정적인 차이점을 정리합니다."
custom_style: true
---

HTML5 이전에는 브라우저에 데이터를 저장하기 위해 주로 쿠키(Cookie)를 사용했지만, HTML5부터는 더 직관적이고 용량이 큰 **웹 스토리지(Web Storage)**가 등장했습니다.

웹 스토리지는 데이터의 지속성에 따라 **`localStorage`**와 **`sessionStorage`** 두 가지로 나뉩니다. 이 둘의 차이점과 사용법을 완벽하게 정리해 보겠습니다. 💾

---

## 1. localStorage (로컬 스토리지)

- **특징:** 브라우저를 종료하거나 탭을 닫아도 **데이터가 삭제되지 않습니다 (영구 저장).**
- **용도:** 자동 로그인 여부, 테마 설정(다크 모드), 장바구니 데이터 등 지워지면 안 되는 데이터 저장.
- **삭제:** 사용자가 직접 브라우저 캐시를 지우거나, 코드로 삭제(`removeItem`, `clear`)해야만 지워집니다.

### 사용법 (CRUD)

데이터를 저장하고 가져오는 방법은 매우 간단합니다. (권장되는 `setItem`, `getItem` 메서드를 사용하세요.)

```javascript
// 1. 저장하기 (Create/Update)
// localStorage.setItem("key", "value");
localStorage.setItem("username", "오윤희");

// 2. 가져오기 (Read)
// const value = localStorage.getItem("key");
const name = localStorage.getItem("username");
console.log(name); // "오윤희"

// 3. 특정 데이터 삭제하기 (Delete)
// localStorage.removeItem("key");
localStorage.removeItem("username");

// 4. 모든 데이터 전체 삭제하기 (Clear)
localStorage.clear();
```

> **⭐ 주의할 점:** 웹 스토리지는 오직 **문자열(String)**만 저장할 수 있습니다. 객체나 배열을 저장할 때는 `JSON.stringify()`를, 가져올 때는 `JSON.parse()`를 사용해야 합니다.

---

## 2. sessionStorage (세션 스토리지)

- **특징:** 브라우저의 탭이나 창을 닫으면 **데이터가 즉시 삭제됩니다 (휘발성 저장).**
- **용도:** 일회성 로그인 정보, 입력 폼 복구 데이터, 비로그인 장바구니 등 임시 데이터.
- **범위:** 같은 도메인이라도 **탭이 다르면** 서로 다른 저장소를 사용합니다.

### 사용법 (CRUD)

`localStorage`와 메서드 이름이 완전히 동일합니다. 객체 이름만 바꿔주면 됩니다.

```javascript
// 1. 저장하기
sessionStorage.setItem("token", "abc-123");

// 2. 가져오기
const token = sessionStorage.getItem("token");

// 3. 특정 데이터 삭제하기
sessionStorage.removeItem("token");

// 4. 모든 데이터 전체 삭제하기
sessionStorage.clear();
```

---

## 3. 요약: 차이점 비교

가장 중요한 차이점은 **'데이터가 얼마나 오래 유지되는가'**입니다.

|      구분       |             localStorage             |         sessionStorage         |
| :-------------: | :----------------------------------: | :----------------------------: |
| **데이터 수명** | **영구적** (직접 지우기 전까지 유지) | **일시적** (탭/창 닫으면 삭제) |
| **데이터 범위** |    동일 도메인의 모든 탭/창 공유     |     해당 탭 내에서만 유효      |
|    **용량**     |   약 5MB ~ 10MB (브라우저별 상이)    |         약 5MB ~ 10MB          |

---

## 마치며

간단한 데이터를 클라이언트에 저장해야 한다면 쿠키보다는 Web Storage를 사용하는 것이 보안과 성능 면에서 유리합니다. 데이터의 특성에 맞춰 **영구 저장이 필요하면 Local**, **임시 저장이 필요하면 Session**을 적절히 선택해 보세요!

혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요. 👋
