---
title: "[three.js] React에서 three.js 사용(3D공간, 재질&스타일링, 빛)"
date: 2023-08-27T23:37:000
categories: [three.js, React]
tags: [three.js, react] #소문자만 가능
---

---

<a href="https://taewok.github.io/posts/react-three.js(1)">React에서 three.js 사용(1)</a><br/>
지난 글에 이어서 평평해 보이는 직육면체를 3D로 보이게 만들고 빛이란 것도 추가해볼 것 이다.

## <b>3D로 만들기</b>

직육면체를 우리가 원하는 <strong>3D</strong>처럼 보이게 하는 것은 간단하다 <strong>OrbitControls</strong> 컴포넌트를 <strong>Canvas</strong> 컴포넌트 안에 넣어주면 된다.

```jsx
//App.js
import { Canvas } from "@react-three/fiber";
import { Box, OrbitControls } from "@react-three/drei";

function App() {
  return (
    <Canvas>
      <OrbitControls />
      <Box />
    </Canvas>
  );
}

export default App;
```

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/646f55ef-adbd-4a87-b9ac-1d058fc3028a"/>

## <b>재질 & 스타일링, 빛</b>

재질이 변하는 것을 쉽게 확인하기 위해 조명에 종류를 <strong>directionalLight</strong> 변경하고 직육면체가 아닌 도넛 모양에 <strong>torusGeometry</strong>로 변경해줍니다.

```jsx
//App.js
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

function App() {
  return (
    <Canvas>
      <OrbitControls />
      <directionalLight position={[1, 1, 1]} />
      <mesh>
        <torusGeometry />
        <meshStandardMaterial color="red" />
      </mesh>
    </Canvas>
  );
}

export default App;
```

<strong>meshStandardMaterial</strong>에 color 속성을 사용해 red로 바꿧지만 검은색으로 나오고 있다.<br/>
이유는 빛! 조명이 없기 떄문이다. 캄캄한 어둠에는 언제나 검은색을 보인다.

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/00ba499c-b1be-41c6-858b-8403419051be" alt="meshStandardMaterial 색상변경"/>

```jsx
//App.js
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

function App() {
  return (
    <Canvas>
      <OrbitControls />
      <directionalLight position={[1, 1, 1]} />
      <mesh>
        <torusGeometry />
        <meshStandardMaterial color="red" />
      </mesh>
    </Canvas>
  );
}

export default App;
```

장면 전체에 균일한 조명을 추가해주는 <strong>ambientLight</strong> 조명을 추가해 <strong>meshStandardMaterial</strong> 속성에 color="red"로 인한 붉은색이 잘 보입니다.

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/a3af00aa-4147-4d48-be71-280bf4294746" alt="ambientLight 적용"/>

여기에 빛을 반사하는 듯한 물체로 재질을 바꾸고 싶다면 빛에 반응하여 물체의 표면을 반사하고 반사광을 만드는 <strong>MeshPhongMaterial</strong>을 사용해야 한다.<br/>
그리고 장면 전체에 균일한 조명으로는 파악하기 힘드므로 특정 방향에서 평행한 빛을 생성하는 <strong>DirectionalLight</strong> 조명을 사용한다.

```jsx
//App.js
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";

function App() {
  return (
    <Canvas>
      <OrbitControls />
      <directionalLight position={[1, 1, 1]} />
      <mesh>
        <torusGeometry />
        <meshPhongMaterial shininess={1700} color="red" />
      </mesh>
    </Canvas>
  );
}

export default App;
```

<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/a5aeca46-d317-49de-a4b1-1b3563b3ccc7" alt="shininess 재질 적용"/>

<h3><blockquote>Material
</blockquote></h3>

현실적인 조명, 그림자, 반사, 굴절 등을 표현하여 물체를 더 현실적으로 보이게 하는 데 사용됩니다

#### 💡 Material 종류

|         컴포넌트명          | 기능                                                                                                        |
| :-------------------------: | ----------------------------------------------------------------------------------------------------------- |
|       <b>Material</b>       | 이 컴포넌트는 주어진 재질로 3D 모델을 렌더링할 수 있도록 도와줍니다. 다양한 재질 옵션을 설정할 수 있습니다. |
|  <b>MeshBasicMaterial</b>   | 기본적인 물체 재질로, 빛에 반응하지 않고 색상만 표현합니다.                                                 |
| <b>MeshStandardMaterial</b> | 표준 재질로, 빛에 반응하여 그림자와 하이라이트를 생성하며 물체를 더 현실적으로 만들어 줍니다.               |
|  <b>MeshPhongMaterial</b>   | 포앙 재질로, 빛에 반응하여 물체의 표면을 반사하고 반사광을 만듭니다.                                        |
| <b>MeshLambertMaterial</b>  | 램버트 재질로, 빛에 대한 광택이 없는 물체를 나타냅니다.                                                     |
|   <b>MeshToonMaterial</b>   | 툰 재질로, 쉐이딩을 통해 물체를 단순하게 나타냅니다. 일반적으로 만화 스타일의 렌더링에 사용됩니다.          |
| <b>MeshPhysicalMaterial</b> | 물리 재질로, 물리 기반의 재질 렌더링을 지원합니다. 실제 재질과 유사한 시각적 효과를 얻을 수 있습니다.       |
|    <b>PointsMaterial</b>    | 포인트 재질로, 점 또는 입자 시스템에 사용됩니다.                                                            |

#### 💡 공통 속성

|       속성명        | 기능                                                      |
| :-----------------: | --------------------------------------------------------- |
|    <b> color</b>    | 재질의 색상을 정의합니다.                                 |
| <b> transparent</b> | 물체가 투명한지 여부를 지정합니다.                        |
|   <b> opacity</b>   | 물체의 투명도를 조절합니다.                               |
|    <b> side</b>     | 물체의 어떤 면을 렌더링할지 설정합니다.                   |
|   <b> visible</b>   | 물체의 가시성 여부를 조절합니다.                          |
|  <b> wireframe</b>  | 물체를 와이어프레임으로 표현할지 여부를 설정합니다.       |
|     <b> map</b>     | 텍스쳐 맵을 지정합니다.                                   |
|  <b> normalMap</b>  | 노말 맵을 지정하여 물체의 표면 노말을 조절할 수 있습니다. |
|   <b> envMap</b>    | 환경 맵을 적용하여 반사 효과를 생성할 수 있습니다.        |

#### 💡 각 Material의 고유 속성

|          Material명          | 기능                                                                                                                                                               |
| :--------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <b> MeshStandardMaterial</b> | roughness: 물체의 거칠기를 조절합니다. 값이 낮을수록 빛이 더 많이 반사됩니다.<br/>metalness: 물체의 금속성을 조절합니다. 값이 높을수록 물체가 금속적으로 보입니다. |
|  <b> MeshPhongMaterial</b>   | shininess: 물체의 광택을 조절합니다. 값이 높을수록 광택이 더 많이 나타납니다.                                                                                      |
|   <b> MeshToonMaterial</b>   | gradientMap: 그라데이션 맵을 적용하여 툰 렌더링 스타일을 설정할 수 있습니다.                                                                                       |
|    <b> PointsMaterial</b>    | size: 포인트(입자)의 크기를 조절합니다.                                                                                                                            |

<h3><blockquote>조명
</blockquote></h3>

3D 장면에 깊이와 입체감을 부여하고 물체 간의 시각적 관계를 강조하는 역할을 합니다. 조명은 장면의 물체에 그림자와 하이라이트를 생성하여 현실적인 느낌을 연출하거나, 특정 분위기나 감정을 전달하는 데 사용됩니다.1

#### 💡 조명 종류

|                조명명                 | 기능                                                                                                                                     |
| :-----------------------------------: | ---------------------------------------------------------------------------------------------------------------------------------------- |
|   <b>AmbientLight (주변 조명) </b>    | 장면 전체에 균일한 조명을 추가합니다. 물체에 그림자나 강조 효과는 없고 모든 방향에서 동일한 밝기로 조명됩니다.                           |
|   <b>DirectionalLight (직사광) </b>   | 특정 방향에서 나오는 평행한 빛을 생성합니다. 햇빛과 비슷한 효과로, 조명이 특정 방향으로 투사되며 그림자가 만들어질 수 있습니다.          |
|       <b>PointLight (점광) </b>       | 특정 지점에서 모든 방향으로 빛을 발산하는 조명입니다. 공간 내에서 빛이 방출되며, 주변에 있는 물체에 그림자와 광택 효과를 줄 수 있습니다. |
|   <b>SpotLight (스포트 라이트) </b>   | 특정 위치에서 특정 방향으로 빛을 집중시켜 조명하는 방식입니다. 터치 라이트나 스포트라이트와 유사한 효과를 만들 수 있습니다.              |
|  <b>HemisphereLight (반구 조명)</b>   | 하늘과 지면처럼 두 개의 다른 색상을 가진 두 개의 빛을 사용해 조명하는 방식입니다. 주변 조명과 그림자의 연출에 활용할 수 있습니다.        |
| <b>RectAreaLight (사각 영역 조명)</b> | 특정 영역에서 빛을 발산하는 조명입니다. 평면 또는 사각형 형태의 영역에서 빛을 투사하며 조명 효과를 만들 수 있습니다.                     |

#### 💡 주요 속성

|                      속성명                       | 기능                                                                                                                                           |
| :-----------------------------------------------: | ---------------------------------------------------------------------------------------------------------------------------------------------- |
|              <b>position (위치) </b>              | 조명의 위치를 지정합니다. 배열로 x, y, z 좌표 값을 입력하여 조명이 어디에서 발산하는지 결정할 수 있습니다.                                     |
|               <b>color (색상) </b>                | 조명의 색상을 지정합니다. RGB 값을 입력하여 조명의 빛 색상을 조절할 수 있습니다.                                                               |
|             <b>intensity (강도) </b>              | 조명의 강도를 조절합니다. 0에서 1 사이의 값을 입력하여 조명의 밝기를 설정할 수 있습니다.                                                       |
|              <b>distance (거리) </b>              | 조명의 최대 효과 거리를 지정합니다. 조명에서 멀어질수록 빛의 강도가 약해집니다.                                                                |
|               <b>decay (감쇠) </b>                | 조명의 감쇠를 설정합니다. 거리에 따라 빛이 어떻게 감쇠하는지를 결정하는 계수입니다.                                                            |
|               <b>angle (각도) </b>                | 스포트 라이트 조명의 조준 각도를 지정합니다. 0에서 Math.PI/2 사이의 값을 가집니다.                                                             |
|       <b>penumbra (빛 중심 영역 비율) </b>        | 스포트 라이트 조명의 빛 중심 영역과 그림자 영역의 비율을 조절합니다. 0에서 1 사이의 값이며, 값이 작을수록 그림자의 경계가 부드럽게 변화합니다. |
|         <b>castShadow (그림자 투사) </b>          | 조명이 그림자를 생성하는지 여부를 나타내는 불리언 값입니다. true로 설정하면 해당 조명이 그림자를 만듭니다.                                     |
| <b>shadowCameraNear (그림자 카메라 최소 거리)</b> | 그림자를 만들 때 사용하는 카메라의 최소 효과 거리를 지정합니다.                                                                                |
| <b>shadowCameraFar (그림자 카메라 최대 거리) </b> | 그림자를 만들 때 사용하는 카메라의 최대 효과 거리를 지정합니다.                                                                                |
|  <b>shadowCameraFov (그림자 카메라 시야각) </b>   | 그림자를 만들 때 사용하는 카메라의 시야각을 지정합니다.                                                                                        |
|       <b>shadowBias (그림자 바이어스) </b>        | 그림자의 경계를 선명하게 하거나 조정하기 위해 사용하는 바이어스 값입니다.                                                                      |

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
