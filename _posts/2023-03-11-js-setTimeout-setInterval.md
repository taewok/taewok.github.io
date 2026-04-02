---
title: "[JavaScript] 타이머 함수 완벽 정리: setTimeout() & setInterval() 활용법"
date: 2023-03-11T21:01:000
categories: [JavaScript]
tags: [javascript] #소문자만 가능
---

웹 페이지에서 특정 시간이 지난 후 동작을 수행하거나, 일정 간격으로 작업을 반복해야 할 때가 있습니다. 자바스크립트에서는 이를 위해 `setTimeout`과 `setInterval`이라는 강력한 타이머 함수를 제공합니다.

오늘은 이 두 함수의 차이점과 올바른 해제 방법까지 알아보겠습니다.

---

## <b style="border-bottom:2px solid gray" class="h2">1. setTimeout(): 딱 한 번만 실행하기</b>

`setTimeout()`은 설정한 밀리초(ms)가 경과한 후, 인자로 전달된 콜백 함수를 **단 한 번** 실행합니다. (1000ms = 1초)

### 기본 문법

```javascript
setTimeout(() => {
  console.log("1초 뒤에 인사를 건넵니다: HI~");
}, 1000);
```

### clearTimeout(): 예약된 작업 취소하기

타이머가 실행되기 전, 특정 조건에서 해당 작업을 취소해야 한다면 `clearTimeout()`을 사용합니다. 이를 위해선 `setTimeout`의 반환값(ID)을 변수에 담아두어야 합니다.

```javascript
const timerId = setTimeout(() => {
  console.log("이 메시지는 보이지 않습니다.");
}, 3000);

// 특정 조건에서 타이머 취소
clearTimeout(timerId);
```

---

## <b style="border-bottom:2px solid gray" class="h2">2. setInterval(): 일정 간격으로 반복하기</b>

`setInterval()`은 설정한 시간마다 콜백 함수를 **무한히 반복**해서 실행합니다. 실시간 시계, 배너 슬라이드 등을 구현할 때 유용합니다.

### 기본 문법

```javascript
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`${count}초 경과...`);
}, 1000);
```

### clearInterval(): 반복 멈추기

반복을 멈추지 않으면 메모리 리소스를 계속 소모하므로, 작업이 끝나면 반드시 `clearInterval()`로 중지시켜야 합니다.

```javascript
let timerVar = setInterval(() => {
  console.log("반복 실행 중");
}, 3000);

// 반복 중지
clearInterval(timerVar);
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. 주의사항: 메모리 누수 방지</b>

프론트엔드 개발(특히 React나 Vue 같은 프레임워크)에서는 컴포넌트가 화면에서 사라질 때(Unmount) 타이머를 반드시 해제해줘야 합니다. 해제하지 않으면 보이지 않는 곳에서 타이머가 계속 돌아가는 **메모리 누수(Memory Leak)** 현상이 발생할 수 있습니다.

| 함수        | 실행 횟수 | 중지 함수       |
| :---------- | :-------- | :-------------- |
| setTimeout  | 1회       | `clearTimeout`  |
| setInterval | 무한 반복 | `clearInterval` |

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>타이머 함수는 비동기 프로그래밍의 기초이자 웹 애플리케이션의 동적인 기능을 담당하는 핵심 요소입니다. <code>setTimeout</code>과 <code>setInterval</code>을 적절히 사용하고, 잊지 말고 <code>clear</code> 함수로 자원을 관리!</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
