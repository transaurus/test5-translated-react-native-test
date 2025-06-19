---
id: virtualizedlist
title: VirtualizedList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

作為更便利的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件的基礎實現，這些元件也有更完善的文檔。通常情況下，只有當你需要比 [`FlatList`](flatlist.md) 提供更多靈活性時才應使用此元件，例如需要與不可變數據而非普通陣列配合使用時。

虛擬化技術通過維護一個有限的可見區域渲染窗口，並將窗口外的項目替換為適當大小的空白空間，大幅改善了大型列表的記憶體消耗和性能表現。窗口會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在當前交互完成後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白內容的可能性。

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

注意事項：

- 當內容滾出渲染窗口時，內部狀態不會保留。請確保所有數據都保存在項目數據或外部存儲（如 Flux、Redux 或 Relay）中。
- 這是個 `PureComponent`，意味著當 `props` 淺層比對相等時不會重新渲染。請確保 `renderItem` 函數依賴的所有內容都通過 prop 傳遞（例如 `extraData`），且更新後不保持 `===` 關係，否則變更時 UI 可能不會更新。這包括 `data` prop 和父元件狀態。
- 為限制記憶體消耗並實現平滑滾動，內容會異步離屏渲染。這可能導致滾動速度超過填充速率時短暫出現空白內容。可根據應用需求調整此權衡方案，我們也在持續優化底層機制。
- 預設情況下，列表會尋找每個項目的 `key` prop 作為 React key。也可通過 `keyExtractor` prop 提供自定義鍵值。

---

# 參考文檔

## 屬性

### [ScrollView 屬性](scrollview.md#props)

繼承 [ScrollView 屬性](scrollview.md#props)。

---

### `data`

傳遞給 `getItem` 和 `getItemCount` 以獲取項目的不透明數據類型。

| Type |
| ---- |
| any  |

---

### <div class="label required basic">Required</div> **`getItem`**

```tsx
(data: any, index: number) => any;
```

用於從任意數據塊中提取項目的通用訪問器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`getItemCount`**

```tsx
(data: any) => number;
```

確定數據塊中包含的項目數量。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`renderItem`**

```tsx
(info: any) => ?React.Element<any>
```

從 `data` 中提取項目並渲染到列表中

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

CellRendererComponent 可自定義由 `renderItem`/`ListItemComponent` 渲染的單元格在放入底層 ScrollView 時的包裝方式。該元件必須接受事件處理程序以通知 VirtualizedList 單元格內的變更。

| Type                                     |
| ---------------------------------------- |
| `React.ComponentType<CellRendererProps>` |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供了 `separators.highlight`/`unhighlight` 來更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自訂屬性。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListItemComponent`

每個資料項目都會使用此元素進行渲染。可以是 React 元件類別或渲染函數。

| Type                |
| ------------------- |
| component, function |

---

### `ListFooterComponent`

在所有項目的底部渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

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

在所有項目的頂部渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

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

`debug` 會開啟額外的日誌記錄和視覺疊加層，以協助除錯使用和實作問題，但會顯著影響效能。

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的效能和記憶體優化，但會完全卸載渲染視窗之外的 React 實例。您應該僅在除錯時需要停用此功能。

| Type    |
| ------- |
| boolean |

---

### `extraData`

一個標記屬性，用於告訴列表重新渲染（因為它實作了 `PureComponent`）。如果您的 `renderItem`、Header、Footer 等函數依賴於 `data` 屬性之外的任何內容，請將其放在這裡並視為不可變的。

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

初始批次中要渲染的項目數量。這應該足以填滿螢幕，但不要太多。請注意，這些項目永遠不會作為視窗渲染的一部分被卸載，以提高滾動到頂部操作的感知效能。

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

每次增量渲染批次中渲染的項目最大數量。一次渲染越多，填充率越好，但響應性可能受影響，因為渲染內容可能會干擾按鈕點擊或其他互動的回應。

| Type   |
| ------ |
| number |

---

### `onEndReached`

當滾動位置接近列表邏輯結尾的 `onEndReachedThreshold` 範圍內時，會呼叫一次。

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

若提供此屬性，會添加標準的 `RefreshControl` 以實現「下拉刷新」功能。請確保同時正確設定 `refreshing` 屬性。

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

用於處理滾動到尚未測量索引時的失敗情況。建議操作是自行計算偏移量並呼叫 `scrollTo`，或先滾動到最遠位置，待更多項目渲染後再重試。

| Type     |
| -------- |
| function |

---

### `onStartReached`

當滾動位置接近列表邏輯開頭的 `onStartReachedThreshold` 範圍內時，會呼叫一次。

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

當行的可見性發生變化時呼叫，可見性由 `viewabilityConfig` 屬性定義。

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

當需要調整載入指示器的偏移量時設置此屬性。

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

此屬性可能提升大型列表的滾動效能。

> 注意：在某些情況下可能出現錯誤（內容缺失）——使用時需自行承擔風險。

| Type    |
| ------- |
| boolean |

---

### `renderScrollComponent`

```tsx
(props: object) => element;
```

渲染自定義的滾動元件，例如使用不同樣式的 `RefreshControl`。

| Type     |
| -------- |
| function |

---

### `viewabilityConfig`

參見 `ViewabilityHelper.js` 中的 flow 類型與進一步文檔。

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

低優先級項目渲染批次間的時間間隔（例如渲染遠離可見區域的項目）。與 `maxToRenderPerBatch` 類似，需權衡填充率與響應速度。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可見區域外渲染的最大項目數量（以可見區域長度為單位）。例如列表填滿螢幕時，預設值 `windowSize={21}` 會渲染可見區域加上前後各 10 個螢幕範圍的內容。減少此值可降低記憶體消耗並可能提升效能，但快速滾動時可能短暫出現未渲染內容的空白區域。

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

提供底層滾動響應器的控制權。注意 `this._scrollRef` 可能不是 `ScrollView`，因此需確認其支援 `getScrollResponder` 方法後再調用。

---

### `scrollToEnd()`

```tsx
scrollToEnd(params?: {animated?: boolean});
```

滾動至內容末端。若未設置 `getItemLayout` 屬性可能出現卡頓。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵值包括：

- `'animated'` (布林值) - 列表滾動時是否執行動畫。預設為 `true`。

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

有效的 `params` 包含：

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

滾動至列表中的特定內容像素偏移位置。

參數 `offset` 表示要滾動到的偏移量。若 `horizontal` 為 true，則偏移量為 x 值；其他情況下偏移量為 y 值。

參數 `animated` (預設為 `true`) 定義列表滾動時是否執行動畫。