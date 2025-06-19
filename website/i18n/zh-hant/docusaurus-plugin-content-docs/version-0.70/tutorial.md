---
id: tutorial
title: Learn the Basics
---

React Native 類似於 React，但它使用原生元件而非網頁元件作為建構基礎。因此，要理解 React Native 應用的基本結構，你需要掌握一些基本的 React 概念，如 JSX、元件、`state` 和 `props`。即使你已經熟悉 React，仍需要學習一些 React Native 特有的內容，例如原生元件。本教程適合所有讀者，無論你是否具有 React 經驗。

讓我們開始吧。

## Hello World

依照我們悠久的傳統，首先我們要建立一個只顯示「Hello, world!」的應用程式。以下是範例：

```SnackPlayer name=Hello%20World
import React from 'react';
import { Text, View } from 'react-native';

const HelloWorldApp = () => {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: "center",
        alignItems: "center"
      }}>
      <Text>Hello, world!</Text>
    </View>
  )
}
export default HelloWorldApp;
```

如果你感到好奇，可以直接在網頁模擬器中試玩這段範例程式碼。你也可以將其貼到你的 `App.js` 檔案中，以在本地機器上建立一個真實的應用程式。

## 這裡發生了什麼？

1. First of all, we need to import `React` to be able to use `JSX`, which will then be transformed to the native components of each platform.
2. On line 2, we import the `Text` and `View` components from `react-native`

接著我們定義了 `HelloWorldApp` 函數，這是一個[函數式元件](https://reactjs.org/docs/components-and-props.html#function-and-class-components)，其行為與網頁版 React 中的相同。此函數返回一個帶有某些樣式的 `View` 元件，並以 `Text` 作為其子元件。

The `Text` component allows us to render a text, while the `View` component renders a container. This container has several styles applied, let's analyze what each one is doing.

第一個樣式是 `flex: 1`，[`flex`](layout-props#flex) 屬性會定義你的項目如何沿主軸「填滿」可用空間。由於我們只有一個容器，它會佔用父元件的所有可用空間。在這個例子中，它是唯一的元件，因此會佔用整個螢幕的可用空間。

接下來的樣式是 [`justifyContent`](layout-props#justifycontent): "center"。這會將容器的子元件沿容器的主軸居中對齊。最後，我們有 [`alignItems`](layout-props#alignitems): "center"，這會將容器的子元件沿容器的交叉軸居中對齊。

這裡的一些內容可能看起來不像 JavaScript。別擔心。_這就是未來_。

首先，ES2015（也稱為 ES6）是對 JavaScript 的一系列改進，現已成為官方標準的一部分，但尚未被所有瀏覽器支援，因此在網頁開發中還不常使用。React Native 自帶 ES2015 支援，因此你可以放心使用這些功能而不必擔心兼容性問題。上面範例中的 `import`、`export`、`const` 和 `from` 都是 ES2015 的特性。如果你不熟悉 ES2015，可以通過閱讀像本教程這樣的範例程式碼來學習。如果需要，[這個頁面](https://babeljs.io/learn-es2015/)提供了 ES2015 特性的詳細概述。

這段程式碼中另一個不尋常的地方是 `<View><Text>Hello world!</Text></View>`。這是 JSX——一種在 JavaScript 中嵌入 XML 的語法。許多框架使用專門的模板語言，讓你能夠在標記語言中嵌入程式碼。在 React 中，這被反轉了。JSX 讓你在程式碼中寫入標記語言。它看起來像網頁上的 HTML，但你不是使用 `<div>` 或 `<span>` 這樣的網頁元素，而是使用 React 元件。在這個例子中，`<Text>` 是一個[核心元件](intro-react-native-components)，用於顯示文字，而 `View` 則類似於 `<div>` 或 `<span>`。

## 元件

So this code is defining `HelloWorldApp`, a new `Component`. When you're building a React Native app, you'll be making new components a lot. Anything you see on the screen is some sort of component.

## Props 屬性

大多數組件在創建時可以通過不同的參數進行定制。這些創建參數稱為 props（屬性）。

你自己的組件也可以使用 `props`。這讓你能夠創建一個單一的組件，並在應用程式的多個地方使用，每個地方稍微調整不同的屬性。在你的函數組件中引用 `props.YOUR_PROP_NAME`，或在類組件中引用 `this.props.YOUR_PROP_NAME`。以下是一個例子：

```SnackPlayer name=Hello%20Props
import React from 'react';
import { Text, View, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center'
  }
})

const Greeting = (props) => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
}

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name='Rexxar' />
      <Greeting name='Jaina' />
      <Greeting name='Valeera' />
    </View>
  );
}

export default LotsOfGreetings;
```

使用 `name` 作為 prop 讓我們可以定制 `Greeting` 組件，這樣我們就能重複使用該組件來顯示不同的問候語。這個例子還展示了如何在 JSX 中使用 `Greeting` 組件。這種能力正是 React 如此強大的原因。

這裡另一個新東西是 [`View`](view.md) 組件。[`View`](view.md) 作為其他組件的容器非常有用，可以幫助控制樣式和佈局。

通過 `props` 和基本的 [`Text`](text.md)、[`Image`](image.md) 和 [`View`](view.md) 組件，你可以構建各種靜態畫面。要學習如何讓你的應用程式隨時間變化，你需要[了解 State（狀態）](#state)。

## State 狀態

與[唯讀的 props](https://reactjs.org/docs/components-and-props.html#props-are-read-only)（不應被修改）不同，`state` 允許 React 組件根據用戶操作、網絡響應等情況改變其輸出。

#### React 中的 state 和 props 有什麼區別？

在 React 組件中，props 是我們從父組件傳遞給子組件的變量。同樣地，state 也是變量，但不同之處在於它們不是作為參數傳遞的，而是由組件內部初始化和管理的。

#### React 和 React Native 在處理 state 上有區別嗎？

<div className="two-columns">

```jsx
// ReactJS Counter Example using Hooks!

import React, { useState } from 'react';



const App = () => {
  const [count, setCount] = useState(0);

  return (
    <div className="container">
      <p>You clicked {count} times</p>
      <button
        onClick={() => setCount(count + 1)}>
        Click me!
      </button>
    </div>
  );
};


// CSS
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

```

```jsx
// React Native Counter Example using Hooks!

import React, {useState} from 'react';
import {View, Text, Button, StyleSheet} from 'react-native';

const App = () => {
  const [count, setCount] = useState(0);

  return (
    <View style={styles.container}>
      <Text>You clicked {count} times</Text>
      <Button
        onPress={() => setCount(count + 1)}
        title="Click me!"
      />
    </View>
  );
};

// React Native Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

</div>

As shown above, there is no difference in handling the `state` between [React](https://reactjs.org/docs/state-and-lifecycle.html) and `React Native`. You can use the state of your components both in classes and in functional components using [hooks](https://reactjs.org/docs/hooks-intro.html)!

在下面的例子中，我們將使用類來展示相同的計數器示例。

```SnackPlayer name=Hello%20Classes
import React, { Component } from 'react'
import {
  StyleSheet,
  TouchableOpacity,
  Text,
  View,
} from 'react-native'

class App extends Component {
  state = {
    count: 0
  }

  onPress = () => {
    this.setState({
      count: this.state.count + 1
    })
  }

 render() {
    return (
      <View style={styles.container}>
        <TouchableOpacity
         style={styles.button}
         onPress={this.onPress}
        >
         <Text>Click me</Text>
        </TouchableOpacity>
        <View>
          <Text>
            You clicked { this.state.count } times
          </Text>
        </View>
      </View>
    )
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
    marginBottom: 10
  }
})

export default App;
```