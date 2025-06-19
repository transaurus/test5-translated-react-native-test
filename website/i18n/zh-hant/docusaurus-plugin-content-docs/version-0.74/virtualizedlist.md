---
id: virtualizedlist
title: VirtualizedList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

這是為更方便使用的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件提供的基礎實現，這些元件也有更完善的文檔。通常情況下，只有當你需要比 [`FlatList`](flatlist.md) 提供更多靈活性時（例如使用不可變數據而非普通陣列）才應該使用此元件。

虛擬化技術通過維護一個有限的可見區域渲染窗口，並將窗口外的項目替換為適當大小的空白空間，大幅提升了大型列表的記憶體使用效率和性能。該窗口會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在當前交互完成後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白區域的可能性。

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

注意事項：

- 當內容滾出渲染窗口時，內部狀態不會保留。請確保所有數據都保存在項目數據或外部存儲（如 Flux、Redux 或 Relay）中。
- 這是個 `PureComponent`，意味著當 `props` 淺層相等時不會重新渲染。請確保 `renderItem` 函數依賴的所有內容都通過 prop 傳遞（例如 `extraData`），且更新後不會 `===`，否則 UI 可能不會隨變化更新。這包括 `data` prop 和父元件狀態。
- 為了限制記憶體使用並實現平滑滾動，內容會在屏幕外異步渲染。這意味著可能滾動速度會快於填充速度，從而暫時看到空白內容。這是可根據應用需求調整的權衡方案，我們正在幕後持續改進。
- 默認情況下，列表會查找每個項目的 `key` prop 並用作 React key。你也可以提供自定義的 `keyExtractor` prop。

---

# 參考

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

用於從任何類型的數據塊中提取項目的通用訪問器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`getItemCount`**

```tsx
(data: any) => number;
```

確定數據塊中包含多少個項目。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">Required</div> **`renderItem`**

```tsx
(info: any) => ?React.Element<any>
```

從 `data` 中獲取項目並將其渲染到列表中

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

CellRendererComponent 允許自定義由 `renderItem`/`ListItemComponent` 渲染的單元格在放入底層 ScrollView 時的包裝方式。此元件必須接受事件處理程序，以通知 VirtualizedList 單元格內的變化。

| Type                                     |
| ---------------------------------------- |
| `React.ComponentType<CellRendererProps>` |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下，會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供 `separators.highlight`/`unhighlight` 來更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自定義屬性。可以是 React 元件（例如 `SomeComponent`），或 React 元素（例如 `<SomeComponent />`）。

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

每個資料項目都會使用此元素渲染。可以是 React 元件類別，或渲染函式。

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

`debug` 會開啟額外的日誌記錄和視覺疊加層，以幫助調試使用和實現，但會顯著影響效能。

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的效能和記憶體優化，但會完全卸載渲染視窗之外的 React 實例。您應該僅在調試時需要禁用此功能。

| Type    |
| ------- |
| boolean |

---

### `extraData`

一個標記屬性，用於告訴列表重新渲染（因為它實現了 `PureComponent`）。如果您的 `renderItem`、Header、Footer 等函式依賴於 `data` 屬性之外的任何內容，請將其放在這裡並將其視為不可變的。

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

不從頂部的第一個項目開始，而是從 `initialScrollIndex` 指定的位置開始。這會停用「滾動至頂部」的優化功能（該功能會始終渲染前 `initialNumToRender` 個項目），並立即從此初始索引開始渲染項目。需要實作 `getItemLayout` 才能使用此功能。

| Type   |
| ------ |
| number |

---

### `inverted`

反轉滾動方向。使用 `-1` 的縮放變換來實現。

| Type    |
| ------- |
| boolean |

---

### `keyExtractor`

```tsx
(item: any, index: number) => string;
```

用於為指定索引的項目提取唯一鍵。此鍵用於緩存和作為 React 的鍵來追蹤項目重新排序。預設的提取器會檢查 `item.key`，然後是 `item.id`，最後回退到使用索引（與 React 的行為相同）。

| Type     |
| -------- |
| function |

---

### `maxToRenderPerBatch`

每個增量渲染批次中要渲染的最大項目數。一次渲染的項目越多，填充率越好，但響應性可能會受到影響，因為渲染內容可能會干擾按鈕點擊或其他互動的響應。

| Type   |
| ------ |
| number |

---

### `onEndReached`

當滾動位置距離列表邏輯結尾的距離在 `onEndReachedThreshold` 範圍內時，會觸發一次此回調。

| Type                                        |
| ------------------------------------------- |
| `(info: {distanceFromEnd: number}) => void` |

---

### `onEndReachedThreshold`

列表的尾部邊緣距離內容結尾多遠（以列表可見長度為單位）時會觸發 `onEndReached` 回調。例如，值為 0.5 表示當內容結尾位於列表可見長度的一半範圍內時，會觸發 `onEndReached`。

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onRefresh`

```tsx
() => void;
```

如果提供此回調，將添加一個標準的 `RefreshControl` 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

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

用於處理滾動到尚未測量的索引時失敗的情況。建議的操作是計算自己的偏移量並調用 `scrollTo`，或者滾動到最遠位置後再嘗試（當更多項目已渲染時）。

| Type     |
| -------- |
| function |

---

### `onStartReached`

當滾動位置距離列表邏輯開頭的距離在 `onStartReachedThreshold` 範圍內時，會觸發一次此回調。

| Type                                          |
| --------------------------------------------- |
| `(info: {distanceFromStart: number}) => void` |

---

### `onStartReachedThreshold`

列表的頭部邊緣距離內容開頭多遠（以列表可見長度為單位）時會觸發 `onStartReached` 回調。例如，值為 0.5 表示當內容開頭位於列表可見長度的一半範圍內時，會觸發 `onStartReached`。

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onViewableItemsChanged`

當行的可見性發生變化時觸發，具體由 `viewabilityConfig` 屬性定義。

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

當需要正確顯示載入指示器時，設定此偏移量。

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

此屬性可提升長列表的滾動效能。

> 注意：在某些情況下可能出現錯誤（內容缺失）——使用時請自行承擔風險。

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

請參閱 `ViewabilityHelper.js` 中的 flow 類型與進一步說明文件。

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

低優先級項目渲染批次間的時間間隔（例如渲染遠離可見區域的項目）。與 `maxToRenderPerBatch` 類似，需在填充率與響應速度之間權衡。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可見區域外渲染的最大項目數量（以可見區域長度為單位）。例如若列表填滿螢幕，預設值 `windowSize={21}` 會渲染可見區域加上前後各 10 個螢幕範圍的內容。減少此數值可降低記憶體消耗並可能提升效能，但會增加快速滾動時短暫顯示空白未渲染內容的機率。

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

提供底層滾動響應器的控制代碼。請注意 `this._scrollRef` 可能不是 `ScrollView`，因此需確認其支援 `getScrollResponder` 方法後再呼叫。

---

### `scrollToEnd()`

```tsx
scrollToEnd(params?: {animated?: boolean});
```

滾動至內容末端。若未設定 `getItemLayout` 屬性，可能出現卡頓。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵值為：

- `'animated'` (布林值) - 控制列表滾動時是否執行動畫效果。預設為 `true`。

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

Valid `params` consist of:

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

Valid `params` consist of:

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

滾動至列表內指定的像素偏移位置。

參數 `offset` 需指定目標滾動偏移量。若 `horizontal` 為 true 時表示 x 軸座標，其餘情況為 y 軸座標。

參數 `animated` (預設 `true`) 控制滾動時是否啟用動畫效果。