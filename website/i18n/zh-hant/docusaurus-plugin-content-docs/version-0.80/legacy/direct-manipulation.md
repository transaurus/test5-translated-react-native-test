---
id: direct-manipulation
title: Direct Manipulation
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

有時需要直接修改元件而不透過 state/props 觸發整個子樹重新渲染。例如在瀏覽器中使用 React 時，您可能需要直接操作 DOM 節點，而在行動應用中操作視圖(view)也是同樣道理。`setNativeProps` 就是 React Native 中相當於直接設置 DOM 節點屬性的方法。

:::caution
請在頻繁重新渲染導致效能瓶頸時才使用 `setNativeProps`！

直接操作不應是您經常使用的工具。通常僅在創建連續動畫以避免渲染元件層級和協調多個視圖的開銷時使用。
`setNativeProps` 是命令式的，它將狀態儲存在原生層（DOM、UIView 等）而非 React 元件中，這會使您的程式碼更難理解。

在使用前，請先嘗試用 `setState` 和 [`shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action) 解決問題。
:::

## 在 TouchableOpacity 中使用 setNativeProps

[TouchableOpacity](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Components/Touchable/TouchableOpacity.js) 內部使用 `setNativeProps` 來更新其子元件的不透明度：

```tsx
const viewRef = useRef<View>();
const setOpacityTo = useCallback(value => {
  // Redacted: animation related code
  viewRef.current.setNativeProps({
    opacity: value,
  });
}, []);
```

這讓我們能編寫以下程式碼，並確知子元件會響應點擊更新不透明度，而無需子元件知曉此事或修改其實作方式：

```tsx
<TouchableOpacity onPress={handlePress}>
  <View>
    <Text>Press me!</Text>
  </View>
</TouchableOpacity>
```

假設 `setNativeProps` 不可用。我們可能的一種實作方式是將不透明度值儲存在 state 中，並在 `onPress` 觸發時更新該值：

```tsx
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

與原始範例相比，這種方式計算量更大——即使視圖及其子元件的其他屬性未改變，React 仍需在每次不透明度變化時重新渲染元件層級。通常這種開銷無需擔心，但在執行連續動畫和響應手勢時，謹慎優化元件可提高動畫的流暢度。

If you look at the implementation of `setNativeProps` in [NativeMethodsMixin](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Renderer/implementations/ReactNativeRenderer-prod.js) you will notice that it is a wrapper around `RCTUIManager.updateView` - this is the exact same function call that results from re-rendering - see [receiveComponent in ReactNativeBaseComponent](https://github.com/facebook/react-native/blob/fb2ec1ea47c53c2e7b873acb1cb46192ac74274e/Libraries/Renderer/oss/ReactNativeRenderer-prod.js#L5793-L5813).

## 複合元件與 setNativeProps

複合元件並非由原生視圖支持，因此無法對其調用 `setNativeProps`。請看此範例：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=setNativeProps%20with%20Composite%20Components&ext=js
import React from 'react';
import {Text, TouchableOpacity, View} from 'react-native';

const MyButton = props => (
  <View style={{marginTop: 50}}>
    <Text>{props.label}</Text>
  </View>
);

const App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=setNativeProps%20with%20Composite%20Components&ext=tsx
import React from 'react';
import {Text, TouchableOpacity, View} from 'react-native';

const MyButton = (props: {label: string}) => (
  <View style={{marginTop: 50}}>
    <Text>{props.label}</Text>
  </View>
);

const App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);

export default App;
```

</TabItem>
</Tabs>

If you run this you will immediately see this error: `Touchable child must either be native or forward setNativeProps to a native component`. This occurs because `MyButton` isn't directly backed by a native view whose opacity should be set. You can think about it like this: if you define a component with `createReactClass` you would not expect to be able to set a style prop on it and have that work - you would need to pass the style prop down to a child, unless you are wrapping a native component. Similarly, we are going to forward `setNativeProps` to a native-backed child component.

#### 將 setNativeProps 轉發給子元件

Since the `setNativeProps` method exists on any ref to a `View` component, it is enough to forward a ref on your custom component to one of the `<View />` components that it renders. This means that a call to `setNativeProps` on the custom component will have the same effect as if you called `setNativeProps` on the wrapped `View` component itself.

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Forwarding%20setNativeProps&ext=js
import React from 'react';
import {Text, TouchableOpacity, View} from 'react-native';

const MyButton = React.forwardRef((props, ref) => (
  <View {...props} ref={ref} style={{marginTop: 50}}>
    <Text>{props.label}</Text>
  </View>
));

const App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Forwarding%20setNativeProps&ext=tsx
import React from 'react';
import {Text, TouchableOpacity, View} from 'react-native';

const MyButton = React.forwardRef<View, {label: string}>((props, ref) => (
  <View {...props} ref={ref} style={{marginTop: 50}}>
    <Text>{props.label}</Text>
  </View>
));

const App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);

export default App;
```

</TabItem>
</Tabs>

You can now use `MyButton` inside of `TouchableOpacity`!

您可能已經注意到，我們使用 `{...props}` 將所有屬性傳遞給子視圖。這樣做的原因是 `TouchableOpacity` 實際上是一個複合元件，因此除了依賴子元件的 `setNativeProps` 外，還要求子元件處理觸控操作。為此，它會傳遞[各種屬性](view.md#onmoveshouldsetresponder)來回調 `TouchableOpacity` 元件。相比之下，`TouchableHighlight` 是由原生視圖支持的，只需要我們實現 `setNativeProps`。

## 使用 setNativeProps 編輯 TextInput 值

Another very common use case of `setNativeProps` is to edit the value of the TextInput. The `controlled` prop of TextInput can sometimes drop characters when the `bufferDelay` is low and the user types very quickly. Some developers prefer to skip this prop entirely and instead use `setNativeProps` to directly manipulate the TextInput value when necessary. For example, the following code demonstrates editing the input when you tap a button:

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Clear%20text&ext=js
import React from 'react';
import {useCallback, useRef} from 'react';
import {
  StyleSheet,
  TextInput,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

const App = () => {
  const inputRef = useRef(null);
  const editText = useCallback(() => {
    inputRef.current.setNativeProps({text: 'Edited Text'});
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
    alignItems: 'center',
    justifyContent: 'center',
  },
  input: {
    height: 50,
    width: 200,
    marginHorizontal: 20,
    borderWidth: 1,
    borderColor: '#ccc',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Clear%20text&ext=tsx
import React from 'react';
import {useCallback, useRef} from 'react';
import {
  StyleSheet,
  TextInput,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

const App = () => {
  const inputRef = useRef<TextInput>(null);
  const editText = useCallback(() => {
    inputRef.current?.setNativeProps({text: 'Edited Text'});
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
    alignItems: 'center',
    justifyContent: 'center',
  },
  input: {
    height: 50,
    width: 200,
    marginHorizontal: 20,
    borderWidth: 1,
    borderColor: '#ccc',
  },
});

export default App;
```

</TabItem>
</Tabs>

您可以使用 [`clear`](../textinput#clear) 方法來清除 `TextInput`，該方法使用相同的方式清除當前輸入文本。

## 避免與渲染函數衝突

如果您更新了一個同時由渲染函數管理的屬性，可能會導致一些不可預測且令人困惑的錯誤，因為每當元件重新渲染且該屬性發生變化時，之前通過 `setNativeProps` 設置的任何值都將被完全忽略並覆蓋。

## setNativeProps 與 shouldComponentUpdate

通過[智能應用 `shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#avoid-reconciliation)，您可以避免在協調未變更的元件子樹時產生不必要的開銷，從而達到足以使用 `setState` 而非 `setNativeProps` 的性能水平。

## 其他原生方法

The methods described here are available on most of the default components provided by React Native. Note, however, that they are _not_ available on composite components that aren't directly backed by a native view. This will generally include most components that you define in your own app.

### measure(callback)

確定給定視圖在屏幕上的位置、寬度和高度，並通過異步回調返回這些值。如果成功，回調將被調用並傳入以下參數：

- x
- y
- width
- height
- pageX
- pageY

請注意，這些測量值只有在原生渲染完成後才可用。如果您需要盡快獲取測量值且不需要 `pageX` 和 `pageY`，可以考慮使用 [`onLayout`](view.md#onlayout) 屬性替代。

此外，`measure()` 返回的寬度和高度是元件在視口中的寬度和高度。如果您需要元件的實際大小，可以考慮使用 [`onLayout`](view.md#onlayout) 屬性替代。

### measureInWindow(callback)

確定給定視圖在窗口中的位置，並通過異步回調返回這些值。如果 React 根視圖嵌入到另一個原生視圖中，這將為您提供絕對坐標。如果成功，回調將被調用並傳入以下參數：

- x
- y
- width
- height

### measureLayout(relativeToNativeComponentRef, onSuccess, onFail)

與 `measure()` 類似，但會測量視圖相對於祖先的位置，該祖先由 `relativeToNativeComponentRef` 引用指定。這意味著返回的坐標是相對於祖先視圖原點 `x`、`y` 的。

:::note
此方法也可以使用 `relativeToNativeNode` 處理程序（而非引用）調用，但這種變體在新架構中已過時。
:::

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=measureLayout%20example&supportedPlatforms=android,ios&ext=js
import React, {useEffect, useRef, useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';

const App = () => {
  const textContainerRef = useRef(null);
  const textRef = useRef(null);
  const [measure, setMeasure] = useState(null);

  useEffect(() => {
    if (textRef.current && textContainerRef.current) {
      textRef.current.measureLayout(
        textContainerRef.current,
        (left, top, width, height) => {
          setMeasure({left, top, width, height});
        },
      );
    }
  }, [measure]);

  return (
    <View style={styles.container}>
      <View ref={textContainerRef} style={styles.textContainer}>
        <Text ref={textRef}>Where am I? (relative to the text container)</Text>
      </View>
      <Text style={styles.measure}>{JSON.stringify(measure)}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  textContainer: {
    backgroundColor: '#61dafb',
    justifyContent: 'center',
    alignItems: 'center',
    padding: 12,
  },
  measure: {
    textAlign: 'center',
    padding: 12,
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=measureLayout%20example&ext=tsx
import React, {useEffect, useRef, useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';

type Measurements = {
  left: number;
  top: number;
  width: number;
  height: number;
};

const App = () => {
  const textContainerRef = useRef<View>(null);
  const textRef = useRef<Text>(null);
  const [measure, setMeasure] = useState<Measurements | null>(null);

  useEffect(() => {
    if (textRef.current && textContainerRef.current) {
      textRef.current?.measureLayout(
        textContainerRef.current,
        (left, top, width, height) => {
          setMeasure({left, top, width, height});
        },
        () => {
          console.error('measurement failed');
        },
      );
    }
  }, [measure]);

  return (
    <View style={styles.container}>
      <View ref={textContainerRef} style={styles.textContainer}>
        <Text ref={textRef}>Where am I? (relative to the text container)</Text>
      </View>
      <Text style={styles.measure}>{JSON.stringify(measure)}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  textContainer: {
    backgroundColor: '#61dafb',
    justifyContent: 'center',
    alignItems: 'center',
    padding: 12,
  },
  measure: {
    textAlign: 'center',
    padding: 12,
  },
});

export default App;
```

</TabItem>
</Tabs>

### focus()

請求將焦點設定於指定的輸入框或視圖上。觸發的具體行為會根據平台和視圖類型而有所不同。

### blur()

移除輸入框或視圖的焦點。此為 `focus()` 的反向操作。