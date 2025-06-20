---
id: height-and-width
title: Height and Width
---

元件的高度和寬度決定了它在螢幕上的大小。

## 固定尺寸

設定元件尺寸的一般方法是透過在樣式中添加固定的 `width` 和 `height`。React Native 中的所有尺寸都是無單位的，代表與密度無關的像素。

```SnackPlayer name=Height%20and%20Width
import React from 'react';
import { View } from 'react-native';

const FixedDimensionsBasics = () => {
  return (
    <View>
      <View style={{
        width: 50, height: 50, backgroundColor: 'powderblue'
      }} />
      <View style={{
        width: 100, height: 100, backgroundColor: 'skyblue'
      }} />
      <View style={{
        width: 150, height: 150, backgroundColor: 'steelblue'
      }} />
    </View>
  );
};

export default FixedDimensionsBasics;
```

這種設定尺寸的方式適用於那些大小應該始終固定為特定點數，而不是根據螢幕大小計算的元件。

:::caution
點數與實際測量單位之間沒有通用的映射關係。這意味著具有固定尺寸的元件在不同設備和螢幕大小上可能不會具有相同的物理大小。然而，對於大多數使用情況來說，這種差異是微不足道的。
:::

## 彈性尺寸

在元件的樣式中使用 `flex` 可以讓元件根據可用空間動態擴展和收縮。通常你會使用 `flex: 1`，這會告訴元件填充所有可用空間，並與具有相同父元件的其他元件均勻共享。`flex` 值越大，元件相對於其兄弟元件所佔的空間比例就越高。

:::info
元件只有在父元件的尺寸大於 `0` 時才能擴展以填充可用空間。如果父元件沒有固定的 `width` 和 `height` 或 `flex`，父元件的尺寸將為 `0`，而 `flex` 子元件將不可見。
:::

```SnackPlayer name=Flex%20Dimensions
import React from 'react';
import { View } from 'react-native';

const FlexDimensionsBasics = () => {
  return (
    // Try removing the `flex: 1` on the parent View.
    // The parent will not have dimensions, so the children can't expand.
    // What if you add `height: 300` instead of `flex: 1`?
    <View style={{ flex: 1 }}>
      <View style={{ flex: 1, backgroundColor: 'powderblue' }} />
      <View style={{ flex: 2, backgroundColor: 'skyblue' }} />
      <View style={{ flex: 3, backgroundColor: 'steelblue' }} />
    </View>
  );
};

export default FlexDimensionsBasics;
```

在你能控制元件的大小之後，下一步是[學習如何在螢幕上佈局](flexbox.md)。

## 百分比尺寸

If you want to fill a certain portion of the screen, but you _don't_ want to use the `flex` layout, you _can_ use **percentage values** in the component's style. Similar to flex dimensions, percentage dimensions require parent with a defined size.

```SnackPlayer name=Percentage%20Dimensions
import React from 'react';
import { View } from 'react-native';

const PercentageDimensionsBasics = () => {
  // Try removing the `height: '100%'` on the parent View.
  // The parent will not have dimensions, so the children can't expand.
  return (
    <View style={{ height: '100%' }}>
      <View style={{
        height: '15%', backgroundColor: 'powderblue'
      }} />
      <View style={{
        width: '66%', height: '35%', backgroundColor: 'skyblue'
      }} />
      <View style={{
        width: '33%', height: '50%', backgroundColor: 'steelblue'
      }} />
    </View>
  );
};

export default PercentageDimensionsBasics;
```