---
title: "[three.js] React three.js에서 조명과 Material 다루기"
date: 2023-08-27T23:37:00
categories: [three.js, react]
tags: [three.js, react, material, light]
description: "React Three Fiber에서 조명과 material을 사용해 3D 물체의 색상, 질감, 입체감을 조정하는 방법을 정리했습니다."
custom_style: true
---

## 들어가며

이전 글에서는 React에서 three.js를 시작하고 기본 박스를 렌더링하는 방법을 살펴봤습니다.

이번 글에서는 3D 장면을 더 입체적으로 보이게 만드는 조명과 material을 정리해볼게요.

3D 물체는 모양만 있다고 끝나지 않습니다.

어떤 재질을 쓰는지, 어떤 방향에서 빛을 받는지에 따라 전혀 다르게 보입니다.

---

## OrbitControls로 3D 장면 조작하기

먼저 마우스로 장면을 둘러볼 수 있도록 `OrbitControls`를 추가합니다.

```tsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

const App = () => {
  return (
    <Canvas>
      <OrbitControls />

      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="red" />
      </mesh>
    </Canvas>
  );
};

export default App;
```

이제 마우스로 드래그하면 장면을 회전해서 볼 수 있습니다.

---

## 색상이 보이지 않는 이유

위 코드에서 `meshStandardMaterial color="red"`를 설정했는데도 물체가 어둡거나 검게 보일 수 있습니다.

이유는 `meshStandardMaterial`이 조명에 반응하는 재질이기 때문입니다.

조명이 없으면 물체 표면이 제대로 밝아지지 않습니다.

---

## 조명 추가하기

```tsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

const App = () => {
  return (
    <Canvas>
      <OrbitControls />
      <ambientLight intensity={0.4} />
      <directionalLight position={[2, 2, 2]} intensity={1} />

      <mesh>
        <torusGeometry />
        <meshStandardMaterial color="red" />
      </mesh>
    </Canvas>
  );
};

export default App;
```

`ambientLight`는 전체를 은은하게 밝히고, `directionalLight`는 특정 방향에서 빛을 비춥니다.

두 조명을 함께 사용하면 물체가 훨씬 자연스럽게 보입니다.

---

## Material 종류

Material은 3D 물체의 표면을 결정합니다.

| Material | 특징 |
| --- | --- |
| `meshBasicMaterial` | 조명 영향을 받지 않고 색상만 표시 |
| `meshStandardMaterial` | 현실적인 표면 표현에 자주 사용 |
| `meshPhongMaterial` | 반짝이는 하이라이트 표현에 유용 |
| `meshLambertMaterial` | 광택이 적은 무광 표면 표현 |
| `meshToonMaterial` | 만화 같은 스타일 표현 |

처음에는 `meshStandardMaterial`을 기본값처럼 사용해도 충분합니다.

---

## meshPhongMaterial 사용하기

반짝이는 표면을 만들고 싶다면 `meshPhongMaterial`을 사용할 수 있습니다.

```tsx
<mesh>
  <torusGeometry />
  <meshPhongMaterial color="red" shininess={100} />
</mesh>
```

`shininess` 값이 클수록 하이라이트가 더 날카롭고 강하게 보입니다.

---

## 자주 사용하는 조명

| 조명 | 역할 |
| --- | --- |
| `ambientLight` | 장면 전체를 균일하게 밝힘 |
| `directionalLight` | 특정 방향에서 비추는 빛 |
| `pointLight` | 한 점에서 모든 방향으로 퍼지는 빛 |
| `spotLight` | 특정 방향으로 집중되는 빛 |
| `hemisphereLight` | 하늘과 지면 색을 나누어 비추는 빛 |

간단한 장면에서는 `ambientLight`와 `directionalLight` 조합만으로도 충분히 보기 좋은 결과를 만들 수 있습니다.

---

## 마무리

3D 장면에서 material과 조명은 물체의 분위기를 결정하는 중요한 요소입니다.

색상을 지정했는데 물체가 어둡게 보인다면 먼저 조명이 있는지 확인해보면 좋습니다.

기본적으로는 `meshStandardMaterial`, `ambientLight`, `directionalLight` 조합으로 시작하고, 필요에 따라 다른 material과 조명을 하나씩 실험해보면 이해하기 쉽습니다.
