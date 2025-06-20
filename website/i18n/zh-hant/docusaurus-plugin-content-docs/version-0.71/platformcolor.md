---
id: platformcolor
title: PlatformColor
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

```js
PlatformColor(color1, [color2, ...colorN]);
```

您可以使用 `PlatformColor` 函數來存取目標平台的原生顏色，只需提供對應的原生顏色字串值。將字串傳遞給 `PlatformColor` 函數後，若該平台存在此顏色，函數將返回對應的原生顏色，您可將其應用於應用程式的任何部分。

若您向 `PlatformColor` 函數傳遞多個字串值，函數會將第一個值視為預設值，其餘作為備用選項。

```js
PlatformColor('bogusName', 'linkColor');
```

由於原生顏色可能對主題或高對比度敏感，此平台特定邏輯也會在您的元件內部轉譯。

### 支援的顏色

完整支援的系統顏色類型清單請參閱：

- Android:
  - [R.attr](https://developer.android.com/reference/android/R.attr) - `?attr` prefix
  - [R.color](https://developer.android.com/reference/android/R.color) - `@android:color` prefix
- iOS (Objective-C and Swift notations):
  - [UIColor Standard Colors](https://developer.apple.com/documentation/uikit/uicolor/standard_colors)
  - [UIColor UI Element Colors](https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors)

#### 開發者注意事項

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["web"])}>

<TabItem value="web">

> If you’re familiar with design systems, another way of thinking about this is that `PlatformColor` lets you tap into the local design system's color tokens so your app can blend right in!

</TabItem>
</Tabs>

## 範例

```SnackPlayer name=PlatformColor%20Example&supportedPlatforms=android,ios
import React from 'react';
import {Platform, PlatformColor, StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.label}>I am a special label color!</Text>
  </View>
);

const styles = StyleSheet.create({
  label: {
    padding: 16,
    ...Platform.select({
      ios: {
        color: PlatformColor('label'),
        backgroundColor: PlatformColor('systemTealColor'),
      },
      android: {
        color: PlatformColor('?android:attr/textColor'),
        backgroundColor: PlatformColor('@android:color/holo_blue_bright'),
      },
      default: {color: 'black'},
    }),
  },
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

提供給 `PlatformColor` 函數的字串值必須與應用程式運行所在原生平台上的字串完全匹配。為避免運行時錯誤，應將該函數包裹在平台檢查中，可透過 `Platform.OS === 'platform'` 或 `Platform.select()` 實現，如上例所示。

> **Note:** You can find a complete example that demonstrates proper, intended use of `PlatformColor` in [PlatformColorExample.js](https://github.com/facebook/react-native/blob/0.71-stable/packages/rn-tester/js/examples/PlatformColor/PlatformColorExample.js).