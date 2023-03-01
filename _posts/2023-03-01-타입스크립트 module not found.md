---
title: "[TypeScropt] 타입스크립트 module not found"
date: 2023-03-01T16:01:000
categories: [TypeScropt,error]
tags: [typeScropt,error] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">문제</b>
<p>TypeScropt로 프로젝트를 진행하던 중 react-router-dom으로 첫url에 화면에 보여질 Home.tsx 컴포넌트를 불러오려고 했더니 아래와 같은 에러가 발생했다.</p>
<img src="https://user-images.githubusercontent.com/88264006/222068777-311f0684-c655-4bcb-a074-384a9442c67a.png" alt="Module not found: Error: Can't resolve"/>

## <b style="border-bottom:2px solid gray">해결방법</b>
- npm install typescript @types/node @types/react @types/react-dom @types/jest 설치

```js
npm install typescript @types/node @types/react @types/react-dom @types/jest
```