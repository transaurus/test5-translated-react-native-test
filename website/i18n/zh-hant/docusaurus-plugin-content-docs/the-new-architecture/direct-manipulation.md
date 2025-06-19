---
id: direct-manipulation-new-architecture
title: Direct Manipulation
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

有時需要直接修改元件而不透過狀態/屬性觸發整個子樹重新渲染。例如在瀏覽器中使用 React 時，有時需要直接修改 DOM 節點，行動應用中的視圖也是如此。`setNativeProps` 是 React Native 中相當於直接設定 DOM 節點屬性的方法。

:::caution
當頻繁重新渲染導致效能瓶頸時才使用 `setNativeProps`！

直接操作不應是經常使用的工具。通常僅在建立連續動畫以避免渲染元件層級和協調多個視圖的開銷時使用。
`setNativeProps` 是命令式的，並將狀態儲存在原生層（DOM、UIView 等）而非 React 元件中，這會使程式碼更難理解。

在使用前，請先嘗試用 `setState` 和 [`shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action) 解決問題。
:::

## 使用 setNativeProps 編輯 TextInput 值

Another very common use case of `setNativeProps` is to edit the value of the TextInput. The `controlled` prop of TextInput can sometimes drop characters when the `bufferDelay` is low and the user types very quickly. Some developers prefer to skip this prop entirely and instead use `setNativeProps` to directly manipulate the TextInput value when necessary.

例如，以下程式碼示範點擊按鈕時編輯輸入框：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=setNativeProps%20on%20TextInput&ext=js
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

你可以使用 [`clear`](../textinput#clear) 方法清除 `TextInput`，該方法採用相同方式清除當前輸入文字。

## 避免與渲染函數衝突

若你更新的屬性同時由渲染函數管理，可能會導致不可預測且混亂的錯誤，因為每當元件重新渲染且該屬性變更時，先前透過 `setNativeProps` 設定的值將被完全忽略並覆寫。