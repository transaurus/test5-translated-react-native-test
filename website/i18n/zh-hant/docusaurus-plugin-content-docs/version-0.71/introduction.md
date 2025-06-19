---
id: getting-started
title: Introduction
description: This helpful guide lays out the prerequisites for learning React Native, using these docs, and setting up your environment.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<div className="content-banner">
  Welcome to the very start of your React Native journey! If you're looking for environment setup instructions, they've moved to <a href="environment-setup">their own section</a>. Continue reading for an introduction to the documentation, Native Components, React, and more!
  <img className="content-banner-img" src="/docs/assets/p_android-ios-devices.svg" alt=" " />
</div>

React Native 的使用者背景多元：從資深 iOS 開發者到 React 初學者，甚至是首次接觸程式設計的轉職者。本文檔的撰寫考量了所有學習者，無論其經驗水平或技術背景為何。

## 如何使用本文檔

您可以從頭開始像閱讀書籍般線性閱讀，或直接查閱特定章節。若已熟悉 React？可跳過[該章節](intro-react)或快速瀏覽作為複習。

## 前置知識

使用 React Native 需具備 JavaScript 基礎知識。若需入門或複習，可參考 Mozilla Developer Network 的[JavaScript 教學](https://developer.mozilla.org/en-US/docs/Web/JavaScript)或[快速回顧指南](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)。

> 雖然我們盡量避免預設讀者具備 React、Android 或 iOS 開發知識，但這些領域的學習對 React Native 開發者極有價值。適當時機我們會提供進階資源連結。

## 互動式範例

本篇導覽讓您能直接在瀏覽器中操作互動範例，如下所示：

```SnackPlayer name=Hello%20World
import React from 'react';
import {Text, View} from 'react-native';

const YourApp = () => {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
      }}>
      <Text>Try editing me! 🎉</Text>
    </View>
  );
};

export default YourApp;
```

上方是 Expo 開發的 Snack Player 工具，可嵌入並運行 React Native 專案，即時預覽 Android 與 iOS 平台的渲染效果。程式碼可即時編輯，您可直接在瀏覽器中嘗試將「Try editing me!」文字改為「Hello, world!」。

> 若需建立本地開發環境，可遵循[本地環境設置指南](environment-setup)，並將程式碼範例貼至您的 `App.js` 檔案中。（網頁開發者可能已具備行動瀏覽器測試環境！）

## 函式元件與類別元件

React 允許使用類別或函式定義元件。早期僅類別元件可擁有狀態，但自 React Hooks API 推出後，函式元件也能管理狀態及其他功能。

[Hooks 於 React Native 0.59 版本引入](/blog/2019/03/12/releasing-react-native-059)。由於 Hooks 代表元件編寫的未來趨勢，本文檔範例主要採用函式元件。必要時會以下拉選單形式補充類別元件說明：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Hello%20World%20Function%20Component
import React from 'react';
import {Text, View} from 'react-native';

const HelloWorldApp = () => {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
      }}>
      <Text>Hello, world!</Text>
    </View>
  );
};

export default HelloWorldApp;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Hello%20World%20Class%20Component
import React, {Component} from 'react';
import {Text, View} from 'react-native';

class HelloWorldApp extends Component {
  render() {
    return (
      <View
        style={{
          flex: 1,
          justifyContent: 'center',
          alignItems: 'center',
        }}>
        <Text>Hello, world!</Text>
      </View>
    );
  }
}

export default HelloWorldApp;
```

</TabItem>
</Tabs>

更多類別元件範例可查閱[歷史版本文件](/versions)。

## 開發者須知

React Native 學習者可能具備網頁、Android、iOS 等不同技術背景。我們會適時提供特定平台的補充說明，例如：

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["android","ios","web"])}>

<TabItem value="android">

> Android developers may be familiar with this concept.

</TabItem>
<TabItem value="ios">

> iOS developers may be familiar with this concept.

</TabItem>
<TabItem value="web">

> Web developers may be familiar with this concept.

</TabItem>
</Tabs>

## 格式規範

Menu paths are written in bold and use carets to navigate submenus. Example: **Android Studio > Preferences**

---

了解本文檔結構後，接下來將深入 React Native 的核心基礎：[原生元件](intro-react-native-components.md)。