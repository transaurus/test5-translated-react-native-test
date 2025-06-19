---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

## 術語

- **VirtualizedList:** The component behind `FlatList` (React Native's implementation of the [`Virtual List`](https://bvaughn.github.io/react-virtualized/#/components/List) concept.)

- **Memory consumption:** How much information about your list is being stored in memory, which could lead to an app crash.

- **Responsiveness:** Application ability to respond to interactions. Low responsiveness, for instance, is when you touch on a component and it waits a bit to respond, instead of responding immediately as expected.

- **Blank areas:** When `VirtualizedList` can't render your items fast enough, you may enter a part of your list with non-rendered components that appear as blank space.

- **Viewport:** The visible area of content that is rendered to pixels.

- **Window:** The area in which items should be mounted, which is generally much larger than the viewport.

## 屬性

以下是一些可以幫助提升 `FlatList` 性能的屬性：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

如果設為 `true`，視口之外的視圖將從原生視圖層級中分離。

**優點:** 這減少了主執行緒上的時間消耗，從而降低了掉幀的風險，因為視口之外的視圖不會參與原生的渲染和繪製遍歷。

**缺點:** 請注意，此實現可能存在錯誤，例如內容缺失（主要在 iOS 上觀察到），尤其是當你進行複雜的變換和/或絕對定位時。另外，這並不會節省大量記憶體，因為視圖並未被釋放，只是被分離。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設置較大的數字意味著滾動時出現的視覺空白區域更少（提高填充率）。

**缺點:** 每批次更多的項目意味著 JavaScript 執行時間更長，可能阻塞其他事件處理，如點擊，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 將此屬性與 `maxToRenderPerBatch` 結合使用，可以讓你實現例如在較不頻繁的批次中渲染更多項目，或在更頻繁的批次中渲染較少項目。

**缺點:** 較不頻繁的批次可能導致空白區域，而更頻繁的批次可能導致響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 為每個設備定義精確的項目數量以覆蓋屏幕。這可以大幅提升初始渲染的性能。

**缺點:** 設置較低的 `initialNumToRender` 可能導致空白區域，尤其是如果初始渲染時數量太小無法覆蓋視口。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

這裡傳遞的數字是一個測量單位，其中 1 相當於你的視口高度。默認值為 21（上方 10 個視口，下方 10 個視口，中間 1 個視口）。

**優點:** 較大的 `windowSize` 將減少滾動時看到空白空間的機會。另一方面，較小的 `windowSize` 將同時掛載更少的項目，節省記憶體。

**缺點:** 對於較大的 `windowSize`，你將消耗更多記憶體。對於較小的 `windowSize`，你將有更大的機會看到空白區域。

## 列表項目

以下是一些關於列表項元件的技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項中使用大量邏輯和嵌套結構。如果這個列表項元件在應用中被頻繁重用，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套層次。

### 使用輕量級元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版或縮略圖，盡可能小）。與設計團隊溝通，在列表中盡量減少效果、互動和資訊的展示，可以將這些內容放在項目的詳細頁面中。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化的元件，只有在傳遞給元件的 props 發生變化時才會重新渲染。我們可以使用這個函數來優化 FlatList 中的元件。

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

在這個例子中，我們確定 MyListItem 只有在標題變化時才需要重新渲染。我們將比較函數作為第二個參數傳遞給 React.memo()，這樣元件就只會在指定的 prop 變化時重新渲染。如果比較函數返回 true，元件將不會重新渲染。

### 使用緩存的優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

如果所有列表項元件的高度相同（對於水平列表則是寬度相同），提供 [getItemLayout](flatlist#getitemlayout) prop 可以讓 `FlatList` 不需要管理異步佈局計算。這是一個非常理想的優化技巧。

如果你的元件有動態大小且確實需要性能優化，可以考慮與設計團隊溝通，看看是否可以重新設計以提高性能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

你也可以在項目元件中使用 `key` prop。

### 避免在 renderItem 中使用匿名函數

對於函數式元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被包裹在 `useCallback` 鉤子中，以防止每次渲染時重新創建。

對於類元件，將 `renderItem` 函數移到 render 函數之外，這樣它就不會在每次調用 render 函數時重新創建。

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