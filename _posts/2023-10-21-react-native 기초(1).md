---
title: "[React Native] 기본 컴포넌트와 스타일링 정리"
date: 2023-10-21T22:30:00
categories: [react-native]
tags: [react-native, component, stylesheet]
description: "React Native에서 자주 사용하는 View, Text, Button, TextInput, FlatList와 StyleSheet 사용법을 정리했습니다."
custom_style: true
---

## 들어가며

React Native는 React 문법을 사용하지만, 웹의 HTML 태그를 그대로 사용하지는 않습니다.

웹에서는 `div`, `p`, `button`, `input`을 사용하지만 React Native에서는 모바일 네이티브 UI에 맞는 컴포넌트를 사용합니다.

이번 글에서는 React Native를 처음 시작할 때 자주 만나는 기본 컴포넌트를 정리해볼게요.

---

## View

`View`는 레이아웃을 구성하는 가장 기본적인 컨테이너입니다.

웹의 `div`와 비슷한 역할을 합니다.

```tsx
import { View } from "react-native";

const App = () => {
  return (
    <View>
      {/* 다른 컴포넌트를 이 안에 배치합니다. */}
    </View>
  );
};

export default App;
```

여러 컴포넌트를 묶거나 화면 구조를 나눌 때 사용합니다.

---

## Text

React Native에서는 텍스트를 반드시 `Text` 컴포넌트 안에 넣어야 합니다.

```tsx
import { Text } from "react-native";

const App = () => {
  return <Text>Hello, React Native!</Text>;
};

export default App;
```

웹처럼 아무 태그 안에나 문자열을 바로 넣는 방식과 다르기 때문에 처음에 자주 실수하는 부분입니다.

---

## Button

`Button`은 기본 버튼 컴포넌트입니다.

```tsx
import { Button } from "react-native";

const App = () => {
  return (
    <Button
      title="클릭"
      onPress={() => {
        console.log("button pressed");
      }}
    />
  );
};

export default App;
```

웹의 `onClick` 대신 React Native에서는 `onPress`를 사용합니다.

---

## TextInput

사용자 입력을 받을 때는 `TextInput`을 사용합니다.

```tsx
import { useState } from "react";
import { TextInput } from "react-native";

const App = () => {
  const [text, setText] = useState("");

  return (
    <TextInput
      value={text}
      onChangeText={setText}
      placeholder="텍스트를 입력해주세요"
    />
  );
};

export default App;
```

웹의 input과 달리 텍스트 변경 이벤트는 `onChangeText`를 자주 사용합니다.

---

## FlatList

목록을 렌더링할 때는 `FlatList`를 사용할 수 있습니다.

```tsx
import { FlatList, Text } from "react-native";

const data = [
  { id: "1", name: "Apple" },
  { id: "2", name: "Banana" },
  { id: "3", name: "Grape" },
];

const App = () => {
  return (
    <FlatList
      data={data}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
};

export default App;
```

`data`에는 목록 배열을 넣고, `renderItem`에서는 각 항목을 어떻게 보여줄지 작성합니다.

---

## StyleSheet 사용하기

React Native에서는 CSS 파일 대신 JavaScript 객체 형태로 스타일을 작성합니다.

```tsx
import { StyleSheet, Text, View } from "react-native";

const App = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello, React Native!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: "lightblue",
  },
  text: {
    fontSize: 18,
    color: "darkblue",
  },
});

export default App;
```

CSS의 `background-color`처럼 kebab-case를 쓰지 않고, `backgroundColor`처럼 camelCase를 사용합니다.

---

## 마무리

React Native는 React와 문법은 비슷하지만 사용하는 기본 컴포넌트가 다릅니다.

레이아웃은 `View`, 텍스트는 `Text`, 입력은 `TextInput`, 목록은 `FlatList`를 사용합니다.

처음에는 웹 태그와 비교하면서 역할을 익히면 훨씬 쉽게 적응할 수 있습니다.
