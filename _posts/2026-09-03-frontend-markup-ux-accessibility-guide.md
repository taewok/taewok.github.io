---
title: "[HTML] 프론트엔드가 알아야 할 마크업 UX 기술"
date: 2026-09-03T01:00:00Z
categories: [html, frontend]
tags: [html, accessibility, markup, ux, tabindex, aria]
description: "프론트엔드 개발자가 사용자 경험을 해치지 않는 마크업을 작성하기 위해 알아야 하는 tabindex, focus, label, button, a, aria 속성, hidden, disabled 같은 기초 UX 지식을 정리했습니다."
custom_style: true
---

## 들어가며: 마크업은 화면에 보이는 구조만 만드는 게 아니에요

프론트엔드 개발을 하다 보면 HTML을 단순히 화면에 요소를 배치하는 도구처럼 생각하기 쉽습니다.

```txt
버튼처럼 보이면 버튼
링크처럼 보이면 링크
input처럼 보이면 input
```

하지만 실제 서비스에서는 보이는 모양만큼이나 중요한 것이 있습니다.

사용자가 그 요소를 **어떻게 찾고, 이동하고, 이해하고, 조작하는지**입니다.

예를 들어 이런 상황을 생각해볼 수 있습니다.

```html
<div class="button" onclick="submitForm()">저장</div>
```

화면에는 버튼처럼 보일 수 있습니다.

하지만 키보드로 `Tab` 이동이 되지 않을 수 있고, `Enter`나 `Space` 키로 실행되지 않을 수 있습니다. 스크린 리더 사용자에게도 이것이 버튼인지 명확하지 않을 수 있습니다.

이런 문제는 JavaScript 로직보다 마크업 단계에서 먼저 생깁니다.

그래서 프론트엔드 개발자는 CSS와 JavaScript만큼이나 **마크업 UX 지식**을 알아야 합니다.

이번 글에서는 실무에서 자주 만나는 마크업 UX 지식을 정리해보겠습니다.

```txt
1. tabindex와 포커스 순서
2. focus-visible과 키보드 사용자
3. button과 a의 차이
4. label과 form 접근성
5. aria-label, aria-labelledby, aria-describedby
6. aria-expanded, aria-controls
7. hidden, aria-hidden, display none
8. disabled와 aria-disabled
9. alt, title, placeholder의 차이
10. skip link와 live region
```

---

## 마크업 UX란?

마크업 UX는 HTML 구조와 속성만으로도 사용자가 더 편하게 화면을 이해하고 조작할 수 있게 만드는 기술입니다.

조금 더 쉽게 말하면 이렇습니다.

```txt
마크업 UX는
HTML을 사용자의 행동 흐름에 맞게 작성하는 일입니다.
```

좋은 마크업은 다음 사용자에게 모두 도움이 됩니다.

- 마우스로 조작하는 사용자
- 키보드만 사용하는 사용자
- 스크린 리더를 사용하는 사용자
- 모바일에서 터치로 조작하는 사용자
- 검색 엔진
- 나중에 코드를 읽는 개발자

반대로 마크업이 잘못되면 화면은 멀쩡해 보여도 사용자는 불편을 겪습니다.

```txt
Tab을 눌렀는데 예상하지 못한 곳으로 이동함
버튼처럼 생겼는데 키보드로 누를 수 없음
input의 이름을 알 수 없음
아이콘 버튼이 무슨 기능인지 읽히지 않음
닫힌 메뉴 안의 링크가 포커스됨
```

이런 문제는 작은 속성 하나로 해결되는 경우가 많습니다.

---

## focus란?

`focus`는 현재 키보드 입력을 받을 수 있는 요소를 의미합니다.

예를 들어 input을 클릭하면 커서가 생기고 글자를 입력할 수 있습니다.

이때 그 input은 focus 상태입니다.

```html
<input type="text" />
```

버튼도 focus될 수 있습니다.

```html
<button type="button">저장</button>
```

키보드 사용자는 보통 `Tab` 키로 focus를 이동합니다.

```txt
Tab
→ 다음 focus 가능한 요소로 이동

Shift + Tab
→ 이전 focus 가능한 요소로 이동

Enter 또는 Space
→ 현재 focus된 버튼 실행
```

즉, focus는 키보드 사용자의 마우스 커서 같은 역할을 합니다.

그래서 focus가 어디에 있는지 보이지 않거나, focus 순서가 이상하면 UX가 크게 나빠집니다.

---

## tabindex란?

`tabindex`는 요소가 키보드 focus를 받을 수 있는지, 그리고 Tab 순서에 들어갈지를 제어하는 HTML 전역 속성입니다.

MDN에서는 `tabindex`가 요소를 focus 가능하게 만들거나, 순차 focus 탐색에서 포함하거나 제외하고, focus 순서를 정할 수 있게 해주는 속성이라고 설명합니다.

```html
<div tabindex="0">키보드로 focus 가능한 div</div>
```

`tabindex`에서 가장 많이 알아야 할 값은 세 가지입니다.

| 값 | 의미 |
| --- | --- |
| `tabindex="0"` | 자연스러운 Tab 순서에 포함 |
| `tabindex="-1"` | Tab으로는 접근 불가, JavaScript로 focus 가능 |
| `tabindex="1"` 이상 | 강제로 우선순위 지정 |

하나씩 보겠습니다.

---

## tabindex="0": 자연스러운 순서에 넣기

`tabindex="0"`은 원래 focus되지 않는 요소를 자연스러운 Tab 순서에 넣습니다.

```html
<div tabindex="0">공지사항 카드</div>
```

이렇게 하면 `Tab` 키로 이 `div`에 이동할 수 있습니다.

하지만 아무 `div`에나 `tabindex="0"`을 붙이는 것은 좋지 않습니다.

focus 가능하다는 것은 사용자가 뭔가 조작할 수 있을 것이라고 기대하게 만들기 때문입니다.

`tabindex="0"`은 보통 이런 경우에 사용합니다.

- 직접 만든 커스텀 위젯
- JavaScript로 조작되는 탭 메뉴
- 키보드로 선택 가능한 카드
- focus를 받아야 하는 스크롤 영역

단순히 디자인 때문에 focus를 주고 싶다면 다시 생각해봐야 합니다.

```txt
이 요소가 키보드 사용자가 멈춰서 조작하거나 읽어야 하는 대상인가?
그렇다면 tabindex="0"을 고려한다.
```

---

## tabindex="-1": 코드로만 focus 이동하기

`tabindex="-1"`은 Tab 순서에는 들어가지 않지만, JavaScript의 `.focus()`로는 focus할 수 있게 만듭니다.

```html
<section id="content" tabindex="-1">
  <h1>본문 제목</h1>
  <p>본문 내용...</p>
</section>
```

```js
const content = document.querySelector("#content");

content.focus();
```

이 패턴은 다음 상황에서 유용합니다.

- 모달이 열렸을 때 모달 컨테이너로 focus 이동
- skip link를 눌렀을 때 main 영역으로 focus 이동
- 페이지 이동 후 본문 제목으로 focus 이동
- 에러가 발생했을 때 에러 요약 영역으로 focus 이동

예를 들어 본문 바로가기 링크를 만들 때 사용할 수 있습니다.

```html
<a class="skip-link" href="#main">본문 바로가기</a>

<main id="main" tabindex="-1">
  <h1>글 제목</h1>
  <p>본문 내용...</p>
</main>
```

`main`은 평소 Tab 순서에 들어갈 필요는 없지만, skip link로 이동했을 때 focus를 받을 수 있어야 합니다.

그럴 때 `tabindex="-1"`이 잘 맞습니다.

---

## tabindex 양수는 거의 쓰지 않기

`tabindex="1"`, `tabindex="2"`처럼 양수를 주면 기본 DOM 순서보다 먼저 focus됩니다.

```html
<button tabindex="2">두 번째</button>
<button tabindex="1">첫 번째</button>
```

이렇게 하면 화면과 DOM 순서와 focus 순서가 달라집니다.

처음에는 편해 보일 수 있지만, 페이지가 커지면 focus 흐름을 예측하기 어려워집니다.

```txt
화면은 위에서 아래로 보이는데
Tab은 엉뚱한 순서로 이동함
```

대부분의 경우 양수 `tabindex`는 피하고, HTML 순서를 올바르게 작성하는 것이 좋습니다.

```html
<button>첫 번째</button>
<button>두 번째</button>
```

focus 순서를 고치고 싶다면 `tabindex` 숫자를 억지로 조정하기보다 DOM 구조를 먼저 고치는 편이 안전합니다.

---

## focus-visible: focus 표시를 없애지 않기

초보자 코드에서 자주 보이는 CSS가 있습니다.

```css
*:focus {
  outline: none;
}
```

디자인상 파란 테두리가 거슬려서 지우는 경우가 많습니다.

하지만 이 코드는 키보드 사용자에게 아주 위험합니다.

focus 표시가 사라지면 현재 어디에 있는지 알 수 없기 때문입니다.

WCAG의 Focus Visible 기준도 키보드 focus가 보이도록 하는 것을 중요하게 다룹니다.

대신 `:focus-visible`을 사용하면 키보드 조작 시에만 focus 스타일을 더 자연스럽게 보여줄 수 있습니다.

```css
button:focus-visible,
a:focus-visible,
input:focus-visible,
textarea:focus-visible,
select:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 3px;
}
```

이렇게 하면 마우스로 클릭할 때는 과하게 보이지 않으면서, 키보드로 이동할 때는 focus 위치가 명확하게 보입니다.

중요한 원칙은 이렇습니다.

```txt
outline을 지우려면
반드시 더 명확한 focus 스타일을 대신 제공한다.
```

---

## button과 a는 역할이 다릅니다

버튼과 링크는 둘 다 클릭할 수 있습니다.

그래서 마크업에서 자주 섞입니다.

하지만 기준은 분명합니다.

```txt
다른 페이지나 위치로 이동한다
→ a

현재 화면에서 어떤 동작을 실행한다
→ button
```

페이지 이동은 `a`입니다.

```html
<a href="/posts">글 목록 보기</a>
```

모달 열기, 저장, 삭제, 토글은 `button`입니다.

```html
<button type="button">모달 열기</button>
<button type="submit">저장</button>
```

다음처럼 `div`에 클릭 이벤트를 붙여 버튼처럼 만드는 것은 피하는 것이 좋습니다.

```html
<!-- 좋지 않음 -->
<div class="button" onclick="openModal()">모달 열기</div>
```

이렇게 하면 직접 처리해야 할 것이 많아집니다.

- focus 가능하게 만들기
- Enter 키 처리
- Space 키 처리
- role 부여
- disabled 상태 처리
- 스크린 리더 이름 제공

반면 `button`은 기본적으로 많은 동작을 이미 가지고 있습니다.

```html
<button type="button">모달 열기</button>
```

좋은 마크업 UX의 기본은 브라우저가 이미 잘 만들어둔 네이티브 요소를 사용하는 것입니다.

---

## button에는 type을 명시하기

`button`을 사용할 때는 `type`을 명시하는 습관이 좋습니다.

```html
<button type="button">모달 열기</button>
```

폼 안의 `button`은 기본값이 `submit`입니다.

그래서 단순 버튼이라고 생각하고 넣었는데 form이 제출되는 문제가 생길 수 있습니다.

```html
<form>
  <input type="text" />

  <!-- 기본적으로 submit 버튼처럼 동작할 수 있음 -->
  <button>닫기</button>
</form>
```

폼을 제출하는 버튼이라면 `submit`을 명시합니다.

```html
<button type="submit">저장</button>
```

일반 동작 버튼이라면 `button`을 명시합니다.

```html
<button type="button">닫기</button>
```

작은 습관이지만 의도하지 않은 제출을 막는 데 도움이 됩니다.

---

## label은 input의 이름입니다

폼 UX에서 가장 중요한 태그 중 하나가 `label`입니다.

```html
<label for="email">이메일</label>
<input id="email" name="email" type="email" />
```

`label`은 사용자에게 이 입력칸이 무엇을 입력하는 곳인지 알려줍니다.

그리고 `for`와 `id`를 연결하면 label을 클릭했을 때 input에 focus가 이동합니다.

```txt
label 클릭
→ 연결된 input으로 focus 이동
```

스크린 리더도 label을 통해 input의 이름을 읽을 수 있습니다.

즉, label은 단순한 텍스트가 아니라 input의 접근 가능한 이름입니다.

placeholder만 label처럼 사용하는 것은 좋지 않습니다.

```html
<!-- 좋지 않음 -->
<input type="email" placeholder="이메일" />
```

placeholder는 입력을 시작하면 사라지고, 값이 들어간 뒤에는 사용자가 이 필드가 무엇인지 다시 확인하기 어렵습니다.

가능하면 보이는 label을 제공합니다.

```html
<label for="email">이메일</label>
<input id="email" name="email" type="email" placeholder="name@example.com" />
```

placeholder는 label 대신이 아니라 입력 예시로 사용하는 것이 좋습니다.

---

## input type을 잘 고르면 UX가 좋아집니다

모든 입력칸을 `type="text"`로 만들 수도 있습니다.

하지만 입력 목적에 맞는 type을 고르면 브라우저가 더 좋은 UX를 제공합니다.

```html
<input type="email" />
<input type="password" />
<input type="number" />
<input type="tel" />
<input type="search" />
<input type="date" />
```

예를 들어 모바일에서 `type="email"`은 이메일 입력에 맞는 키보드를 보여줄 수 있습니다.

```html
<label for="email">이메일</label>
<input id="email" name="email" type="email" autocomplete="email" />
```

전화번호라면 `type="tel"`이 자연스럽습니다.

```html
<label for="phone">전화번호</label>
<input id="phone" name="phone" type="tel" autocomplete="tel" />
```

검색창이라면 `type="search"`를 사용할 수 있습니다.

```html
<label for="keyword">검색어</label>
<input id="keyword" name="keyword" type="search" />
```

마크업만 잘 작성해도 모바일 키보드, 자동완성, 기본 검증 같은 UX가 좋아집니다.

---

## autocomplete도 사용자 경험입니다

회원가입이나 결제 폼에서 `autocomplete`은 꽤 중요합니다.

브라우저가 사용자의 정보를 자동완성할 수 있기 때문입니다.

```html
<label for="name">이름</label>
<input id="name" name="name" autocomplete="name" />

<label for="email">이메일</label>
<input id="email" name="email" type="email" autocomplete="email" />

<label for="password">비밀번호</label>
<input
  id="password"
  name="password"
  type="password"
  autocomplete="current-password"
/>
```

새 비밀번호를 만드는 화면이라면 `new-password`를 사용할 수 있습니다.

```html
<input type="password" autocomplete="new-password" />
```

자동완성을 막기보다, 브라우저가 어떤 값인지 이해할 수 있게 적절한 값을 주는 편이 사용자에게 편합니다.

---

## aria-label: 보이는 텍스트가 없을 때 이름 주기

아이콘만 있는 버튼은 화면을 보는 사람에게는 의미가 있을 수 있습니다.

하지만 스크린 리더는 아이콘 모양을 그대로 이해하지 못합니다.

```html
<!-- 좋지 않음 -->
<button type="button">
  <svg>...</svg>
</button>
```

이럴 때 `aria-label`로 버튼의 이름을 줄 수 있습니다.

```html
<button type="button" aria-label="검색">
  <svg aria-hidden="true">...</svg>
</button>
```

`aria-label`은 화면에는 보이지 않지만 보조 기술이 읽을 수 있는 이름을 제공합니다.

다만 보이는 텍스트가 이미 있다면 굳이 `aria-label`을 반복해서 넣을 필요는 없습니다.

```html
<!-- 충분함 -->
<button type="button">검색</button>
```

초보 단계에서는 이렇게 기억하면 좋습니다.

```txt
보이는 텍스트가 있으면 그 텍스트를 사용한다.
아이콘만 있다면 aria-label을 고려한다.
```

---

## aria-labelledby와 aria-describedby

`aria-labelledby`는 다른 요소의 텍스트를 이 요소의 이름으로 연결합니다.

```html
<h2 id="dialog-title">프로필 수정</h2>

<div role="dialog" aria-labelledby="dialog-title">
  ...
</div>
```

이 경우 dialog의 이름은 `프로필 수정`이 됩니다.

`aria-describedby`는 더 긴 설명을 연결할 때 사용합니다.

```html
<label for="password">비밀번호</label>
<input
  id="password"
  type="password"
  aria-describedby="password-help"
/>

<p id="password-help">
  영문, 숫자, 특수문자를 포함해 8자 이상 입력해주세요.
</p>
```

둘의 차이는 이렇게 보면 됩니다.

| 속성 | 역할 |
| --- | --- |
| `aria-labelledby` | 짧은 이름 |
| `aria-describedby` | 부가 설명 |

input의 이름은 label이 담당하고, 추가 안내 문구는 `aria-describedby`로 연결하는 구조가 좋습니다.

---

## aria-expanded와 aria-controls

드롭다운, 아코디언, 메뉴처럼 열리고 닫히는 UI에서는 현재 상태를 알려주는 것이 중요합니다.

```html
<button
  type="button"
  aria-expanded="false"
  aria-controls="category-menu"
>
  카테고리
</button>

<ul id="category-menu" hidden>
  <li><a href="/html">HTML</a></li>
  <li><a href="/css">CSS</a></li>
  <li><a href="/javascript">JavaScript</a></li>
</ul>
```

메뉴가 열리면 이렇게 바뀝니다.

```html
<button
  type="button"
  aria-expanded="true"
  aria-controls="category-menu"
>
  카테고리
</button>

<ul id="category-menu">
  <li><a href="/html">HTML</a></li>
  <li><a href="/css">CSS</a></li>
  <li><a href="/javascript">JavaScript</a></li>
</ul>
```

`aria-expanded`는 현재 펼쳐져 있는지 알려줍니다.

`aria-controls`는 이 버튼이 어떤 영역을 제어하는지 연결합니다.

JavaScript에서는 열림 상태에 맞춰 이 값을 함께 바꿔야 합니다.

```js
button.setAttribute("aria-expanded", "true");
menu.hidden = false;
```

UI가 바뀌면 시각적인 상태뿐 아니라 접근성 상태도 함께 바꿔야 합니다.

---

## hidden, display none, aria-hidden 차이

요소를 숨기는 방법도 여러 가지가 있습니다.

```html
<div hidden>숨겨진 내용</div>
```

```css
.hidden {
  display: none;
}
```

```html
<div aria-hidden="true">보조 기술에서 숨김</div>
```

차이는 이렇게 볼 수 있습니다.

| 방법 | 화면 | 접근성 트리 |
| --- | --- | --- |
| `hidden` | 보이지 않음 | 보통 노출되지 않음 |
| `display: none` | 보이지 않음 | 보통 노출되지 않음 |
| `aria-hidden="true"` | 보일 수 있음 | 보조 기술에서 숨김 |

`aria-hidden="true"`는 화면에서 숨기는 속성이 아닙니다.

보조 기술에게 이 요소를 무시하라고 알려주는 속성입니다.

그래서 focus 가능한 요소에 `aria-hidden="true"`를 주는 것은 피해야 합니다.

```html
<!-- 좋지 않음 -->
<button type="button" aria-hidden="true">삭제</button>
```

화면에는 보이고 focus도 될 수 있는데, 스크린 리더에서는 사라진 것처럼 처리될 수 있기 때문입니다.

장식용 아이콘처럼 의미가 없는 요소에는 사용할 수 있습니다.

```html
<button type="button">
  <svg aria-hidden="true">...</svg>
  삭제
</button>
```

아이콘은 숨기고, 버튼 텍스트 `삭제`는 그대로 읽히게 하는 구조입니다.

---

## disabled와 aria-disabled

버튼을 비활성화할 때는 보통 `disabled`를 사용합니다.

```html
<button type="submit" disabled>저장</button>
```

`disabled`가 붙은 버튼은 클릭되지 않고, 보통 Tab focus 순서에서도 제외됩니다.

이 동작은 대부분 자연스럽습니다.

하지만 어떤 컴포넌트에서는 비활성 상태를 사용자에게 설명하기 위해 focus는 가능하게 두고 싶을 때가 있습니다.

그럴 때는 `aria-disabled="true"`를 사용할 수 있습니다.

```html
<button type="button" aria-disabled="true">
  다음 단계
</button>
```

주의할 점은 `aria-disabled`는 의미만 전달합니다.

실제 클릭을 막아주지는 않습니다.

```js
button.addEventListener("click", (event) => {
  if (button.getAttribute("aria-disabled") === "true") {
    event.preventDefault();
    return;
  }

  goNext();
});
```

대부분의 기본 버튼에는 `disabled`가 더 적절합니다.

`aria-disabled`는 커스텀 위젯이나 focus 유지가 필요한 특별한 상황에서 고려하면 됩니다.

---

## alt: 이미지를 설명하는 텍스트

이미지에는 `alt`를 작성해야 합니다.

```html
<img src="/profile.jpg" alt="김개발 프로필 사진" />
```

`alt`는 이미지가 보이지 않거나 스크린 리더로 읽을 때 이미지의 대체 텍스트 역할을 합니다.

하지만 모든 이미지에 자세한 설명을 넣어야 하는 것은 아닙니다.

장식용 이미지는 빈 `alt`를 사용할 수 있습니다.

```html
<img src="/decorative-line.png" alt="" />
```

빈 `alt`는 이 이미지가 중요한 정보가 아니라는 뜻입니다.

반대로 정보가 있는 이미지라면 의미를 전달해야 합니다.

```html
<img
  src="/chart.png"
  alt="2026년 1분기부터 4분기까지 방문자가 꾸준히 증가한 그래프"
/>
```

좋은 `alt`는 파일 이름을 설명하는 것이 아니라 이미지가 전달하는 정보를 설명합니다.

---

## title은 좋은 설명 방법이 아닐 때가 많아요

HTML에는 `title` 속성이 있습니다.

```html
<button type="button" title="삭제">
  X
</button>
```

마우스를 올리면 툴팁처럼 보일 수 있습니다.

하지만 `title`은 모바일이나 키보드 사용자에게 일관되게 전달되지 않을 수 있습니다.

아이콘 버튼의 이름을 제공하려면 `title`보다 `aria-label`이나 보이는 텍스트가 더 명확합니다.

```html
<button type="button" aria-label="삭제">
  <svg aria-hidden="true">...</svg>
</button>
```

가능하면 중요한 정보를 `title`에만 의존하지 않는 편이 좋습니다.

---

## placeholder는 label이 아닙니다

placeholder는 입력 예시나 힌트입니다.

```html
<input placeholder="name@example.com" />
```

하지만 label 대신 사용하면 문제가 생깁니다.

```html
<!-- 좋지 않음 -->
<input placeholder="이메일" />
```

사용자가 값을 입력하면 placeholder는 사라집니다.

그러면 사용자는 이 필드가 무엇이었는지 다시 확인하기 어렵습니다.

더 좋은 구조는 label과 placeholder를 함께 쓰는 것입니다.

```html
<label for="email">이메일</label>
<input id="email" type="email" placeholder="name@example.com" />
```

label은 필드 이름이고, placeholder는 입력 예시입니다.

---

## 오류 메시지는 input과 연결하기

폼 검증 오류를 보여줄 때 화면에 빨간 글씨만 추가하면 충분하지 않을 수 있습니다.

input과 오류 메시지를 연결해주는 것이 좋습니다.

```html
<label for="email">이메일</label>
<input
  id="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>

<p id="email-error">올바른 이메일 형식으로 입력해주세요.</p>
```

여기서 `aria-invalid="true"`는 현재 값이 유효하지 않다는 상태를 나타냅니다.

`aria-describedby`는 오류 메시지와 input을 연결합니다.

이렇게 하면 사용자가 input에 focus했을 때 어떤 문제가 있는지 더 잘 이해할 수 있습니다.

---

## live region: 동적으로 바뀌는 메시지 알려주기

화면의 내용이 JavaScript로 바뀌었을 때, 스크린 리더가 그 변화를 자동으로 알아차리지 못할 수 있습니다.

예를 들어 저장이 완료되었을 때 토스트 메시지를 띄운다고 해보겠습니다.

```html
<p class="toast">저장되었습니다.</p>
```

시각적으로는 보이지만 보조 기술 사용자에게는 변화가 전달되지 않을 수 있습니다.

이럴 때 `aria-live`를 사용할 수 있습니다.

```html
<div aria-live="polite" id="status-message"></div>
```

```js
const statusMessage = document.querySelector("#status-message");

statusMessage.textContent = "저장되었습니다.";
```

`aria-live="polite"`는 현재 읽고 있는 흐름을 방해하지 않고 적절한 시점에 변경 내용을 알려줍니다.

에러처럼 더 즉각적으로 알려야 하는 경우에는 `role="alert"`를 사용할 수 있습니다.

```html
<p role="alert">결제에 실패했습니다. 다시 시도해주세요.</p>
```

다만 너무 많은 메시지를 live region으로 보내면 오히려 방해가 될 수 있습니다.

정말 사용자가 알아야 하는 상태 변화에만 사용하는 것이 좋습니다.

---

## skip link: 반복 메뉴 건너뛰기

키보드 사용자는 페이지를 이동할 때마다 상단 메뉴를 계속 지나가야 할 수 있습니다.

이때 본문으로 바로 이동하는 링크를 제공하면 좋습니다.

```html
<a class="skip-link" href="#main">본문 바로가기</a>

<header>
  <nav>
    <a href="/">홈</a>
    <a href="/posts">글</a>
    <a href="/about">소개</a>
  </nav>
</header>

<main id="main" tabindex="-1">
  <h1>페이지 제목</h1>
  <p>본문 내용...</p>
</main>
```

CSS로 평소에는 숨기고 focus될 때만 보이게 만들 수 있습니다.

```css
.skip-link {
  position: absolute;
  left: 16px;
  top: 16px;
  transform: translateY(-120%);
  background-color: #111827;
  color: white;
  padding: 8px 12px;
  border-radius: 6px;
}

.skip-link:focus-visible {
  transform: translateY(0);
}
```

이렇게 하면 키보드 사용자가 `Tab`을 눌렀을 때 본문으로 바로 이동할 수 있습니다.

작은 기능이지만 반복되는 탐색 피로를 줄여줍니다.

---

## 클릭 가능한 영역은 충분히 크게 만들기

마크업 UX는 접근성 속성만의 문제가 아닙니다.

터치와 클릭 영역도 중요합니다.

예를 들어 아이콘만 클릭 가능하게 만들면 사용하기 불편할 수 있습니다.

```html
<!-- 클릭 영역이 작을 수 있음 -->
<button type="button" aria-label="삭제">
  <svg width="16" height="16" aria-hidden="true">...</svg>
</button>
```

버튼 자체의 padding을 충분히 주는 것이 좋습니다.

```css
.icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 40px;
  min-height: 40px;
  padding: 8px;
}
```

사용자는 정확히 아이콘 선을 누르는 것이 아니라 버튼 영역을 누릅니다.

마크업은 올바른 태그를 고르고, CSS는 조작하기 편한 크기를 보장해야 합니다.

---

## 커스텀 위젯을 만들 때는 ARIA 패턴을 확인하기

탭, 메뉴, 콤보박스, 슬라이더, 트리뷰 같은 컴포넌트는 생각보다 복잡합니다.

예를 들어 탭 UI는 이런 것들을 고려해야 합니다.

- 어떤 탭이 선택되어 있는지
- 탭 버튼과 패널이 어떻게 연결되는지
- 화살표 키로 탭 사이를 이동할 수 있는지
- Tab 키는 탭 목록을 어떻게 빠져나가는지

단순히 `div`와 `onClick`으로만 만들면 키보드 UX가 부족할 수 있습니다.

복잡한 커스텀 위젯을 직접 만들 때는 WAI-ARIA Authoring Practices Guide의 패턴을 확인하는 것이 좋습니다.

초보 단계에서는 직접 모든 패턴을 외우기보다 이렇게 생각하면 충분합니다.

```txt
브라우저 기본 요소로 만들 수 있으면 기본 요소를 쓴다.
기본 요소로 어려운 복잡한 위젯이면 ARIA 패턴을 확인한다.
```

실무에서는 Radix UI, React Aria, Headless UI 같은 접근성 처리가 포함된 라이브러리를 사용하는 것도 좋은 선택입니다.

---

## 자주 하는 실수

### 1. 클릭 이벤트가 있는 div를 버튼처럼 쓰기

```html
<div onclick="save()">저장</div>
```

가능하면 `button`을 사용합니다.

```html
<button type="button">저장</button>
```

### 2. outline을 완전히 제거하기

```css
*:focus {
  outline: none;
}
```

대신 명확한 `:focus-visible` 스타일을 제공합니다.

```css
:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 3px;
}
```

### 3. tabindex 양수로 순서를 억지로 맞추기

focus 순서가 이상하다면 숫자를 주기보다 DOM 순서를 먼저 확인합니다.

```html
<!-- 피하기 -->
<button tabindex="3">저장</button>
```

### 4. label 없이 input 만들기

```html
<!-- 부족함 -->
<input placeholder="이메일" />
```

```html
<label for="email">이메일</label>
<input id="email" type="email" placeholder="name@example.com" />
```

### 5. aria를 만능 해결책처럼 쓰기

ARIA는 HTML을 보완하는 도구입니다.

네이티브 HTML로 해결할 수 있다면 먼저 HTML을 올바르게 쓰는 편이 좋습니다.

```html
<!-- 굳이 이렇게 만들기보다 -->
<div role="button" tabindex="0">저장</div>

<!-- 이렇게 쓰는 것이 좋음 -->
<button type="button">저장</button>
```

---

## 체크리스트

마크업을 작성한 뒤 아래 질문을 해보면 좋습니다.

```txt
1. 클릭 가능한 요소는 키보드로도 focus 가능한가?
2. 버튼과 링크를 역할에 맞게 사용했는가?
3. button에는 type을 명시했는가?
4. input에는 label이 연결되어 있는가?
5. placeholder를 label 대신 사용하지 않았는가?
6. 아이콘 버튼에는 접근 가능한 이름이 있는가?
7. focus 표시가 눈에 보이는가?
8. tabindex 양수를 사용하지 않았는가?
9. 열고 닫히는 UI는 aria-expanded 상태를 갱신하는가?
10. 숨긴 요소 안의 focus 가능한 요소가 Tab 순서에 남아 있지 않은가?
11. 오류 메시지는 input과 연결되어 있는가?
12. 동적 상태 메시지는 필요한 경우 aria-live로 전달되는가?
```

이 체크리스트를 습관처럼 보면 마크업 단계에서 막을 수 있는 UX 문제가 꽤 줄어듭니다.

---

## 정리

프론트엔드에서 마크업은 단순히 DOM 구조를 만드는 일이 아닙니다.

사용자가 화면을 어떻게 탐색하고, 어디에 focus가 있는지 확인하고, 어떤 요소를 조작할 수 있는지 이해하게 만드는 UX의 시작점입니다.

핵심만 다시 정리하면 다음과 같습니다.

- `tabindex="0"`은 자연스러운 Tab 순서에 요소를 포함합니다.
- `tabindex="-1"`은 JavaScript로 focus를 옮길 때 유용합니다.
- 양수 `tabindex`는 대부분 피하는 것이 좋습니다.
- focus 표시를 없애지 말고 `:focus-visible`로 명확하게 제공합니다.
- 이동은 `a`, 동작은 `button`을 사용합니다.
- `button`에는 `type`을 명시합니다.
- `input`에는 `label`을 연결합니다.
- 아이콘 버튼에는 `aria-label`을 제공합니다.
- 설명 문구와 오류 메시지는 `aria-describedby`로 연결할 수 있습니다.
- 접히고 펼쳐지는 UI는 `aria-expanded` 상태를 갱신합니다.
- `aria-hidden`은 화면 숨김이 아니라 보조 기술에서 숨김입니다.
- 중요한 동적 메시지는 `aria-live`나 `role="alert"`를 고려합니다.

한 줄로 정리하면 이렇습니다.

```txt
좋은 마크업은
사용자가 화면을 보기 전에 이미 사용 방법을 설명하고 있다
```

HTML을 조금 더 정확하게 쓰는 것만으로도 접근성, 키보드 UX, 모바일 입력 경험, 유지보수성이 함께 좋아집니다.

## 참고

- [MDN: tabindex](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/tabindex)
- [MDN: Keyboard accessible](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Keyboard)
- [MDN: Keyboard-navigable JavaScript widgets](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Keyboard-navigable_JavaScript_widgets)
- [MDN: aria-label](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-label)
- [MDN: aria-labelledby](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-labelledby)
- [MDN: ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)
- [WAI: Understanding Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html)
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
