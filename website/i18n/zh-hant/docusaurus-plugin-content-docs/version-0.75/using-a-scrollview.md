---
id: using-a-scrollview
title: Using a ScrollView
---

The [ScrollView](scrollview.md) is a generic scrolling container that can contain multiple components and views. The scrollable items can be heterogeneous, and you can scroll both vertically and horizontally (by setting the `horizontal` property).

此範例建立了一個垂直的 `ScrollView`，其中混合了圖片與文字內容。

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

可透過 `pagingEnabled` 屬性設定 ScrollView 以允許使用者透過滑動手勢進行分頁瀏覽。在 Android 上，也可使用 [ViewPager](https://github.com/react-native-community/react-native-viewpager) 元件實現水平滑動切換視圖的功能。

在 iOS 上，單一項目的 ScrollView 可用來讓使用者縮放內容。設定 `maximumZoomScale` 與 `minimumZoomScale` 屬性後，使用者便能透過捏合與展開手勢進行縮放。

ScrollView 最適合用於呈現數量有限且尺寸不大的項目。所有 `ScrollView` 中的元素與視圖均會被渲染，即使它們目前未顯示在畫面上。若您有大量無法完整顯示於畫面的列表項目，則應改用 `FlatList`。接下來讓我們[學習列表視圖的使用方式](using-a-listview.md)。