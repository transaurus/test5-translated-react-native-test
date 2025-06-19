---
id: view-style-props
title: View Style Props
---

### 範例

```SnackPlayer name=ViewStyleProps
import React from 'react';
import {View, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
      <View style={styles.top} />
      <View style={styles.middle} />
      <View style={styles.bottom} />
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-between',
    padding: 20,
    margin: 10,
  },
  top: {
    flex: 0.3,
    backgroundColor: 'grey',
    borderWidth: 5,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
  },
  middle: {
    flex: 0.3,
    backgroundColor: 'beige',
    borderWidth: 5,
  },
  bottom: {
    flex: 0.3,
    backgroundColor: 'pink',
    borderWidth: 5,
    borderBottomLeftRadius: 20,
    borderBottomRightRadius: 20,
  },
});

export default App;
```

# 參考文獻

## 屬性

### `backfaceVisibility`

| Type                          |
| ----------------------------- |
| enum(`'visible'`, `'hidden'`) |

---

### `backgroundColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomEndRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomLeftRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomRightRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomStartRadius`

| Type   |
| ------ |
| number |

---

### `borderStartEndRadius`

| Type   |
| ------ |
| number |

---

### `borderStartStartRadius`

| Type   |
| ------ |
| number |

---

### `borderEndEndRadius`

| Type   |
| ------ |
| number |

---

### `borderEndStartRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomWidth`

| Type   |
| ------ |
| number |

---

### `borderColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderCurve` <div class="label ios">iOS</div>

在 iOS 13+ 上，可以變更邊框的角落曲線。

| Type                               |
| ---------------------------------- |
| enum(`'circular'`, `'continuous'`) |

---

### `borderEndColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftWidth`

| Type   |
| ------ |
| number |

---

### `borderRadius`

如果圓角邊框不可見，請嘗試同時套用 `overflow: 'hidden'`。

| Type   |
| ------ |
| number |

---

### `borderRightColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderRightWidth`

| Type   |
| ------ |
| number |

---

### `borderStartColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderStyle`

| Type                                    |
| --------------------------------------- |
| enum(`'solid'`, `'dotted'`, `'dashed'`) |

---

### `borderTopColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderTopEndRadius`

| Type   |
| ------ |
| number |

---

### `borderTopLeftRadius`

| Type   |
| ------ |
| number |

---

### `borderTopRightRadius`

| Type   |
| ------ |
| number |

---

### `borderTopStartRadius`

| Type   |
| ------ |
| number |

---

### `borderTopWidth`

| Type   |
| ------ |
| number |

---

### `borderWidth`

| Type   |
| ------ |
| number |

### `boxShadow`

:::note
`boxShadow` 僅在[新架構](/architecture/landing-page)中可用。外陰影僅支援 **Android 9+**。內陰影僅支援 **Android 10+**。
:::

為元素添加陰影效果，可控制陰影的位置、顏色、大小和模糊程度。此陰影可根據是否設置為 _inset_ 出現在元素邊框框的外部或內部。這是符合規範的[同名網頁樣式屬性](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow)實現。詳見[BoxShadowValue](./boxshadowvalue)文檔中所有可用參數的說明。

這些陰影可以組合使用，因此單個 `boxShadow` 可由多個不同的陰影組成。

`boxShadow` 接受模仿[網頁語法](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow#syntax)的字串，或[BoxShadowValue](./boxshadowvalue)物件的陣列。

| Type |
| --------------------------- |
| array of BoxShadowValue ojects \| string |

### `cursor` <div class="label ios">iOS</div>

在 iOS 17+ 中，設置為 `pointer` 可在指針（如 iOS 上的觸控板或觸控筆，或 visionOS 上的用戶視線）懸停於視圖上時啟用懸停效果。

| Type                        |
| --------------------------- |
| enum(`'auto'`, `'pointer'`) |

---

### `elevation` <div class="label android">Android</div>

使用 Android 底層的[高程 API](https://developer.android.com/training/material/shadows-clipping.html#Elevation)設置視圖的高程。這會為項目添加投射陰影，並影響重疊視圖的 z 軸順序。僅支援 Android 5.0+，對更早版本無效。

| Type   |
| ------ |
| number |

---

### `filter`

:::note
`filter` is only available on the [New Architecture](/architecture/landing-page)
:::

Adds a graphical filter to the `View`. This filter is comprised of any number of _filter functions_, which each represent some atomic change to the graphical composition of the `View`. The complete list of valid filter functions is defined below. `filter` will apply to descendants of the `View` as well as the `View` itself. `filter` implies `overflow: hidden`, so descendants will be clipped to fit the bounds of the `View`.

以下濾鏡函數在所有平台均可使用：

- `brightness`: 改變 `View` 的亮度。接受非負數或百分比。
- `opacity`: 改變 `View` 的不透明度或 alpha 值。接受非負數或百分比。

:::note
由於性能和規範合規性問題，iOS 上僅提供這兩種濾鏡函數。計劃探索使用 SwiftUI 替代 UIKit 實現的潛在解決方案。
:::

<div class="label basic android">Android</div>

以下濾鏡函數僅在 Android 上可用：

- `blur`: Blurs the `View` with a [Guassian blur](https://en.wikipedia.org/wiki/Gaussian_blur), where the specified length represents the radius used in the blurring algorithm. Any non-negative DIP value is valid (no percents). The larger the value, the blurrier the result.
- `contrast`: Changes the contrast of the `View`. Takes a non-negative number or percentage.
- `dropShadow`: Adds a shadow around the alpha mask of the `View` (only non-zero alpha pixels in the `View` will cast a shadow). Takes an optional color representing the shadow color, and 2 or 3 lengths. If 2 lengths are specified they are interperted as `offsetX` and `offsetY` which will translate the shadow in the X and Y dimensions respectfully. If a 3rd length is given it is interpreted as the standard deviation of the Guassian blur used on the shadow - so a larger value will blur the shadow more. Read more about the arguments in [DropShadowValue](./dropshadowvalue.md).
- `grayscale`: Converts the `View` to [grayscale](https://en.wikipedia.org/wiki/Grayscale) by the specified amount. Takes a non-negative number or percentage, where `1` or `100%` represents complete grayscale.
- `hueRotate`: Changes the [hue](https://en.wikipedia.org/wiki/Hue) of the View. The argument of this function defines the angle of a color wheel around which the hue will be rotated, so e.g., `360deg` would have no effect. This angle can have either `deg` or `rad` units.
- `invert`: Inverts the colors in the `View`. Takes a non-negative number or percentage, where `1` or `100%` represents complete inversion.
- `sepia`: Converts the `View` to [sepia](<https://en.wikipedia.org/wiki/Sepia_(color)>). Takes a non-negative number or percentage, where `1` or `100%` represents complete sepia.
- `saturate`: Changes the [saturation](https://en.wikipedia.org/wiki/Colorfulness) of the `View`. Takes a non-negative number or percentage.

:::note
`blur`和`dropShadow`僅支援 **Android 12+**
:::

`filter`可接受由上述濾鏡函數組成的物件陣列，或模仿[網頁語法](https://developer.mozilla.org/en-US/docs/Web/CSS/filter#syntax)的字串。

| Type |
| ------ |
| array of objects: `{brightness: number\|string}`, `{opacity: number\|string}`, `{blur: number\|string}`, `{contrast: number\|string}`, `{dropShadow: DropShadowValue\|string}`, `{grayscale: number\|string}`, `{hueRotate: number\|string}`, `{invert: number\|string}`, `{sepia: number\|string}`, `{saturate: number\|string}` or string|

---

### `opacity`

| Type   |
| ------ |
| number |

---

### `outlineColor`

:::note
`outlineColor` is only available on the [New Architecture](/architecture/landing-page)
:::

設定元素輪廓線的顏色。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-color)。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `outlineOffset`

:::note
`outlineOffset` is only available on the [New Architecture](/architecture/landing-page)
:::

設定輪廓線與元素邊界之間的間距。不影響佈局。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-offset)。

| Type   |
| ------ |
| number |

---

### `outlineStyle`

:::note
`outlineStyle` is only available on the [New Architecture](/architecture/landing-page)
:::

設定元素輪廓線的樣式。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-style)。

| Type                                    |
| --------------------------------------- |
| enum(`'solid'`, `'dotted'`, `'dashed'`) |

---

### `outlineWidth`

:::note
`outlineWidth` is only available on the [New Architecture](/architecture/landing-page)
:::

設定元素外圍繪製的輪廓線寬度（位於邊框之外）。不影響佈局。詳見 [網頁文件說明](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-width)。

| Type   |
| ------ |
| number |

---

### `pointerEvents`

控制 `View` 是否能成為觸控事件的目標。

- `'auto'`: View 可成為觸控事件的目標。
- `'none'`: View 永遠不會成為觸控事件的目標。
- `'box-none'`: View 本身不會成為觸控事件目標，但其子視圖可以。
- `'box-only'`: View 可成為觸控事件目標，但其子視圖不可。

| Type                                                  |
| ----------------------------------------------------- |
| enum(`'auto'`, `'box-none'`, `'box-only'`, `'none'` ) |