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

以下是一些可以幫助提高 `FlatList` 性能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

如果為 `true`，視口之外的視圖將從原生視圖層次結構中分離。

**優點:** 這減少了主線程上花費的時間，從而降低了掉幀的風險，通過將視口之外的視圖排除在原生渲染和繪製遍歷之外。

**缺點:** 請注意，此實現可能存在錯誤，例如內容缺失（主要在 iOS 上觀察到），尤其是如果你正在進行複雜的變換和/或絕對定位操作。另外請注意，這不會節省大量記憶體，因為視圖並未被釋放，只是被分離。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設置較大的數字意味著滾動時視覺空白區域較少（提高填充率）。

**缺點:** 每批次更多的項目意味著 JavaScript 執行的時間更長，可能阻塞其他事件處理，如點擊，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 將此屬性與 `maxToRenderPerBatch` 結合使用，可以讓你例如在較不頻繁的批次中渲染更多項目，或在更頻繁的批次中渲染較少的項目。

**缺點:** 較不頻繁的批次可能導致空白區域，更頻繁的批次可能導致響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 定義精確的項目數量，這些項目將覆蓋每個設備的屏幕。這可以大大提升初始渲染的性能。

**缺點:** 設置較低的 `initialNumToRender` 可能導致空白區域，特別是如果它在初始渲染時太小而無法覆蓋視口。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

這裡傳遞的數字是一個測量單位，其中 1 相當於你的視口高度。默認值為 21（上方 10 個視口，下方 10 個視口，中間一個視口）。

**優點:** 較大的 `windowSize` 將導致滾動時看到空白空間的機會較少。另一方面，較小的 `windowSize` 將導致同時掛載的項目較少，節省記憶體。

**缺點:** 對於較大的 `windowSize`，你將有更多的記憶體消耗。對於較低的 `windowSize`，你將有更大的機會看到空白區域。

## 列表項目

以下是關於列表項元件的一些技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項中使用過多邏輯和嵌套結構。如果這個列表項元件在應用中被大量重複使用，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套層次。

### 使用輕量級元件

元件越重，渲染速度越慢。避免使用高解析度圖片（在列表項中使用裁剪版或縮略圖，尺寸越小越好）。與設計團隊溝通，盡量減少列表中的特效、互動和資訊量，將這些內容保留在項目的詳細頁面中展示。

### 使用 shouldComponentUpdate

為元件實現更新驗證。React 的 `PureComponent` 通過淺比較實現了 [`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。這在列表項中代價較高，因為需要檢查所有 props。如果需要位元級的性能優化，可以為列表項元件制定最嚴格的更新規則，只檢查可能變化的 props。如果列表結構非常簡單，甚至可以考慮使用

```tsx
shouldComponentUpdate() {
  return false
}
```

### 使用緩存的優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

如果所有列表項元件具有相同高度（或水平列表的寬度），提供 [getItemLayout](flatlist#getitemlayout) prop 可以讓 `FlatList` 無需處理異步佈局計算。這是一個非常理想的優化技術。

如果元件尺寸動態變化且確實需要性能優化，可以考慮與設計團隊討論是否通過重新設計來提升性能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可以在項目元件中使用 `key` prop。

### 避免在 renderItem 中使用匿名函數

對於函數式元件，將 `renderItem` 函數移到返回的 JSX 之外。同時確保使用 `useCallback` 鉤子包裹，防止每次渲染時重新創建。

對於類元件，將 `renderItem` 函數移到 render 函數之外，這樣就不會在每次調用 render 時重新創建。

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