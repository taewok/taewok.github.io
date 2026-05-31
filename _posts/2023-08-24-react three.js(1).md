---
title: "[three.js] React에서 three.js 시작하기"
date: 2023-08-24T17:04:00
categories: [three.js, react]
tags: [three.js, react, react-three-fiber, drei]
description: "React에서 three.js를 사용하기 위해 @react-three/fiber와 drei를 설치하고 기본 박스를 렌더링하는 방법을 정리했습니다."
custom_style: true
---

## three.js란?

three.js는 브라우저에서 3D 그래픽을 만들 수 있게 도와주는 JavaScript 라이브러리입니다.

WebGL을 직접 다루지 않아도 카메라, 조명, 물체, 재질 같은 3D 요소를 비교적 쉽게 사용할 수 있습니다.

React 프로젝트에서는 `@react-three/fiber`를 함께 사용하면 React 컴포넌트 방식으로 3D 장면을 구성할 수 있습니다.

---

## 설치하기

React에서 three.js를 사용할 때는 보통 아래 패키지를 함께 설치합니다.

```bash
npm install three @react-three/fiber @react-three/drei
```

각 패키지의 역할은 다음과 같습니다.

| 패키지 | 역할 |
| --- | --- |
| `three` | 3D 그래픽의 핵심 기능 |
| `@react-three/fiber` | three.js를 React 방식으로 사용할 수 있게 해주는 렌더러 |
| `@react-three/drei` | 자주 쓰는 helper 컴포넌트 모음 |

---

## Canvas 만들기

`Canvas`는 3D 장면이 렌더링되는 영역입니다.

```tsx
import { Canvas } from "@react-three/fiber";

const App = () => {
  return (
    <Canvas>
      {/* 3D 요소가 이 안에 들어갑니다. */}
    </Canvas>
  );
};

export default App;
```

React 앱 안에서 3D 세계를 담는 컨테이너라고 생각하면 됩니다.

---

## 기본 박스 렌더링하기

가장 기본적인 3D 물체를 만들어보겠습니다.

```tsx
import { Canvas } from "@react-three/fiber";

const App = () => {
  return (
    <Canvas>
      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
};

export default App;
```

`mesh`는 3D 물체를 의미합니다.

`boxGeometry`는 물체의 모양을 박스로 정합니다.

`meshStandardMaterial`은 물체의 색상과 표면 재질을 정합니다.

---

## 조명 추가하기

`meshStandardMaterial`은 조명에 영향을 받는 재질입니다.

그래서 조명이 없으면 물체가 어둡게 보일 수 있습니다.

```tsx
import { Canvas } from "@react-three/fiber";

const App = () => {
  return (
    <Canvas>
      <ambientLight intensity={0.5} />
      <directionalLight position={[2, 2, 2]} intensity={1} />

      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
};

export default App;
```

`ambientLight`는 장면 전체를 은은하게 밝힙니다.

`directionalLight`는 특정 방향에서 비추는 빛입니다.

---

## OrbitControls 추가하기

마우스로 3D 장면을 회전하고 확대하고 싶다면 `OrbitControls`를 사용할 수 있습니다.

```tsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

const App = () => {
  return (
    <Canvas>
      <ambientLight intensity={0.5} />
      <directionalLight position={[2, 2, 2]} />
      <OrbitControls />

      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
};

export default App;
```

`OrbitControls`를 넣으면 마우스로 장면을 직접 둘러볼 수 있습니다.

---

## 마무리

React에서 three.js를 시작하려면 `Canvas`, `mesh`, `geometry`, `material`, `light`의 역할을 먼저 이해하면 좋습니다.

`Canvas` 안에 3D 요소를 넣고, `mesh`에 모양과 재질을 연결하면 기본 물체를 렌더링할 수 있습니다.

처음에는 박스 하나를 띄운 뒤 조명과 컨트롤을 하나씩 추가해보면 흐름을 익히기 쉽습니다.
