---
title: "[three.js] React에서 three.js 사용(라이브러리 설치와 직육면체 생성)"
date: 2023-08-24T17:04:000
categories: [three.js, React]
tags: [three.js, react] #소문자만 가능
---

---

## <b>three.js란?</b>

<img src="https://images.velog.io/images/9rganizedchaos/post/65b3a471-7b9c-4e23-9eae-718018bb33ef/img.png" alt="three.js logo"/>

웹 브라우저에서 3D 그래픽을 생성하고 렌더링하기 위한 JavaScript 라이브러리입니다. <br/>
Three.js를 사용하면 웹 페이지에서 다양한 3D 요소를 만들고 조작할 수 있습니다.

## <b>라이브러리 설치</b>

```js
npm install three
npm install @react-three/fiber
npm install @react-three/drei
```

- <b>@react-three/fiber</b>: React와 Three.js를 결합하여 웹 브라우저에서 3D 그래픽을 렌더링하기 위한 라이브러리이며 더욱 쉽게 3D 그래픽을 웹 애플리케이션에 통합할 수 있도록 도와줍니다.
- <b>@react-three/drei</b>: Three.js의 다양한 설정과 기능들을 간소화해주며 편리한 컴포넌트 제공하여 다양한 종류의 3D 그래픽 요소를 더 간단하게 생성하고 설정할 수 있습니다.

## <b>직육면체 생성</b>

웹에 3D 직육면체를 생성하기 위한 기본적인 코드이다

```jsx
//App.js
import { Canvas } from "@react-three/fiber";
import { Box } from "@react-three/drei";

function App() {
  return (
    <Canvas>
      <mesh>
        <Box />
      </mesh>
    </Canvas>
  );
}

export default App;
```

생성된 직육면체
<img src="https://github.com/TWOGATH3R/twogather-web-frontend/assets/88264006/5a317378-725c-4d1d-aa61-36dfea4c8dc2"/>

이제 사용된 태그와 컴포넌트 들에 대해 알아보자

<h3><blockquote>Canvas 컴포넌트
</blockquote></h3>

이 태그를 사용하면 Three.js와 React를 결합하여 3D 그래픽을 화면에 표시하고 조작하는 환경을 만들 수 있습니다.

#### 💡 주요 속성

|         속성명         | 기능                                                                     |
| :--------------------: | ------------------------------------------------------------------------ |
|     <b>camera</b>      | 렌더링에 사용할 카메라를 설정합니다. 기본값은 PerspectiveCamera 입니다.  |
|       <b>gl</b>        | WebGL 렌더링 컨텍스트를 커스터마이징하려면 이 속성을 사용할 수 있습니다. |
|    <b>onCreated</b>    | 렌더링 컨텍스트가 생성된 후 실행될 콜백 함수를 지정합니다.               |
|   <b>pixelRatio</b>    | 렌더링의 픽셀 비율을 설정합니다.                                         |
|    <b>shadowMap</b>    | 그림자 맵을 활성화하거나 비활성화합니다.                                 |
| <b>colorManagement</b> | 컬러 매니지먼트를 설정하여 색상을 조절할 수 있습니다.                    |
|   <b>concurrent</b>    | 병렬 렌더링을 활성화하거나 비활성화합니다. 기본값은 true입니다.          |

<h3><blockquote>Box 컴포넌트
</blockquote></h3>

Three.js의 BoxGeometry와 MeshStandardMaterial을 내부적으로 생성하고 조합하여 쉽게 3D 박스 모양의 그래픽을 생성하고 렌더링할 수 있는 방법을 제공합니다. 이를 사용하면 코드를 더 간결하게 작성할 수 있습니다.

#### 💡 주요 속성

|     속성명      | 기능                                                                                                           |
| :-------------: | -------------------------------------------------------------------------------------------------------------- |
|   <b>args</b>   | 배열 형태로 [width, height, depth] 값을 받아 박스의 너비, 높이 및 깊이를 설정합니다. 기본값은 [1, 1, 1]입니다. |
| <b>position</b> | 배열 형태로 [x, y, z] 값을 받아 박스의 위치를 설정합니다.                                                      |
| <b>rotation</b> | 배열 형태로 [x, y, z] 값을 받아 박스의 회전을 설정합니다.                                                      |
|  <b>scale</b>   | 배열 형태로 [x, y, z] 값을 받아 박스의 크기를 설정합니다.                                                      |
|  <b>color</b>   | 3D 요소에 색을 지정                                                                                            |

박스 모양만이 아닌 여러가지 모양에 대한 컴포넌트가 있다.

|     컴포넌트명      | 설명                                      |
| :-----------------: | ----------------------------------------- |
|    <b>Sphere</b>    | 구 형태의 3D 모델을 생성합니다.           |
|   <b>Cylinder</b>   | 원기둥 형태의 3D 모델을 생성합니다.       |
|    <b>Plane</b>     | 평면 형태의 3D 모델을 생성합니다.         |
|    <b>Torus</b>     | 토러스(고리) 형태의 3D 모델을 생성합니다. |
|  <b>TorusKnot</b>   | 토러스 노트 형태의 3D 모델을 생성합니다.  |
|     <b>Text</b>     | 텍스트를 3D 모델로 생성합니다.            |
| <b>Icosahedron</b>  | 이코사헤드론 형태의 3D 모델을 생성합니다. |
|  <b>Octahedron</b>  | 옥타헤드론 형태의 3D 모델을 생성합니다.   |
| <b>Tetrahedron</b>  | 테트라헤드론 형태의 3D 모델을 생성합니다. |
| <b>Dodecahedron</b> | 도데카헤드론 형태의 3D 모델을 생성합니다. |

<h3><blockquote>mesh 태그
</blockquote></h3>

Three.js의 Mesh 객체를 생성하고 렌더링하는 일들을 축약해놓은 아주 편리한 태그다.

#### 💡 주요 속성

|        속성명        | 기능                                                                                       |
| :------------------: | ------------------------------------------------------------------------------------------ |
|   <b>position</b>    | 3D 공간에서 해당 Mesh의 위치를 지정합니다. [x, y, z] 형식으로 좌표를 설정할 수 있습니다.   |
|   <b>rotation</b>    | 3D 공간에서 해당 Mesh의 회전을 지정합니다. [x, y, z] 형식으로 회전값을 설정할 수 있습니다. |
|     <b>scale</b>     | 해당 Mesh의 크기를 지정합니다. [x, y, z] 형식으로 크기를 설정할 수 있습니다.               |
|   <b>geometry</b>    | Mesh의 지오메트리(형태)를 지정합니다. 이 속성을 사용하여 Mesh의 모양을 결정합니다.         |
|   <b>material</b>    | Mesh의 재질을 지정합니다. 재질은 Mesh의 색상, 텍스처, 광택 등을 결정합니다.                |
|    <b>visible</b>    | Mesh의 가시성 여부를 지정합니다. 기본값은 true입니다.                                      |
|  <b>castShadow</b>   | Mesh가 그림자를 만들어내는지 여부를 지정합니다.                                            |
| <b>receiveShadow</b> | Mesh가 그림자를 받는지 여부를 지정합니다.                                                  |

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
