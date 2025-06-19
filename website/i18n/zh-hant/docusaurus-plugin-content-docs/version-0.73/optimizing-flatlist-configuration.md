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

設為 `true` 時，位於視口外的視圖會從原生視圖層級中分離。

**優點:** 通過排除視口外視圖的原生渲染與繪製流程，可減少主執行緒負擔，降低掉幀風險。

**缺點:** 需注意此實現可能存在錯誤（尤其在iOS平台），特別是當使用複雜變換或絕對定位時，可能導致內容缺失。另請注意此設定不會顯著節省記憶體，因視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 增大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目會導致JavaScript執行時間延長，可能阻塞點擊等事件處理，影響響應速度。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 結合此屬性與 `maxToRenderPerBatch` 可實現靈活配置，例如低頻次大批量渲染或高頻次小批量渲染。

**缺點:** 低頻批次可能導致空白區域，高頻批次可能引發響應延遲。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義覆蓋各裝置螢幕的初始項目數，可大幅提升初始渲染效能。

**缺點:** 過低的 `initialNumToRender` 可能導致空白區域，特別是當初始渲染無法完整覆蓋視口時。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視口高度為單位（1代表一個視口高度），預設值為21（上方10視口+下方10視口+中間1視口）。

**Pros:** Bigger `windowSize` will result in less chance of seeing blank space while scrolling. On the other hand, smaller `windowSize` will result in fewer items mounted simultaneously, saving memory.

**Cons:** For a bigger `windowSize`, you will have more memory consumption. For a lower `windowSize`, you will have a bigger chance of seeing blank areas.

## 列表項目

以下是一些關於列表項目元件的技巧。它們是列表的核心，因此需要保持高效運作。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項目中加入過多邏輯和嵌套結構。若該列表項目元件在應用程式中被頻繁使用，建議為大型列表專門創建一個元件，並盡可能減少邏輯和嵌套層級。

### 使用輕量級元件

元件越龐大，渲染速度越慢。避免使用高解析度圖片（改用裁剪版或縮略圖，尺寸越小越好）。與設計團隊溝通，在列表中盡量減少特效、互動和資訊量，可將這些內容放在項目詳情頁展示。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化元件，僅當傳入的 props 發生變化時才會重新渲染。我們可以用此函數來優化 FlatList 中的元件。

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

此範例中，我們設定 MyListItem 僅在 title 變化時重新渲染。通過向 React.memo() 傳入比較函數作為第二參數，可指定僅在特定 prop 變更時重新渲染。若比較函數返回 true，元件將不會重新渲染。

### 使用快取優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

If all your list item components have the same height (or width, for a horizontal list), providing the [getItemLayout](flatlist#getitemlayout) prop removes the need for your `FlatList` to manage async layout calculations. This is a very desirable optimization technique.

若元件尺寸動態變化且極需效能提升，可考慮請設計團隊評估是否調整設計以提升效能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可在項目元件中使用 `key` prop。

### 避免在 renderItem 使用匿名函數

對於函數元件，請將 `renderItem` 函數移至返回的 JSX 外部，並用 `useCallback` 鉤子包裹以避免每次渲染時重建。

對於類別元件，請將 `renderItem` 函數移至 render 函數外部，避免每次調用 render 時重建。

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