---
id: usecolorscheme
title: useColorScheme
---

```tsx
import {useColorScheme} from 'react-native';
```

The `useColorScheme` React hook provides and subscribes to color scheme updates from the [`Appearance`](appearance) module. The return value indicates the current user preferred color scheme. The value may be updated later, either through direct user action (e.g. theme selection in device settings) or on a schedule (e.g. light and dark themes that follow the day/night cycle).

### 支援的色彩方案

- `"light"`：使用者偏好淺色主題。
- `"dark"`：使用者偏好深色主題。
- `null`：使用者未指定偏好的色彩主題。

---

## 範例

```SnackPlayer
import React from 'react';
import {Text, StyleSheet, useColorScheme} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const colorScheme = useColorScheme();
  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text>useColorScheme(): {colorScheme}</Text>
      </SafeAreaView>
    </SafeAreaProvider>
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

你可以在 [`AppearanceExample.js`](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/Appearance/AppearanceExample.js) 找到完整的範例，該範例展示了如何將此鉤子與 React 上下文一起使用，為你的應用程式添加淺色與深色主題支援。