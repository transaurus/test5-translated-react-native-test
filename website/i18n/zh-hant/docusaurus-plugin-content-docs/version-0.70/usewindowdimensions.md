---
id: usewindowdimensions
title: useWindowDimensions
---

```jsx
import {useWindowDimensions} from 'react-native';
```

`useWindowDimensions` 會在螢幕尺寸或字體縮放比例變更時自動更新所有數值。您可以透過以下方式取得應用程式視窗的寬度和高度：

```jsx
const {height, width} = useWindowDimensions();
```

## 範例

```SnackPlayer name=useWindowDimensions&supportedPlatforms=ios,android
import React from "react";
import { View, StyleSheet, Text, useWindowDimensions } from "react-native";

const App = () => {
  const { height, width, scale, fontScale } = useWindowDimensions();
  return (
    <View style={styles.container}>
      <Text style={styles.header}>
        Window Dimension Data
      </Text>
      <Text>Height: {height}</Text>
      <Text>Width: {width}</Text>
      <Text>Font scale: {fontScale}</Text>
      <Text>Pixel ratio: {scale}</Text>
    </View>
  );
}
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center"
  },
  header: {
    fontSize: 20,
    marginBottom: 12
  }
});

export default App;
```

## 屬性

### `fontScale`

```jsx
useWindowDimensions().fontScale;
```

當前使用的字體縮放比例。部分作業系統允許使用者調整字體大小以提升閱讀舒適度，此屬性可讓您獲知當前生效的縮放值。

---

### `height`

```jsx
useWindowDimensions().height;
```

應用程式所佔用的視窗或螢幕高度（像素單位）。

---

### `scale`

```jsx
useWindowDimensions().scale;
```

應用程式運行裝置的像素密度比率，可能值包括：

- `1`：表示一個點等於一個像素（通常為 96 PPI/DPI，部分平台為 76）。
- `2` 或 `3`：表示 Retina 或高 DPI 顯示器。

---

### `width`

```jsx
useWindowDimensions().width;
```

應用程式所佔用的視窗或螢幕寬度（像素單位）。