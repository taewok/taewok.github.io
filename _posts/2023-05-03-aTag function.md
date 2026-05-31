---
title: "[HTML] a 태그로 할 수 있는 다양한 기능"
date: 2023-05-03T16:01:00
categories: [html]
tags: [html, anchor, link]
description: "HTML a 태그의 기본 링크 이동부터 새 탭 열기, 이메일 보내기, 전화 연결, 파일 다운로드까지 정리했습니다."
custom_style: true
---

## 들어가며

HTML에서 `a` 태그는 링크를 만들 때 가장 많이 사용하는 태그입니다.

처음에는 단순히 다른 페이지로 이동할 때만 쓰는 것처럼 보이지만, 실제로는 이메일 보내기, 전화 연결, 파일 다운로드처럼 여러 기능을 처리할 수 있습니다.

이번 글에서는 `a` 태그에서 자주 사용하는 패턴을 하나씩 정리해볼게요.

---

## 기본 페이지 이동

가장 기본적인 사용법은 `href`에 이동할 주소를 넣는 것입니다.

```html
<a href="https://www.google.com">Google로 이동</a>
```

클릭하면 해당 주소로 이동합니다.

내부 페이지로 이동할 때도 사용할 수 있습니다.

```html
<a href="/about">소개 페이지</a>
```

---

## 새 탭에서 열기

외부 사이트로 이동할 때는 현재 페이지를 유지하고 새 탭으로 열고 싶을 때가 많습니다.

이럴 때는 `target="_blank"`를 사용합니다.

```html
<a href="https://www.google.com" target="_blank" rel="noopener noreferrer">
  새 탭에서 Google 열기
</a>
```

여기서 `rel="noopener noreferrer"`도 함께 넣는 것이 좋습니다.

새 탭으로 열린 페이지가 기존 페이지에 접근하는 것을 막아 보안상 더 안전합니다.

---

## 이메일 보내기

`mailto:`를 사용하면 사용자의 기본 메일 앱을 열 수 있습니다.

```html
<a href="mailto:test@example.com">이메일 보내기</a>
```

제목과 본문을 미리 넣을 수도 있습니다.

```html
<a href="mailto:test@example.com?subject=문의드립니다&body=안녕하세요.">
  문의 메일 보내기
</a>
```

다만 사용자의 환경에 메일 앱이 설정되어 있지 않으면 기대한 대로 동작하지 않을 수 있습니다.

---

## 전화 연결하기

모바일 환경에서는 `tel:`을 사용해 전화 연결 링크를 만들 수 있습니다.

```html
<a href="tel:01012345678">전화 걸기</a>
```

PC에서는 전화 앱이 연결되어 있지 않으면 동작하지 않을 수 있지만, 모바일 웹에서는 꽤 자주 사용하는 방식입니다.

---

## 파일 다운로드

`download` 속성을 사용하면 링크된 파일을 다운로드하도록 유도할 수 있습니다.

```html
<a href="/files/sample.pdf" download>PDF 다운로드</a>
```

다운로드될 파일명을 직접 지정할 수도 있습니다.

```html
<a href="/files/sample.pdf" download="guide.pdf">가이드 다운로드</a>
```

브라우저나 파일 출처에 따라 동작이 조금 달라질 수 있으니, 실제 배포 환경에서 꼭 확인해보는 것이 좋습니다.

---

## 페이지 안 특정 위치로 이동하기

`id`와 `#`을 조합하면 같은 페이지 안의 특정 위치로 이동할 수 있습니다.

```html
<a href="#section-contact">문의 영역으로 이동</a>

<section id="section-contact">
  <h2>문의하기</h2>
</section>
```

문서가 긴 페이지에서 목차를 만들 때 유용합니다.

---

## 마무리

`a` 태그는 단순한 링크 태그처럼 보이지만, 실제로는 여러 사용자 행동을 연결하는 중요한 HTML 요소입니다.

페이지 이동에는 기본 `href`를 사용하고, 외부 링크는 `target="_blank"`와 `rel="noopener noreferrer"`를 함께 사용하면 좋습니다.

이메일, 전화, 다운로드처럼 목적이 다른 링크도 `mailto:`, `tel:`, `download` 속성을 활용하면 간단하게 구현할 수 있습니다.
