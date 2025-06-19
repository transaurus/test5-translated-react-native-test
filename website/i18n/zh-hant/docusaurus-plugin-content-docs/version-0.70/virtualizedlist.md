---
id: virtualizedlist
title: VirtualizedList
---

這是為更方便的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件提供的基礎實現，這些元件也有更完善的文檔。通常只有在需要比 [`FlatList`](flatlist.md) 提供更高靈活性的情況下才應使用此元件，例如需配合不可變數據而非普通陣列時。

虛擬化技術通過維護一個有限的可見區域渲染窗口，並將窗口外的項目替換為適當大小的空白空間，大幅提升了大型列表的記憶體使用效率和性能表現。該窗口會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在運行中的交互之後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白空間的可能性。

## 範例

```SnackPlayer name=VirtualizedListExample
import React from 'react';
import { SafeAreaView, View, VirtualizedList, StyleSheet, Text, StatusBar } from 'react-native';

const DATA = [];

const getItem = (data, index) => ({
  id: Math.random().toString(12).substring(0),
  title: `Item ${index+1}`
});

const getItemCount = (data) => 50;

const Item = ({ title }) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => {
  return (
    <SafeAreaView style={styles.container}>
      <VirtualizedList
        data={DATA}
        initialNumToRender={4}
        renderItem={({ item }) => <Item title={item.title} />}
        keyExtractor={item => item.key}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight,
  },
  item: {
    backgroundColor: '#f9c2ff',
    height: 150,
    justifyContent: 'center',
    marginVertical: 8,
    marginHorizontal: 16,
    padding: 20,
  },
  title: {
    fontSize: 32,
  },
});

export default App;
```

---

需注意事項：

- 當內容滾出渲染窗口時，內部狀態不會保留。請確保所有數據都保存在項目數據或外部存儲（如 Flux、Redux 或 Relay）中。
- 這是個 `PureComponent`，意味著若 `props` 進行淺層比較後相等則不會重新渲染。請確保 `renderItem` 函數依賴的所有內容都通過 prop（例如 `extraData`）傳遞，且這些 prop 在更新後不應保持 `===` 關係，否則變更時 UI 可能不會更新。這包括 `data` prop 和父元件狀態。
- 為限制記憶體使用並實現平滑滾動，內容會異步離屏渲染。這可能導致滾動速度超過填充速率而暫時看到空白內容。此為可根據應用需求調整的權衡方案，我們正在幕後持續改進此機制。
- 預設情況下，列表會尋找每個項目的 `key` prop 並將其作為 React key。您也可以通過 `keyExtractor` prop 提供自定義鍵值。

---

# 參考文獻

## 屬性

### [ScrollView 屬性](scrollview.md#props)

繼承 [ScrollView 屬性](scrollview.md#props)。

---

### <div class="label required basic">Required</div> **`data`**

預設訪問函數假設這是形如 `{key: string}` 的物件陣列，但您可通過覆寫 `getItem`、`getItemCount` 和 `keyExtractor` 來處理任何基於索引的數據類型。

| Type |
| ---- |
| any  |

---

### <div class="label required basic">Required</div> **`getItem`**

```jsx
(data: any, index: number) => object;
```

用於從任意數據塊中提取項目的通用訪問器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`getItemCount`**

```jsx
(data: any) => number;
```

確定數據塊中包含的項目數量。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`renderItem`**

```jsx
(info: any) => ?React.Element<any>
```

從 `data` 中提取項目並將其渲染至列表

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

每個單元格都使用此元件進行渲染。可以是 React 元件類或渲染函數。預設使用 [`View`](view.md)。

| Type                |
| ------------------- |
| component, function |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供 `separators.highlight`/`unhighlight` 來更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自訂屬性。可以是 React 元件（例如 `SomeComponent`），或 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 元件（例如 `SomeComponent`），或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListItemComponent`

每個數據項目都會使用此元素渲染。可以是 React 元件類別，或渲染函數。

| Type                |
| ------------------- |
| component, function |

---

### `ListFooterComponent`

在所有項目的底部渲染。可以是 React 元件（例如 `SomeComponent`），或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponentStyle`

用於 `ListFooterComponent` 內部 View 的樣式設定。

| Type          | Required |
| ------------- | -------- |
| ViewStyleProp | No       |

---

### `ListHeaderComponent`

在所有項目的頂部渲染。可以是 React 元件（例如 `SomeComponent`），或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListHeaderComponentStyle`

用於 `ListHeaderComponent` 內部 View 的樣式設定。

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `debug`

`debug` 會開啟額外的日誌記錄和視覺疊加層，以幫助調試使用和實現問題，但會顯著影響性能。

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的性能和記憶體優化，但會完全卸載渲染窗口之外的 React 實例。您應該僅在調試時需要禁用此功能。

| Type    |
| ------- |
| boolean |

---

### `extraData`

一個標記屬性，用於通知列表重新渲染（因為它實現了 `PureComponent`）。如果您的 `renderItem`、Header、Footer 等函數依賴於 `data` 屬性之外的任何內容，請將其放在這裡並視為不可變。

| Type |
| ---- |
| any  |

---

### `getItemLayout`

```jsx
(
    data: any,
    index: number,
  ) => {length: number, offset: number, index: number}
```

| Type     |
| -------- |
| function |

---

### `horizontal`

如果為 `true`，則將項目水平並排渲染，而不是垂直堆疊。

| Type    |
| ------- |
| boolean |

---

### `initialNumToRender`

初始批次中要渲染的項目數量。這應該足以填滿屏幕，但不要太多。請注意，這些項目永遠不會作為窗口渲染的一部分被卸載，以提高滾動到頂部操作的感知性能。

| Type   | Default |
| ------ | ------- |
| number | `10`    |

---

### `initialScrollIndex`

不從頂部的第一個項目開始，而是從 `initialScrollIndex` 指定的索引處開始。這會停用「滾動至頂部」的優化功能（該功能會始終渲染前 `initialNumToRender` 個項目），並立即從此初始索引處開始渲染項目。需實作 `getItemLayout` 方法。

| Type   |
| ------ |
| number |

---

### `inverted`

反轉滾動方向。使用 `-1` 的縮放變換實現。

| Type    |
| ------- |
| boolean |

---

### `listKey`

此列表的唯一標識符。如果在另一個 VirtualizedList 中有多個同層級嵌套的 VirtualizedLists，此鍵是虛擬化正常運作的必要條件。

| Type   | Required |
| ------ | -------- |
| string | True     |

---

### `keyExtractor`

```jsx
(item: object, index: number) => string;
```

用於為指定索引處的項目提取唯一鍵。該鍵用於緩存及作為 React 的 key 來追蹤項目重新排序。默認提取器會檢查 `item.key`，其次是 `item.id`，最後回退到使用索引（與 React 的行為一致）。

| Type     |
| -------- |
| function |

---

### `maxToRenderPerBatch`

每次增量渲染批次中渲染項目的最大數量。一次渲染越多，填充率越好，但響應性可能下降，因為渲染內容可能會干擾按鈕點擊或其他互動的響應。

| Type   |
| ------ |
| number |

---

### `onEndReached`

```jsx
(info: {distanceFromEnd: number}) => void
```

當滾動位置接近渲染內容的 `onEndReachedThreshold` 範圍內時，會觸發一次此回調。

| Type     |
| -------- |
| function |

---

### `onEndReachedThreshold`

列表底部邊緣距離內容末尾多遠時（以列表可見長度為單位）觸發 `onEndReached` 回調。例如，值為 0.5 表示當內容末尾處於列表可見長度的一半範圍內時觸發 `onEndReached`。

| Type   |
| ------ |
| number |

---

### `onRefresh`

```jsx
() => void
```

若提供此屬性，將添加標準的 `RefreshControl` 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

| Type     |
| -------- |
| function |

---

### `onScrollToIndexFailed`

```jsx
(info: {
    index: number,
    highestMeasuredFrameIndex: number,
    averageItemLength: number,
  }) => void
```

用於處理滾動到尚未測量的索引時失敗的情況。建議操作是自行計算偏移量並調用 `scrollTo`，或先滾動到最遠可能位置，待更多項目渲染後再重試。

| Type     |
| -------- |
| function |

---

### `onViewableItemsChanged`

當行的可見性發生變化時調用，可見性由 `viewabilityConfig` 屬性定義。

| Type                                                                                                  |
| ----------------------------------------------------------------------------------------------------- |
| `md (callback: {changed: [ViewToken](viewtoken)[], viewableItems: [ViewToken](viewtoken)[]}) => void` |

---

### `persistentScrollbar`

| Type |
| ---- |
| bool |

---

### `progressViewOffset`

當需要偏移量以使加載指示器正確顯示時，設置此屬性。

| Type   |
| ------ |
| number |

---

### `refreshControl`

自定義的刷新控制元件。設定後會覆蓋內建預設的 `<RefreshControl>` 元件，同時忽略 onRefresh 和 refreshing 屬性。僅適用於垂直方向的 VirtualizedList。

| Type    |
| ------- |
| element |

---

### `refreshing`

在等待刷新資料時將此屬性設為 true。

| Type    |
| ------- |
| boolean |

---

### `removeClippedSubviews`

此屬性可提升長列表的滾動效能。

> 注意：某些情況下可能導致內容顯示異常（如內容缺失）—— 請自行承擔使用風險。

| Type    |
| ------- |
| boolean |

---

### `renderScrollComponent`

```jsx
(props: object) => element;
```

渲染自訂滾動元件，例如使用不同樣式的 `RefreshControl`。

| Type     |
| -------- |
| function |

---

### `viewabilityConfig`

參見 `ViewabilityHelper.js` 中的 flow 類型與詳細文件說明。

| Type              |
| ----------------- |
| ViewabilityConfig |

---

### `viewabilityConfigCallbackPairs`

List of `ViewabilityConfig`/`onViewableItemsChanged` pairs. A specific `onViewableItemsChanged` will be called when its corresponding `ViewabilityConfig`'s conditions are met. See `ViewabilityHelper.js` for flow type and further documentation.

| Type                                   |
| -------------------------------------- |
| array of ViewabilityConfigCallbackPair |

---

### `updateCellsBatchingPeriod`

低優先級項目渲染批次間的時間間隔（例如用於渲染遠離可見區域的項目）。與 `maxToRenderPerBatch` 類似，需在填充率與響應速度之間取得平衡。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可見區域外渲染的項目數量上限（以可見區域長度為單位）。例如列表填滿螢幕時，預設值 `windowSize={21}` 會渲染可見區域外加最多10頁上方與10頁下方的內容。調低此值可減少記憶體消耗並可能提升效能，但快速滾動時可能短暫出現未渲染內容的空白區域。

| Type   |
| ------ |
| number |

## 方法

### `flashScrollIndicators()`

```jsx
flashScrollIndicators();
```

---

### `getChildContext()`

```jsx
getChildContext () => Object;
```

回傳的 `Object` 包含以下內容：

- 'virtualizedList' (Object). This object consist of the following
  - getScrollMetrics' (Function). Returns an object with following properties: `{ contentLength: number, dOffset: number, dt: number, offset: number, timestamp: number, velocity: number, visibleLength: number }`.
  - 'horizontal' (boolean) - Optional.
  - 'getOutermostParentListRef' (Function).
  - 'getNestedChildState' (Function) - Returns ChildListState .
  - 'registerAsNestedChild' (Function). This accept an object with following properties `{ cellKey: string, key: string, ref: VirtualizedList, parentDebugInfo: ListDebugInfo }`. It returns a ChildListState
  - 'unregisterAsNestedChild' (Function). This takes an object with following properties, `{ key: string, state: ChildListState }`
  - 'debugInfo' (ListDebugInfo).

---

### `getScrollableNode()`

```jsx
getScrollableNode () => ?number;
```

---

### `getScrollRef()`

```jsx
getScrollRef () => | ?React.ElementRef<typeof ScrollView>
    | ?React.ElementRef<typeof View>;
```

---

### `getScrollResponder()`

```jsx
getScrollResponder () => ?ScrollResponderType;
```

提供底層滾動響應器的控制權。請注意 `this._scrollRef` 可能不是 `ScrollView`，因此在呼叫前需確認其是否響應 `getScrollResponder` 方法。

---

### `hasMore()`

```jsx
hasMore () => boolean;
```

---

### `scrollToEnd()`

```jsx
scrollToEnd(([options]: {animated: boolean}));
```

滾動至內容末端。若未設置 `getItemLayout` 屬性，可能出現卡頓現象。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵值包括：

- `'animated'` (布林值) - 滾動時是否啟用動畫效果。預設為 `true`。

---

### `scrollToIndex()`

```jsx
scrollToIndex((params: object));
```

有效的 `params` 包含：

- 'animated' (布林值)。可選。
- 'index' (數字)。必填。
- 'viewOffset' (數字)。可選。
- 'viewPosition' (數字)。可選。

---

### `scrollToItem()`

```jsx
scrollToItem((params: object));
```

有效的 `params` 包含：

- 'animated' (布林值)。可選。
- 'item' (項目)。必填。
- 'viewPosition' (數字)。可選。

---

### `scrollToOffset()`

```jsx
scrollToOffset((params: object));
```

滾動至列表中的特定像素偏移位置。

參數 `offset` 需指定目標滾動偏移量。若 `horizontal` 為 true，偏移量為 x 軸值；否則為 y 軸值。

參數 `animated` (預設 `true`) 定義滾動時是否啟用動畫效果。

---

### `recordInteraction()`

```jsx
recordInteraction();
```

---

### `setNativeProps()`

```jsx
setNativeProps((props: Object));
```