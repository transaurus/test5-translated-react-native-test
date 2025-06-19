---
id: sectionlist
title: SectionList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程序時，安全性常常被忽視。確實，構建完全無法穿透的軟件是不可能的——我們尚未發明完全無法穿透的鎖（畢竟，銀行金庫仍然會被闖入）。然而，成為惡意攻擊的受害者或暴露安全漏洞的概率與您願意投入保護應用程序免受此類事件影響的努力成反比。儘管普通的掛鎖可以被撬開，但它仍然比櫥櫃掛鉤難得多！

- 完全跨平台。
- 可配置的可見性回調。
- 列表標頭支援。
- 列表頁尾支援。
- 項目分隔線支援。
- 分區標頭支援。
- 分區分隔線支援。
- 異質資料與項目渲染支援。
- 下拉刷新。
- 滾動載入。

若不需要分區支援且希望更簡單的介面，請使用 [`<FlatList>`](flatlist.md)。

## 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=SectionList%20Example
import React from "react";
import { StyleSheet, Text, View, SafeAreaView, SectionList, StatusBar } from "react-native";

const DATA = [
  {
    title: "Main dishes",
    data: ["Pizza", "Burger", "Risotto"]
  },
  {
    title: "Sides",
    data: ["French Fries", "Onion Rings", "Fried Shrimps"]
  },
  {
    title: "Drinks",
    data: ["Water", "Coke", "Beer"]
  },
  {
    title: "Desserts",
    data: ["Cheese Cake", "Ice Cream"]
  }
];

const Item = ({ title }) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => (
  <SafeAreaView style={styles.container}>
    <SectionList
      sections={DATA}
      keyExtractor={(item, index) => item + index}
      renderItem={({ item }) => <Item title={item} />}
      renderSectionHeader={({ section: { title } }) => (
        <Text style={styles.header}>{title}</Text>
      )}
    />
  </SafeAreaView>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: StatusBar.currentHeight,
    marginHorizontal: 16
  },
  item: {
    backgroundColor: "#f9c2ff",
    padding: 20,
    marginVertical: 8
  },
  header: {
    fontSize: 32,
    backgroundColor: "#fff"
  },
  title: {
    fontSize: 24
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=SectionList%20Example
import React, { Component } from "react";
import { StyleSheet, Text, View, SafeAreaView, SectionList, StatusBar } from "react-native";

const DATA = [
  {
    title: "Main dishes",
    data: ["Pizza", "Burger", "Risotto"]
  },
  {
    title: "Sides",
    data: ["French Fries", "Onion Rings", "Fried Shrimps"]
  },
  {
    title: "Drinks",
    data: ["Water", "Coke", "Beer"]
  },
  {
    title: "Desserts",
    data: ["Cheese Cake", "Ice Cream"]
  }
];

const Item = ({ title }) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

class App extends Component {
  render() {
    return (
      <SafeAreaView style={styles.container}>
        <SectionList
          sections={DATA}
          keyExtractor={(item, index) => item + index}
          renderItem={({ item }) => <Item title={item} />}
          renderSectionHeader={({ section: { title } }) => (
            <Text style={styles.header}>{title}</Text>
          )}
        />
      </SafeAreaView>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: StatusBar.currentHeight,
    marginHorizontal: 16
  },
  item: {
    backgroundColor: "#f9c2ff",
    padding: 20,
    marginVertical: 8
  },
  header: {
    fontSize: 32,
    backgroundColor: "#fff"
  },
  title: {
    fontSize: 24
  }
});

export default App;
```

</TabItem>
</Tabs>

這是 [`<VirtualizedList>`](virtualizedlist.md) 的便利封裝，因此繼承了其未在此明確列出的屬性（以及 [`<ScrollView>`](scrollview.md) 的屬性），並有以下注意事項：

- 當內容滾動超出渲染視窗時，內部狀態不會保留。請確保所有資料都儲存在項目資料或外部儲存（如 Flux、Redux 或 Relay）中。
- 這是一個 `PureComponent`，意味著若 `props` 保持淺層相等，它不會重新渲染。請確保 `renderItem` 函數依賴的所有內容都作為 prop 傳遞（例如 `extraData`），且在更新後不會 `===`，否則 UI 可能不會隨變更更新。這包括 `data` prop 和父元件狀態。
- 為了限制記憶體使用並實現平滑滾動，內容會異步在螢幕外渲染。這意味著可能滾動速度超過填充速率，暫時看到空白內容。這是可根據應用需求調整的權衡，我們正在幕後改進此機制。
- 預設情況下，列表會尋找每個項目的 `key` prop 並將其用作 React key。或者，您可以提供自訂的 `keyExtractor` prop。

---

# 參考

## 屬性

### [VirtualizedList 屬性](virtualizedlist.md#props)

繼承 [VirtualizedList 屬性](virtualizedlist.md#props)。

---

### <div class="label required basic">Required</div>**`renderItem`**

每個分區中每個項目的預設渲染器。可依分區覆寫。應返回一個 React 元素。

| Type     |
| -------- |
| function |

渲染函數將傳入包含以下鍵的物件：

- 'item' (object) - 此分區 `data` 鍵中指定的項目物件
- 'index' (number) - 項目在分區中的索引。
- 'section' (object) - `sections` 中指定的完整分區物件。
- 'separators' (object) - 包含以下鍵的物件：
  - 'highlight' (function) - `() => void`
  - 'unhighlight' (function) - `() => void`
  - 'updateProps' (function) - `(select, newProps) => void`
    - 'select' (enum) - 可能值為 'leading'、'trailing'
    - 'newProps' (object)

---

### <div class="label required basic">Required</div>**`sections`**

The actual data to render, akin to the `data` prop in [`FlatList`](flatlist.md).

| Type                                        |
| ------------------------------------------- |
| array of [Section](sectionlist.md#section)s |

---

### `extraData`

一個標記屬性，用於告知列表重新渲染（因為它實作了 `PureComponent`）。若您的 `renderItem`、標頭、頁尾等函數依賴於 `data` prop 之外的任何內容，請將其放在此處並視為不可變。

| Type |
| ---- |
| any  |

---

### `initialNumToRender`

初始批次中要渲染的項目數量。這個數量應該足以填滿螢幕，但不宜過多。請注意，這些項目永遠不會因為視窗化渲染而被卸載，以提升滾動至頂部操作的感知效能。

| Type   | Default |
| ------ | ------- |
| number | `10`    |

---

### `inverted`

反轉滾動方向。使用 -1 的比例變換。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下，會提供 `highlighted`、`section` 和 `[leading/trailing][Item/Section]` 屬性。`renderItem` 提供 `separators.highlight`/`unhighlight` 來更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自訂屬性。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `keyExtractor`

用於為指定索引的項目提取唯一鍵。鍵用於快取和作為 React 鍵來追蹤項目重新排序。預設提取器會檢查 `item.key`，如果沒有則回退使用索引，就像 React 的做法一樣。請注意，這會為每個項目設置鍵，但每個整體區段仍需要自己的鍵。

| Type                                    |
| --------------------------------------- |
| (item: object, index: number) => string |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponent`

在列表的最末端渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListHeaderComponent`

在列表的最開頭渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `onEndReached`

當滾動位置接近渲染內容的 `onEndReachedThreshold` 範圍內時，會呼叫一次。

| Type                                          |
| --------------------------------------------- |
| `(info: { distanceFromEnd: number }) => void` |

---

### `onEndReachedThreshold`

列表底部邊緣距離內容末端多遠（以列表可見長度為單位）時，會觸發 `onEndReached` 回呼。因此，值為 0.5 時，當內容末端在列表可見長度的一半範圍內時，會觸發 `onEndReached`。

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onRefresh`

如果提供，將添加標準的 RefreshControl 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。若要讓 RefreshControl 從頂部偏移（例如 100 點），請使用 `progressViewOffset={100}`。

| Type     |
| -------- |
| function |

---

### `onViewableItemsChanged`

當行的可見性發生變化時呼叫，由 `viewabilityConfig` 屬性定義。

| Type                                                                                                  |
| ----------------------------------------------------------------------------------------------------- |
| `md (callback: {changed: [ViewToken](viewtoken)[], viewableItems: [ViewToken](viewtoken)[]}) => void` |

---

### `refreshing`

在等待刷新數據時將其設為 true。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `removeClippedSubviews`

> 注意：在某些情況下可能存在錯誤（內容缺失）——使用時需自行承擔風險。

這可能會提高大型列表的滾動性能。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `renderSectionFooter`

在每個區段的底部渲染。

| Type                                                                      |
| ------------------------------------------------------------------------- |
| `md (info: {section: [Section](sectionlist#section)}) => element ｜ null` |

---

### `renderSectionHeader`

在每個區段的頂部渲染。在 iOS 上，這些默認會固定在 `ScrollView` 的頂部。請參閱 `stickySectionHeadersEnabled`。

| Type                                                                      |
| ------------------------------------------------------------------------- |
| `md (info: {section: [Section](sectionlist#section)}) => element ｜ null` |

---

### `SectionSeparatorComponent`

在每個區段的頂部和底部渲染（請注意，這與僅在項目之間渲染的 `ItemSeparatorComponent` 不同）。這些旨在將區段與上方和下方的標頭分開，通常具有與 `ItemSeparatorComponent` 相同的高亮響應。還會接收 `highlighted`、`[leading/trailing][Item/Section]` 以及來自 `separators.updateProps` 的任何自定義屬性。

| Type               |
| ------------------ |
| component, element |

---

### `stickySectionHeadersEnabled`

使區段標頭固定在屏幕頂部，直到下一個標頭將其推離。僅在 iOS 上默認啟用，因為這是該平台的標準。

| Type    | Default                                                                                          |
| ------- | ------------------------------------------------------------------------------------------------ |
| boolean | `false` <div class="label android">Android</div><hr/>`true` <div className="label ios">iOS</div> |

## 方法

### `flashScrollIndicators()` <div class="label ios">iOS</div>

```jsx
flashScrollIndicators();
```

暫時顯示滾動指示器。

---

### `recordInteraction()`

```jsx
recordInteraction();
```

通知列表已發生交互，這應觸發可見性計算，例如，如果 `waitForInteractions` 為 true 且用戶尚未滾動。通常由點擊項目或導航操作調用。

---

### `scrollToLocation()`

```jsx
scrollToLocation(params);
```

滾動到指定 `sectionIndex` 和 `itemIndex`（在區段內）的項目，將其定位在可見區域中，其中 `viewPosition` 為 0 時將其置於頂部（可能被固定標頭覆蓋），1 時置於底部，0.5 時置於中間。

> 注意：如果未指定 `getItemLayout` 或 `onScrollToIndexFailed` 屬性，則無法滾動到渲染窗口之外的位置。

**參數：**

| Name                                                    | Type   |
| ------------------------------------------------------- | ------ |
| params <div class="label basic required">Required</div> | object |

有效的 `params` 鍵為：

- 'animated' (boolean) - 列表滾動時是否執行動畫。默認為 `true`。
- 'itemIndex' (number) - 要滾動到的項目在區段內的索引。必需。
- 'sectionIndex' (number) - 包含要滾動到項目的區段索引。必需。
- 'viewOffset' (number) - 固定像素數以偏移最終目標位置，例如用於補償固定標頭。
- 'viewPosition' (number) - 值為 `0` 時將指定索引的項目置於頂部，`1` 時置於底部，`0.5` 時置於中間。

## 類型定義

### Section

標識要為給定區段渲染的數據的對象。

| Type |
| ---- |
| any  |

**Properties:**

| Name                                                  | Type               | Description                                                                                                                                                         |
| ----------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| data <div class="label basic required">Required</div> | array              | The data for rendering items in this section. Array of objects, much like [`FlatList`'s data prop](flatlist#required-data).                                         |
| key                                                   | string             | Optional key to keep track of section re-ordering. If you don't plan on re-ordering sections, the array index will be used by default.                              |
| renderItem                                            | function           | Optionally define an arbitrary item renderer for this section, overriding the default [`renderItem`](sectionlist#renderitem) for the list.                          |
| ItemSeparatorComponent                                | component, element | Optionally define an arbitrary item separator for this section, overriding the default [`ItemSeparatorComponent`](sectionlist#itemseparatorcomponent) for the list. |
| keyExtractor                                          | function           | Optionally define an arbitrary key extractor for this section, overriding the default [`keyExtractor`](sectionlist#keyextractor).                                   |