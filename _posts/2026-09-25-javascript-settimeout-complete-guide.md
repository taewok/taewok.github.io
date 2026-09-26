---
title: "[JavaScript] setTimeout 완벽 이해하기: 이벤트 루프, 지연 실행, cleanup까지"
date: 2026-09-25T00:30:00Z
categories: [javascript]
tags: [javascript, settimeout, timer, event-loop, async, react]
description: "setTimeout이 정확히 언제 실행되는지, 이벤트 루프와 task queue, Promise와 실행 순서 차이, clearTimeout, setTimeout 0, React cleanup, debounce 직접 구현까지 정리했습니다."
custom_style: true
excerpt_separator: <!--more-->
---

<!--more-->

{% raw %}

## 들어가며: setTimeout은 정확히 몇 초 뒤에 실행될까?

JavaScript를 처음 배울 때 `setTimeout`은 꽤 단순해 보입니다.

```js
setTimeout(() => {
  console.log("1초 뒤 실행");
}, 1000);
```

이 코드를 보면 자연스럽게 이렇게 이해합니다.

```txt
1000ms 뒤에 함수가 실행된다.
```

하지만 실제로는 조금 더 정확히 이해해야 합니다.

`setTimeout`은 "정확히 그 시간 뒤에 실행"을 보장하는 함수가 아닙니다.

더 정확히 말하면 이렇습니다.

```txt
지정한 시간이 지난 뒤,
콜백을 실행 가능한 작업 큐에 넣는다.
그리고 JavaScript call stack이 비었을 때 실행된다.
```

즉, `setTimeout(fn, 1000)`은 "1초 뒤 무조건 실행"이 아니라, "최소 1초가 지난 뒤 실행될 수 있는 상태가 된다"에 가깝습니다.

이번 글에서는 `setTimeout`을 단순 사용법이 아니라 이벤트 루프, 실행 순서, `clearTimeout`, `setTimeout(0)`, React cleanup까지 연결해서 이해해보겠습니다.

---

## setTimeout 기본 사용법

`setTimeout`은 일정 시간이 지난 뒤 콜백 함수를 한 번 실행합니다.

```js
setTimeout(() => {
  console.log("실행");
}, 1000);
```

첫 번째 인자는 실행할 함수입니다.

두 번째 인자는 지연 시간입니다.

단위는 밀리초입니다.

```txt
1000ms = 1초
3000ms = 3초
```

아래 코드는 3초 뒤에 실행됩니다.

```js
setTimeout(() => {
  console.log("3초 뒤 실행");
}, 3000);
```

`setTimeout`은 타이머 id를 반환합니다.

```js
const timerId = setTimeout(() => {
  console.log("나중에 실행");
}, 1000);
```

이 id는 나중에 예약된 작업을 취소할 때 사용합니다.

```js
clearTimeout(timerId);
```

---

## clearTimeout으로 예약 취소하기

`setTimeout`으로 예약한 작업은 아직 실행되기 전이라면 취소할 수 있습니다.

```js
const timerId = setTimeout(() => {
  console.log("실행되지 않습니다.");
}, 3000);

clearTimeout(timerId);
```

이 코드는 3초가 지나기 전에 `clearTimeout`을 호출했기 때문에 콜백이 실행되지 않습니다.

실무에서는 다음 상황에서 자주 사용합니다.

- 사용자가 입력을 계속 바꿀 때 이전 검색 예약 취소
- 컴포넌트가 사라질 때 예약된 타이머 정리
- 모달이 닫혔을 때 뒤늦은 상태 변경 방지
- 알림 메시지를 자동으로 숨기되, 사용자가 직접 닫으면 예약 취소

예를 들어 토스트 메시지를 3초 뒤 숨기고 싶다면 이렇게 작성할 수 있습니다.

```js
const timerId = setTimeout(() => {
  hideToast();
}, 3000);

closeButton.addEventListener("click", () => {
  clearTimeout(timerId);
  hideToast();
});
```

핵심은 이것입니다.

```txt
setTimeout은 미래의 작업을 예약한다.
clearTimeout은 아직 실행되지 않은 예약을 취소한다.
```

---

## setTimeout은 동기 코드를 멈추지 않는다

`setTimeout`은 JavaScript 실행을 멈추는 함수가 아닙니다.

아래 코드를 봅시다.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");
```

출력 순서는 이렇습니다.

```txt
A
C
B
```

`setTimeout`을 만났다고 해서 1초 동안 아래 코드가 멈추지 않습니다.

JavaScript는 타이머를 예약해두고 바로 다음 코드를 실행합니다.

```txt
console.log("A") 실행
→ setTimeout 콜백 예약
→ console.log("C") 실행
→ 1초 뒤 콜백 실행 가능
→ console.log("B") 실행
```

이 점을 이해하지 못하면 아래처럼 착각하기 쉽습니다.

```js
let result = null;

setTimeout(() => {
  result = "완료";
}, 1000);

console.log(result);
```

출력은 `"완료"`가 아니라 `null`입니다.

타이머 콜백은 나중에 실행되기 때문입니다.

타이머 결과를 사용하려면 콜백 안에서 처리하거나 Promise로 감싸야 합니다.

```js
setTimeout(() => {
  const result = "완료";
  console.log(result);
}, 1000);
```

---

## 이벤트 루프 관점에서 이해하기

`setTimeout`을 제대로 이해하려면 이벤트 루프를 알아야 합니다.

JavaScript는 기본적으로 한 번에 하나의 작업을 실행합니다.

이때 현재 실행 중인 함수들은 call stack에 쌓입니다.

`setTimeout`을 호출하면 브라우저는 타이머를 관리하다가, 지정한 시간이 지나면 콜백을 task queue에 넣습니다.

이후 call stack이 비면 이벤트 루프가 task queue에서 작업을 꺼내 실행합니다.

흐름은 이렇습니다.

```txt
1. setTimeout 호출
2. 브라우저가 타이머 시작
3. 지정한 시간이 지남
4. 콜백이 task queue에 들어감
5. call stack이 비어야 함
6. 이벤트 루프가 콜백을 꺼내 실행
```

중요한 부분은 5번입니다.

call stack이 비어야 콜백이 실행됩니다.

그래서 아래 코드는 1초보다 늦게 실행될 수 있습니다.

```js
setTimeout(() => {
  console.log("타이머 콜백");
}, 1000);

const start = Date.now();

while (Date.now() - start < 3000) {
  // 3초 동안 call stack을 비우지 않음
}

console.log("긴 작업 끝");
```

출력 흐름은 이렇습니다.

```txt
3초 동안 while문 실행
→ 긴 작업 끝
→ 타이머 콜백
```

타이머는 1초 뒤 실행 가능한 상태가 되었을 수 있습니다.

하지만 JavaScript가 3초 동안 긴 작업을 처리하느라 call stack이 비지 않았기 때문에 콜백은 기다려야 합니다.

그래서 `setTimeout`은 정확한 예약 시간이 아니라 최소 지연 시간으로 이해하는 편이 좋습니다.

---

## setTimeout 0은 즉시 실행일까?

많이 헷갈리는 코드입니다.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

출력은 이렇습니다.

```txt
A
C
B
```

`setTimeout(..., 0)`이라고 해서 즉시 실행되는 것이 아닙니다.

0ms 뒤에 실행 가능한 task로 예약될 뿐입니다.

현재 실행 중인 동기 코드가 모두 끝난 뒤에야 실행됩니다.

```txt
현재 call stack의 코드 실행
→ setTimeout 콜백은 task queue로 이동
→ call stack이 빔
→ task queue의 콜백 실행
```

그래서 `setTimeout(0)`은 "즉시 실행"이 아니라 "현재 실행 흐름이 끝난 뒤 나중에 실행"입니다.

이 패턴은 긴 작업을 쪼개거나, 현재 call stack 이후에 어떤 작업을 미루고 싶을 때 사용되기도 했습니다.

```js
setTimeout(() => {
  console.log("현재 동기 코드 이후 실행");
}, 0);
```

하지만 Promise microtask나 `queueMicrotask`, `requestAnimationFrame`과는 실행 타이밍이 다르므로 목적에 맞게 골라야 합니다.

---

## Promise와 setTimeout 실행 순서

다음 코드는 프론트엔드 면접이나 학습 자료에서 자주 등장합니다.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve().then(() => {
  console.log("C");
});

console.log("D");
```

출력은 이렇습니다.

```txt
A
D
C
B
```

왜 `C`가 `B`보다 먼저 나올까요?

Promise의 `then` 콜백은 microtask queue에 들어갑니다.

`setTimeout` 콜백은 task queue에 들어갑니다.

현재 동기 코드가 끝난 뒤, 브라우저는 보통 task를 처리하기 전에 microtask queue를 먼저 비웁니다.

흐름은 이렇습니다.

```txt
1. console.log("A")
2. setTimeout 콜백을 task queue에 예약
3. Promise.then 콜백을 microtask queue에 예약
4. console.log("D")
5. call stack 비어 있음
6. microtask queue 처리 → C
7. task queue 처리 → B
```

정리하면 이렇습니다.

```txt
동기 코드
→ microtask
→ task
```

대표적으로 Promise callback은 microtask이고, `setTimeout` callback은 task입니다.

그래서 `setTimeout(0)`보다 `Promise.then`이 먼저 실행됩니다.

---

## setTimeout의 delay는 숫자로 변환된다

`setTimeout`의 두 번째 인자는 숫자 delay입니다.

하지만 JavaScript에서는 값이 숫자로 변환될 수 있습니다.

```js
setTimeout(() => {
  console.log("실행");
}, "1000");
```

문자열 `"1000"`은 숫자 `1000`으로 변환될 수 있으므로 1초 뒤 실행됩니다.

하지만 이런 코드는 피하는 것이 좋습니다.

```js
setTimeout(() => {
  console.log("실행");
}, "1 second");
```

명확하게 숫자를 전달하는 편이 안전합니다.

```js
setTimeout(() => {
  console.log("실행");
}, 1000);
```

실무 코드에서는 상수로 의미를 드러내면 더 좋습니다.

```js
const TOAST_HIDE_DELAY = 3000;

setTimeout(() => {
  hideToast();
}, TOAST_HIDE_DELAY);
```

---

## 중첩 setTimeout과 최소 지연 시간

브라우저는 아주 짧은 타이머가 계속 중첩될 때 최소 지연 시간을 적용할 수 있습니다.

HTML 표준에서는 중첩된 타이머가 일정 수준을 넘고 delay가 4ms보다 작으면 delay를 4ms로 맞추는 동작을 정의합니다.

쉽게 말하면 이런 코드가 매번 정확히 0ms로 도는 것은 아닙니다.

```js
function loop() {
  setTimeout(() => {
    console.log("tick");
    loop();
  }, 0);
}

loop();
```

브라우저는 너무 촘촘한 타이머가 메인 스레드를 과도하게 점유하지 않도록 최소 지연 시간을 적용할 수 있습니다.

또 브라우저 탭이 백그라운드에 있거나, 기기가 절전 모드이거나, 브라우저가 성능을 조절하는 상황에서는 타이머가 더 늦게 실행될 수 있습니다.

그래서 `setTimeout`은 정밀한 시간 측정 도구가 아닙니다.

정확한 애니메이션 프레임이 필요하다면 `requestAnimationFrame`을 고려해야 합니다.

정확한 시간 경과 측정이 필요하다면 콜백 실행 횟수만 믿지 말고 `Date.now()`나 `performance.now()`로 실제 흐른 시간을 계산해야 합니다.

---

## this가 달라지는 문제

객체의 메서드를 `setTimeout`에 그대로 넘기면 `this`가 기대와 다를 수 있습니다.

```js
const user = {
  name: "Kim",
  sayName() {
    console.log(this.name);
  },
};

setTimeout(user.sayName, 1000);
```

이 코드는 `"Kim"`을 출력하지 않을 수 있습니다.

`user.sayName`이라는 함수만 전달했기 때문에, 호출될 때 `this`가 `user`로 유지되지 않습니다.

해결 방법은 감싸서 호출하는 것입니다.

```js
setTimeout(() => {
  user.sayName();
}, 1000);
```

또는 `bind`를 사용할 수 있습니다.

```js
setTimeout(user.sayName.bind(user), 1000);
```

요즘 코드에서는 화살표 함수로 감싸는 방식이 가장 읽기 쉽습니다.

---

## 인자를 전달하는 방법

`setTimeout`은 콜백 뒤에 추가 인자를 전달할 수 있습니다.

```js
setTimeout(
  (name) => {
    console.log(`Hello, ${name}`);
  },
  1000,
  "Kim",
);
```

출력은 1초 뒤 다음과 같습니다.

```txt
Hello, Kim
```

하지만 실무에서는 화살표 함수로 감싸는 방식이 더 자주 쓰입니다.

```js
setTimeout(() => {
  greet("Kim");
}, 1000);
```

이 방식이 여러 값을 다루거나 로직을 추가하기 더 편합니다.

---

## setTimeout과 setInterval의 차이

`setTimeout`은 한 번 실행됩니다.

```js
setTimeout(() => {
  console.log("한 번 실행");
}, 1000);
```

`setInterval`은 일정 간격마다 반복 실행됩니다.

```js
setInterval(() => {
  console.log("반복 실행");
}, 1000);
```

비교하면 이렇습니다.

| 구분 | setTimeout | setInterval |
| --- | --- | --- |
| 실행 횟수 | 한 번 | 반복 |
| 취소 함수 | `clearTimeout` | `clearInterval` |
| 대표 용도 | 지연 실행, 예약 취소 | 주기적 폴링, 시계 |
| 주의점 | cleanup 필요 | 누적 실행과 cleanup 더 중요 |

반복 작업이라도 `setInterval`보다 재귀적 `setTimeout`이 더 나을 때가 있습니다.

```js
function poll() {
  setTimeout(async () => {
    await fetchData();
    poll();
  }, 3000);
}

poll();
```

이 방식은 이전 작업이 끝난 뒤 다음 타이머를 예약할 수 있습니다.

반면 `setInterval`은 이전 작업이 오래 걸려도 정해진 간격마다 다음 실행을 시도합니다.

API polling처럼 비동기 작업 시간이 일정하지 않다면 재귀적 `setTimeout`이 더 제어하기 쉬울 수 있습니다.

---

## React에서 setTimeout 사용할 때

React 컴포넌트에서 `setTimeout`을 사용할 때는 cleanup이 중요합니다.

나쁜 예시입니다.

```tsx
function Toast({ message }: { message: string }) {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    setTimeout(() => {
      setVisible(false);
    }, 3000);
  }, []);

  if (!visible) {
    return null;
  }

  return <div>{message}</div>;
}
```

이 코드는 컴포넌트가 3초 전에 언마운트되어도 타이머 콜백이 나중에 실행될 수 있습니다.

그래서 cleanup에서 `clearTimeout`을 호출하는 편이 안전합니다.

```tsx
function Toast({ message }: { message: string }) {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    const timerId = window.setTimeout(() => {
      setVisible(false);
    }, 3000);

    return () => {
      window.clearTimeout(timerId);
    };
  }, []);

  if (!visible) {
    return null;
  }

  return <div>{message}</div>;
}
```

React에서 타이머를 쓸 때 기본 원칙은 이렇습니다.

```txt
useEffect 안에서 setTimeout을 만들었다면
cleanup에서 clearTimeout으로 정리한다.
```

---

## dependency가 바뀔 때 이전 타이머 취소하기

검색어가 바뀐 뒤 500ms 동안 추가 입력이 없을 때만 검색하고 싶다고 해봅시다.

이것은 debounce 패턴입니다.

```tsx
function SearchBox() {
  const [keyword, setKeyword] = useState("");

  useEffect(() => {
    if (!keyword) {
      return;
    }

    const timerId = window.setTimeout(() => {
      search(keyword);
    }, 500);

    return () => {
      window.clearTimeout(timerId);
    };
  }, [keyword]);

  return (
    <input
      value={keyword}
      onChange={(event) => setKeyword(event.target.value)}
    />
  );
}
```

흐름은 이렇습니다.

```txt
r 입력
→ 500ms 뒤 search("r") 예약

re 입력
→ 이전 타이머 취소
→ 500ms 뒤 search("re") 예약

rea 입력
→ 이전 타이머 취소
→ 500ms 뒤 search("rea") 예약
```

입력이 계속 바뀌면 이전 타이머는 cleanup에서 취소됩니다.

마지막 입력 이후 500ms가 지나야 검색이 실행됩니다.

이 패턴은 `setTimeout`의 대표적인 실무 활용입니다.

---

## 직접 debounce 함수 만들기

`setTimeout`을 이해하면 debounce를 직접 만들 수 있습니다.

```js
function debounce(fn, delay) {
  let timerId;

  return (...args) => {
    clearTimeout(timerId);

    timerId = setTimeout(() => {
      fn(...args);
    }, delay);
  };
}
```

사용 예시입니다.

```js
const search = (keyword) => {
  console.log("검색:", keyword);
};

const debouncedSearch = debounce(search, 500);

debouncedSearch("r");
debouncedSearch("re");
debouncedSearch("rea");
debouncedSearch("react");
```

마지막 호출인 `"react"`만 500ms 뒤 실행됩니다.

왜 그럴까요?

매번 새 호출이 들어올 때 이전 타이머를 취소하기 때문입니다.

```txt
호출
→ 이전 타이머 취소
→ 새 타이머 예약
```

debounce는 `setTimeout`과 `clearTimeout` 조합으로 만들어지는 대표적인 패턴입니다.

---

## stale closure 주의하기

React에서 타이머를 사용할 때 오래된 값을 참조하는 문제가 생길 수 있습니다.

아래 코드를 봅시다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setTimeout(() => {
      console.log(count);
    }, 1000);
  }

  return (
    <button type="button" onClick={handleClick}>
      {count}
    </button>
  );
}
```

버튼을 누른 시점의 `count`가 1이라면, 1초 뒤에도 그 콜백은 1을 기억합니다.

그 사이에 `count`가 5로 바뀌어도 예약된 콜백 안의 `count`는 버튼을 누르던 렌더링의 값일 수 있습니다.

이것은 closure 때문입니다.

항상 최신 값을 써야 한다면 `useRef`를 사용할 수 있습니다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count;
  }, [count]);

  function handleClick() {
    setTimeout(() => {
      console.log(countRef.current);
    }, 1000);
  }

  return (
    <button type="button" onClick={() => setCount((prev) => prev + 1)}>
      {count}
    </button>
  );
}
```

반대로 "클릭한 순간의 값"을 기억해야 하는 경우라면 기존 closure 동작이 오히려 맞습니다.

중요한 것은 의도를 구분하는 것입니다.

```txt
예약한 순간의 값을 써야 하는가?
실행되는 순간의 최신 값을 써야 하는가?
```

---

## 상태 업데이트에는 updater function이 안전하다

타이머 안에서 이전 상태를 기반으로 업데이트할 때는 updater function이 안전합니다.

```tsx
setTimeout(() => {
  setCount(count + 1);
}, 1000);
```

이 코드는 `count`가 오래된 값일 수 있습니다.

이럴 때는 이렇게 작성합니다.

```tsx
setTimeout(() => {
  setCount((prevCount) => prevCount + 1);
}, 1000);
```

`prevCount`는 React가 최신 상태를 기준으로 전달해줍니다.

그래서 타이머, 이벤트 핸들러, 비동기 콜백에서 이전 상태 기반 업데이트를 할 때 자주 사용하는 패턴입니다.

---

## setTimeout과 requestAnimationFrame 비교

화면 렌더링 타이밍과 관련된 작업에는 `setTimeout`보다 `requestAnimationFrame`이 더 적합할 수 있습니다.

예를 들어 DOM을 측정하거나 스크롤 위치를 조정하거나 애니메이션을 만들 때입니다.

```js
setTimeout(() => {
  element.scrollIntoView();
}, 0);
```

이 코드가 동작할 때도 있지만, 브라우저 렌더링 타이밍과 정확히 맞는다는 보장은 없습니다.

이런 경우에는 `requestAnimationFrame`을 고려할 수 있습니다.

```js
requestAnimationFrame(() => {
  element.scrollIntoView();
});
```

비교하면 이렇습니다.

| 상황 | 추천 |
| --- | --- |
| 몇 초 뒤 작업 실행 | `setTimeout` |
| 예약된 작업 취소 | `setTimeout` + `clearTimeout` |
| 입력이 멈춘 뒤 실행 | `setTimeout` 기반 debounce |
| 화면 그리기 직전 DOM 작업 | `requestAnimationFrame` |
| 애니메이션 프레임 제어 | `requestAnimationFrame` |
| Promise 후속 처리 | `Promise.then`, `queueMicrotask` |

`setTimeout`은 시간 지연 도구입니다.

`requestAnimationFrame`은 렌더링 프레임에 맞춘 도구입니다.

둘은 비슷해 보이지만 목적이 다릅니다.

---

## setTimeout으로 sleep 만들기

`setTimeout`은 Promise와 함께 사용해서 `sleep` 함수를 만들 수 있습니다.

```js
function sleep(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
```

사용 예시입니다.

```js
async function run() {
  console.log("시작");

  await sleep(1000);

  console.log("1초 뒤");
}
```

이 패턴은 테스트, 데모, 재시도 간격, 애니메이션 시퀀스 등에서 사용할 수 있습니다.

다만 실제 서비스 로직에서 무작정 `sleep`으로 타이밍을 맞추는 것은 조심해야 합니다.

```txt
서버 응답이 끝났는지 기다려야 한다
→ Promise나 상태를 기다리는 구조가 더 적합

DOM이 렌더링된 뒤 작업해야 한다
→ requestAnimationFrame이나 effect 흐름 검토

단순히 일정 시간 쉬어야 한다
→ sleep 사용 가능
```

---

## AbortController와 함께 취소 가능한 sleep 만들기

조금 더 실무적으로는 취소 가능한 sleep이 필요할 수 있습니다.

```js
function sleep(ms, signal) {
  return new Promise((resolve, reject) => {
    if (signal?.aborted) {
      reject(new DOMException("Aborted", "AbortError"));
      return;
    }

    const timerId = setTimeout(resolve, ms);

    signal?.addEventListener(
      "abort",
      () => {
        clearTimeout(timerId);
        reject(new DOMException("Aborted", "AbortError"));
      },
      { once: true },
    );
  });
}
```

사용 예시입니다.

```js
const controller = new AbortController();

sleep(3000, controller.signal)
  .then(() => {
    console.log("완료");
  })
  .catch((error) => {
    if (error.name === "AbortError") {
      console.log("취소됨");
    }
  });

controller.abort();
```

이렇게 하면 타이머도 비동기 작업처럼 취소 흐름을 가질 수 있습니다.

---

## 실무에서 자주 쓰는 패턴

### 1. 토스트 자동 닫기

```tsx
useEffect(() => {
  const timerId = window.setTimeout(() => {
    closeToast(id);
  }, 3000);

  return () => {
    window.clearTimeout(timerId);
  };
}, [id, closeToast]);
```

### 2. 검색 debounce

```tsx
useEffect(() => {
  const timerId = window.setTimeout(() => {
    search(keyword);
  }, 500);

  return () => {
    window.clearTimeout(timerId);
  };
}, [keyword]);
```

### 3. 버튼 중복 클릭 잠깐 막기

```tsx
function SubmitButton() {
  const [disabled, setDisabled] = useState(false);

  function handleClick() {
    setDisabled(true);

    setTimeout(() => {
      setDisabled(false);
    }, 1000);
  }

  return (
    <button type="button" disabled={disabled} onClick={handleClick}>
      제출
    </button>
  );
}
```

다만 API 중복 요청 방지는 타이머보다 `isSubmitting`, `isPending` 같은 실제 요청 상태로 막는 편이 더 안전합니다.

### 4. 에러 메시지 잠깐 보여주기

```tsx
useEffect(() => {
  if (!errorMessage) {
    return;
  }

  const timerId = window.setTimeout(() => {
    clearErrorMessage();
  }, 3000);

  return () => {
    window.clearTimeout(timerId);
  };
}, [errorMessage]);
```

---

## setTimeout을 피해야 하는 경우

`setTimeout`은 편하지만, 모든 문제의 답은 아닙니다.

다음 상황에서는 다른 방식이 더 적합할 수 있습니다.

### 1. 데이터 로딩 완료를 기다릴 때

나쁜 예시입니다.

```js
setTimeout(() => {
  renderData();
}, 1000);
```

서버가 1초 안에 응답한다는 보장은 없습니다.

데이터 로딩은 Promise나 async/await 흐름으로 처리해야 합니다.

```js
const data = await fetchData();
renderData(data);
```

### 2. DOM 렌더링 타이밍을 억지로 맞출 때

```js
setTimeout(() => {
  measureElement();
}, 0);
```

DOM 측정이나 스크롤 보정이라면 `requestAnimationFrame`, `useLayoutEffect`, `ResizeObserver` 같은 도구가 더 적합할 수 있습니다.

### 3. 복잡한 상태 전환을 시간으로 때울 때

```js
setTimeout(() => {
  setStep("next");
}, 500);
```

애니메이션 종료 후 상태를 바꾸고 싶다면 `transitionend`, 애니메이션 라이브러리의 callback, 상태 머신 같은 구조가 더 명확할 수 있습니다.

타이머는 실제 원인을 기다리는 것이 아니라 시간을 기다리는 도구입니다.

그래서 "무엇이 끝났는지"가 중요하다면 이벤트나 Promise를 기다리는 편이 더 안전합니다.

---

## 흔한 실수 정리

### 1. setTimeout이 동기 코드를 멈춘다고 생각한다

```js
setTimeout(() => {
  value = 1;
}, 1000);

console.log(value);
```

타이머 콜백은 나중에 실행됩니다.

### 2. clearTimeout을 안 한다

React 컴포넌트에서 타이머를 만들었다면 cleanup에서 정리해야 합니다.

```tsx
return () => {
  clearTimeout(timerId);
};
```

### 3. setTimeout 0을 즉시 실행으로 오해한다

```js
setTimeout(callback, 0);
```

즉시 실행이 아니라 현재 call stack 이후 task queue에서 실행됩니다.

### 4. Promise보다 먼저 실행될 거라고 생각한다

```js
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise"));
```

일반적으로 `promise`가 먼저 출력됩니다.

### 5. 오래된 state를 참조한다

```tsx
setTimeout(() => {
  setCount(count + 1);
}, 1000);
```

이전 상태 기반 업데이트라면 updater function을 고려합니다.

```tsx
setCount((prev) => prev + 1);
```

---

## 정리

`setTimeout`은 지정한 시간이 지난 뒤 콜백을 한 번 실행하도록 예약하는 함수입니다.

하지만 정확히 이해하려면 아래 문장이 더 중요합니다.

```txt
setTimeout은 지정한 시간이 지난 뒤 콜백을 task queue에 넣고,
call stack이 비었을 때 실행되도록 예약한다.
```

그래서 delay는 "정확한 실행 시각"이 아니라 "최소 지연 시간"에 가깝습니다.

핵심을 정리하면 이렇습니다.

```txt
setTimeout은 동기 코드를 멈추지 않는다.
setTimeout 0은 즉시 실행이 아니다.
Promise.then은 setTimeout보다 먼저 실행될 수 있다.
clearTimeout으로 실행 전 예약을 취소할 수 있다.
React에서는 useEffect cleanup에서 타이머를 정리해야 한다.
debounce는 setTimeout과 clearTimeout으로 만들 수 있다.
DOM 렌더링 타이밍에는 requestAnimationFrame이 더 적합할 수 있다.
```

`setTimeout`은 단순한 타이머 함수처럼 보이지만, 이벤트 루프와 함께 이해하면 JavaScript 비동기 흐름을 보는 눈이 훨씬 선명해집니다.

---

## 참고 자료

- [MDN - Window: setTimeout()](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)
- [MDN - Using microtasks in JavaScript](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
- [MDN - Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [WHATWG HTML Standard - Timers](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html)

{% endraw %}
