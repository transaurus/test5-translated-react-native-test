---
id: style
title: Style
---

在 React Native 中，您使用 JavaScript 來為應用程式設定樣式。所有核心元件都接受名為 `style` 的屬性。樣式名稱和[值](colors.md)通常與網頁上的 CSS 工作方式相符，只是名稱使用駝峰式命名法，例如 `backgroundColor` 而不是 `background-color`。

The `style` prop can be a plain old JavaScript object. That's what we usually use for example code. You can also pass an array of styles - the last style in the array has precedence, so you can use this to inherit styles.

隨著元件複雜度的增加，通常更清晰的做法是使用 `StyleSheet.create` 在一個地方定義多個樣式。以下是一個範例：

```SnackPlayer name=Style
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const LotsOfStyles = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.red}>just red</Text>
      <Text style={styles.bigBlue}>just bigBlue</Text>
      <Text style={[styles.bigBlue, styles.red]}>bigBlue, then red</Text>
      <Text style={[styles.red, styles.bigBlue]}>red, then bigBlue</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginTop: 50,
  },
  bigBlue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

export default LotsOfStyles;
```

一個常見的模式是讓您的元件接受一個 `style` 屬性，該屬性又用於為子元件設定樣式。您可以使用這個來實現樣式的「層疊」效果，就像在 CSS 中一樣。

有更多方法可以自訂文字樣式。請查看[文字元件參考](text.md)以獲取完整列表。

現在您可以讓您的文字變得美觀。成為樣式專家的下一步是[學習如何控制元件大小](height-and-width.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：在某些情況下，React Native 與網頁上的 CSS 工作方式不符，例如觸控區域永遠不會超出父視圖的邊界，並且在 Android 上不支援負邊距。