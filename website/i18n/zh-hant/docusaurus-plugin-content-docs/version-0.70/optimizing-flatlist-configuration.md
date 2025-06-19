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

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，視窗區域外的子視圖會從原生視圖層級中分離。

**優點:** 減少主執行緒處理時間，透過排除視窗區域外的視圖來降低掉幀風險，避免不必要的原生渲染與繪製遍歷。

**缺點:** 需注意此實作可能存在錯誤（尤其在 iOS 上），例如內容缺失（常見於使用複雜變形或絕對定位時）。另請注意此設定不會顯著節省記憶體，因為視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

It is a `VirtualizedList` prop that can be passed through `FlatList`. This controls the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**優點:** 設定較大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目意味著 JavaScript 執行時間更長，可能阻塞點擊等事件處理，降低響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod` tells your `VirtualizedList` the delay in milliseconds between batch renders (how frequently your component will be rendering the windowed items).

**優點:** 搭配 `maxToRenderPerBatch` 可靈活調配，例如「較少批次處理更多項目」或「較多批次處理較少項目」。

**缺點:** 批次頻率過低可能導致空白區域；過高則可能引發響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義適合所有裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 設定過低的 `initialNumToRender` 可能導致空白區域，尤其在初始渲染時數量不足以覆蓋視窗區域的情況下。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（1 = 1倍視窗高度），預設值為 21（上方 10 倍、下方 10 倍、中間 1 倍區域）。

**優點:** 較大的 `windowSize` 可降低滾動時出現空白區域的機率；較小的 `windowSize` 則能減少同時掛載的項目數，節省記憶體。

**缺點：** 較大的 `windowSize` 會消耗更多記憶體。較小的 `windowSize` 則可能導致空白區域出現的機率增加。

## 列表項目

以下是一些關於列表項目元件的建議。它們是列表的核心，因此需要保持高效能。

### 使用基礎元件

元件越複雜，渲染速度越慢。盡量避免在列表項目中加入過多邏輯或嵌套結構。如果該列表項目元件在應用中重複使用多次，建議為大型列表專門創建一個元件，並盡可能減少邏輯和嵌套層級。

### 使用輕量元件

元件越「重」，渲染速度越慢。避免使用過大的圖片（在列表項目中使用裁剪版本或縮圖，尺寸越小越好）。與設計團隊溝通，盡量減少列表中的特效、互動和資訊量，可將這些內容放在項目詳情頁中展示。

### 使用 shouldComponentUpdate

為元件實作更新檢查。React 的 `PureComponent` 已內建透過淺比較實作的 [`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。但在列表中使用此方法成本較高，因為需檢查所有 props。若追求極致效能，可為列表項目元件制定嚴格的更新規則，僅檢查可能變動的 props。若列表結構非常簡單，甚至可以直接使用

```jsx
shouldComponentUpdate() {
  return false
}
```

### 使用快取優化圖片

You can use the community packages (such as [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann)) for more performant images. Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your JavaScript thread will be free again.

### 使用 getItemLayout

若所有列表項目元件高度相同（水平列表則為寬度），提供 [getItemLayout](flatlist#getitemlayout) prop 可讓 `FlatList` 跳過非同步佈局計算，這是一項極具價值的優化技巧。

若元件尺寸動態變化且需兼顧效能，可考慮與設計團隊討論是否調整設計以提升效能。

### 使用 keyExtractor 或 key

You can set the [`keyExtractor`](flatlist#keyextractor) to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering.

也可直接在項目元件中使用 `key` prop。

### 避免在 renderItem 中使用匿名函式

將 `renderItem` 函式移至 render 函式外部，避免每次呼叫 render 時重新創建函式。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="classical">

```jsx
renderItem = ({ item }) => (<View key={item.key}><Text>{item.title}</Text></View>);

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

```jsx
const renderItem = ({ item }) => (
   <View key={item.key}>
      <Text>{item.title}</Text>
   </View>
 );

return (
  // ...

  <FlatList data={items} renderItem={renderItem} />;
  // ...
);
```

</TabItem>
</Tabs>