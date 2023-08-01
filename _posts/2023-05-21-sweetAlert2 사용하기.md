---
title: "[React] sweetAlert2 사용하기"
date: 2023-05-21T16:27:000
categories: [React]
tags: [react] #소문자만 가능
---

---

## <b style="border-bottom:2px solid gray">sweetAlert2 설치</b>

```js
npm install sweetalert2
```

---

## <b style="border-bottom:2px solid gray">sweetAlert2 사용</b>

### <b style="color:green">기본 Alert</b>

```js
Swal.fire({
  title: "Alert 실행",
  text: "질문이나 전달할 내용을 나타내는 곳",
  icon: "success",
});
```

![image](https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/983fff85-fe30-4521-b1ea-3f39fa9262fc)

### <b style="color:green">Confirm(확인) Alert</b>

어떤 버튼을 눌렀는지에 따라 후처리 이벤트(then)를 이용해 다음 동작을 설정할 수 있습니다.

```js
Swal.fire({
  title: "로그아웃",
  text: "로그아웃 하시겠습니까?",
  icon: "question",
  showCancelButton: true, // cancel버튼 보이기. 기본은 원래 없음
  confirmButtonColor: "#3085d6", // confrim 버튼 색깔 지정
  cancelButtonColor: "#d33", // cancel 버튼 색깔 지정
  confirmButtonText: "확인", // confirm 버튼 텍스트 지정
  cancelButtonText: "취소", // cancel 버튼 텍스트 지정
}).then((result) => {
  if (result.isConfirmed) {
    // 만약 모달창에서 confirm 버튼을 눌렀다면
  } else if (result.isDismissed) {
    // 만약 모달창에서 cancel 버튼을 눌렀다면
  }
});
```

![image](https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/a79a898d-78d4-4dca-b63c-113ad6006de6)

### <b style="color:green">Prompt Alert</b>

텍스트를 입력받는 알림창, 체크박스, 라디오 버튼, 셀렉트 박스, 파일 등 input 태그로 만들 수 있는 모든 것들을 prompt로 구현할 수 있다

```js
Swal.fire({
  title: "당신의 이름을 입력하세요.",
  text: "그냥 예시일 뿐입니다.",
  input: "text",
  inputPlaceholder: "이름을 입력..",
}).then((res) => {
  //이름 입력후 console로 값이 찍힌다.
  console.log(res.value);
});
```

![image](https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/1e6ee4d1-c435-403d-a295-e738c74ca710)

### <b style="color:green">버튼 처리</b>

|    속성명     | 설명                    |
| :-----------: | :---------------------- |
| `isConfirmed` | "확인" 버튼을 눌렀을 때 |
|  `isDenied`   | "거부" 버튼을 눌렀을 때 |
| `isDismissed` | "취소" 버튼을 눌렀을 때 |
|    `value`    | Prompt에 입력된 값      |

---

## <b style="border-bottom:2px solid gray">sweetAlert2 속성</b>

|        속성명        | 설명                                                                                                                              |
| :------------------: | :-------------------------------------------------------------------------------------------------------------------------------- |
|       `title`        | 팝업 제목 font:30px                                                                                                               |
|        `text`        | 팝업에 대한 설명 font:18px                                                                                                        |
|        `icon`        | SweetAlert2에서 제공하는 5가지 기본 icon <br/>EX: warning, error, success, info, question                                         |
|     `iconColor`      | 아이콘의 색상을 변경                                                                                                              |
|     `showClass`      | 팝업 표시 시 애니메이션용 CSS 클래스<br/>popup: "swal2-show",<br/>backdrop: "swal2-backdrop-show",<br/>icon: "swal2-icon-show"    |
|     `hideClass`      | 팝업을 숨길 때 애니메이션용 CSS 클래스<br/>popup: 'swal2-hide',<br/>backdrop: 'swal2-backdrop-hide',<br/> icon: 'swal2-icon-hide' |
|       `footer`       | 팝업의 바닥글입니다. 일반 텍스트 또는 HTML일 수 있다.                                                                             |
|      `backdrop`      | SweetAlert2가 활성화되면 뒤 배경을 어둡게 할지 안할지 기본값:true                                                                 |
|       `input`        | 입력 필드 유형 정의 <br/>EX:text, email, password, number, tel, range, textarea, select, radio, checkbox, file, url               |
|  `inputPlaceholder`  | 입력 필드 Placeholder                                                                                                             |
|     `inputValue`     | 입력 필드 초기값 설정                                                                                                             |
|       `width`        | 패딩을 포함한 팝업 창 넓이 기본값:32em                                                                                            |
|      `padding`       | 팝업 창 패딩. 모든 CSS 단위 기본값:0 0 1.25em                                                                                     |
|       `color`        | 제목, 내용 및 바닥글 색상                                                                                                         |
|     `background`     | 팝업 창 배경(CSS 배경 속성). 기본값:'#fff'                                                                                        |
|      `position`      | 'top', 'top-start', 'top-end', 'center', 'center-start', 'center-end', 'bottom', 'bottom-start', 'bottom-end'                     |
|       `timer`        | 팝업의 자동 닫기 타이머. 1000이 1초                                                                                               |
|  `timerProgressBar`  | timer과 같이 써야하며 timer에 남은 시간을 bar 하단에 표시 해준다                                                                  |
| `allowOutsideClick`  | 사용자가 팝업 외부를 클릭하여 팝업 해제 여부 기본값:true                                                                          |
|   `allowEnterKey`    | 사용자가 Enter key로 팝업 확인 여부 기본값:true                                                                                   |
| `showConfirmButton`  | "확인" 버튼이 표시 여부 기본값:true                                                                                               |
|   `showDenyButton`   | "거부" 버튼이 표시 여부 기본값:false                                                                                              |
|  `showCancelButton`  | 사용자가 모달을 해제하기 위해 클릭할 수 있는 "취소" 버튼 표시 여부 기본값:false                                                   |
| `confirmButtonText`  | "확인" 버튼의 텍스트 기본값:"OK"                                                                                                  |
|   `denyButtonText`   | "거부" 버튼의 텍스트 기본값:"No"                                                                                                  |
|  `cancelButtonText`  | "취소" 버튼의 텍스트 기본값:"Cancel"                                                                                              |
| `confirmButtonColor` | "확인" 버튼의 배경색을 변경 기본값:#3085d6                                                                                        |
|  `denyButtonColor`   | "거부" 버튼의 배경색을 변경 기본값:#dd6b55                                                                                        |
| `cancelButtonColor`  | "취소" 버튼의 배경색을 변경 기본값:#aaa                                                                                           |
|   `reverseButtons`   | 기본 버튼 위치를 반전하려는 경우 기본값:false                                                                                     |
|     `focusDeny`      | "거부" 버튼에 focus                                                                                                               |
|    `focusCancel`     | "취소" 버튼에 focus                                                                                                               |
|      `imageUrl`      | 팝업에 대한 사용자 지정 아이콘을 추가                                                                                             |
|     `imageWidth`     | imageUrl이 설정된 경우 이미지 너비를 설정                                                                                         |
|    `imageHeight`     | 사용자 정의 이미지 높이                                                                                                           |
|      `imageAlt`      | 사용자 정의 이미지 아이콘의 대체 텍스트                                                                                           |


---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>