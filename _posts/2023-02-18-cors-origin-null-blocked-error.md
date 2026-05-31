---
title: "[HTML/JS] origin null CORS 에러 해결하기"
date: 2023-02-18T18:31:00
categories: [html, error]
tags: [html, javascript, error, cors, http-server]
description: "로컬에서 HTML 파일을 직접 열었을 때 발생하는 origin null CORS 에러의 원인과 http-server를 이용한 해결 방법을 정리했습니다."
custom_style: true
---

## 🧐 origin null CORS 에러가 왜 발생할까요?

HTML과 JavaScript만으로 간단한 예제를 만들다 보면 파일을 브라우저에서 바로 열어 확인하는 경우가 있습니다.

예를 들어 VS Code에서 HTML 파일을 바로 열거나, 파일을 더블 클릭해서 브라우저로 실행하는 방식입니다.

그런데 JavaScript 모듈을 사용할 때 다음과 같은 에러가 발생할 수 있습니다.

```txt
Access to script at '...' from origin 'null' has been blocked by CORS policy
```

이 에러는 보통 웹 서버를 통하지 않고 `file://` 경로로 HTML 파일을 열었을 때 발생합니다.

브라우저는 보안상 `file://`에서 실행된 페이지의 origin을 `null`로 취급할 수 있습니다. 그리고 모듈 스크립트나 외부 리소스를 불러올 때 CORS 정책에 막힐 수 있습니다.

---

## 💡 해결 방법은 로컬 서버로 실행하기

해결 방법은 간단합니다.

HTML 파일을 직접 여는 대신, 로컬 서버를 띄워서 `http://localhost` 주소로 접속하면 됩니다.

```txt
file://...
→ CORS 문제가 생길 수 있음

http://localhost:8080
→ 로컬 서버를 통해 정상 실행
```

---

## 🛠️ http-server 사용하기

Node.js 환경이 있다면 `http-server` 패키지를 사용할 수 있습니다.

### 1. http-server 실행하기

설치 없이 바로 실행하고 싶다면 `npx`를 사용하면 됩니다.

```bash
npx http-server
```

전역으로 설치해서 사용하고 싶다면 다음 명령어로 설치할 수 있습니다.

```bash
npm install http-server -g
```

설치 후에는 HTML 파일이 있는 폴더에서 다음 명령어를 실행합니다.

```bash
http-server
```

---

## 🌐 브라우저에서 접속하기

명령어를 실행하면 터미널에 접속 가능한 주소가 표시됩니다.

예를 들어 다음과 같은 주소가 나올 수 있습니다.

```txt
http://127.0.0.1:8080
http://localhost:8080
```

이 주소를 브라우저에 입력하면 HTML 파일을 로컬 서버를 통해 실행할 수 있습니다.

이제 `file://`이 아니라 `http://` 주소로 접근하기 때문에 origin null CORS 에러를 피할 수 있습니다.

---

## 🧩 Live Server를 사용해도 좋아요

VS Code를 사용한다면 `Live Server` 확장 프로그램을 사용하는 방법도 편합니다.

Live Server를 설치한 뒤 HTML 파일에서 `Open with Live Server`를 실행하면 자동으로 로컬 서버가 열립니다.

간단한 예제를 자주 확인해야 한다면 `http-server`보다 Live Server가 더 편할 수도 있습니다.

---

## ✅ 정리

`origin null` CORS 에러는 보통 HTML 파일을 `file://` 경로로 직접 열었을 때 발생합니다.

- 브라우저는 `file://` 환경을 일반적인 웹 origin처럼 다루지 않습니다.
- 모듈 스크립트나 외부 리소스 로딩이 CORS 정책에 막힐 수 있습니다.
- 해결하려면 로컬 서버를 띄워 `http://localhost`로 접속하면 됩니다.
- `http-server`나 VS Code의 `Live Server`를 사용할 수 있습니다.

HTML/JS 예제를 만들 때 스크립트가 이상하게 로드되지 않는다면, 먼저 파일을 직접 열고 있는지 확인해보면 좋습니다.
