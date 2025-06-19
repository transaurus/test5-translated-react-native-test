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

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

若設為 `true`，位於視窗可見區域外的視圖會從原生視圖層級中分離。

**優點:** 透過排除視窗外的視圖，減少主執行緒處理時間，降低掉幀風險。

**缺點:** 需注意此實作可能存在錯誤（尤其在iOS平台），例如內容缺失（常見於使用複雜變形或絕對定位時）。另請注意此設定不會顯著節省記憶體，因視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設定較大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目會導致JavaScript執行時間延長，可能阻塞點擊等事件處理，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 結合此屬性與 `maxToRenderPerBatch` 可實現靈活策略，例如「高數量低頻次」或「低數量高頻次」的批次渲染。

**缺點:** 低頻次批次可能導致空白區域，高頻次批次可能引發響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 設定過低的 `initialNumToRender` 可能導致空白區域，特別是初始渲染時無法完整覆蓋視窗可見區域。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（1代表等同視窗高度），預設值為21（上方10單位、下方10單位、中間1單位）。

**優點:** 較大的 `windowSize` 可降低滾動時出現空白的機率；較小的 `windowSize` 則能減少同時掛載的項目數，節省記憶體。

**缺點:** 較大的 `windowSize` 會增加記憶體消耗，較小的 `windowSize` 則提高出現空白區域的可能性。

## 列表項目

以下是一些關於列表項元件的技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度越慢。盡量避免在列表項中加入過多邏輯和嵌套。如果該列表項元件在應用中重複使用多次，請專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套。

### 使用輕量級元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版或縮略圖，盡可能小）。與設計團隊溝通，在列表中盡量減少效果、互動和資訊的展示，將它們放在項目的詳細頁面中。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化的元件，僅當傳遞給元件的 props 發生變化時才會重新渲染。我們可以使用此函數來優化 FlatList 中的元件。

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

在此範例中，我們確定 MyListItem 僅在標題變更時才需要重新渲染。我們將比較函數作為第二個參數傳遞給 React.memo()，這樣元件僅在指定的 prop 變更時才會重新渲染。如果比較函數返回 true，則元件不會重新渲染。

### 使用緩存的優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

如果所有列表項元件的高度相同（或水平列表的寬度相同），提供 [getItemLayout](flatlist#getitemlayout) prop 可以讓 `FlatList` 無需管理異步佈局計算。這是一個非常理想的優化技術。

如果元件具有動態大小且確實需要性能，可以考慮請設計團隊評估是否重新設計以提升性能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

你也可以在項目元件中使用 `key` prop。

### 避免在 renderItem 中使用匿名函數

對於函數元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被包裹在 `useCallback` 鉤子中，以防止每次渲染時重新創建。

對於類別元件，將 `renderItem` 函數移到 render 函數之外，這樣它就不會在每次調用 render 函數時重新創建。

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