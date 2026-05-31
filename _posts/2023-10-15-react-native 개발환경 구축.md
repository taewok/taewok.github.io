---
title: "[React Native] 개발 환경 구축하기"
date: 2023-10-15T18:39:00
categories: [react-native]
tags: [react-native, expo, android-studio]
description: "React Native를 시작하기 위해 Node.js, Expo, Android Studio를 준비하는 기본 개발 환경 구축 흐름을 정리했습니다."
custom_style: true
---

## 들어가며

React Native는 JavaScript와 React를 기반으로 모바일 앱을 만들 수 있는 프레임워크입니다.

React를 공부한 경험이 있다면 컴포넌트, props, state 같은 개념을 이어서 사용할 수 있다는 장점이 있습니다.

이번 글에서는 React Native를 처음 시작할 때 필요한 개발 환경 구축 흐름을 정리해볼게요.

---

## React Native란?

React Native는 iOS와 Android 앱을 만들 수 있는 모바일 앱 개발 프레임워크입니다.

웹에서 사용하는 React 문법과 비슷한 방식으로 UI를 구성하지만, 실제 렌더링은 모바일 네이티브 컴포넌트를 기반으로 동작합니다.

처음 학습할 때는 Expo를 사용하면 환경 설정 부담을 줄이고 빠르게 시작할 수 있습니다.

---

## Node.js 준비하기

React Native와 Expo를 사용하려면 Node.js가 필요합니다.

이미 Node.js가 설치되어 있다면 버전을 확인합니다.

```bash
node -v
npm -v
```

여러 프로젝트에서 Node 버전을 바꿔야 한다면 nvm 같은 버전 관리 도구를 사용하는 것이 편합니다.

Windows에서는 `nvm-windows`를 사용할 수 있습니다.

```bash
nvm install 18
nvm use 18
```

프로젝트나 Expo 버전에 따라 권장 Node 버전이 다를 수 있으니, 사용 중인 도구의 문서를 함께 확인하는 것이 좋습니다.

---

## Expo로 시작하기

Expo는 React Native 앱을 쉽게 만들고 실행할 수 있게 도와주는 도구입니다.

새 프로젝트는 아래 명령어로 만들 수 있습니다.

```bash
npx create-expo-app my-app
```

프로젝트 폴더로 이동한 뒤 실행합니다.

```bash
cd my-app
npm start
```

개발 서버가 실행되면 QR 코드가 표시되고, Expo Go 앱으로 실제 기기에서 확인할 수 있습니다.

---

## Android Studio 설치하기

Android 에뮬레이터에서 앱을 실행하려면 Android Studio가 필요합니다.

Android Studio를 설치하면 Android SDK, 에뮬레이터, 가상 기기 관리 도구를 함께 사용할 수 있습니다.

설치 후에는 보통 아래 항목을 확인합니다.

- Android SDK 설치 여부
- Android SDK Platform 설치 여부
- Android Emulator 설치 여부
- 가상 기기 생성 여부

에뮬레이터를 사용할 계획이라면 Android Studio의 Device Manager에서 가상 기기를 하나 만들어두면 됩니다.

---

## 실제 기기로 확인하기

Expo를 사용하면 처음에는 실제 기기에서 확인하는 방식이 가장 간단합니다.

1. 휴대폰에 Expo Go 앱을 설치합니다.
2. PC와 휴대폰을 같은 네트워크에 연결합니다.
3. `npm start`로 개발 서버를 실행합니다.
4. 터미널이나 브라우저에 표시된 QR 코드를 Expo Go로 스캔합니다.

이렇게 하면 Android Studio 에뮬레이터 설정 없이도 빠르게 앱을 확인할 수 있습니다.

---

## 개발 환경에서 자주 만나는 문제

React Native 환경 구축은 운영체제, Node 버전, Android SDK 설정에 따라 문제가 달라질 수 있습니다.

문제가 생기면 아래 순서로 확인해보면 좋습니다.

1. Node.js 버전이 프로젝트와 맞는지 확인합니다.
2. 패키지를 다시 설치합니다.
3. Expo 개발 서버를 재시작합니다.
4. Android Studio SDK 설정을 확인합니다.
5. PC와 모바일 기기가 같은 네트워크에 있는지 확인합니다.

처음에는 Expo Go로 실제 기기에서 실행해보고, 이후 필요할 때 에뮬레이터 환경을 잡는 방식도 좋습니다.

---

## 마무리

React Native를 처음 시작할 때는 Expo를 사용하면 훨씬 가볍게 출발할 수 있습니다.

기본적으로 Node.js를 준비하고, `create-expo-app`으로 프로젝트를 만든 뒤, Expo Go나 Android Studio 에뮬레이터로 실행하면 됩니다.

환경 구축은 한 번에 완벽하게 끝내기보다, 실제 실행이 되는 최소 흐름부터 잡고 필요한 설정을 하나씩 추가하는 방식이 가장 편합니다.
