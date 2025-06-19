---
id: using-a-scrollview
title: Using a ScrollView
---

The [ScrollView](scrollview.md) is a generic scrolling container that can contain multiple components and views. The scrollable items can be heterogeneous, and you can scroll both vertically and horizontally (by setting the `horizontal` property).

這個範例創建了一個垂直的 `ScrollView`，其中混合了圖片和文字。

```SnackPlayer name=Using%20ScrollView
import React from 'react';
import {Image, ScrollView, Text} from 'react-native';

const logo = {
  uri: 'https://reactnative.dev/img/tiny_logo.png',
  width: 64,
  height: 64,
};

const App = () => (
  <ScrollView>
    <Text style={{fontSize: 96}}>Scroll me plz</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>If you like</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>Scrolling down</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>What's the best</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 96}}>Framework around?</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{fontSize: 80}}>React Native</Text>
  </ScrollView>
);

export default App;
```

可以通過使用 `pagingEnabled` 屬性來配置 ScrollView，允許用戶通過滑動手勢在視圖之間進行分頁切換。在 Android 上，也可以使用 [ViewPager](https://github.com/react-native-community/react-native-viewpager) 組件來實現水平滑動切換視圖。

在 iOS 上，帶有單個項目的 ScrollView 可以用來允許用戶縮放內容。設置 `maximumZoomScale` 和 `minimumZoomScale` 屬性，用戶就可以使用捏合和展開手勢來放大和縮小。

ScrollView 最適合用於展示數量有限且大小有限的內容。`ScrollView` 的所有元素和視圖都會被渲染，即使它們當前不在屏幕上顯示。如果您有一個很長的列表，無法在屏幕上完全顯示，則應該使用 `FlatList`。接下來，讓我們[學習關於列表視圖的內容](using-a-listview.md)。