---
id: settings
title: Settings
---

`Settings` 作為 [`NSUserDefaults`](https://developer.apple.com/documentation/foundation/nsuserdefaults) 的封裝層，這是一個僅在 iOS 上可用的持久化鍵值儲存系統。

## 範例

```SnackPlayer name=Settings%20Example&supportedPlatforms=ios
import React, { useState } from "react";
import { Button, Settings, StyleSheet, Text, View } from "react-native";

const App = () => {
  const [data, setData] = useState(Settings.get("data"));

  const storeData = data => {
    Settings.set(data);
    setData(Settings.get("data"));
  };

  return (
    <View style={styles.container}>
      <Text>Stored value:</Text>
      <Text style={styles.value}>{data}</Text>
      <Button
        onPress={() => storeData({ data: "React" })}
        title="Store 'React'"
      />
      <Button
        onPress={() => storeData({ data: "Native" })}
        title="Store 'Native'"
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center"
  },
  value: {
    fontSize: 24,
    marginVertical: 12
  }
});

export default App;
```

---

# 參考文檔

## 方法

### `clearWatch()`

```jsx
static clearWatch(watchId: number)
```

`watchId` 是當初訂閱時由 `watchKeys()` 返回的數字識別碼。

---

### `get()`

```jsx
static get(key: string): mixed
```

Get the current value for a given `key` in `NSUserDefaults`.

---

### `set()`

```jsx
static set(settings: object)
```

在 `NSUserDefaults` 中設定一個或多個值。

---

### `watchKeys()`

```jsx
static watchKeys(keys: string | array<string>, callback: function): number
```

Subscribe to be notified when the value for any of the keys specified by the `keys` parameter has been changed in `NSUserDefaults`. Returns a `watchId` number that may be used with `clearWatch()` to unsubscribe.

> **注意：** `watchKeys()` 的設計會忽略內部 `set()` 呼叫，僅在 React Native 程式碼外部觸發變更時才會執行回調函數。