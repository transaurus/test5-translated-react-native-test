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

## 屬性參數

以下是有助於提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，位於可視區域外的視圖會從原生視圖層級中分離。

**優點:** 透過排除可視區域外的視圖，減少主執行緒處理時間，從而降低掉幀風險。

**缺點:** 需注意此實作可能存在錯誤（尤其在 iOS 上），例如內容缺失（常見於使用複雜變形或絕對定位時）。另請注意此設定不會顯著節省記憶體，因視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設定較大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目可能導致 JavaScript 執行時間過長，阻塞點擊等事件處理，影響響應速度。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 結合此屬性與 `maxToRenderPerBatch` 可靈活調配，例如實現「少量高頻」或「多量低頻」的渲染策略。

**缺點:** 低頻批次可能導致空白區域，高頻批次可能引發響應遲緩。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 過低的 `initialNumToRender` 可能導致空白區域，尤其在初始渲染時無法覆蓋可視區域的情況下。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以「1個視窗高度」為單位，預設值為21（即可視區域上方10單位、下方10單位，加上中間1單位）。

**優點:** 較大的 `windowSize` 可降低滾動時出現空白的機率；較小的 `windowSize` 則能減少同時掛載的項目數，節省記憶體。

**缺點:** 較大的 `windowSize` 會增加記憶體消耗，較小的 `windowSize` 則可能提高出現空白區域的風險。

## 列表項目元件

以下是一些關於列表項元件的技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項中使用過多邏輯和嵌套。如果在應用中頻繁重用該列表項元件，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套。

### 使用輕量元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版本或縮略圖，盡可能小）。與設計團隊溝通，在列表中盡量減少效果、互動和資訊的展示，可以在項目的詳細頁面中顯示這些內容。

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

如果元件大小動態變化且確實需要性能優化，可以考慮與設計團隊溝通，看看是否可以重新設計以提高性能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可以在項目元件中使用 `key` 屬性。

### 避免在 renderItem 中使用匿名函數

對於函數式元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被包裹在 `useCallback` 鉤子中，以防止每次渲染時重新創建。

對於類元件，將 `renderItem` 函數移到渲染函數之外，這樣它就不會在每次渲染函數調用時重新創建。

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