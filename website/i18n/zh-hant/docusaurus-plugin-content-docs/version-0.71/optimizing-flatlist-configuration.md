---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 術語

- **VirtualizedList:** The component behind `FlatList` (React Native's implementation of the [`Virtual List`](https://bvaughn.github.io/react-virtualized/#/components/List) concept.)

- **Memory consumption:** How much information about your list is being stored in memory, which could lead to an app crash.

- **Responsiveness:** Application ability to respond to interactions. Low responsiveness, for instance, is when you touch on a component and it waits a bit to respond, instead of responding immediately as expected.

- **Blank areas:** When `VirtualizedList` can't render your items fast enough, you may enter a part of your list with non-rendered components that appear as blank space.

- **Viewport:** The visible area of content that is rendered to pixels.

- **Window:** The area in which items should be mounted, which is generally much larger than the viewport.

## 屬性

以下是可以幫助提高 `FlatList` 性能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

如果為 `true`，視口之外的視圖將從原生視圖層次結構中分離。

**優點:** 這減少了主線程上花費的時間，從而降低了掉幀的風險，通過將視口之外的視圖排除在原生渲染和繪製遍歷之外。

**缺點:** 請注意，此實現可能存在錯誤，例如缺少內容（主要在 iOS 上觀察到），尤其是如果你正在進行複雜的變換和/或絕對定位操作。另外，這不會顯著節省記憶體，因為視圖並未被釋放，只是被分離。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設置更大的數字意味著滾動時視覺空白區域更少（提高填充率）。

**缺點:** 每批次更多的項目意味著更長的 JavaScript 執行時間，可能阻塞其他事件處理，如點擊，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

雖然 `maxToRenderPerBatch` 告訴每批次渲染的項目數量，但設置 `updateCellsBatchingPeriod` 告訴你的 `VirtualizedList` 批次渲染之間的延遲（以毫秒為單位）（你的組件將多頻繁地渲染窗口中的項目）。

**優點:** 將此屬性與 `maxToRenderPerBatch` 結合使用，可以讓你，例如，在較不頻繁的批次中渲染更多項目，或在更頻繁的批次中渲染更少的項目。

**缺點:** 較不頻繁的批次可能導致空白區域，更頻繁的批次可能導致響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 為每個設備定義精確的項目數量，以覆蓋屏幕。這可以大大提升初始渲染的性能。

**缺點:** 設置較低的 `initialNumToRender` 可能導致空白區域，特別是如果它太小而無法在初始渲染時覆蓋視口。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

這裡傳遞的數字是一個測量單位，其中 1 相當於你的視口高度。默認值為 21（上方 10 個視口，下方 10 個視口，中間一個）。

**優點:** 更大的 `windowSize` 將導致滾動時看到空白空間的機會更少。另一方面，更小的 `windowSize` 將導致同時掛載的項目更少，節省記憶體。

**缺點：** 較大的 `windowSize` 會消耗更多記憶體。較小的 `windowSize` 則可能增加出現空白區域的機率。

## 列表項目

以下是一些關於列表項目元件的建議。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度越慢。盡量避免在列表項目中使用過多邏輯和嵌套結構。如果該列表項目元件在應用中頻繁使用，建議為大型列表專門創建一個元件，並盡可能減少邏輯和嵌套層級。

### 使用輕量元件

元件越「重」，渲染速度越慢。避免使用高解析度圖片（改用裁剪版或縮圖，尺寸越小越好）。與設計團隊溝通，在列表中盡量減少特效、互動和資訊量，可將這些內容保留在項目詳情頁中展示。

### 使用 shouldComponentUpdate

為元件實作更新驗證機制。React 的 `PureComponent` 會透過淺比較實作 [`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)，但這在列表中成本較高，因為需檢查所有 props。若要追求位元級效能，應為列表項目元件制定嚴格的更新規則，僅檢查可能變動的 props。若列表結構非常簡單，甚至可以直接使用

```tsx
shouldComponentUpdate() {
  return false
}
```

### 使用快取優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

若所有列表項目元件高度相同（水平列表則為寬度），提供 [getItemLayout](flatlist#getitemlayout) prop 可讓 `FlatList` 跳過非同步佈局計算，這是非常理想的優化技巧。

若元件尺寸動態變化且亟需效能提升，可考慮請設計團隊評估是否調整設計以優化表現。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可直接在項目元件中使用 `key` prop。

### 避免在 renderItem 使用匿名函式

將 `renderItem` 函式移至 render 函式外部，避免每次呼叫 render 時重新創建函式實例。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="classical">

```tsx
renderItem = ({item}) => (<View key={item.key}><Text>{item.title}</Text></View>);

render(){
  // ...

  <FlatList
    data={items}
    renderItem={this.renderItem}
  />

  // ...
}

```

</TabItem>
<TabItem value="functional">

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

</TabItem>
</Tabs>