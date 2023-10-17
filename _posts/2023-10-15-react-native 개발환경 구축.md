---
title: "[React-Native] 개발환경 구축"
date: 2023-10-15T18:39:000
categories: [React-Native]
tags: [react-native] #소문자만 가능
---

---

## React-Native 시작하게된 계기

<p>기존 react를 다뤘다면 크게 어렵지 않을 것이라는 지인의 추천으로 react-native에 흥미를 가지게 되어 도전하게 되었다. 그래서 제일 먼저 react-native가 무엇이며 개발 환경을 세팅하려면 어떻게 해야 하는지 알아보았다.
</p>

<br/>

## React-Native란?

<p>
리액트 네이티브(React Native)는 Facebook에서 개발한 오픈 소스 프레임워크로, 모바일 애플리케이션을 개발하기 위한 도구입니다. JavaScript와 리액트(React)를 기반으로 하여 iOS와 안드로이드 모바일 플랫폼용 네이티브 애플리케이션을 구축할 수 있습니다. 
</p>

<br/>

## React-Native 개발 환경

### 1. Node.js 설치

<p>본인은 기존에 npm 패키지를 사용하고 있어 따로 설치는 안했지만 Expo cli를 설치하는 중에 버전 호환에 문제가 생겨 nvm을 통해 17버전에서 16버전으로 다운그레이드 해주었다.</p>

- 다운그레이드 과정

  1. 아래 경로로 이동해서 Windows용 nvm설치 파일을 다운로드 한다. nvm-setup.zip 파일을 다운로드 한다.<br/><a href="https://github.com/coreybutler/nvm-windows/releases">https://github.com/coreybutler/nvm-windows/releases</a>

  2. 다음 명령어를 통해 node 16버전 다운 받기

  ```js
  nvm install 16
  ```

  3. 사용할 버전으로 변경해준다.

  ```js
  nvm use {사용할 버전}
  ```

<br/>

### 2. Expo CLI(Expo Command Line Interface) 설치

<p>리액트 네이티브 기반의 모바일 앱을 쉽게 개발할 수 있도록 도와주는 도구로 Expo Cli의 가장 큰 장점은 초기 환경 세팅이 간단하다는 것 이다. 다른 선택지로 react-native-cli를 사용하는 방법도 있지만 초기 환경 세팅이 꽤 귀찮다는 단점으로 인해 Expo CLI를 사용하기로 하였다.</p>

```js
//expo-cli 설치
npm install -g expo-cli
```

<br/>

### 3. Android Studio 설치

<p>Android Studio를 사용하면 애뮬레이터를 화면에 띄어서 안드로이드 화면을 보며 개발을 할 수 있습니다.</p>

<p>최신 버전에 Andriod 스튜디오를 설치한다</p>
<a href="https://developer.android.com/studio">https://developer.android.com/studio</a>

- 설치과정

1. android-studio exe 파일 실행

2. <p>Next 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Foy1SN%2FbtscxbCQVkt%2Fa0lTA25rLVqWVckKzukU80%2Fimg.png"><br/>

3. <p>Next 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FTAx1k%2FbtscwEZxsI7%2FXWbJzqqXKAiq2yy0Poeoqk%2Fimg.png"/>
   <br/>

4. <p>Next 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRiF1s%2FbtscwKL8yuG%2FdYEvDLfkHo8ruVgiF4aST1%2Fimg.png"/>
   <br/>

5. <p>install 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdim1AN%2FbtscxnpCIB7%2FgKPHkOqJmFynLhh3GLWK60%2Fimg.png"/>
   <br/>

6. <p>Finish 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeJaqY3%2FbtscHd6SYsh%2FEgHxn6wd7D3K6B805LVKB0%2Fimg.png"/> 
   <br/>

7. <p>안드로이드 스튜디오 실행 후 ok 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbKsI1D%2FbtscFDrxjUj%2FkICWMWNm0cvwR4VDihVZZK%2Fimg.png"/>
   <br/>

8. <p>Don’t send 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdNxeDf%2FbtscGyczJ2t%2F8HhDNg1immi6TWRObKe8KK%2Fimg.png"/>
   <br/>

9. <p>Next 클릭</p>
   <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCbpob%2FbtsctK0lqy1%2FvoP4fmO68gO40d3CTP2xZ0%2Fimg.png"/>
   <br/>

10. <p>Custom은 할 줄 모르는 초보니 Standard 그대로 Next 클릭</p>
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCbpob%2FbtsctK0lqy1%2FvoP4fmO68gO40d3CTP2xZ0%2Fimg.png"/>
    <br/>

11. <p>눈 건강을 위해 Darcula Mode Next 클릭</p>
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fsxk6u%2Fbtscwnqcse6%2FqQbwsiMmCFP7acYJXIkgd1%2Fimg.png"/>
    <br/>

12. <p>Next 클릭</p>
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdGkcjd%2FbtscyHnV2KJ%2F6ntFKfMtKKnSEVKfWDvjKk%2Fimg.png"/>
    <br/>

13. <p>왼쪽 Licenses들을 하나씩 클릭하여 모두 Accept하고 Finish 클릭</p>
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcALOic%2FbtscGMocQo9%2F2Nl38rhfoFNxOnWFMNvlzk%2Fimg.png"/>
    <br/>

14. <p>설치완료되면 Finish 클릭</p>
    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcaKfyP%2FbtscGITDUrl%2FCTGf5POrQmgcFSc0eAt3V0%2Fimg.png"/>

<br/>

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
