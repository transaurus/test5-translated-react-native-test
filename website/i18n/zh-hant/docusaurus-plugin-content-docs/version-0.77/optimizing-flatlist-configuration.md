---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

## 術語

- **VirtualizedList:** `FlatList` 底層的元件（React Native 對 [`Virtual List`](https://bvaughn.github.io/react-virtualized/#/components/List) 概念的實現。）

- **記憶體消耗:** 關於你的列表有多少資訊被儲存在記憶體中，這可能導致應用程式崩潰。

- **響應性:** 應用程式對互動的反應能力。低響應性，例如當你觸摸一個元件時，它會等待一下才回應，而不是如預期般立即回應。

- **空白區域:** 當 `VirtualizedList` 無法足夠快地渲染你的項目時，你可能會進入列表的一部分，其中未渲染的元件會顯示為空白空間。

- **視口:** 渲染為像素的可見內容區域。

- **窗口:** 應該掛載項目的區域，通常比視口大得多。

## 屬性

以下是可以幫助提升 `FlatList` 性能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

如果設為 `true`，視口之外的視圖會從原生視圖層次結構中分離。

**優點:** 這減少了主線程上花費的時間，從而降低了掉幀的風險，因為視口之外的視圖被排除在原生渲染和繪製遍歷之外。

**缺點:** 請注意，這個實現可能存在錯誤，例如內容缺失（主要在 iOS 上觀察到），特別是如果你正在進行複雜的變換和/或絕對定位操作。另外請注意，這不會節省大量記憶體，因為視圖並未被釋放，只是被分離。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設置更大的數字意味著滾動時視覺空白區域更少（提高填充率）。

**缺點:** 每批次更多的項目意味著 JavaScript 執行時間更長，可能阻塞其他事件處理，如點擊，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 將此屬性與 `maxToRenderPerBatch` 結合使用，可以讓你在例如更不頻繁的批次中渲染更多項目，或在更頻繁的批次中渲染更少項目之間進行權衡。

**缺點:** 更不頻繁的批次可能導致空白區域，更頻繁的批次可能導致響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 為每個設備定義精確的項目數量以覆蓋屏幕。這可以大大提升初始渲染的性能。

**缺點:** 設置較低的 `initialNumToRender` 可能導致空白區域，特別是如果初始渲染時它太小而無法覆蓋視口。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

這裡傳遞的數字是一個測量單位，其中 1 相當於你的視口高度。默認值為 21（上方 10 個視口，下方 10 個視口，中間 1 個視口）。

**優點:** 更大的 `windowSize` 將減少滾動時看到空白空間的機會。另一方面，更小的 `windowSize` 將同時掛載更少的項目，節省記憶體。

**缺點:** 對於更大的 `windowSize`，你將有更高的記憶體消耗。對於更低的 `windowSize`，你將有更大的機會看到空白區域。

## 列表項目

以下是一些關於列表項元件的技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度越慢。盡量避免在列表項中加入過多邏輯和嵌套。如果這個列表項元件在應用中被大量重複使用，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套。

### 使用輕量元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版本或縮略圖，盡可能小）。與設計團隊溝通，在列表中盡量減少效果、互動和信息的展示，將這些內容放在項目的詳細頁面中。

### 使用 `memo()`

`React.memo()` creates a memoized component that will be re-rendered only when the props passed to the component change. We can use this function to optimize the components in the FlatList.

```tsx
import React, {memo} from 'react';
import {View, Text} from 'react-native';

const MyListItem = memo(
  ({title}: {title: string}) => (
    <View>
      <Text>{title}</Text>
    </View>
  ),
  (prevProps, nextProps) => {
    return prevProps.title === nextProps.title;
  },
);

export default MyListItem;
```

In this example, we have determined that MyListItem should be re-rendered only when the title changes. We passed the comparison function as the second argument to React.memo() so that the component is re-rendered only when the specified prop is changed. If the comparison function returns true, the component will not be re-rendered.

### 使用緩存的優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

If all your list item components have the same height (or width, for a horizontal list), providing the [getItemLayout](flatlist#getitemlayout) prop removes the need for your `FlatList` to manage async layout calculations. This is a very desirable optimization technique.

如果元件尺寸動態變化且確實需要性能優化，可以考慮請設計團隊評估是否重新設計以提升性能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可以在項目元件中使用 `key` 屬性。

### 避免在 renderItem 中使用匿名函數

對於函數元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被 `useCallback` 鉤子包裹，以防止每次渲染時重新創建。

For class components, move the `renderItem` function outside of the render function, so it won't recreate itself each time the render function is called.

```tsx
const renderItem = useCallback(({item}) => (
   <View key={item.key}>
      <Text>{item.title}</Text>
   </View>
 ), []);

return (
  // ...

  <FlatList data={items} renderItem={renderItem} />;
  // ...
);
```