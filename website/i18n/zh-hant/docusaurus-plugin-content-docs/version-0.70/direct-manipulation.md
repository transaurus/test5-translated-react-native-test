---
id: direct-manipulation
title: Direct Manipulation
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

有時需要直接修改元件而不透過 state/props 觸發整個子樹重新渲染。例如在瀏覽器中使用 React 時，有時需要直接操作 DOM 節點，在行動應用中操作視圖也是如此。`setNativeProps` 就是 React Native 中直接設定 DOM 節點屬性的等效方法。

:::caution
請在頻繁重新渲染導致效能瓶頸時使用 `setNativeProps`！

直接操作不應是頻繁使用的工具。通常僅在建立連續動畫以避免渲染元件層級和協調多個視圖的開銷時使用。
`setNativeProps` 是命令式的，它將狀態儲存在原生層（DOM、UIView 等）而非 React 元件中，這會使程式碼更難理解。

在使用前，請先嘗試用 `setState` 和 [`shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action) 解決問題。
:::

## 在 TouchableOpacity 中使用 setNativeProps

[TouchableOpacity](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Components/Touchable/TouchableOpacity.js) 內部使用 `setNativeProps` 來更新其子元件的不透明度：

```jsx
const viewRef = useRef();
const setOpacityTo = useCallback(value => {
  // Redacted: animation related code
  viewRef.current.setNativeProps({
    opacity: value,
  });
}, []);
```

這讓我們能編寫以下程式碼，並確知子元件會根據點擊更新不透明度，而無需子元件知曉此邏輯或修改其實作：

```jsx
<TouchableOpacity onPress={handlePress}>
  <View>
    <Text>Press me!</Text>
  </View>
</TouchableOpacity>
```

假設 `setNativeProps` 不可用。我們可能透過將不透明度值儲存在 state 中，並在 `onPress` 觸發時更新該值來實作：

```jsx
const [buttonOpacity, setButtonOpacity] = useState(1);
return (
  <TouchableOpacity
    onPressIn={() => setButtonOpacity(0.5)}
    onPressOut={() => setButtonOpacity(1)}>
    <View style={{opacity: buttonOpacity}}>
      <Text>Press me!</Text>
    </View>
  </TouchableOpacity>
);
```

與原始範例相比，這種做法計算量較大——即使視圖及其子元件的其他屬性未改變，React 仍需在每次不透明度變化時重新渲染元件層級。通常這種開銷無需擔心，但在執行連續動畫和響應手勢時，謹慎優化元件可提升動畫的流暢度。

If you look at the implementation of `setNativeProps` in [NativeMethodsMixin](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Renderer/implementations/ReactNativeRenderer-prod.js) you will notice that it is a wrapper around `RCTUIManager.updateView` - this is the exact same function call that results from re-rendering - see [receiveComponent in ReactNativeBaseComponent](https://github.com/facebook/react-native/blob/fb2ec1ea47c53c2e7b873acb1cb46192ac74274e/Libraries/Renderer/oss/ReactNativeRenderer-prod.js#L5793-L5813).

## 複合元件與 setNativeProps

複合元件沒有對應的原生視圖，因此無法對其呼叫 `setNativeProps`。參考以下範例：

```SnackPlayer name=setNativeProps%20with%20Composite%20Components
import React from "react";
import { Text, TouchableOpacity, View } from "react-native";

const MyButton = (props) => (
  <View style={{ marginTop: 50 }}>
    <Text>{props.label}</Text>
  </View>
);

export default App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);
```

If you run this you will immediately see this error: `Touchable child must either be native or forward setNativeProps to a native component`. This occurs because `MyButton` isn't directly backed by a native view whose opacity should be set. You can think about it like this: if you define a component with `createReactClass` you would not expect to be able to set a style prop on it and have that work - you would need to pass the style prop down to a child, unless you are wrapping a native component. Similarly, we are going to forward `setNativeProps` to a native-backed child component.

#### 將 setNativeProps 轉發給子元件

Since the `setNativeProps` method exists on any ref to a `View` component, it is enough to forward a ref on your custom component to one of the `<View />` components that it renders. This means that a call to `setNativeProps` on the custom component will have the same effect as if you called `setNativeProps` on the wrapped `View` component itself.

```SnackPlayer name=Forwarding%20setNativeProps
import React from "react";
import { Text, TouchableOpacity, View } from "react-native";

const MyButton = React.forwardRef((props, ref) => (
  <View {...props} ref={ref} style={{ marginTop: 50 }}>
    <Text>{props.label}</Text>
  </View>
));

export default App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);
```

You can now use `MyButton` inside of `TouchableOpacity`!

你可能注意到我們使用 `{...props}` 將所有屬性傳遞給子視圖。這是因為 `TouchableOpacity` 實際上是一個複合元件，除了依賴子元件的 `setNativeProps` 外，還需要子元件處理觸控操作。為此，它會傳遞[多種屬性](view.md#onmoveshouldsetresponder)回調到 `TouchableOpacity` 元件。相比之下，`TouchableHighlight` 由原生視圖支持，僅需我們實現 `setNativeProps`。

## 使用 setNativeProps 編輯 TextInput 值

Another very common use case of `setNativeProps` is to edit the value of the TextInput. The `controlled` prop of TextInput can sometimes drop characters when the `bufferDelay` is low and the user types very quickly. Some developers prefer to skip this prop entirely and instead use `setNativeProps` to directly manipulate the TextInput value when necessary. For example, the following code demonstrates editing the input when you tap a button:

```SnackPlayer name=Clear%20text
import React from "react";
import { useCallback, useRef } from "react";
import { StyleSheet, TextInput, Text, TouchableOpacity, View } from "react-native";

const App = () => {
  const inputRef = useRef();
  const editText = useCallback(() => {
    inputRef.current.setNativeProps({ text: "Edited Text" });
  }, []);

  return (
    <View style={styles.container}>
      <TextInput ref={inputRef} style={styles.input} />
      <TouchableOpacity onPress={editText}>
        <Text>Edit text</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  input: {
    height: 50,
    width: 200,
    marginHorizontal: 20,
    borderWidth: 1,
    borderColor: "#ccc",
  },
});

export default App;
```

你可以使用 [`clear`](textinput#clear) 方法清除 `TextInput`，該方法採用相同方式清空當前輸入文字。

## 避免與渲染函數衝突

若你更新的屬性同時由渲染函數管理，可能導致不可預測的錯誤，因為元件重新渲染時若該屬性變動，任何先前透過 `setNativeProps` 設定的值都會被完全忽略並覆寫。

## setNativeProps 與 shouldComponentUpdate

透過[智慧應用 `shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#avoid-reconciliation)，可避免協調未變更元件子樹的不必要開銷，甚至可能使 `setState` 的性能足以取代 `setNativeProps`。

## 其他原生方法

The methods described here are available on most of the default components provided by React Native. Note, however, that they are _not_ available on composite components that aren't directly backed by a native view. This will generally include most components that you define in your own app.

### measure(callback)

測量指定視圖在螢幕上的位置、寬度和高度，並透過非同步回調返回數值。若成功，回調將包含以下參數：

- x
- y
- width
- height
- pageX
- pageY

請注意這些測量值需等待原生渲染完成後才能取得。若需盡快獲取測量值且不需要 `pageX` 和 `pageY`，建議改用 [`onLayout`](view.md#onlayout) 屬性。

Also the width and height returned by `measure()` are the width and height of the component in the viewport. If you need the actual size of the component, consider using the [`onLayout`](view.md#onlayout) property instead.

### measureInWindow(callback)

測量指定視圖在視窗中的位置，並透過非同步回調返回數值。若 React 根視圖嵌入其他原生視圖中，此方法將提供絕對座標。若成功，回調將包含以下參數：

- x
- y
- width
- height

### measureLayout(relativeToNativeComponentRef, onSuccess, onFail)

類似 `measure()`，但測量結果是相對於以 `relativeToNativeComponentRef` 參照的祖先視圖。這意味著返回的座標是相對於祖先視圖原點 `x`, `y` 的相對值。

:::note
此方法也可使用 `relativeToNativeNode` 處理程序（而非參照）呼叫，但此變體已被棄用。
:::

```SnackPlayer name=measureLayout%20example&supportedPlatforms=android,ios
import React, { useEffect, useRef, useState } from "react";
import { Text, View, StyleSheet } from "react-native";

const App = () => {
  const textContainerRef = useRef(null);
  const textRef = useRef(null);
  const [measure, setMeasure] = useState(null);

  useEffect(() => {
    if (textRef.current && textContainerRef.current) {
      textRef.current.measureLayout(
        textContainerRef.current,
        (left, top, width, height) => {
          setMeasure({ left, top, width, height });
        }
      );
    }
  }, [measure]);

  return (
    <View style={styles.container}>
      <View
        ref={textContainerRef}
        style={styles.textContainer}
      >
        <Text ref={textRef}>
          Where am I? (relative to the text container)
        </Text>
      </View>
      <Text style={styles.measure}>
        {JSON.stringify(measure)}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
  },
  textContainer: {
    backgroundColor: "#61dafb",
    justifyContent: "center",
    alignItems: "center",
    padding: 12,
  },
  measure: {
    textAlign: "center",
    padding: 12,
  },
});

export default App;
```

### focus()

請求對指定的輸入框或視圖進行聚焦。觸發的具體行為將取決於平台和視圖類型。

### blur()

移除輸入框或視圖的聚焦狀態。此為 `focus()` 的反向操作。