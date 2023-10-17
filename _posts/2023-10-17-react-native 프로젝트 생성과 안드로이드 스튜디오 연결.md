---
title: "[React-Native] 프로젝트 생성과 Android Studio 연결"
date: 2023-10-17T19:20:000
categories: [React-Native]
tags: [react-native] #소문자만 가능
---

---

<p>지난 시간 <a href="https://taewok.github.io/posts/react-native-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95">React-Native 개발환경 구축</a> 후 프로젝트 생성과 안드로이드 앱 테스트를 위해 가상 기기를 화면에 만들어야한다. 앱을 개발하며 테스트하는 방법은 여러가지가 있지만 고등학교 시절 Android Studio를 사용해봤기에 Android Studio 에뮬레이터를 채택했다.</p>

- 테스트 방법
  - Android Studio 에뮬레이터 사용
  - 실제 안드로이드 기기 사용
  - Expo Go 앱 사용

## React-Native 프로젝트 생성

1. Expo Cli가 설치되어 있다는 가정하에 아래의 명령어를 사용한다.

```js
expo init 프로젝트이름
```

<br/>

2. 아래와 같이 템플릿을 선택하라는 말이 나온다 위/아래 키로 변경할 수 있으며 Enter로 선택한다.
   <img src="https://github.com/taewok/taewok/assets/88264006/ab9def50-4cf1-48fa-9ddd-b636e6a48d54"/>

- 각 템플릿 설명

  - <strong>blank</strong>: 미니멀한 앱 템플릿으로, 초기 앱은 비어있으며 개발자가 필요한 모든 구성 요소를 직접 추가해야 합니다.

  - <strong>blank (TypeScript)</strong>: 미니멀한 앱 템플릿이지만 TypeScript를 사용하여 프로젝트를 초기화합니다.

  - <strong>tabs (TypeScript)</strong>: 여러 예제 화면과 탭을 사용하여 시작하는 템플릿입니다. 이 템플릿은 React Navigation을 사용하여 탭과 스택 내비게이션을 구성하고 있습니다.

  - <strong>minimal</strong>: 이 템플릿은 Expo에서 제공하는 많은 기능과 라이브러리를 사용하지 않고 앱을 시작하고자 하는 경우 유용합니다. 추가 기능은 개발자가 직접 구현해야 합니다.

<br/>
<p>본인은 여러 예제가 있는 tabs (TypeScript) 템플릿 선택 후
아래와 같이 구성된 프로젝트가 생성된 걸 볼 수 있다.</p>

<img src="https://github.com/taewok/taewok/assets/88264006/8a1b5ae2-d69d-445c-b4df-cb146ab40034"/>

<br/>

## Android Studio를 사용해 가상 기기 띄우기

3-1 Android Studio 실행
<img src="https://github.com/taewok/taewok/assets/88264006/1e436fe0-8776-40b0-bc43-596883e96c8e"/>
<br/>

3-2 More Actions 클릭 후 Virtual Device Manager 클릭
<img src="https://github.com/taewok/taewok/assets/88264006/9446ebe5-a3fe-4576-b28e-13359a087d68"/>
<br/>

3-3 아래와 같은 화면이 뜨는데 재생 버튼을 눌러줍니다.
<img src="https://github.com/taewok/taewok/assets/88264006/ae3663ec-e19b-4f9a-90bc-dc731c6d9305"/>

 <p>그러면 아래와 같이 가상 안드로이드 기기를 띄우는데 성공합니다.</p>
<br/>

## React-Native 프로젝트 실행

<p>가상 안드로이드 기기를 띄운 상태에서 프로젝트에 터미널을 열고 npm start 명령어를 실행합니다. 그럼 아래와 같이 결과가 나오는데요.</p>

<img src="https://github.com/taewok/taewok/assets/88264006/e1e0e706-72ed-460a-8ad8-4e27fb6c4a5e"/>

- <strong>s (Switch to Development Build)</strong>: 개발 빌드로 전환합니다. 이 명령은 개발 및 디버깅에 사용됩니다.

- <strong>a (Open Android)</strong>: Expo Go 앱 또는 Android 에뮬레이터에서 앱을 엽니다. 이 명령은 Android 기기에서 앱을 열거나 실행하는 데 사용됩니다.

- <strong>w (Open Web)</strong>: 웹 브라우저에서 Expo 프로젝트를 엽니다. 이 명령은 웹 브라우저를 사용하여 앱을 테스트하는 데 사용됩니다.

- <strong>j (Open Debugger)</strong>: Expo 개발 도구를 엽니다. 이 도구는 JavaScript 디버깅을 지원하며, 코드 디버깅 및 오류 추적에 사용됩니다.

- <strong>r (Reload App)</strong>: 앱을 다시 로드합니다. 이 명령은 앱의 변경 사항을 즉시 적용하거나 오류를 수정한 후 앱을 다시 시작하는 데 사용됩니다.

- <strong>m (Toggle Menu)</strong>: Expo Go 앱에서 개발자 메뉴를 엽니다. 이 메뉴에서 다양한 설정 및 디버깅 도구에 액세스할 수 있습니다.

- <strong>o (Open Project Code in Your Editor)</strong>: 선택한 텍스트 에디터 또는 IDE에서 프로젝트 코드를 엽니다. Expo CLI와 연결된 편집기를 열어 프로젝트 파일을 편집할 수 있습니다.

- <strong>? (Show All Commands)</strong>: 사용 가능한 모든 명령과 단축키를 표시합니다. 명령어 창에서 ?를 입력하면 이러한 명령 및 단축키 목록을 확인할 수 있습니다.

<p>안드로이드 개발을 위해서니 a키를 눌러준다.</p>

<p>정상적이게 작동했다면 가상 기기 화면이 아래의 이미지 같이 변한다.</p>

<img src="https://github.com/taewok/taewok/assets/88264006/bbb8b79c-5e2a-4c25-8b8b-5f27b5bd9f7c"/>

<p>그리고 화면에 나와있듯이 app/(tabs)/index.tsx에 내용을 바꾸면 실시간으로 가상 기기 화면에 적용된다. 이렇게 되면 React-Native 개발을 하면 안드로이드 테스트는 문제가 없다!</p>

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
