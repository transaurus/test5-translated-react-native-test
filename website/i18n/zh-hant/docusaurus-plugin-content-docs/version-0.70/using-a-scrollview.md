---
id: using-a-scrollview
title: Using a ScrollView
---

The [ScrollView](scrollview.md) is a generic scrolling container that can contain multiple components and views. The scrollable items can be heterogeneous, and you can scroll both vertically and horizontally (by setting the `horizontal` property).

此範例創建了一個垂直方向的 `ScrollView`，其中混合了圖片和文字內容。

```SnackPlayer name=Using%20ScrollView
import React from 'react';
import { Image, ScrollView, Text } from 'react-native';

const logo = {
  uri: 'https://reactnative.dev/img/tiny_logo.png',
  width: 64,
  height: 64
};

const App = () => (
  <ScrollView>
    <Text style={{ fontSize: 96 }}>Scroll me plz</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>If you like</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>Scrolling down</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>What's the best</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 96 }}>Framework around?</Text>
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Image source={logo} />
    <Text style={{ fontSize: 80 }}>React Native</Text>
  </ScrollView>
);

export default App;
```

可通過 `pagingEnabled` 屬性將 ScrollView 配置為支持滑動手勢分頁瀏覽視圖。在 Android 上，也可使用 [ViewPager](https://github.com/react-native-community/react-native-viewpager) 組件實現水平滑動切換視圖。

在 iOS 上，單一項目的 ScrollView 可支持內容縮放功能。設置 `maximumZoomScale` 和 `minimumZoomScale` 屬性後，用戶便能使用捏合手勢進行縮放操作。

The ScrollView works best to present a small number of things of a limited size. All the elements and views of a `ScrollView` are rendered, even if they are not currently shown on the screen. If you have a long list of items which cannot fit on the screen, you should use a `FlatList` instead. So let's [learn about list views](using-a-listview.md) next.