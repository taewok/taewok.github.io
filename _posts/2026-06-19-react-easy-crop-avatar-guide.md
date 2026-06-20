---
title: "[React] react-easy-crop으로 대표 이미지 크롭 기능 구현하기"
date: 2026-06-19T13:00:00Z
categories: [frontend]
tags: [react, react-easy-crop, image-crop, canvas, modal]
description: "react-easy-crop으로 이미지 업로드, 비율 선택, 드래그, 줌, canvas 저장까지 연결해 대표 이미지 크롭 기능을 구현하는 방법을 정리했습니다."
custom_style: true
---

## 🧐 들어가며: 단순 업로드만으로는 조금 아쉬웠어요

캐릭터 생성 페이지의 대표 이미지 업로드 기능을 만들다 보면 단순히 파일을 올리는 것만으로는 부족할 때가 있습니다.

사용자가 원하는 부분을 직접 잘라서 대표 이미지로 저장할 수 있어야 하기 때문입니다.

예를 들어 이런 경험이 필요합니다.

- 이미지를 업로드하고
- 원하는 비율을 고르고
- 마우스로 위치를 조정하고
- 확대/축소까지 한 뒤
- 최종 결과만 대표 이미지로 저장하기

이번 글에서는 `react-easy-crop`을 사용해서 이 흐름을 어떻게 만들었는지 정리해보겠습니다.

핵심은 단순합니다.

```txt
업로드한 원본 이미지를 바로 저장하지 않고
크롭 결과가 완성된 뒤에만 최종 이미지로 반영한다
```

---

## 💡 왜 직접 구현하지 않았을까요?

이미지 크롭은 브라우저의 `canvas`와 포인터 이벤트만으로도 직접 구현할 수 있습니다.

하지만 직접 만들기 시작하면 생각보다 챙길 것이 많습니다.

- 드래그 좌표 계산
- 이미지 경계 제한
- 비율별 프레임 계산
- 확대/축소 배율 관리
- 화면 좌표를 실제 이미지 픽셀 좌표로 변환

처음에는 간단해 보여도 기능이 커질수록 유지보수가 부담스러워집니다.

그래서 이번에는 `react-easy-crop`을 사용했습니다.

장점은 분명했습니다.

- 상태가 단순해집니다.
- 코드가 선언적으로 읽힙니다.
- 드래그와 줌 동작이 안정적입니다.
- 비율 변경을 붙이기 쉽습니다.
- 최종 크롭 좌표를 픽셀 단위로 받을 수 있습니다.

---

## 📦 설치하기

프로젝트에 라이브러리를 추가합니다.

```bash
npm install react-easy-crop --legacy-peer-deps
```

현재 프로젝트에서는 기존 의존성과 peer 충돌이 있어서 `--legacy-peer-deps`를 함께 사용했습니다.

그리고 `react-easy-crop` 스타일을 불러옵니다.

공식 문서에서는 기본적으로 스타일이 자동 주입되지만, 수동으로 사용하고 싶다면 패키지 CSS를 import할 수 있습니다.

```css
@import "react-easy-crop/react-easy-crop.css";
```

만약 자동 주입을 끄고 싶다면 `disableAutomaticStylesInjection` 옵션을 확인하면 됩니다.

---

## 🧭 전체 흐름

대표 이미지 크롭 기능은 보통 이런 순서로 흘러갑니다.

1. 사용자가 대표 이미지 파일을 선택합니다.
2. 바로 저장하지 않고 크롭 모달을 엽니다.
3. 모달에서 비율을 선택합니다.
4. 드래그와 확대/축소로 원하는 영역을 맞춥니다.
5. `onCropComplete`에서 실제 픽셀 좌표를 저장합니다.
6. 적용 버튼을 누르면 `canvas`로 잘라낸 이미지를 만듭니다.
7. 최종 결과만 폼 값에 반영합니다.

이 흐름의 핵심은 원본 파일과 최종 저장 파일을 분리하는 것입니다.

```txt
원본 파일
→ 크롭 중간 상태
→ 최종 저장 이미지
```

이렇게 나누면 업로드 원본을 잃지 않으면서도, 크롭 결과만 깔끔하게 저장할 수 있습니다.

---

## 🛠️ 업로드 컴포넌트 만들기

파일을 읽은 뒤 바로 폼 값에 넣지 않고, 먼저 크롭 대상만 따로 잡아둡니다.

```tsx
import {ChangeEvent, useState} from "react";

type CropTarget = {
  src: string;
  type: string;
};

export default function ImageUploadField() {
  const [cropTarget, setCropTarget] = useState<CropTarget | null>(null);

  const handleImageChange = (event: ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (!file) return;

    const reader = new FileReader();

    reader.onloadend = () => {
      if (typeof reader.result !== "string") return;

      setCropTarget({
        src: reader.result,
        type: file.type
      });
    };

    reader.readAsDataURL(file);
  };

  return (
    <div>
      <input type="file" accept="image/*" onChange={handleImageChange} />
      {cropTarget ? <p>크롭 모달을 열 준비가 됐어요.</p> : null}
    </div>
  );
}
```

이 단계에서는 `FileReader`로 이미지를 읽고, base64 문자열 형태로 보관합니다.

중요한 점은 여기서 최종 저장을 끝내지 않는다는 것입니다.

```txt
파일 선택
→ base64로 읽기
→ cropTarget에 저장
→ 모달에서 크롭
→ 적용 시 최종 저장
```

---

## 🪟 크롭 모달 상태

`react-easy-crop`은 필요한 상태가 생각보다 단순합니다.

```tsx
import {Area, Point} from "react-easy-crop";
import {useState} from "react";

const [crop, setCrop] = useState<Point>({ x: 0, y: 0 });
const [zoom, setZoom] = useState(1);
const [selectedRatio, setSelectedRatio] = useState("square");
const [croppedAreaPixels, setCroppedAreaPixels] = useState<Area | null>(null);
const [imageAspect, setImageAspect] = useState(1);
```

각 상태의 역할은 다음과 같습니다.

| 상태 | 역할 |
| --- | --- |
| `crop` | 이미지가 얼마나 이동했는지 |
| `zoom` | 확대/축소 값 |
| `selectedRatio` | 사용자가 선택한 비율 |
| `croppedAreaPixels` | 실제 잘라낼 픽셀 좌표 |
| `imageAspect` | 원본 이미지의 가로세로 비율 |

이 중에서 특히 중요한 값은 `croppedAreaPixels`입니다.

화면에서 보이는 크롭 영역이 아니라, 실제 이미지 픽셀 기준으로 어디를 잘라야 하는지가 들어 있기 때문입니다.

---

## 🧱 비율 선택 만들기

크롭 비율은 직접 정의해서 사용하면 편합니다.

```tsx
const CROP_RATIO_VALUES = {
  original: null,
  square: 1,
  landscape: 4 / 3,
  portrait: 3 / 4,
  widescreen: 16 / 9
} as const;
```

`original`은 원본 이미지 비율을 그대로 쓰고, 나머지는 고정 비율로 적용합니다.

이렇게 해두면 사용자가 상황에 따라 비율을 쉽게 바꿀 수 있습니다.

```txt
원본 비율
정사각형
가로형
세로형
와이드형
```

프로필 이미지나 대표 썸네일처럼 서비스마다 요구 비율이 다른 경우에 잘 맞습니다.

---

## 🧩 Cropper 사용하기

실제 UI는 `Cropper` 컴포넌트로 구성합니다.

```tsx
import Cropper from "react-easy-crop";

<Cropper
  image={imageSrc}
  crop={crop}
  zoom={zoom}
  aspect={aspect}
  cropShape="rect"
  showGrid
  objectFit="contain"
  onCropChange={setCrop}
  onZoomChange={setZoom}
  onCropComplete={handleCropComplete}
  onMediaLoaded={({ naturalWidth, naturalHeight }) => {
    setImageAspect(naturalWidth / naturalHeight);
  }}
/>
```

여기서 중요한 포인트를 하나씩 보면 다음과 같습니다.

`image`
: 크롭할 이미지 소스입니다.

`crop`
: 이미지가 현재 어디에 위치해 있는지 나타냅니다.

`zoom`
: 확대/축소 값입니다.

`aspect`
: 크롭 비율입니다.

`cropShape`
: `rect` 또는 `round` 형태로 사용할 수 있습니다.

`showGrid`
: 가이드 그리드를 보여줍니다.

`objectFit`
: 이미지가 크롭 영역 안에서 어떻게 맞춰질지 정합니다.

`onCropChange`, `onZoomChange`
: 사용자가 드래그하거나 확대할 때 상태를 갱신합니다.

`onCropComplete`
: 실제 크롭 좌표를 넘겨줍니다.

`onMediaLoaded`
: 이미지가 로드된 뒤 원본 크기 정보를 받을 수 있습니다.

공식 README에서도 `Cropper`는 부모 요소 안에서 `position: absolute`로 동작하기 때문에, 바깥쪽에 `position: relative`가 있는 래퍼가 필요하다고 안내합니다.

```txt
Cropper
→ absolute

바깥 래퍼
→ relative
```

이 구조를 빼먹으면 크롭 영역이 페이지 전체를 덮어버릴 수 있습니다.

---

## 📐 onCropComplete가 중요한 이유

`react-easy-crop`은 사용자가 움직인 결과를 두 가지 형태로 넘깁니다.

- 퍼센트 기준 좌표
- 픽셀 기준 좌표

대표 이미지처럼 실제 저장이 필요한 경우에는 픽셀 좌표를 사용하는 편이 더 자연스럽습니다.

```tsx
const handleCropComplete = (
  _croppedArea: Area,
  nextCroppedAreaPixels: Area
) => {
  setCroppedAreaPixels(nextCroppedAreaPixels);
};
```

공식 문서도 `onCropComplete`를 크롭 영역을 저장할 때 쓰라고 안내합니다.

이렇게 저장해두면 적용 버튼을 눌렀을 때 canvas로 실제 이미지를 만들 수 있습니다.

---

## 🖼️ 실제 이미지로 잘라내기

화면에서만 잘라 보이는 걸로는 부족합니다.

최종적으로는 실제 이미지 데이터가 필요합니다.

그래서 canvas 유틸을 따로 분리합니다.

```tsx
const loadImage = (src: string) =>
  new Promise<HTMLImageElement>((resolve, reject) => {
    const image = new Image();

    image.onload = () => resolve(image);
    image.onerror = () => reject(new Error("이미지를 불러오지 못했습니다."));
    image.src = src;
  });

export const createCroppedImageDataUrl = async ({
  imageSrc,
  cropArea,
  outputType
}: {
  imageSrc: string;
  cropArea: { x: number; y: number; width: number; height: number };
  outputType: string;
}) => {
  const image = await loadImage(imageSrc);
  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d");

  if (!context) {
    throw new Error("Failed to create canvas context.");
  }

  canvas.width = Math.max(1, Math.round(cropArea.width));
  canvas.height = Math.max(1, Math.round(cropArea.height));

  context.drawImage(
    image,
    cropArea.x,
    cropArea.y,
    cropArea.width,
    cropArea.height,
    0,
    0,
    canvas.width,
    canvas.height
  );

  return canvas.toDataURL(outputType, 0.92);
};
```

이 함수의 역할은 명확합니다.

```txt
원본 이미지
→ cropArea에 맞게 자르기
→ canvas에 다시 그리기
→ data URL로 반환
```

즉, 화면 UI와 저장 로직을 분리하는 셈입니다.

---

## ▶ 적용 버튼 연결하기

이제 적용 버튼을 누르면 실제 크롭 결과를 만들 수 있습니다.

```tsx
const handleApply = async () => {
  if (!croppedAreaPixels) return;

  const croppedImage = await createCroppedImageDataUrl({
    imageSrc,
    cropArea: croppedAreaPixels,
    outputType: imageType === "image/png" ? "image/png" : "image/jpeg"
  });

  onApply(croppedImage);
};
```

여기서 포인트는 `image/png`와 `image/jpeg`를 구분해 저장 형식을 선택하는 부분입니다.

서비스 정책에 따라 투명도가 필요하면 PNG를, 일반 프로필 이미지라면 JPEG를 사용하는 식으로 나눌 수 있습니다.

---

## 🧩 폼 값에 반영하기

최종적으로는 크롭된 결과만 폼 값에 넣습니다.

```tsx
const handleCropApply = (croppedImage: string) => {
  setValue("representativeImage", croppedImage, {
    shouldDirty: true,
    shouldValidate: true
  });

  setCropTarget(null);
};
```

이 방식의 장점은 흐름이 분명해진다는 점입니다.

```txt
업로드 원본
→ 크롭 대상
→ 최종 대표 이미지
```

폼 상태에는 이미 잘라진 최종 결과만 들어가고, 중간 과정은 모달 내부에서만 관리됩니다.

---

## 📌 구현하면서 좋았던 점

`react-easy-crop`을 쓰면서 가장 좋았던 점은 직접 계산해야 할 부분이 크게 줄었다는 것입니다.

직접 만들었다면 많이 붙었을 코드들이 라이브러리 안에 이미 정리되어 있습니다.

- 드래그 포인터 이벤트
- 프레임 경계 계산
- 이미지 중심 정렬
- 줌에 따른 이동 범위
- 크롭 좌표 계산

우리는 다음 정도만 관리하면 됩니다.

- 어떤 비율을 쓸지
- 줌 범위를 얼마나 줄지
- 크롭 결과를 어디에 저장할지
- 최종 이미지를 어떤 형식으로 만들지

이 차이가 꽤 큽니다.

---

## ⚠️ 아쉬운 점과 주의할 점

물론 완전히 공짜는 아닙니다.

- 라이브러리 의존성이 하나 늘어납니다.
- 스타일을 함께 관리해야 합니다.
- 아주 특이한 커스텀 UI는 라이브러리 구조에 맞춰야 합니다.

그리고 `react-easy-crop`은 모달 안에서 쓸 때 크기 계산이 어긋나지 않도록 주의해야 합니다.

공식 문서에서도 모달의 opening animation이 크롭퍼 크기를 바꿔버리면 문제가 생길 수 있다고 안내합니다.

즉, 모달 크기가 열리는 동안 scale이 바뀌는 애니메이션은 피하고, fade나 slide처럼 크기 자체를 바꾸지 않는 방식이 더 안전합니다.

---

## ✅ 정리

이번 구현의 핵심은 단순합니다.

```txt
원본 이미지를 바로 저장하지 않는다
→ 크롭 모달에서 원하는 영역을 먼저 고른다
→ onCropComplete로 픽셀 좌표를 저장한다
→ canvas로 실제 이미지를 만든다
→ 최종 결과만 폼 값에 반영한다
```

이 흐름을 잡아두면 대표 이미지 크롭 기능을 꽤 안정적으로 만들 수 있습니다.

정리해보면:

- 간단한 크롭은 직접 구현도 가능하지만
- 유지보수와 가독성을 생각하면 `react-easy-crop`이 꽤 좋은 선택입니다.
- 화면 UI와 실제 저장 로직은 분리하는 편이 좋습니다.
- 최종 저장은 canvas를 통해 따로 처리해야 완성됩니다.

이제 사용자는 이미지를 업로드하고, 원하는 비율을 고르고, 위치와 줌을 조절한 뒤, 정말 원하는 부분만 대표 이미지로 저장할 수 있게 됩니다.
