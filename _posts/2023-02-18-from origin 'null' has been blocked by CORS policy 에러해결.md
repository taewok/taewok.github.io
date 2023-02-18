---
title: "[Html] from origin 'null' has been blocked by CORS policy 에러발생"
date: 2023-02-18T18:31:000
categories: [html,error]
tags: [html,error,JavaScript] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">에러 발생</b>
<img src="https://user-images.githubusercontent.com/88264006/219853915-12d10a06-3b65-4a07-ba3e-f5b6ce545470.png" alt="from origin 'null' has been blocked by CORS policy 에러발생"/>
<span>index.html 파일을 Open In Default Browser(Alt+B)로 실행하니 index.js를 불러오려고 하는 부분에서 위와 같은 Error가 발생했다.</span>

## <b style="border-bottom:2px solid gray">해결방법</b>
### 1. http-server가 설치돼 있지 않다면
```
npm install http-server -g
```
### 2. http-server 실행
```
npx http-server
```
<p>위 코드를 터미널에 입력하고 엔터를 치면 아래와 같은 결과가 뜬다.</p>
<img src="https://user-images.githubusercontent.com/88264006/219854246-ee0eb20d-33aa-4c02-a298-45d94cc79aed.png" alt="npx http-server"/>
<p>그럼 주소가 2개 뜨는데 하나 골라서 주소창에 입력하고 들어가면 에러 없이 index.html 파일을 로컬에 실행시킬 수 있다.</p>