---
title: "[React Native] 프로젝트 생성과 Android Studio 연결하기"
date: 2023-10-17T19:20:00
categories: [react-native]
tags: [react-native, expo, android-studio]
description: "Expo로 React Native 프로젝트를 생성하고 Android Studio 에뮬레이터에서 실행하는 흐름을 정리했습니다."
custom_style: true
---

## 들어가며

React Native 개발을 시작하려면 프로젝트를 생성한 뒤, 앱을 실행해볼 환경이 필요합니다.

실행 방법은 여러 가지가 있습니다.

- Expo Go로 실제 기기에서 확인하기
- Android Studio 에뮬레이터에서 확인하기
- 개발 빌드로 실행하기

이번 글에서는 Expo 프로젝트를 만들고 Android Studio 에뮬레이터에서 실행하는 흐름을 정리해볼게요.

---

## Expo 프로젝트 생성하기

새 Expo 프로젝트는 아래 명령어로 만들 수 있습니다.

```bash
npx create-expo-app my-app
```

프로젝트가 생성되면 폴더로 이동합니다.

```bash
cd my-app
```

개발 서버는 다음 명령어로 실행합니다.

```bash
npm start
```

---

## 템플릿 선택하기

Expo 프로젝트를 만들 때 템플릿을 선택할 수 있습니다.

처음 학습할 때는 기본 템플릿이나 TypeScript 템플릿을 선택하면 충분합니다.

탭 구조나 라우팅 예제가 포함된 템플릿을 선택하면 파일이 조금 더 많지만, 앱 구조를 빠르게 살펴보기 좋습니다.

---

## Android Studio에서 에뮬레이터 만들기

Android Studio를 실행한 뒤 Device Manager에서 가상 기기를 만들 수 있습니다.

흐름은 대략 다음과 같습니다.

1. Android Studio를 실행합니다.
2. `More Actions`에서 `Virtual Device Manager`를 엽니다.
3. 새 기기를 생성합니다.
4. 원하는 Android 기기와 시스템 이미지를 선택합니다.
5. 생성한 에뮬레이터를 실행합니다.

에뮬레이터가 켜진 상태에서 Expo 프로젝트를 실행하면 Android 기기로 앱을 열 수 있습니다.

---

## Expo 개발 서버에서 Android로 실행하기

`npm start`를 실행하면 터미널에 여러 단축키가 표시됩니다.

Android 에뮬레이터에서 앱을 실행하려면 보통 `a`를 누릅니다.

```txt
a - open Android
w - open web
r - reload app
m - toggle menu
```

에뮬레이터가 정상적으로 실행 중이라면 Expo 앱이 Android 화면에 열립니다.

---

## 코드 변경 확인하기

Expo는 개발 중 코드 변경을 빠르게 반영해줍니다.

예를 들어 `app/(tabs)/index.tsx` 또는 `App.tsx`의 텍스트를 바꾸면 에뮬레이터 화면에도 변경된 내용이 반영됩니다.

반영이 늦거나 멈춘 것처럼 보이면 `r`을 눌러 앱을 다시 로드해볼 수 있습니다.

---

## 문제가 생길 때 확인할 것

Android 에뮬레이터 연결이 잘 안 된다면 아래를 확인해보면 좋습니다.

1. Android Studio 에뮬레이터가 실행 중인지 확인합니다.
2. Expo 개발 서버가 켜져 있는지 확인합니다.
3. 터미널에서 `a`를 눌러 Android 실행을 시도합니다.
4. 에뮬레이터가 너무 느리면 재시작해봅니다.
5. SDK나 emulator 설정이 누락되지 않았는지 Android Studio에서 확인합니다.

처음에는 환경 문제가 낯설 수 있지만, 한 번 연결해두면 이후 개발 흐름은 훨씬 편해집니다.

---

## 마무리

React Native 프로젝트는 Expo를 사용하면 비교적 간단하게 시작할 수 있습니다.

`create-expo-app`으로 프로젝트를 만들고, Android Studio에서 에뮬레이터를 실행한 뒤, Expo 개발 서버에서 `a`를 누르면 Android 환경에서 앱을 확인할 수 있습니다.

처음에는 실제 기기와 에뮬레이터 중 하나만 안정적으로 연결해두어도 개발을 시작하기에 충분합니다.
