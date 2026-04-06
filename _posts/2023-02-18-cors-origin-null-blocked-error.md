---
title: "[HTML/JS] CORS 에러 해결: Access to script from origin 'null' has been blocked..."
date: 2023-02-18T18:31:00
categories: [hTML, error]
tags: [html, javascript, error, cors, http-server]
description: "로컬에서 HTML 파일을 열었을 때 발생하는 'from origin null has been blocked by CORS policy' 에러의 원인과 http-server를 이용한 해결 방법을 알아봅니다."
custom_style: true
---

HTML/JS 개발을 하다 보면 VS Code에서 `Alt+B` 등으로 브라우저를 바로 열었을 때(로컬 실행), 스크립트가 동작하지 않고 아래와 같은 무시무시한 에러를 마주할 때가 있습니다. 😱

**Access to script at '...' from origin 'null' has been blocked by CORS policy**

![CORS 에러 발생 화면](https://user-images.githubusercontent.com/88264006/219853915-12d10a06-3b65-4a07-ba3e-f5b6ce545470.png)

이 에러는 주로 자바스크립트 모듈(`type="module"`)을 사용할 때, 웹 서버를 통하지 않고 로컬 파일 시스템(`file://`)으로 직접 접근하면 브라우저 보안 정책(CORS)에 막혀 발생합니다.

해결 방법은 간단합니다. 로컬에 **가상 서버**를 띄워서 실행하면 됩니다!

---

## 해결 방법: http-server 사용하기

Node.js 환경이 있다면 `http-server` 패키지를 사용하여 아주 간단하게 로컬 서버를 구동할 수 있습니다.

### 1. http-server 설치하기

만약 아직 설치되어 있지 않다면, 터미널(Terminal)에 아래 명령어를 입력하여 전역(Global)으로 설치합니다.

```bash
npm install http-server -g
```

### 2. 서버 실행하기

설치가 완료되었다면, HTML 파일이 있는 경로에서 다음 명령어를 입력합니다. (`npx`를 사용하면 설치 없이 바로 실행도 가능합니다.)

```bash
npx http-server
```

### 3. 결과 확인 및 접속

명령어를 입력하고 엔터를 치면 가상 서버가 작동하며 접속 가능한 주소(IP)들을 보여줍니다.

![http-server 실행 결과](https://user-images.githubusercontent.com/88264006/219854246-ee0eb20d-33aa-4c02-a298-45d94cc79aed.png)

이제 터미널에 뜬 주소 중 하나(예: `http://127.0.0.1:8080`)를 복사해서 브라우저 주소창에 입력해 보세요.
CORS 에러 없이 `index.js` 파일이 정상적으로 로드되는 것을 확인할 수 있습니다. 🎉

---

## 마치며

로컬 파일 실행 시 발생하는 CORS 에러는 보안을 위한 브라우저의 정책 때문입니다.
`http-server` 외에도 VS Code의 **'Live Server'** 확장 프로그램을 사용하는 것도 좋은 방법이니 편한 방식을 선택해서 즐거운 코딩 하시길 바랍니다!

궁금한 점이나 피드백이 있다면 언제든 댓글로 남겨주세요. 👋
