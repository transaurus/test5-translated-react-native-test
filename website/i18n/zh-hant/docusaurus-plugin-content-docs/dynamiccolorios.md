---
id: dynamiccolorios
title: DynamicColorIOS
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The `DynamicColorIOS` function is a platform color type specific to iOS.

```tsx
DynamicColorIOS({
  light: color,
  dark: color,
  highContrastLight: color, // (optional) will fallback to "light" if not provided
  highContrastDark: color, // (optional) will fallback to "dark" if not provided
});
```

`DynamicColorIOS` 接收一個物件作為參數，該物件包含兩個必要鍵值：`dark` 和 `light`，以及兩個可選鍵值 `highContrastLight` 和 `highContrastDark`。這些分別對應 iOS 在「淺色模式」、「深色模式」以及啟用高對比度無障礙模式時所要使用的色彩。

在運行時，系統會根據當前系統外觀和無障礙設定自動選擇要顯示的色彩。動態色彩特別適用於品牌色彩或其他需要自動適應系統設定變更的應用程式特定色彩。

#### 開發者注意事項

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["ios", "web"])}>

<TabItem value="web">

> If you’re familiar with `@media (prefers-color-scheme: dark)` in CSS, this is similar! Only instead of defining all the colors in a media query, you define which color to use under what circumstances right there where you're using it. Neat!

</TabItem>
<TabItem value="ios">

> The `DynamicColorIOS` function is similar to the iOS native methods [`UIColor colorWithDynamicProvider:`](https://developer.apple.com/documentation/uikit/uicolor/3238040-colorwithdynamicprovider)

</TabItem>
</Tabs>

## 範例

```tsx
import {DynamicColorIOS} from 'react-native';

const customDynamicTextColor = DynamicColorIOS({
  dark: 'lightskyblue',
  light: 'midnightblue',
});

const customContrastDynamicTextColor = DynamicColorIOS({
  dark: 'darkgray',
  light: 'lightgray',
  highContrastDark: 'black',
  highContrastLight: 'white',
});
```