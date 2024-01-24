---
title: "[React-Native] 기초 학습"
date: 2023-10-21T22:30:000
categories: [React-Native]
tags: [react-native] #소문자만 가능
---

---

<p>React-Native를 학습하기 위해 노마드 코더의 <a href="https://nomadcoders.co/react-native-for-beginners/lectures/3252">왕초보를 위한 React Native 101</a> 무료 강의를 들으며 새롭게 알게 된 React-Native의 지식을 써 내려가며 다시 한번 학습해야겠다.</p>
<br/>

## React-Native 컴포넌트란?

<p>React와 비슷한 구문 및 컨셉을 사용하지만 React-Native는 HTML태그가 아닌 react-native에서 지원하는 자체 컴포넌트를 사용한다.
<br/>
<br/>
예를 들어, 웹에서는 <strong>&lt;div&gt;,  &lt;p&gt;,  &lt;button&gt;,  &lt;input&gt;</strong>와 같은 HTML 태그를 사용하지만, React Native에서는 <strong>&lt;View&gt;,  &lt;Text&gt;,  &lt;Button&gt;,  &lt;TextInput&gt;</strong>과 같은 리액트 네이티브 컴포넌트를 사용하며 이러한 컴포넌트는 모바일 앱의 네이티브 UI 구성 요소와 일치하도록 변환된다.
</p>
<br/>

### &lt;View&gt; (View == div)

<p>View는 리액트 네이티브에서 컨테이너로 사용되는 컴포너트로 &lt;div&gt;와 마찬가지로 여러 컴포넌트나 요소를 감싸고, 레이아웃을 구성할 때 사용합니다.</p>

```jsx
import { View } from "react-native";

<View style={styles.container}>
  {/* 다른 컴포넌트들을 이 안에 배치할 수 있음 */}
</View>;
```

<br/>

### &lt;Text&gt; (Text == p, span, h1)

<p>Text는 컴포넌트는 기존의 모든 텍스트를 표시하는 데 사용되며, 여러 형식의 텍스트 표시에 유용합니다. HTML의 <strong>&lt;p&gt;, &lt;span&gt;, &lt;h1&gt;</strong>과 유사한 역할을 하며 텍스트 관련 스타일링을 적용할 수 있습니다.</p>

```jsx
import { Text } from "react-native";

<Text>{/* 이 텍스트는 리액트 네이티브 Text 컴포넌트로 표시됩니다. */}</Text>;
```

<br/>

### &lt;Button&gt; (Button == button)

<p>Button은 클릭 가능한 버튼을 생성하는 데 사용되며
HTML의 <strong>&lt;button&gt;</strong>과 유사한 역할을 하며, 사용자 상호작용을 처리할 때 유용하다.</p>

```jsx
import { Button } from "react-native";

<Button
  title="클릭하세요"
  onPress={() => {
    // 클릭 이벤트 핸들러
  }}
/>;
```

<br/>

### &lt;TextInput&gt; (Button == input, textarea)

<p>TextInput은 사용자의 입력을 받기 위해 사용되는 컴포넌트이며
HTML의 <strong>&lt;input&gt;, &lt;textarea&gt;</strong>와 유사한 역할을 하며, 텍스트 입력 필드를 생성하는 데 사용된다.</p>

```jsx
import { TextInput } from "react-native";

<TextInput
  placeholder="여기에 입력하세요"
  value=""
  onChangeText={(text) => {
    // 입력 값 변경 이벤트 핸들러
  }}
/>;
```

<br/>

### &lt;FlatList&gt; (FlatList == ol, ul)

<p>FlatList 컴포넌트는 React-Native에서 목록 데이터를 효과적으로 렌더링하기 위한 도구이며 HTML의 <strong>&lt;ol&gt;, &lt;ul&gt;</strong>와 유사하다.</p>

```jsx
import { FlatList } from "react-native";

const data = [
  { key: "item1", name: "Item 1" },
  { key: "item2", name: "Item 2" },
  { key: "item3", name: "Item 3" },
];

<FlatList
  data={data}
  keyExtractor={(item) => item.key} // key를 추출하는 함수
  renderItem={({ item }) => <Text>{item.name}</Text>}
/>;
```

- FlatList속성
  - data: 표시할 배열 데이터를 지정하는 속성
  - renderItem: data속성에서 지정한 데이터를 매개변수로 받고 해당 항목을 렌더링하는 JSX를 반환
  - keyExtractor: data속성에서 지정한 데이터를 매개변수로 받고 해당 데이터에서 키를 지정하거나 추출

<br/>

## 스타일링

<p>React Native에서는 CSS와 비슷한 스타일 속성 및 값들을 사용하며(background-color==backgroundColor) 스타일을 최적화하기 위해 StyleSheet.create 함수를 제공하는데 이 함수를 사용하여 스타일 객체를 생성하고 컴포넌트의 style 속성에 적용할 수 있다..</p>

```jsx
import { StyleSheet, View, Text } from "react-native";

const styles = StyleSheet.create({
  container: {
    backgroundColor: "lightblue",
    padding: 10,
  },
  text: {
    fontSize: 16,
    color: "darkblue",
  },
});

const MyComponent = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello, React Native!</Text>
    </View>
  );
};
```

<p>더 복잡한 앱에서는 외부 스타일 파일을 만들고 불러와 사용할 수도 있다.</p>
<br/>

```jsx
// MyComponentStyles.js
import { StyleSheet } from "react-native";

export default StyleSheet.create({
  container: {
    backgroundColor: "lightblue",
    padding: 10,
  },
  text: {
    fontSize: 16,
    color: "darkblue",
  },
});
```

```jsx
// MyComponent.js
import React from "react";
import { View, Text } from "react-native";
import styles from "./MyComponentStyles";

const MyComponent = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello, React Native!</Text>
    </View>
  );
};
```

---

## <b style="border-bottom:2px solid gray"><b>마치며</b></b>

<P>혹시 잘못된 정보나 궁금하신 게 있다면 편하게 댓글 달아주세요.<br/>
지적이나 피드백은 언제나 환영입니다.</p>
