---
id: keyboard
title: Keyboard
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`Keyboard` module to control keyboard events.

### 使用方法

Keyboard 模組允許您監聽原生事件並作出反應，同時也能對鍵盤進行操作，例如關閉鍵盤。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Keyboard%20Function%20Component%20Example&supportedPlatforms=ios,android
import React, { useState, useEffect } from "react";
import { Keyboard, Text, TextInput, StyleSheet, View } from "react-native";

const Example = () => {
  const [keyboardStatus, setKeyboardStatus] = useState(undefined);

  useEffect(() => {
    const showSubscription = Keyboard.addListener("keyboardDidShow", () => {
      setKeyboardStatus("Keyboard Shown");
    });
    const hideSubscription = Keyboard.addListener("keyboardDidHide", () => {
      setKeyboardStatus("Keyboard Hidden");
    });

    return () => {
      showSubscription.remove();
      hideSubscription.remove();
    };
  }, []);

  return (
    <View style={style.container}>
      <TextInput
        style={style.input}
        placeholder='Click here…'
        onSubmitEditing={Keyboard.dismiss}
      />
      <Text style={style.status}>{keyboardStatus}</Text>
    </View>
  );
}

const style = StyleSheet.create({
  container: {
    flex: 1,
    padding: 36
  },
  input: {
    padding: 10,
    borderWidth: 0.5,
    borderRadius: 4
  },
  status: {
    padding: 10,
    textAlign: "center"
  }
});

export default Example;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Keyboard%20Class%20Component%20Example&supportedPlatforms=ios,android
import React, { Component } from 'react';
import { Keyboard, Text, TextInput, StyleSheet, View } from 'react-native';

class Example extends Component {
  state = {
    keyboardStatus: undefined
  };

  componentDidMount() {
    this.keyboardDidShowSubscription = Keyboard.addListener(
      'keyboardDidShow',
      () => {
        this.setState({ keyboardStatus: 'Keyboard Shown' });
      },
    );
    this.keyboardDidHideSubscription = Keyboard.addListener(
      'keyboardDidHide',
      () => {
        this.setState({ keyboardStatus: 'Keyboard Hidden' });
      },
    );
  }

  componentWillUnmount() {
    this.keyboardDidShowSubscription.remove();
    this.keyboardDidHideSubscription.remove();
  }

  render() {
    return (
      <View style={style.container}>
        <TextInput
          style={style.input}
          placeholder='Click here…'
          onSubmitEditing={Keyboard.dismiss}
        />
        <Text style={style.status}>
          {this.state.keyboardStatus}
        </Text>
      </View>
    )
  }
}

const style = StyleSheet.create({
  container: {
    flex: 1,
    padding: 36
  },
  input: {
    padding: 10,
    borderWidth: 0.5,
    borderRadius: 4
  },
  status: {
    padding: 10,
    textAlign: "center"
  }
});

export default Example;
```

</TabItem>
</Tabs>

---

# 參考文檔

## 方法

### `addListener()`

```jsx
static addListener(eventName, callback)
```

The `addListener` function connects a JavaScript function to an identified native keyboard notification event.

此函數會返回該監聽器的引用。

**參數：**

| Name                                                                     | Type     | Description                                                                    |
| ------------------------------------------------------------------------ | -------- | ------------------------------------------------------------------------------ |
| eventName <div className="label basic two-lines required">Required</div> | string   | The string that identifies the event you're listening for. See the list below. |
| callback <div className="label basic two-lines required">Required</div>  | function | The function to be called when the event fires                                 |

**`eventName`**

可為以下任一事件：

- `keyboardWillShow`
- `keyboardDidShow`
- `keyboardWillHide`
- `keyboardDidHide`
- `keyboardWillChangeFrame`
- `keyboardDidChangeFrame`

> 注意：Android 上僅支援 `keyboardDidShow` 和 `keyboardDidHide` 事件。若您的 activity 將 `android:windowSoftInputMode` 設為 `adjustNothing`，則在 Android 10 及以下版本中這些事件不會觸發。

---

### `removeListener()`

```jsx
static removeListener(eventName, callback)
```

> **Deprecated.** Use the `remove()` method on the event subscription returned by [`addListener()`](#addlistener).

**參數：**

| Name      | Type     | Required | Description                                                                    |
| --------- | -------- | -------- | ------------------------------------------------------------------------------ |
| eventName | string   | Yes      | The `nativeEvent` is the string that identifies the event you're listening for |
| callback  | function | Yes      | The function to be called when the event fires                                 |

---

### `removeAllListeners()`

```jsx
static removeAllListeners(eventName)
```

移除特定事件類型的所有監聽器。

**參數：**

| Name      | Type   | Required | Description                                                          |
| --------- | ------ | -------- | -------------------------------------------------------------------- |
| eventType | string | Yes      | The native event string listeners are watching which will be removed |

---

### `dismiss()`

```jsx
static dismiss()
```

關閉當前鍵盤並移除焦點。

---

### `scheduleLayoutAnimation`

```jsx
static scheduleLayoutAnimation(event)
```

用於同步 TextInput（或其他鍵盤附屬視圖）的尺寸或位置變化與鍵盤移動。