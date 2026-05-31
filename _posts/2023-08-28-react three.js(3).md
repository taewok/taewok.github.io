---
title: "[three.js] Leva useControls로 3D 값 조절하기"
date: 2023-08-28T00:24:00
categories: [three.js, react]
tags: [three.js, react, leva, usecontrols]
description: "React Three Fiber에서 Leva의 useControls를 사용해 3D 객체의 위치나 색상을 실시간으로 조절하는 방법을 정리했습니다."
custom_style: true
---

## useControls란?

`useControls`는 Leva 라이브러리에서 제공하는 훅입니다.

화면에 조작 패널을 만들고, 그 패널에서 값을 변경하면 React 컴포넌트에 실시간으로 반영할 수 있습니다.

React Three Fiber로 3D 장면을 만들 때는 위치, 회전, 크기, 색상 같은 값을 테스트하기에 매우 편합니다.

---

## 설치하기

```bash
npm install leva
```

설치 후 `useControls`를 import해서 사용할 수 있습니다.

```tsx
import { useControls } from "leva";
```

---

## 위치 조절하기

아래 예시는 Leva 패널에서 mesh의 위치를 조절하는 코드입니다.

```tsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
import { useControls } from "leva";

const Box = () => {
  const { positionX, positionY, positionZ } = useControls({
    positionX: { value: 0, min: -5, max: 5, step: 0.1 },
    positionY: { value: 0, min: -5, max: 5, step: 0.1 },
    positionZ: { value: 0, min: -5, max: 5, step: 0.1 },
  });

  return (
    <mesh position={[positionX, positionY, positionZ]}>
      <boxGeometry />
      <meshStandardMaterial color="orange" />
    </mesh>
  );
};

const App = () => {
  return (
    <Canvas>
      <ambientLight intensity={0.5} />
      <directionalLight position={[2, 2, 2]} />
      <OrbitControls />
      <Box />
    </Canvas>
  );
};

export default App;
```

패널에서 값을 움직이면 박스 위치가 바로 바뀝니다.

---

## 색상도 조절할 수 있어요

Leva는 숫자뿐 아니라 색상 값도 다룰 수 있습니다.

```tsx
const { color } = useControls({
  color: "#ff8a00",
});

return (
  <mesh>
    <boxGeometry />
    <meshStandardMaterial color={color} />
  </mesh>
);
```

색상 선택 UI가 자동으로 생기기 때문에 material 색상을 실험하기 좋습니다.

---

## 여러 값을 묶어서 관리하기

회전이나 크기까지 함께 조절할 수 있습니다.

```tsx
const { scale, rotationY } = useControls({
  scale: { value: 1, min: 0.1, max: 3, step: 0.1 },
  rotationY: { value: 0, min: 0, max: Math.PI * 2, step: 0.01 },
});

return (
  <mesh scale={scale} rotation={[0, rotationY, 0]}>
    <boxGeometry />
    <meshStandardMaterial color="orange" />
  </mesh>
);
```

값을 코드에서 계속 수정하고 새로고침하지 않아도 되기 때문에 작업 속도가 빨라집니다.

---

## 언제 사용하면 좋을까요?

`useControls`는 최종 사용자용 UI라기보다 개발 중 값을 조정하는 도구에 가깝습니다.

예를 들면 이런 상황에서 유용합니다.

- 조명 위치 찾기
- 카메라 위치 조정
- 물체의 색상과 크기 실험
- 애니메이션 파라미터 조절
- 3D 장면의 초기값 탐색

---

## 마무리

Leva의 `useControls`를 사용하면 3D 객체의 값을 실시간으로 조절할 수 있습니다.

React Three Fiber에서 위치, 회전, 크기, 색상 같은 값을 찾을 때 매우 편리합니다.

특히 감으로 숫자를 바꿔가며 조정해야 하는 3D 작업에서는 개발 시간을 많이 줄여줍니다.
