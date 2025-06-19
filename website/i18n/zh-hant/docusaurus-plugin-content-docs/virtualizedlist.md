---
id: virtualizedlist
title: VirtualizedList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

這是更便利的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件的基礎實現，這些元件也有更完善的文件說明。通常只有在需要比 [`FlatList`](flatlist.md) 提供更多彈性的情況下才應使用此元件，例如需搭配不可變數據而非普通陣列時。

虛擬化技術透過維護一個有限的有效渲染視窗，並將視窗外的項目替換為適當大小的空白空間，大幅改善了大型列表的記憶體消耗與效能表現。此視窗會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在執行中的互動之後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白空間的可能性。

## 範例

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=VirtualizedListExample&ext=js
import React from 'react';
import {View, VirtualizedList, StyleSheet, Text, StatusBar} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const getItem = (_data, index) => ({
  id: Math.random().toString(12).substring(0),
  title: `Item ${index + 1}`,
});

const getItemCount = _data => 50;

const Item = ({title}) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container} edges={['top']}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  </SafeAreaProvider>
);

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

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=VirtualizedListExample&ext=tsx
import React from 'react';
import {View, VirtualizedList, StyleSheet, Text, StatusBar} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

type ItemData = {
  id: string;
  title: string;
};

const getItem = (_data: unknown, index: number): ItemData => ({
  id: Math.random().toString(12).substring(0),
  title: `Item ${index + 1}`,
});

const getItemCount = (_data: unknown) => 50;

type ItemProps = {
  title: string;
};

const Item = ({title}: ItemProps) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container} edges={['top']}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  </SafeAreaProvider>
);

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

</TabItem>
</Tabs>

---

需注意事項：

- 當內容滾出渲染視窗時，內部狀態不會保留。請確保所有數據都儲存在項目數據或外部狀態管理工具（如 Flux、Redux 或 Relay）中。
- 此為 `PureComponent`，意味著若 `props` 為淺層相等時不會重新渲染。請確認 `renderItem` 函式依賴的所有內容都透過 prop 傳遞（例如 `extraData`），且更新後不為 `===`，否則變更時 UI 可能不會更新。這包含 `data` prop 和父元件狀態。
- 為限制記憶體使用並實現平滑滾動，內容會以非同步方式在螢幕外渲染。這可能導致滾動速度超過填充速率而短暫看到空白內容。此為可依應用需求調整的權衡取捨，我們也正持續優化此機制。
- 預設情況下，列表會尋找每個項目的 `key` prop 作為 React key。您也可透過自訂 `keyExtractor` prop 提供 key。

---

# 參考文件

## 屬性

### [ScrollView 屬性](scrollview.md#props)

繼承 [ScrollView 屬性](scrollview.md#props)。

---

### `data`

傳遞給 `getItem` 和 `getItemCount` 以取得項目的不透明數據類型。

| Type |
| ---- |
| any  |

---

### <div class="label required basic">Required</div> **`getItem`**

```tsx
(data: any, index: number) => any;
```

用於從任何類型的數據塊中提取項目的通用存取器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`getItemCount`**

```tsx
(data: any) => number;
```

決定數據塊中包含多少項目。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`renderItem`**

```tsx
(info: any) => ?React.Element<any>
```

從 `data` 取得項目並渲染至列表中

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

CellRendererComponent 可自訂由 `renderItem`/`ListItemComponent` 渲染的單元格在放入底層 ScrollView 時的包裝方式。此元件必須接受事件處理程序以通知 VirtualizedList 單元格內的變更。

| Type                                     |
| ---------------------------------------- |
| `React.ComponentType<CellRendererProps>` |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。默認情況下，會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供了 `separators.highlight`/`unhighlight` 來更新 `highlighted` 屬性，但您也可以通過 `separators.updateProps` 添加自定義屬性。可以是 React 組件（例如 `SomeComponent`），也可以是 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 組件（例如 `SomeComponent`），也可以是 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListItemComponent`

每個數據項目都使用此元素進行渲染。可以是 React 組件類，或渲染函數。

| Type                |
| ------------------- |
| component, function |

---

### `ListFooterComponent`

在所有項目的底部渲染。可以是 React 組件（例如 `SomeComponent`），也可以是 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponentStyle`

用於 `ListFooterComponent` 內部 View 的樣式。

| Type          | Required |
| ------------- | -------- |
| ViewStyleProp | No       |

---

### `ListHeaderComponent`

在所有項目的頂部渲染。可以是 React 組件（例如 `SomeComponent`），也可以是 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListHeaderComponentStyle`

用於 `ListHeaderComponent` 內部 View 的樣式。

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `debug`

`debug` 會開啟額外的日誌記錄和視覺疊加層，以幫助調試使用和實現，但會顯著影響性能。

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的性能和內存優化，但會完全卸載渲染窗口之外的 React 實例。您應該僅在調試時禁用此功能。

| Type    |
| ------- |
| boolean |

---

### `extraData`

一個標記屬性，用於告訴列表重新渲染（因為它實現了 `PureComponent`）。如果您的 `renderItem`、Header、Footer 等函數依賴於 `data` 屬性之外的任何內容，請將其放在這裡並視為不可變。

| Type |
| ---- |
| any  |

---

### `getItemLayout`

```tsx
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

不從頂部的第一個項目開始，而是從 `initialScrollIndex` 指定的位置開始。這會停用「滾動至頂部」的優化功能（該功能會保持前 `initialNumToRender` 個項目始終渲染），並立即從此初始索引開始渲染項目。需要實作 `getItemLayout` 方法。

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

### `keyExtractor`

```tsx
(item: any, index: number) => string;
```

用於為指定索引的項目提取唯一鍵值。該鍵值用於緩存，並作為 React 的 key 來追蹤項目重新排序。預設提取器會檢查 `item.key`，其次是 `item.id`，最後回退到使用索引（與 React 的行為一致）。

| Type     |
| -------- |
| function |

---

### `maxToRenderPerBatch`

每次增量渲染批次中最多渲染的項目數。一次渲染越多，填充率越好，但響應性可能下降，因為渲染內容可能會干擾按鈕點擊或其他互動操作。

| Type   |
| ------ |
| number |

---

### `onEndReached`

當滾動位置接近列表邏輯末尾（距離為 `onEndReachedThreshold`）時觸發一次。

| Type                                        |
| ------------------------------------------- |
| `(info: {distanceFromEnd: number}) => void` |

---

### `onEndReachedThreshold`

How far from the end (in units of visible length of the list) the trailing edge of the list must be from the end of the content to trigger the `onEndReached` callback. Thus, a value of 0.5 will trigger `onEndReached` when the end of the content is within half the visible length of the list.

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onRefresh`

```tsx
() => void;
```

若提供此屬性，會添加標準的 `RefreshControl` 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

| Type     |
| -------- |
| function |

---

### `onScrollToIndexFailed`

```tsx
(info: {
  index: number,
  highestMeasuredFrameIndex: number,
  averageItemLength: number,
}) => void;
```

用於處理滾動到尚未測量索引時的失敗情況。建議操作是自行計算偏移量並調用 `scrollTo`，或先滾動到最遠位置後等待更多項目渲染完成再重試。

| Type     |
| -------- |
| function |

---

### `onStartReached`

當滾動位置接近列表邏輯開頭（距離為 `onStartReachedThreshold`）時觸發一次。

| Type                                          |
| --------------------------------------------- |
| `(info: {distanceFromStart: number}) => void` |

---

### `onStartReachedThreshold`

How far from the start (in units of visible length of the list) the leading edge of the list must be from the start of the content to trigger the `onStartReached` callback. Thus, a value of 0.5 will trigger `onStartReached` when the start of the content is within half the visible length of the list.

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onViewableItemsChanged`

當行的可見性發生變化時觸發（由 `viewabilityConfig` 屬性定義）。

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

設定此參數以確保載入指示器的偏移量顯示正確。

| Type   |
| ------ |
| number |

---

### `refreshControl`

A custom refresh control element. When set, it overrides the default `<RefreshControl>` component built internally. The onRefresh and refreshing props are also ignored. Only works for vertical VirtualizedList.

| Type    |
| ------- |
| element |

---

### `refreshing`

Set this true while waiting for new data from a refresh.

| Type    |
| ------- |
| boolean |

---

### `removeClippedSubviews`

此設定可提升大型列表的滾動效能。

> 注意：在某些情況下可能會有錯誤（內容缺失）— 使用時請自行承擔風險。

| Type    |
| ------- |
| boolean |

---

### `renderScrollComponent`

```tsx
(props: object) => element;
```

渲染自訂的滾動元件，例如使用不同樣式的 `RefreshControl`。

| Type     |
| -------- |
| function |

---

### `viewabilityConfig`

請參閱 `ViewabilityHelper.js` 以獲取流類型及進一步文件說明。

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

低優先級項目渲染批次間的時間間隔（例如渲染遠離螢幕可見區域的項目）。與 `maxToRenderPerBatch` 類似，需在填充率與響應性之間取捨。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可見區域外渲染的項目數量上限（以可見區域長度為單位）。例如列表填滿螢幕時，預設值 `windowSize={21}` 會渲染可見區域加上前後各 10 個螢幕範圍的內容。降低此數值可減少記憶體消耗並可能提升效能，但快速滾動時可能短暫出現未渲染內容的空白區域。

| Type   |
| ------ |
| number |

## 方法

### `flashScrollIndicators()`

```tsx
flashScrollIndicators();
```

---

### `getScrollableNode()`

```tsx
getScrollableNode(): any;
```

---

### `getScrollRef()`

```tsx
getScrollRef():
  | React.ElementRef<typeof ScrollView>
  | React.ElementRef<typeof View>
  | null;
```

---

### `getScrollResponder()`

```tsx
getScrollResponder () => ScrollResponderMixin | null;
```

提供底層滾動響應器的控制權。請注意 `this._scrollRef` 可能不是 `ScrollView`，因此需確認其支援 `getScrollResponder` 方法後再呼叫。

---

### `scrollToEnd()`

```tsx
scrollToEnd(params?: {animated?: boolean});
```

滾動至內容底部。若未設定 `getItemLayout` 屬性，滾動可能不流暢。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵值包括：

- `'animated'` (布林值) - 控制列表滾動時是否顯示動畫。預設為 `true`。

---

### `scrollToIndex()`

```tsx
scrollToIndex(params: {
  index: number;
  animated?: boolean;
  viewOffset?: number;
  viewPosition?: number;
});
```

有效的 `params` 參數包含：

- 'index' (數字)。必填。
- 'animated' (布林值)。選填。
- 'viewOffset' (數字)。選填。
- 'viewPosition' (數字)。選填。

---

### `scrollToItem()`

```tsx
scrollToItem(params: {
  item: ItemT;
  animated?: boolean;
  viewOffset?: number;
  viewPosition?: number;
);
```

有效的 `params` 參數包含：

- 'item' (項目)。必填。
- 'animated' (布林值)。選填。
- 'viewOffset' (數字)。選填。
- 'viewPosition' (數字)。選填。

---

### `scrollToOffset()`

```tsx
scrollToOffset(params: {
  offset: number;
  animated?: boolean;
});
```

滾動至列表中的特定像素偏移位置。

參數 `offset` 為目標滾動偏移值。若 `horizontal` 為 true，偏移值為 x 軸座標，否則為 y 軸座標。

參數 `animated` (預設 `true`) 控制滾動時是否顯示動畫。