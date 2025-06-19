---
id: usecolorscheme
title: useColorScheme
---

```tsx
import {useColorScheme} from 'react-native';
```

The `useColorScheme` React hook provides and subscribes to color scheme updates from the [`Appearance`](appearance) module. The return value indicates the current user preferred color scheme. The value may be updated later, either through direct user action (e.g. theme selection in device settings) or on a schedule (e.g. light and dark themes that follow the day/night cycle).

### 支援的色彩方案

- `"light"`: 使用者偏好淺色主題。
- `"dark"`: 使用者偏好深色主題。
- `null`: 使用者未指定偏好色彩方案。

> **Note:** Currently due to technical constraints, when the Chrome debugger is enabled, this hook will _always_ return `"light"`.

---

## 範例

```SnackPlayer
import React from 'react';
import {Text, StyleSheet, useColorScheme, View} from 'react-native';

const App = () => {
  const colorScheme = useColorScheme();
  return (
    <View style={styles.container}>
      <Text>useColorScheme(): {colorScheme}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

你可以在 [`AppearanceExample.js`](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/Appearance/AppearanceExample.js) 找到完整範例，該範例示範如何搭配 React context 使用此 Hook，為你的應用程式加入淺色與深色主題支援。