---
id: tutorial
title: Learn the Basics
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 類似 React，但它使用原生元件而非網頁元件作為建構基礎。因此要理解 React Native 應用的基本結構，您需要掌握一些 React 的核心概念，如 JSX、元件、`state` 和 `props`。即使您已熟悉 React，仍需學習 React Native 特有的知識，例如原生元件。本教程適合所有讀者，無論您是否有 React 經驗。

讓我們開始吧。

## Hello World

遵循悠久的傳統，我們首先構建一個只顯示「Hello, world!」的應用程式。如下所示：

```SnackPlayer name=Hello%20World
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

若您感到好奇，可以直接在網頁模擬器中試玩這段範例代碼。也可以將其貼到本機的 `App.js` 檔案中來創建實際應用。

## 這段代碼做了什麼？

1. First of all, we need to import `React` to be able to use `JSX`, which will then be transformed to the native components of each platform.
2. On line 2, we import the `Text` and `View` components from `react-native`

接著定義 `HelloWorldApp` 函數，這是一個[函數式元件](https://reactjs.org/docs/components-and-props.html#function-and-class-components)，其行為與網頁版 React 完全相同。該函數返回帶有樣式的 `View` 元件及其子元件 `Text`。

The `Text` component allows us to render a text, while the `View` component renders a container. This container has several styles applied, let's analyze what each one is doing.

第一個樣式是 `flex: 1`，[`flex`](layout-props#flex) 屬性決定子元件如何沿主軸「填滿」可用空間。由於我們只有一個容器，它會佔滿父元件的所有空間。本例中它是唯一元件，因此會佔據整個螢幕空間。

下一個樣式是 [`justifyContent`](layout-props#justifycontent): "center"，這會將容器子元件沿主軸居中對齊。最後是 [`alignItems`](layout-props#alignitems): "center"，使子元件沿交叉軸居中對齊。

這裡有些語法可能看起來不像 JavaScript。別擔心，_這是未來趨勢_。

首先，ES2015（又稱 ES6）是 JavaScript 的改進集，現已成為官方標準，但尚未被所有瀏覽器支持，因此在網頁開發中還不普及。React Native 內建支援 ES2015，您可以放心使用這些特性。上例中的 `import`、`export`、`const` 和 `from` 都是 ES2015 特性。如果不熟悉 ES2015，可以透過閱讀本教程的範例代碼來學習。[這個頁面](https://babeljs.io/learn-es2015/)詳細介紹了 ES2015 特性。

這段代碼另一個特殊之處是 `<View><Text>Hello world!</Text></View>`。這是 JSX —— 一種在 JavaScript 中嵌入 XML 的語法。許多框架使用專用模板語言來在標記語言中嵌入代碼，而 React 反其道而行。JSX 允許您在代碼中編寫標記語言，看起來像網頁的 HTML，但您使用的是 React 元件而非 `<div>` 或 `<span>` 等網頁元素。本例中，`<Text>` 是[核心元件](intro-react-native-components)用於顯示文字，`View` 則類似 `<div>` 或 `<span>`。

## 元件

So this code is defining `HelloWorldApp`, a new `Component`. When you're building a React Native app, you'll be making new components a lot. Anything you see on the screen is some sort of component.

## Props 屬性

大多數元件在創建時可透過不同參數進行客製化，這些創建參數稱為「props」（屬性）。

自定義元件也能使用 `props`。這讓你能創建單一元件並在應用程式的多個地方使用，每個地方只需微調屬性。在函式元件中透過 `props.YOUR_PROP_NAME` 存取，類別元件則用 `this.props.YOUR_PROP_NAME`。範例如下：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Hello%20Props&ext=js
import React from 'react';
import {Text, View, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center',
  },
});

const Greeting = props => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Hello%20Props&ext=tsx
import React from 'react';
import {Text, View, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center',
  },
});

type GreetingProps = {
  name: string;
};

const Greeting = (props: GreetingProps) => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
</Tabs>

透過 `name` 屬性可客製化 `Greeting` 元件，讓我們能重複使用同個元件來顯示不同問候語。此範例也示範了在 JSX 中使用 `Greeting` 元件，這種靈活性正是 React 的強大之處。

這裡還出現另一個新元件 [`View`](view.md)。[`View`](view.md) 作為其他元件的容器，能協助控制樣式與佈局。

結合 `props` 與基礎的 [`Text`](text.md)、[`Image`](image.md) 和 [`View`](view.md) 元件，你已能建構各種靜態畫面。若要讓應用程式能隨時間變化，請繼續學習[狀態管理](#state)。

## State 狀態

與[唯讀的 props](https://reactjs.org/docs/components-and-props.html#props-are-read-only) 不同，`state` 允許 React 元件根據用戶操作、網路回應等情況動態改變輸出內容。

#### React 中的 state 和 props 有何區別？

在 React 元件中，props 是從父元件傳遞給子元件的變數；而 state 同樣是變數，關鍵差異在於 state 由元件自身初始化並管理，而非透過參數傳遞。

#### React 與 React Native 的狀態處理有差異嗎？

<div className="two-columns">

```tsx
// ReactJS Counter Example using Hooks!

import React, {useState} from 'react';



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

```tsx
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

以下範例將用類別形式實現相同的計數器功能。

```SnackPlayer name=Hello%20Classes
import React, {Component} from 'react';
import {StyleSheet, TouchableOpacity, Text, View} from 'react-native';

class App extends Component {
  state = {
    count: 0,
  };

  onPress = () => {
    this.setState({
      count: this.state.count + 1,
    });
  };

  render() {
    return (
      <View style={styles.container}>
        <TouchableOpacity style={styles.button} onPress={this.onPress}>
          <Text>Click me</Text>
        </TouchableOpacity>
        <View>
          <Text>You clicked {this.state.count} times</Text>
        </View>
      </View>
    );
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
    marginBottom: 10,
  },
});

export default App;
```