---
id: intro-react-native-components
title: Core Components and Native Components
description: 'React Native lets you compose app interfaces using Native Components. Conveniently, it comes with a set of these components for you to get started with right now—the Core Components!'
---

import ThemedImage from '@theme/ThemedImage';

React Native 是一個開源框架，用於使用 [React](https://reactjs.org/) 和平台原生功能來構建 Android 和 iOS 應用程式。透過 React Native，你可以使用 JavaScript 存取平台的 API，並使用 React 元件（可重複使用、可嵌套的程式碼包）來描述 UI 的外觀和行為。你可以在下一節中了解更多關於 React 的內容。但首先，讓我們來看看 React Native 中的元件是如何運作的。

## 視圖與行動開發

In Android and iOS development, a **view** is the basic building block of UI: a small rectangular element on the screen which can be used to display text, images, or respond to user input. Even the smallest visual elements of an app, like a line of text or a button, are kinds of views. Some kinds of views can contain other views. It’s views all the way down!

<figure>
  <img src="/docs/assets/diagram_ios-android-views.svg" width="1000" alt="Diagram of Android and iOS app showing them both built on top of atomic elements called views." />
  <figcaption>Just a sampling of the many views used in Android and iOS apps.</figcaption>
</figure>

## 原生元件

In Android development, you write views in Kotlin or Java; in iOS development, you use Swift or Objective-C. With React Native, you can invoke these views with JavaScript using React components. At runtime, React Native creates the corresponding Android and iOS views for those components. Because React Native components are backed by the same views as Android and iOS, React Native apps look, feel, and perform like any other apps. We call these platform-backed components **Native Components.**

React Native 提供了一組現成的、可直接使用的原生元件，你可以立即開始構建你的應用程式。這些是 React Native 的**核心元件（Core Components）**。

:::caution
本文件參考了一組舊版 API，需要更新以反映新架構
:::

React Native 還允許你為 [Android](legacy/native-components-android.md) 和 [iOS](legacy/native-components-ios.md) 構建自己的原生元件，以滿足應用的獨特需求。我們還有一個活躍的生態系統，提供**社群貢獻的元件**。請查看 [Native Directory](https://reactnative.directory) 以了解社群創建的內容。

## 核心元件

React Native 有許多核心元件，從控制項到活動指示器應有盡有。你可以在 [API 部分](components-and-apis) 找到所有這些元件的文件。你主要會使用以下核心元件：

| React Native UI Component | Android View   | iOS View         | Web Analog              | Description                                                                                           |
| ------------------------- | -------------- | ---------------- | ----------------------- | ----------------------------------------------------------------------------------------------------- |
| `<View>`                  | `<ViewGroup>`  | `<UIView>`       | A non-scrolling `<div>` | A container that supports layout with flexbox, style, some touch handling, and accessibility controls |
| `<Text>`                  | `<TextView>`   | `<UITextView>`   | `<p>`                   | Displays, styles, and nests strings of text and even handles touch events                             |
| `<Image>`                 | `<ImageView>`  | `<UIImageView>`  | `<img>`                 | Displays different types of images                                                                    |
| `<ScrollView>`            | `<ScrollView>` | `<UIScrollView>` | `<div>`                 | A generic scrolling container that can contain multiple components and views                          |
| `<TextInput>`             | `<EditText>`   | `<UITextField>`  | `<input type="text">`   | Allows the user to enter text                                                                         |

在下一節中，你將開始結合這些核心元件，了解 React 的工作原理。現在就來試試它們吧！

```SnackPlayer name=Hello%20World
import React from 'react';
import {View, Text, Image, ScrollView, TextInput} from 'react-native';

const App = () => {
  return (
    <ScrollView>
      <Text>Some text</Text>
      <View>
        <Text>Some more text</Text>
        <Image
          source={{
            uri: 'https://reactnative.dev/docs/assets/p_cat2.png',
          }}
          style={{width: 200, height: 200}}
        />
      </View>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1,
        }}
        defaultValue="You can type in me"
      />
    </ScrollView>
  );
};

export default App;
```

---

由於 React Native 使用與 React 元件相同的 API 結構，你需要了解 React 元件 API 才能開始使用。[下一節](intro-react) 提供了關於該主題的快速介紹或複習。不過，如果你已經熟悉 React，可以[直接跳過](handling-text-input)。

<ThemedImage
alt="A diagram showing React Native's Core Components are a subset of React Components that ship with React Native."
sources={{
  light: '/docs/assets/diagram_react-native-components.svg',
  dark: '/docs/assets/diagram_react-native-components_dark.svg',
}}
/>