---
id: imagebackground
title: ImageBackground
---

熟悉網頁開發的開發者常會要求實現`background-image`功能。為處理此類需求，您可以使用`<ImageBackground>`組件，該組件具有與`<Image>`相同的屬性，並可在其上疊加任意子組件。

在某些情況下，您可能不想使用`<ImageBackground>`，因為其實現較為基礎。如需更多細節，請參考`<ImageBackground>`的[源代碼](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Image/ImageBackground.js)，並在需要時創建自定義組件。

請注意，您必須指定寬度和高度的樣式屬性。

## 範例

```SnackPlayer name=ImageBackground
import React from 'react';
import {ImageBackground, StyleSheet, Text, View} from 'react-native';

const image = {uri: 'https://legacy.reactjs.org/logo-og.png'};

const App = () => (
  <View style={styles.container}>
    <ImageBackground source={image} resizeMode="cover" style={styles.image}>
      <Text style={styles.text}>Inside</Text>
    </ImageBackground>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  image: {
    flex: 1,
    justifyContent: 'center',
  },
  text: {
    color: 'white',
    fontSize: 42,
    lineHeight: 84,
    fontWeight: 'bold',
    textAlign: 'center',
    backgroundColor: '#000000c0',
  },
});

export default App;
```

---

# 參考文獻

## 屬性

### [圖片屬性](image.md#props)

繼承[圖片屬性](image.md#props)。

---

### `imageStyle`

| Type                                |
| ----------------------------------- |
| [Image Style](image-style-props.md) |

---

### `imageRef`

允許設置對內部`Image`組件的引用

| Type                                                  |
| ----------------------------------------------------- |
| [Ref](https://reactjs.org/docs/refs-and-the-dom.html) |

---

### `style`

| Type                              |
| --------------------------------- |
| [View Style](view-style-props.md) |