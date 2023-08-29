---
title: "[three.js] React에서 three.js 사용(useControls)"
date: 2023-08-28T00:24:000
categories: [three.js, React]
tags: [three.js, react] #소문자만 가능
---

---

## <b>useControls란?</b>

<strong>useControls</strong>는 leva 라이브러리에서 제공하는 함수로, UI 컴포넌트를 생성하고 웹화면에서 실시간으로 상호작용적으로 값을 변경할 수 있게 해줍니다. 3D 객체의 속성을 조작하기 위해 사용할 수 있으며, 이를 R3F에서 사용하면 편리한 UI를 제공할 수 있습니다.

## <b>설치</b>

```js
npm install leva
```

## <b>useControls 사용</b>

여러가지 속성을 사용할 수 있지만 기본적인 position으로 사용해보겠습니다.

```jsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
import { useControls } from "leva";
import React from "react";

function MyScene() {
  // useControls로 UI 컴포넌트 생성 및 초기값 설정
  const { positionX, positionY, positionZ } = useControls({
    positionX: { value: 0, min: -10, max: 10 },
    positionY: { value: 0, min: -10, max: 10 },
    positionZ: { value: 0, min: -10, max: 10 },
  });

  return (
    <Canvas>
      <OrbitControls />
      <mesh position={[positionX, positionY, positionZ]}>
        <boxGeometry />
        <meshBasicMaterial color="red" />
      </mesh>
    </Canvas>
  );
}

export default MyScene;
```

그러면 아래와 같이 물체 옆에 못 보던 조작패드가 생긴걸 볼 수 있다.<br/>
아까 지정해줬던 속성에 이름과 value 값으로 구성되어 있으며 수치를 마우스를 통해서 드래그하면 실시간으로 mesh에 position에 값이 적용되는 걸 볼 수 있다.

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/645eb109-38b2-40ea-8a86-2e83b904c9b3" alt="useControls 적용"/>

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
