---
title: "[JavaScript] 클래스 제어 완벽 가이드: classList (추가, 삭제, 토글)"
date: 2023-03-10T17:01:000
categories: [JavaScript]
tags: [javascript] #소문자만 가능
---

프론트엔드 개발에서 특정 요소에 애니메이션을 트리거하거나 상태(State)를 변경할 때, 클래스(Class)를 조작하는 것은 매우 빈번한 작업입니다. 과거에는 `className` 문자열을 직접 수정했지만, 이제는 더 안전하고 편리한 **classList** API를 사용합니다.

오늘은 `classList`를 활용한 클래스 읽기, 추가, 변경, 삭제 방법을 정리

---

## <b style="border-bottom:2px solid gray" class="h2">1. classList란?</b>

`classList`는 요소의 클래스 속성에 접근할 수 있는 읽기 전용 프로퍼티입니다. `element.className`이 단순한 공백 구분 문자열을 반환하는 것과 달리, `classList`는 다양한 메서드를 가진 유연한 객체(DOMTokenList)를 반환합니다.

---

## <b style="border-bottom:2px solid gray" class="h2">2. 주요 메서드 활용법</b>

### <b>add( ) : 클래스 추가</b>

중복을 허용하지 않으며, 여러 개의 클래스를 동시에 추가할 수도 있습니다.

```javascript
const box = document.querySelector("#box");

box.classList.add("active"); // 한 개 추가
box.classList.add("fade-in", "highlight"); // 여러 개 추가

console.log(box.classList);
// 출력 결과: ["active", "fade-in", "highlight"]
```

### <b>remove( ) : 클래스 삭제</b>

지정한 클래스를 삭제합니다. 존재하지 않는 클래스를 삭제하려고 해도 에러가 발생하지 않습니다.

```javascript
const box = document.querySelector("#box");
box.classList.remove("fade-in");

console.log(box.className);
// 출력 결과: "active highlight"
```

### <b>toggle( ) : 클래스 토글</b>

클래스가 있으면 제거하고, 없으면 추가합니다. `if` 조건문을 길게 쓰는 대신 한 줄로 해결할 수 있어 매우 유용합니다.

```javascript
const box = document.querySelector("#box");

// 첫 번째 호출: 클래스가 없으므로 추가됨
box.classList.toggle("hidden");

// 두 번째 호출: 클래스가 있으므로 제거됨
box.classList.toggle("hidden");
```

### <b>contains( ) : 존재 여부 확인</b>

특정 클래스가 포함되어 있는지 확인하여 `true` 또는 `false`를 반환합니다.

```javascript
if (box.classList.contains("active")) {
  console.log("현재 활성화 상태입니다.");
}
```

---

## <b style="border-bottom:2px solid gray" class="h2">3. className vs classList</b>

| 비교 항목   | className                 | classList                     |
| :---------- | :------------------------ | :---------------------------- |
| 반환 타입   | 문자열 (String)           | 객체 (DOMTokenList)           |
| 조작 방식   | 문자열 전체를 교체        | 메서드 기반 (add, remove 등)  |
| 적합한 상황 | 클래스 전체를 초기화할 때 | 클래스를 부분적으로 수정할 때 |

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<p>클래스 조작은 CSS 애니메이션을 적용하거나 사용자의 인터랙션에 반응하는 가장 직관적인 방법입니다. 특히 <code>toggle</code> 메서드는 다크모드 전환이나 메뉴 열림/닫힘 기능을 구현할 때 유용합니다.</p>

<p>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
