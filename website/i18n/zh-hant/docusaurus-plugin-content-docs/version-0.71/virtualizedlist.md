---
id: virtualizedlist
title: VirtualizedList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

作為更便利的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件的基礎實現，這些元件也有更完善的說明文件。通常只有在需要比 [`FlatList`](flatlist.md) 提供更高靈活性的情況下才應使用此元件，例如處理不可變資料而非普通陣列時。

虛擬化技術透過維護一個有限的可見項目渲染視窗，並將視窗外的項目替換為適當大小的空白空間，大幅改善了大型列表的記憶體消耗與效能表現。該視窗會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在執行中的互動之後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白空間的可能性。

## 範例

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=VirtualizedListExample&ext=js
import React from 'react';
import {
  SafeAreaView,
  View,
  VirtualizedList,
  StyleSheet,
  Text,
  StatusBar,
} from 'react-native';

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

const App = () => {
  return (
    <SafeAreaView style={styles.container}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  );
};

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
import {
  SafeAreaView,
  View,
  VirtualizedList,
  StyleSheet,
  Text,
  StatusBar,
} from 'react-native';

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

const App = () => {
  return (
    <SafeAreaView style={styles.container}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  );
};

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

- 當內容滾出渲染視窗時，內部狀態不會保留。請確保所有資料都儲存在項目資料或外部狀態管理工具（如 Flux、Redux 或 Relay）中。
- 此為 `PureComponent`，意味著若 `props` 進行淺層比較後相等則不會重新渲染。請確認 `renderItem` 函式所依賴的所有內容都透過 prop 傳遞（例如 `extraData`），且更新後不會保持 `===` 相等，否則 UI 可能不會隨變更更新。這包含 `data` prop 和父元件狀態。
- 為限制記憶體使用並實現平滑滾動，內容會以非同步方式在螢幕外渲染。這可能導致滾動速度超過填充速率而暫時看到空白內容。此為可根據應用需求調整的權衡取捨，我們也持續在底層進行改進。
- 預設情況下，列表會尋找每個項目的 `key` prop 作為 React key。您也可以提供自訂的 `keyExtractor` prop 來覆寫此行為。

---

# 參考文獻

## Props

### [ScrollView Props](scrollview.md#props)

繼承 [ScrollView Props](scrollview.md#props)。

---

### `data`

傳遞給 `getItem` 和 `getItemCount` 以取得項目的不透明資料類型。

| Type |
| ---- |
| any  |

---

### <div class="label required basic">Required</div> **`getItem`**

```tsx
(data: any, index: number) => any;
```

用於從任何資料塊中提取項目的通用存取器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`getItemCount`**

```tsx
(data: any) => number;
```

決定資料塊中包含多少個項目。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`renderItem`**

```tsx
(info: any) => ?React.Element<any>
```

從 `data` 中取得項目並將其渲染至列表中

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

每個單元格都使用此元件進行渲染。可以是 React 元件類別或渲染函式。預設使用 [`View`](view.md)。

| Type                |
| ------------------- |
| component, function |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下，會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供的 `separators.highlight`/`unhighlight` 會更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自訂屬性。可以是 React 元件（例如 `SomeComponent`），或是 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 元件（例如 `SomeComponent`），或是 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListItemComponent`

每個資料項目都會使用此元素進行渲染。可以是 React 元件類別，或是渲染函數。

| Type                |
| ------------------- |
| component, function |

---

### `ListFooterComponent`

在所有項目的底部渲染。可以是 React 元件（例如 `SomeComponent`），或是 React 元素（例如 `<SomeComponent />`）。

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

在所有項目的頂部渲染。可以是 React 元件（例如 `SomeComponent`），或是 React 元素（例如 `<SomeComponent />`）。

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

`debug` will turn on extra logging and visual overlays to aid with debugging both usage and implementation, but with a significant perf hit.

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的性能和記憶體優化，但會完全卸載渲染視窗之外的 React 實例。您應該僅在調試時需要禁用此功能。

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

初始批次中要渲染的項目數量。這應該足以填滿螢幕，但不要過多。請注意，這些項目永遠不會作為視窗渲染的一部分被卸載，以提高滾動到頂部操作的感知性能。

| Type   | Default |
| ------ | ------- |
| number | `10`    |

---

### `initialScrollIndex`

不從頂部的第一個項目開始，而是從 `initialScrollIndex` 指定的位置開始。這會停用「滾動至頂部」的優化功能（該功能會保持前 `initialNumToRender` 個項目始終渲染），並立即從此初始索引開始渲染項目。需要實作 `getItemLayout`。

| Type   |
| ------ |
| number |

---

### `inverted`

反轉滾動方向。使用 `-1` 的縮放變換。

| Type    |
| ------- |
| boolean |

---

### `keyExtractor`

```tsx
(item: any, index: number) => string;
```

用於為指定索引的項目提取唯一鍵。此鍵用於快取及作為 React 的 key 來追蹤項目重新排序。預設提取器會檢查 `item.key`，然後是 `item.id`，最後回退到使用索引（與 React 的行為相同）。

| Type     |
| -------- |
| function |

---

### `maxToRenderPerBatch`

每次增量渲染批次中要渲染的最大項目數。一次渲染越多，填充率越好，但響應性可能受影響，因為渲染內容可能會干擾按鈕點擊或其他互動的回應。

| Type   |
| ------ |
| number |

---

### `onEndReached`

```tsx
(info: {distanceFromEnd: number}) => void;
```

當滾動位置接近渲染內容的 `onEndReachedThreshold` 範圍內時，會觸發一次。

| Type     |
| -------- |
| function |

---

### `onEndReachedThreshold`

How far from the end (in units of visible length of the list) the bottom edge of the list must be from the end of the content to trigger the `onEndReached` callback. Thus a value of 0.5 will trigger `onEndReached` when the end of the content is within half the visible length of the list.

| Type   |
| ------ |
| number |

---

### `onRefresh`

```tsx
() => void;
```

若提供此屬性，將添加標準的 `RefreshControl` 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

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

用於處理滾動到尚未測量索引時的失敗情況。建議操作是自行計算偏移量並調用 `scrollTo`，或滾動到最遠位置後等待更多項目渲染完成再重試。

| Type     |
| -------- |
| function |

---

### `onViewableItemsChanged`

當行的可見性發生變化時調用，其定義由 `viewabilityConfig` 屬性決定。

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

當需要偏移量以正確顯示加載指示器時設置此屬性。

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

在等待刷新數據時將其設為 true。

| Type    |
| ------- |
| boolean |

---

### `removeClippedSubviews`

這可能提升大型列表的滾動效能。

> 注意：在某些情況下可能會有錯誤（內容缺失）——使用時需自行承擔風險。

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

低優先級項目渲染批次之間的時間間隔，例如用於渲染遠離螢幕的項目。與 `maxToRenderPerBatch` 類似，需在填充率與響應性之間取得平衡。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可見區域外渲染的最大項目數量，以可見長度為單位。例如，若您的列表填滿螢幕，則 `windowSize={21}`（預設值）將渲染可見螢幕區域加上視窗上方和下方各 10 個螢幕的內容。減少此數值將降低記憶體消耗並可能提升效能，但會增加快速滾動時可能出現短暫空白未渲染內容區域的風險。

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

提供底層滾動響應器的控制權。請注意，`this._scrollRef` 可能不是 `ScrollView`，因此在呼叫前需確認其是否響應 `getScrollResponder`。

---

### `scrollToEnd()`

```tsx
scrollToEnd(params?: {animated?: boolean});
```

滾動至內容末端。若未設定 `getItemLayout` 屬性，可能會出現卡頓。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵為：

- `'animated'` (布林值) - 滾動時是否執行動畫。預設為 `true`。

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

有效的 `params` 包含：

- 'index' (數字). 必填。
- 'animated' (布林值). 選填。
- 'viewOffset' (數字). 選填。
- 'viewPosition' (數字). 選填。

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

- 'item' (Item)。必填。
- 'animated' (boolean)。選填。
- 'viewOffset' (number)。選填。
- 'viewPosition' (number)。選填。

---

### `scrollToOffset()`

```tsx
scrollToOffset(arams: {
  offset: number;
  animated?: boolean;
});
```

滾動至列表中的特定內容像素偏移位置。

參數 `offset` 為要滾動至的偏移值。若 `horizontal` 為 true，則偏移值為 x 軸座標，否則為 y 軸座標。

參數 `animated`（預設為 `true`）定義列表滾動時是否執行動畫效果。