---
title: "[HTML] 시맨틱 태그란? 상황별로 알맞은 태그 고르는 법"
date: 2026-08-27T01:30:00Z
categories: [html]
tags: [html, semantic-html, accessibility, seo, markup]
description: "HTML 시맨틱 태그의 의미와 header, nav, main, section, article, aside, footer 같은 태그를 어떤 상황에서 사용하는지 초보자도 헷갈리지 않도록 정리했습니다."
custom_style: true
---

## 들어가며: 이럴 때 이 태그를 쓰는 게 맞을까?

HTML을 작성하다 보면 어느 순간부터 `div`만으로는 부족하다는 이야기를 듣게 됩니다.

그래서 `header`, `nav`, `main`, `section`, `article`, `aside`, `footer` 같은 시맨틱 태그를 사용하려고 하는데, 막상 쓰려고 하면 헷갈립니다.

```txt
이 영역은 section일까 article일까?
header는 페이지 맨 위에만 써야 할까?
footer는 페이지 맨 아래에만 써야 할까?
aside는 꼭 오른쪽 사이드바일 때만 쓰는 걸까?
nav는 링크만 있으면 무조건 써야 할까?
main은 여러 번 써도 될까?
```

이런 고민은 자연스럽습니다.

시맨틱 태그는 모양을 바꾸는 태그가 아니라 **콘텐츠의 의미와 역할을 설명하는 태그**이기 때문입니다.

즉, 시맨틱 태그를 고를 때는 이렇게 질문해야 합니다.

```txt
이 박스가 어떻게 보이는가?
```

가 아니라,

```txt
이 콘텐츠는 문서 안에서 어떤 역할을 하는가?
```

이번 글에서는 시맨틱 태그의 의미를 단순 목록으로 외우지 않고, 실제로 어떤 상황에서 어떤 태그를 쓰면 좋은지 판단 기준 중심으로 정리해보겠습니다.

---

## 시맨틱 태그란?

시맨틱은 의미라는 뜻입니다.

HTML에서 시맨틱 태그는 브라우저, 검색 엔진, 스크린 리더, 개발자에게 **이 콘텐츠가 어떤 역할을 하는지** 알려주는 태그입니다.

예를 들어 두 코드를 비교해보겠습니다.

```html
<div class="header">
  <div class="logo">My Blog</div>
  <div class="nav">
    <a href="/">Home</a>
    <a href="/posts">Posts</a>
  </div>
</div>
```

```html
<header>
  <h1>My Blog</h1>

  <nav>
    <a href="/">Home</a>
    <a href="/posts">Posts</a>
  </nav>
</header>
```

화면에 보이는 결과는 비슷할 수 있습니다.

하지만 의미는 다릅니다.

두 번째 코드는 브라우저와 보조 기술에게 이렇게 알려줍니다.

```txt
여기는 페이지의 소개 영역입니다.
여기는 주요 탐색 링크 영역입니다.
이 텍스트는 페이지의 큰 제목입니다.
```

즉, 시맨틱 태그는 단순히 개발자 취향 문제가 아니라 문서를 더 잘 이해할 수 있게 만드는 장치입니다.

---

## div는 나쁜 태그일까?

그렇지는 않습니다.

`div`는 아무 의미가 없는 일반 컨테이너입니다.

그래서 스타일링을 위해 묶거나, 특별한 의미가 없는 레이아웃 박스가 필요할 때는 `div`를 쓰는 것이 맞습니다.

```html
<div class="card-grid">
  <article class="card">...</article>
  <article class="card">...</article>
</div>
```

여기서 `.card-grid`는 카드들을 배치하기 위한 레이아웃 wrapper입니다.

이 wrapper 자체가 독립적인 문서 구역은 아닙니다.

이럴 때는 `section`보다 `div`가 더 자연스럽습니다.

초보 단계에서는 이렇게 기억하면 좋습니다.

```txt
의미가 있으면 시맨틱 태그
스타일이나 배치용이면 div
```

`div`를 안 쓰는 것이 목표가 아닙니다.

의미가 있는 곳에 의미 있는 태그를 쓰는 것이 목표입니다.

---

## 전체 페이지 구조 먼저 보기

가장 흔한 페이지 구조는 다음처럼 만들 수 있습니다.

```html
<body>
  <header>
    <h1>프론트엔드 블로그</h1>

    <nav>
      <a href="/">홈</a>
      <a href="/posts">글</a>
      <a href="/about">소개</a>
    </nav>
  </header>

  <main>
    <article>
      <header>
        <h2>시맨틱 태그란?</h2>
        <p>작성일: <time datetime="2026-08-27">2026년 8월 27일</time></p>
      </header>

      <section>
        <h3>시맨틱 태그를 쓰는 이유</h3>
        <p>문서의 의미를 더 명확하게 전달하기 위해 사용합니다.</p>
      </section>
    </article>
  </main>

  <aside>
    <h2>관련 글</h2>
    <a href="/posts/html-dom-element-control-guide">DOM이란?</a>
  </aside>

  <footer>
    <p>Copyright 2026. taewok.</p>
  </footer>
</body>
```

이 구조를 그림처럼 보면 이렇습니다.

```txt
body
├─ header: 사이트 소개와 주요 탐색
├─ main: 이 페이지의 핵심 콘텐츠
│  └─ article: 독립적인 글 한 편
│     ├─ header: 글 제목과 작성 정보
│     └─ section: 글 안의 한 주제
├─ aside: 본문과 관련된 보조 정보
└─ footer: 페이지의 마무리 정보
```

이제 각 태그를 하나씩 보겠습니다.

---

## header: 소개 영역

`header`는 소개 콘텐츠를 담는 태그입니다.

보통 제목, 로고, 작성자 정보, 검색 폼, 주요 탐색 같은 내용이 들어갑니다.

가장 익숙한 예시는 페이지 상단입니다.

```html
<header>
  <h1>프론트엔드 블로그</h1>

  <nav>
    <a href="/">홈</a>
    <a href="/posts">글</a>
    <a href="/about">소개</a>
  </nav>
</header>
```

하지만 `header`는 페이지 맨 위에만 쓰는 태그가 아닙니다.

`article`이나 `section` 안에서도 사용할 수 있습니다.

```html
<article>
  <header>
    <h2>React에서 모달 만들기</h2>
    <p>작성자: 김개발</p>
    <p>작성일: <time datetime="2026-08-27">2026년 8월 27일</time></p>
  </header>

  <p>모달을 만들 때는 포커스 관리가 중요합니다.</p>
</article>
```

여기서 `header`는 사이트 전체의 헤더가 아니라 글 한 편의 헤더입니다.

즉, `header`는 위치가 아니라 역할로 판단합니다.

```txt
이 영역이 어떤 콘텐츠의 시작 정보인가?
그렇다면 header를 고려한다.
```

---

## nav: 주요 탐색 링크

`nav`는 페이지 이동이나 문서 내 이동을 위한 주요 탐색 영역에 사용합니다.

```html
<nav>
  <a href="/">홈</a>
  <a href="/posts">글 목록</a>
  <a href="/about">소개</a>
</nav>
```

중요한 점은 링크가 있다고 무조건 `nav`를 쓰는 것이 아니라는 점입니다.

다음처럼 본문 중간에 있는 단순 참고 링크는 `nav`가 아닙니다.

```html
<p>
  자세한 내용은
  <a href="/posts/html-dom-element-control-guide">DOM 정리 글</a>에서 볼 수 있습니다.
</p>
```

이 링크는 문장 안의 참고 링크입니다.

사이트나 페이지의 주요 이동 구조가 아니므로 `nav`로 감쌀 필요가 없습니다.

`nav`를 쓰기 좋은 상황은 이런 경우입니다.

- 사이트 전체 메뉴
- 문서 목차
- 사이드바 카테고리 메뉴
- 페이지네이션
- 탭처럼 주요 화면 구역을 전환하는 메뉴

```html
<nav aria-label="글 목차">
  <a href="#intro">들어가며</a>
  <a href="#section">section과 article</a>
  <a href="#summary">정리</a>
</nav>
```

한 페이지에 `nav`가 여러 개 있을 수도 있습니다.

그럴 때는 `aria-label`로 구분해주면 보조 기술이 더 잘 이해할 수 있습니다.

```html
<nav aria-label="주요 메뉴">...</nav>
<nav aria-label="글 목차">...</nav>
```

---

## main: 페이지의 핵심 콘텐츠

`main`은 현재 페이지의 핵심 콘텐츠를 담는 태그입니다.

사이트 로고, 전체 메뉴, 푸터, 반복되는 사이드바가 아니라 **이 페이지에서만 중요한 본문**을 담습니다.

```html
<body>
  <header>사이트 헤더</header>

  <main>
    <h1>블로그 글 목록</h1>
    <article>첫 번째 글</article>
    <article>두 번째 글</article>
  </main>

  <footer>사이트 푸터</footer>
</body>
```

`main`에는 보통 페이지마다 달라지는 핵심 내용이 들어갑니다.

예를 들어 페이지별로 보면 이렇게 생각할 수 있습니다.

| 페이지 | main에 들어갈 내용 |
| --- | --- |
| 홈 | 주요 소개, 최신 글, 핵심 섹션 |
| 글 상세 | 글 본문 |
| 상품 상세 | 상품 정보, 가격, 설명 |
| 로그인 | 로그인 폼 |
| 검색 결과 | 검색 결과 목록 |

`main`은 문서의 주요 콘텐츠를 나타내는 영역이므로 보통 한 페이지에 하나만 사용하는 것이 좋습니다.

여러 개의 `main`을 동시에 보여주는 구조는 피하는 편이 안전합니다.

---

## section: 하나의 주제 구역

`section`은 문서 안에서 하나의 주제를 이루는 구역입니다.

가장 중요한 판단 기준은 제목을 붙일 수 있는지입니다.

```html
<section>
  <h2>시맨틱 태그를 쓰는 이유</h2>
  <p>접근성과 문서 구조를 개선할 수 있습니다.</p>
</section>
```

`section`은 보통 heading과 함께 사용합니다.

이 영역을 목차에 넣을 수 있을 정도로 독립적인 주제라면 `section`을 고려할 수 있습니다.

반대로 단순히 스타일을 주려고 묶은 박스라면 `section`보다 `div`가 맞습니다.

```html
<!-- 좋은 예: 하나의 주제가 있음 -->
<section>
  <h2>기능 소개</h2>
  <p>이 서비스에서 제공하는 기능을 소개합니다.</p>
</section>
```

```html
<!-- 애매한 예: 레이아웃을 위한 wrapper라면 div가 더 적절 -->
<section class="container">
  <div class="card">...</div>
  <div class="card">...</div>
</section>
```

위 코드에서 `.container`가 단순히 너비와 여백을 잡기 위한 wrapper라면 `section`을 쓸 이유가 약합니다.

이럴 때는 이렇게 쓰는 편이 더 명확합니다.

```html
<section>
  <h2>추천 글</h2>

  <div class="container">
    <article class="card">...</article>
    <article class="card">...</article>
  </div>
</section>
```

`section`은 주제를 나타내고, `div.container`는 배치를 담당합니다.

---

## article: 독립적으로 떼어낼 수 있는 콘텐츠

`article`은 문서 안에서 독립적으로 배포하거나 재사용할 수 있는 콘텐츠에 사용합니다.

대표적인 예시는 다음과 같습니다.

- 블로그 글 한 편
- 뉴스 기사
- 게시판 글
- 댓글 하나
- 상품 카드
- 사용자 리뷰
- 독립적인 위젯

예를 들어 블로그 글 목록에서는 각 글 카드가 `article`이 될 수 있습니다.

```html
<article>
  <h2>
    <a href="/posts/html-dom-element-control-guide">
      DOM이란? 요소 선택과 제어 방법 정리
    </a>
  </h2>
  <p>DOM의 개념과 JavaScript로 요소를 제어하는 방법을 정리했습니다.</p>
</article>
```

이 카드 하나만 떼어내도 제목, 링크, 설명이 있어 독립적인 콘텐츠로 이해할 수 있습니다.

그래서 `article`이 자연스럽습니다.

댓글도 `article`로 표현할 수 있습니다.

```html
<article>
  <header>
    <h3>김개발</h3>
    <time datetime="2026-08-27T10:00:00">10분 전</time>
  </header>

  <p>덕분에 section과 article 차이를 이해했습니다.</p>
</article>
```

핵심 질문은 이겁니다.

```txt
이 콘텐츠를 따로 떼어내도 하나의 완성된 단위인가?
그렇다면 article을 고려한다.
```

---

## section과 article 차이 확실히 이해하기

가장 많이 헷갈리는 조합이 `section`과 `article`입니다.

둘 다 콘텐츠를 묶는 태그처럼 보이기 때문입니다.

차이는 이렇게 잡으면 좋습니다.

```txt
section
→ 문서 안의 주제 구역

article
→ 독립적으로 떼어내도 의미가 있는 콘텐츠 단위
```

예를 들어 블로그 글 상세 페이지를 보겠습니다.

```html
<article>
  <header>
    <h1>시맨틱 태그란?</h1>
    <p>작성일: <time datetime="2026-08-27">2026년 8월 27일</time></p>
  </header>

  <section>
    <h2>시맨틱 태그를 쓰는 이유</h2>
    <p>HTML 구조를 더 명확하게 만들기 위해 사용합니다.</p>
  </section>

  <section>
    <h2>section과 article의 차이</h2>
    <p>section은 주제 구역이고 article은 독립 콘텐츠입니다.</p>
  </section>
</article>
```

전체 글 한 편은 독립적으로 배포될 수 있으므로 `article`입니다.

그 글 안의 각 소제목 구역은 `section`입니다.

반대로 글 목록 페이지에서는 여러 개의 `article`이 들어갈 수 있습니다.

```html
<main>
  <h1>최근 글</h1>

  <article>
    <h2>DOM이란?</h2>
    <p>DOM의 개념을 정리한 글입니다.</p>
  </article>

  <article>
    <h2>시맨틱 태그란?</h2>
    <p>시맨틱 태그의 사용 기준을 정리한 글입니다.</p>
  </article>
</main>
```

각 글 카드가 독립적인 콘텐츠 단위이기 때문입니다.

---

## aside: 본문을 보조하는 관련 콘텐츠

`aside`는 주요 콘텐츠와 직접 같은 흐름은 아니지만, 관련이 있는 보조 콘텐츠에 사용합니다.

많은 사람들이 `aside`를 오른쪽 사이드바 전용 태그라고 생각하지만, 위치가 핵심은 아닙니다.

역할이 핵심입니다.

```html
<aside>
  <h2>관련 글</h2>
  <ul>
    <li><a href="/posts/css-animation-beginner-guide">CSS 애니메이션 기초</a></li>
    <li><a href="/posts/html-dom-element-control-guide">DOM 제어 방법</a></li>
  </ul>
</aside>
```

`aside`에 들어가기 좋은 콘텐츠는 다음과 같습니다.

- 관련 글
- 작성자 프로필
- 광고
- 참고 링크
- 용어 설명 박스
- 본문 옆 보충 설명

예를 들어 글 중간의 보충 설명도 `aside`가 될 수 있습니다.

```html
<aside>
  <h3>잠깐: SEO와 시맨틱 태그</h3>
  <p>시맨틱 태그는 검색 엔진이 문서 구조를 이해하는 데 도움을 줄 수 있습니다.</p>
</aside>
```

판단 기준은 이렇습니다.

```txt
본문의 핵심 흐름에서는 빠져도 되지만
관련 정보로 함께 보면 좋은가?
그렇다면 aside를 고려한다.
```

---

## footer: 마무리 정보

`footer`는 가장 가까운 문서나 구역의 마무리 정보를 담습니다.

페이지 맨 아래의 사이트 푸터가 대표적입니다.

```html
<footer>
  <p>Copyright 2026. taewok.</p>
  <a href="/privacy">개인정보 처리방침</a>
</footer>
```

하지만 `footer` 역시 페이지 맨 아래에만 쓰는 태그가 아닙니다.

`article` 안에서도 사용할 수 있습니다.

```html
<article>
  <h2>시맨틱 태그란?</h2>
  <p>본문 내용...</p>

  <footer>
    <p>작성자: 김개발</p>
    <a href="/tags/html">HTML 글 더 보기</a>
  </footer>
</article>
```

여기서 `footer`는 글 한 편의 마무리 정보입니다.

`footer`에 들어가기 좋은 내용은 다음과 같습니다.

- 저작권
- 작성자 정보
- 관련 문서 링크
- 연락처
- 사이트맵
- 태그 목록

마찬가지로 위치보다 역할을 기준으로 봅니다.

```txt
이 영역이 어떤 콘텐츠의 마무리 정보인가?
그렇다면 footer를 고려한다.
```

---

## h1부터 h6: 제목은 문서의 뼈대

시맨틱 HTML에서 heading은 매우 중요합니다.

`h1`부터 `h6`는 단순히 글자 크기를 키우는 태그가 아니라 문서의 제목 구조를 만듭니다.

```html
<h1>시맨틱 태그란?</h1>

<h2>시맨틱 태그를 쓰는 이유</h2>
<h3>접근성</h3>
<h3>SEO</h3>

<h2>자주 헷갈리는 태그</h2>
<h3>section과 article</h3>
```

제목 단계는 문서의 목차처럼 생각하면 쉽습니다.

```txt
h1: 페이지 또는 글의 큰 제목
h2: 큰 주제
h3: h2 안의 하위 주제
h4: h3 안의 더 작은 주제
```

초보자가 자주 하는 실수는 디자인 때문에 heading 단계를 고르는 것입니다.

```html
<!-- 글자가 작았으면 좋겠어서 h4를 쓰는 것은 좋지 않음 -->
<h4>페이지의 가장 중요한 제목</h4>
```

글자 크기는 CSS로 조절하고, heading은 문서 구조에 맞게 사용해야 합니다.

```html
<h1 class="small-title">페이지의 가장 중요한 제목</h1>
```

---

## figure와 figcaption: 설명이 붙은 독립 콘텐츠

이미지, 코드, 표, 차트처럼 본문에서 따로 참조할 수 있는 콘텐츠에는 `figure`를 사용할 수 있습니다.

```html
<figure>
  <img src="/images/semantic-layout.png" alt="시맨틱 HTML 레이아웃 예시" />
  <figcaption>시맨틱 태그로 구성한 기본 페이지 구조</figcaption>
</figure>
```

`figcaption`은 그 콘텐츠의 설명입니다.

이미지 아래에 설명이 붙어 있고, 그 묶음이 하나의 단위라면 `figure`가 잘 어울립니다.

코드 예제에도 사용할 수 있습니다.

```html
<figure>
  <pre>
    <code>const title = document.querySelector("h1");</code>
  </pre>
  <figcaption>DOM 요소를 선택하는 코드</figcaption>
</figure>
```

단순 장식용 이미지를 감싸기 위해 무조건 `figure`를 쓸 필요는 없습니다.

본문에서 설명이 필요한 독립 콘텐츠인지가 기준입니다.

---

## time: 날짜와 시간

날짜나 시간을 표시할 때는 `time` 태그를 사용할 수 있습니다.

```html
<time datetime="2026-08-27">2026년 8월 27일</time>
```

화면에는 사람이 읽기 좋은 문장을 보여주고, `datetime`에는 기계가 이해하기 쉬운 형식을 넣습니다.

블로그 작성일, 수정일, 이벤트 일정 등에 잘 어울립니다.

```html
<p>
  작성일:
  <time datetime="2026-08-27">2026년 8월 27일</time>
</p>
```

이렇게 하면 사람과 기계가 모두 날짜를 이해하기 쉬워집니다.

---

## address: 연락처 정보

`address`는 연락처 정보를 담을 때 사용합니다.

```html
<address>
  문의:
  <a href="mailto:hello@example.com">hello@example.com</a>
</address>
```

주의할 점은 모든 주소에 `address`를 쓰는 것이 아니라는 점입니다.

예를 들어 음식점 위치를 본문에서 설명하는 일반 주소라면 꼭 `address`일 필요는 없습니다.

`address`는 문서나 글 작성자, 사이트 소유자, 조직과 연결되는 연락처 정보에 더 적합합니다.

```html
<footer>
  <address>
    Contact:
    <a href="mailto:hello@example.com">hello@example.com</a>
  </address>
</footer>
```

---

## strong, em, b, i 차이

텍스트 강조 태그도 헷갈리기 쉽습니다.

`strong`과 `em`은 의미가 있는 강조입니다.

```html
<p><strong>주의:</strong> 삭제한 데이터는 복구할 수 없습니다.</p>
```

`strong`은 중요하다는 의미를 전달합니다.

```html
<p>이 버튼은 <em>반드시</em> 한 번만 눌러주세요.</p>
```

`em`은 문장 안에서 강조하는 뉘앙스를 전달합니다.

반면 `b`와 `i`는 예전에는 단순 스타일처럼 많이 사용되었지만, HTML5에서는 각각 특정한 의미를 가질 수 있습니다.

하지만 초보 단계에서는 이렇게 기억해도 충분합니다.

```txt
중요하다는 의미가 있으면 strong
문장 안에서 강조하는 의미가 있으면 em
단순히 굵게 또는 기울임 스타일만 원하면 CSS
```

디자인 때문에 글자를 굵게 만들고 싶다면 태그보다 CSS를 사용하는 편이 좋습니다.

```html
<span class="bold-text">굵게 보이는 텍스트</span>
```

```css
.bold-text {
  font-weight: 700;
}
```

---

## button과 a: 클릭하면 다 버튼일까?

실무에서 정말 자주 헷갈리는 태그가 `button`과 `a`입니다.

둘 다 클릭할 수 있기 때문입니다.

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

모달 열기, 저장, 삭제, 탭 전환처럼 동작을 실행하는 것은 `button`입니다.

```html
<button type="button">모달 열기</button>
<button type="submit">저장</button>
```

다음처럼 `a`에 `href` 없이 버튼처럼 쓰는 방식은 피하는 것이 좋습니다.

```html
<!-- 좋지 않음 -->
<a onclick="openModal()">모달 열기</a>
```

이럴 때는 `button`이 더 알맞습니다.

```html
<button type="button">모달 열기</button>
```

태그를 잘 고르면 키보드 조작, 접근성, 브라우저 기본 동작이 자연스럽게 따라옵니다.

---

## ul, ol, dl: 목록도 의미가 있어요

여러 항목을 나열할 때는 목록 태그를 사용할 수 있습니다.

순서가 중요하지 않은 목록은 `ul`입니다.

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

순서가 중요한 목록은 `ol`입니다.

```html
<ol>
  <li>HTML 구조를 작성한다.</li>
  <li>CSS 스타일을 적용한다.</li>
  <li>JavaScript 이벤트를 연결한다.</li>
</ol>
```

용어와 설명의 쌍은 `dl`, `dt`, `dd`를 사용할 수 있습니다.

```html
<dl>
  <dt>DOM</dt>
  <dd>HTML 문서를 JavaScript로 다룰 수 있게 만든 객체 구조입니다.</dd>

  <dt>Semantic HTML</dt>
  <dd>콘텐츠의 의미를 태그로 표현하는 HTML 작성 방식입니다.</dd>
</dl>
```

단순히 줄바꿈을 위해 `div`를 여러 개 쓰기보다, 항목의 성격이 목록이라면 목록 태그를 쓰는 것이 좋습니다.

---

## form, label, fieldset, legend

폼을 만들 때도 시맨틱 태그가 중요합니다.

기본 구조는 `form` 안에 입력 요소를 넣는 것입니다.

```html
<form>
  <label for="email">이메일</label>
  <input id="email" name="email" type="email" />

  <button type="submit">가입하기</button>
</form>
```

`label`은 입력 요소의 이름을 연결합니다.

`for` 값과 `input`의 `id` 값이 같아야 합니다.

```html
<label for="email">이메일</label>
<input id="email" type="email" />
```

이렇게 하면 label을 클릭해도 input에 포커스가 이동합니다.

관련 입력 그룹은 `fieldset`과 `legend`로 묶을 수 있습니다.

```html
<fieldset>
  <legend>알림 설정</legend>

  <label>
    <input type="checkbox" name="email-notification" />
    이메일 알림 받기
  </label>

  <label>
    <input type="checkbox" name="sms-notification" />
    문자 알림 받기
  </label>
</fieldset>
```

폼에서는 태그를 제대로 쓰는 것만으로도 접근성이 크게 좋아집니다.

---

## 상황별 판단표

헷갈릴 때는 아래 표처럼 판단해볼 수 있습니다.

| 상황 | 추천 태그 |
| --- | --- |
| 사이트 상단 로고와 메뉴 | `header` |
| 주요 메뉴, 목차, 페이지네이션 | `nav` |
| 현재 페이지의 핵심 본문 | `main` |
| 하나의 주제 구역 | `section` |
| 독립적으로 떼어낼 수 있는 글, 카드, 댓글 | `article` |
| 본문을 보조하는 관련 정보 | `aside` |
| 페이지나 글의 마무리 정보 | `footer` |
| 제목 구조 | `h1` ~ `h6` |
| 설명이 붙은 이미지, 표, 코드 | `figure`, `figcaption` |
| 날짜와 시간 | `time` |
| 연락처 정보 | `address` |
| 이동 링크 | `a` |
| 동작 실행 | `button` |
| 순서 없는 목록 | `ul` |
| 순서 있는 목록 | `ol` |
| 용어와 설명 | `dl`, `dt`, `dd` |
| 입력 양식 | `form`, `label`, `fieldset`, `legend` |
| 의미 없는 스타일 wrapper | `div` |

이 표를 외우기보다 각 태그의 질문을 기억하는 편이 좋습니다.

---

## 실전 예시: div만 쓴 구조 바꾸기

처음에는 이런 구조를 자주 작성합니다.

```html
<div class="page">
  <div class="top">
    <div class="title">프론트엔드 블로그</div>
    <div class="menu">
      <a href="/">홈</a>
      <a href="/posts">글</a>
    </div>
  </div>

  <div class="content">
    <div class="post">
      <div class="post-title">시맨틱 태그란?</div>
      <div class="post-body">시맨틱 태그는 의미를 가진 태그입니다.</div>
    </div>
  </div>

  <div class="bottom">Copyright 2026.</div>
</div>
```

화면은 문제없이 보일 수 있습니다.

하지만 문서 구조의 의미가 거의 드러나지 않습니다.

시맨틱 태그를 적용하면 이렇게 바꿀 수 있습니다.

```html
<body>
  <header>
    <h1>프론트엔드 블로그</h1>

    <nav>
      <a href="/">홈</a>
      <a href="/posts">글</a>
    </nav>
  </header>

  <main>
    <article>
      <h2>시맨틱 태그란?</h2>
      <p>시맨틱 태그는 의미를 가진 태그입니다.</p>
    </article>
  </main>

  <footer>
    <p>Copyright 2026.</p>
  </footer>
</body>
```

이제 구조만 봐도 페이지의 역할이 훨씬 잘 보입니다.

```txt
header: 사이트 소개와 메뉴
main: 페이지 핵심 내용
article: 글 한 편
footer: 페이지 마무리
```

---

## 자주 하는 실수

### 1. section을 div처럼 쓰기

`section`은 단순 wrapper가 아닙니다.

하나의 주제를 나타낼 때 사용합니다.

```html
<!-- 애매함 -->
<section class="flex gap-4">
  <button>취소</button>
  <button>확인</button>
</section>
```

버튼 배치를 위한 wrapper라면 `div`가 더 적절합니다.

```html
<div class="button-group">
  <button>취소</button>
  <button>확인</button>
</div>
```

### 2. article을 아무 카드에나 쓰기

카드라고 무조건 `article`은 아닙니다.

독립적으로 의미가 있는 카드라면 `article`이 좋습니다.

```html
<article>
  <h2>CSS 애니메이션 기초</h2>
  <p>transition과 keyframes를 정리한 글입니다.</p>
</article>
```

하지만 단순 통계 카드처럼 독립 콘텐츠라기보다 대시보드 일부 값이라면 `section`이나 `div`가 더 자연스러울 수 있습니다.

```html
<section aria-labelledby="sales-title">
  <h2 id="sales-title">오늘 매출</h2>
  <p>1,240,000원</p>
</section>
```

### 3. nav를 모든 링크 묶음에 쓰기

`nav`는 주요 탐색 영역입니다.

본문 안의 참고 링크 몇 개를 묶었다고 무조건 `nav`가 되는 것은 아닙니다.

### 4. header와 footer를 위치로만 판단하기

`header`는 위쪽, `footer`는 아래쪽이라는 뜻이 아닙니다.

각각 소개 정보와 마무리 정보라는 의미입니다.

그래서 `article` 안에도 `header`와 `footer`가 들어갈 수 있습니다.

### 5. 제목 크기 때문에 h 태그를 고르기

`h1`부터 `h6`는 디자인이 아니라 문서 구조입니다.

글자 크기는 CSS로 조절하고, heading 단계는 내용의 계층에 맞춰야 합니다.

---

## 시맨틱 태그를 잘 고르는 질문

태그가 헷갈릴 때는 아래 질문을 순서대로 해보면 좋습니다.

```txt
1. 이 페이지의 핵심 본문인가?
   → main

2. 독립적으로 떼어내도 의미가 있는 콘텐츠인가?
   → article

3. 하나의 제목을 붙일 수 있는 주제 구역인가?
   → section

4. 주요 탐색 링크 모음인가?
   → nav

5. 본문을 보조하는 관련 정보인가?
   → aside

6. 어떤 콘텐츠의 시작 소개인가?
   → header

7. 어떤 콘텐츠의 마무리 정보인가?
   → footer

8. 특별한 의미 없이 스타일이나 배치만 위한 wrapper인가?
   → div
```

이 질문들만 익숙해져도 대부분의 시맨틱 태그 선택이 훨씬 쉬워집니다.

---

## 정리

시맨틱 태그는 화면을 꾸미는 태그가 아니라 콘텐츠의 의미를 설명하는 태그입니다.

`div`만 사용해도 화면은 만들 수 있지만, 시맨틱 태그를 사용하면 브라우저, 검색 엔진, 스크린 리더, 그리고 함께 일하는 개발자가 문서 구조를 더 잘 이해할 수 있습니다.

핵심만 다시 정리하면 다음과 같습니다.

- `header`는 소개 정보입니다.
- `nav`는 주요 탐색 링크입니다.
- `main`은 현재 페이지의 핵심 본문입니다.
- `section`은 하나의 주제 구역입니다.
- `article`은 독립적으로 떼어낼 수 있는 콘텐츠입니다.
- `aside`는 본문을 보조하는 관련 정보입니다.
- `footer`는 마무리 정보입니다.
- `figure`는 설명이 붙은 독립 콘텐츠입니다.
- `time`은 날짜와 시간을 기계가 이해할 수 있게 표현합니다.
- `a`는 이동, `button`은 동작입니다.
- 특별한 의미가 없다면 `div`를 쓰면 됩니다.

한 줄로 정리하면 이렇습니다.

```txt
시맨틱 태그를 고를 때는
모양이 아니라 콘텐츠의 역할을 먼저 생각한다
```

이 기준이 잡히면 `section`과 `article` 사이에서 덜 흔들리고, `header`, `footer`, `aside`, `nav`도 훨씬 자연스럽게 사용할 수 있습니다.

## 참고

- [MDN: Semantics](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)
- [MDN: HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements)
- [MDN: Structuring documents](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Structuring_documents)
- [MDN: article](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/article)
- [MDN: section](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/section)
- [MDN: header](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/header)
- [MDN: footer](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/footer)
- [WHATWG HTML Standard: Sections](https://html.spec.whatwg.org/multipage/sections.html)
