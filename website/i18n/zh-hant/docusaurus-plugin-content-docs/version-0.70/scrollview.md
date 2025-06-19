---
id: scrollview
title: ScrollView
---

該元件封裝了平台原生的 ScrollView，同時整合了觸控鎖定的「響應者」系統。

請注意，ScrollView 必須具有固定高度才能正常運作，因為它們將無限高度的子元件封裝在有限容器中（透過滾動互動）。若要限制 ScrollView 的高度，可以直接設定視圖高度（不建議）或確保所有父視圖都具有固定高度。忘記將 `{flex: 1}` 傳遞至下層視圖結構可能導致錯誤，此時使用元素檢查器能快速除錯。

目前尚不支援防止其他內含響應者阻擋此滾動視圖成為主響應者。

`<ScrollView>` 與 [`<FlatList>`](flatlist.md) 該如何選擇？

`ScrollView` 會一次性渲染所有子 React 元件，但這會帶來效能問題。

假設您需要顯示非常長的項目列表，可能包含多個畫面的內容。一次性建立所有 JS 元件和原生視圖（其中許多可能根本不會顯示），將導致渲染速度變慢和記憶體使用量增加。

這時 `FlatList` 就派上用場。`FlatList` 會延遲渲染項目（僅在即將顯示時才渲染），並移除滾動超出畫面的項目以節省記憶體和處理時間。

`FlatList` is also handy if you want to render separators between your items, multiple columns, infinite scroll loading, or any number of other features it supports out of the box.

## 範例

```SnackPlayer name=ScrollView
import React from 'react';
import { StyleSheet, Text, SafeAreaView, ScrollView, StatusBar } from 'react-native';

const App = () => {
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scrollView}>
        <Text style={styles.text}>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
          minim veniam, quis nostrud exercitation ullamco laboris nisi ut
          aliquip ex ea commodo consequat. Duis aute irure dolor in
          reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
          pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
          culpa qui officia deserunt mollit anim id est laborum.
        </Text>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: StatusBar.currentHeight,
  },
  scrollView: {
    backgroundColor: 'pink',
    marginHorizontal: 20,
  },
  text: {
    fontSize: 42,
  },
});

export default App;
```

---

# 參考文獻

## 屬性

### [視圖屬性](view.md#props)

繼承自[視圖屬性](view#props)。

---

### `StickyHeaderComponent`

用於渲染黏性標頭的 React 元件，需與 `stickyHeaderIndices` 搭配使用。若您的黏性標頭使用自訂變形效果（例如想讓列表具有可動畫隱藏的標頭），可能需要設定此元件。若未提供元件，將使用預設的 [`ScrollViewStickyHeader`](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Components/ScrollView/ScrollViewStickyHeader.js) 元件。

| Type               |
| ------------------ |
| component, element |

---

### `alwaysBounceHorizontal` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，水平滾動到邊界時仍會彈跳。

| Type | Default                                               |
| ---- | ----------------------------------------------------- |
| bool | `true` when `horizontal={true}`<hr/>`false` otherwise |

---

### `alwaysBounceVertical` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，垂直滾動到邊界時仍會彈跳。

| Type | Default                                             |
| ---- | --------------------------------------------------- |
| bool | `false` when `vertical={true}`<hr/>`true` otherwise |

---

### `automaticallyAdjustContentInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整位於導覽列或標籤列/工具列後方之滾動視圖的內容插邊距。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `automaticallyAdjustKeyboardInsets` <div class="label ios">iOS</div>

控制當鍵盤大小改變時，ScrollView 是否應自動調整其 `contentInset` 和 `scrollViewInsets`。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `automaticallyAdjustsScrollIndicatorInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整滾動指示器的內邊距。詳見蘋果官方[屬性說明文件](https://developer.apple.com/documentation/uikit/uiscrollview/3198043-automaticallyadjustsscrollindica)。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `bounces` <div class="label ios">iOS</div>

When true, the scroll view bounces when it reaches the end of the content if the content is larger than the scroll view along the axis of the scroll direction. When `false`, it disables all bouncing even if the `alwaysBounce*` props are `true`.

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `bouncesZoom` <div class="label ios">iOS</div>

When `true`, gestures can drive zoom past min/max and the zoom will animate to the min/max value at gesture end, otherwise the zoom will not exceed the limits.

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `canCancelContentTouches` <div class="label ios">iOS</div>

When `false`, once tracking starts, won't try to drag if the touch moves.

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `centerContent` <div class="label ios">iOS</div>

When `true`, the scroll view automatically centers the content when the content is smaller than the scroll view bounds; when the content is larger than the scroll view, this property has no effect.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `contentContainerStyle`

這些樣式將應用於包裹所有子視圖的滾動視圖內容容器。範例：

```
return (
  <ScrollView contentContainerStyle={styles.contentContainer}>
  </ScrollView>
);
...
const styles = StyleSheet.create({
  contentContainer: {
    paddingVertical: 20
  }
});
```

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `contentInset` <div class="label ios">iOS</div>

設定滾動視圖內容與邊緣的內縮距離。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `contentInsetAdjustmentBehavior` <div class="label ios">iOS</div>

此屬性指定如何利用安全區域內邊距來調整滾動視圖的內容區域。僅適用於 iOS 11 及以上版本。

| Type                                                           | Default   |
| -------------------------------------------------------------- | --------- |
| enum(`'automatic'`, `'scrollableAxes'`, `'never'`, `'always'`) | `'never'` |

---

### `contentOffset`

用於手動設定初始滾動偏移量。

| Type  | Default        |
| ----- | -------------- |
| Point | `{x: 0, y: 0}` |

---

### `decelerationRate`

A floating-point number that determines how quickly the scroll view decelerates after the user lifts their finger. You may also use string shortcuts `"normal"` and `"fast"` which match the underlying iOS settings for `UIScrollViewDecelerationRateNormal` and `UIScrollViewDecelerationRateFast` respectively.

- `'normal'` 0.998 on iOS, 0.985 on Android.
- `'fast'`, 0.99 on iOS, 0.9 on Android.

| Type                               | Default    |
| ---------------------------------- | ---------- |
| enum(`'fast'`, `'normal'`), number | `'normal'` |

---

### `directionalLockEnabled` <div class="label ios">iOS</div>

設為 true 時，滾動視圖在拖拽過程中會鎖定單一方向（垂直或水平）的滾動。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableIntervalMomentum`

當設為 true 時，無論手勢速度多快，滾動視圖都會在釋放時相對於滾動位置的下一個索引處停止。這可用於當頁面寬度小於水平 ScrollView 或高度小於垂直 ScrollView 時的分頁效果。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableScrollViewPanResponder`

當設為 true 時，ScrollView 上預設的 JS 手勢響應器會被停用，其子元件將完全控制觸摸事件。此屬性特別適用於啟用 `snapToInterval` 的情況，因其不符合常規觸摸模式。若未啟用 `snapToInterval`，請勿在常規 ScrollView 中使用此屬性，否則可能導致滾動時出現非預期觸摸行為。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `endFillColor` <div class="label android">Android</div>

當滾動視圖佔用空間大於內容區域時，此屬性可填充剩餘區域以避免設定背景造成不必要的過度繪製。此為進階優化選項，一般情況下不需使用。

| Type            |
| --------------- |
| [color](colors) |

---

### `fadingEdgeLength` <div class="label android">Android</div>

淡化滾動內容邊緣效果。

若值大於 `0`，會根據當前滾動方向與位置顯示淡出邊緣，提示尚有更多內容可瀏覽。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `horizontal`

設為 `true` 時，滾動視圖的子元件會水平排列成行，而非垂直排列成列。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `indicatorStyle` <div class="label ios">iOS</div>

滾動指示器的樣式。

- `'default'` same as `black`.
- `'black'`, scroll indicator is `black`. This style is good against a light background.
- `'white'`, scroll indicator is `white`. This style is good against a dark background.

| Type                                    | Default     |
| --------------------------------------- | ----------- |
| enum(`'default'`, `'black'`, `'white'`) | `'default'` |

---

### `invertStickyHeaders`

設定黏性標頭應固定在 ScrollView 底部而非頂部，通常與反向滾動的 ScrollView 搭配使用。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `keyboardDismissMode`

決定拖曳時是否關閉鍵盤。

- `'none'`，拖曳不會關閉鍵盤
- `'on-drag'`，開始拖曳時關閉鍵盤

**僅限 iOS**

- `'interactive'`, the keyboard is dismissed interactively with the drag and moves in synchrony with the touch, dragging upwards cancels the dismissal. On Android this is not supported and it will have the same behavior as `'none'`.

| Type                                                                                                                                                            | Default  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| enum(`'none'`, `'on-drag'`) <div className="label android">Android</div><hr />enum(`'none'`, `'on-drag'`, `'interactive'`) <div className="label ios">iOS</div> | `'none'` |

---

### `keyboardShouldPersistTaps`

決定點擊後鍵盤應保持顯示的時機。

- `'never'` tapping outside of the focused text input when the keyboard is up dismisses the keyboard. When this happens, children won't receive the tap.
- `'always'`, the keyboard will not dismiss automatically, and the scroll view will not catch taps, but children of the scroll view can catch taps.
- `'handled'`, the keyboard will not dismiss automatically when the tap was handled by children of the scroll view (or captured by an ancestor).
- `false`, **_deprecated_**, use `'never'` instead
- `true`, **_deprecated_**, use `'always'` instead

| Type                                                      | Default   |
| --------------------------------------------------------- | --------- |
| enum(`'always'`, `'never'`, `'handled'`, `false`, `true`) | `'never'` |

---

### `maintainVisibleContentPosition` <div class="label ios">iOS</div>

設定後，滾動視圖會調整滾動位置，使當前可見且位於或超過 `minIndexForVisible` 的第一個子元件保持位置不變。這適用於雙向載入內容的列表（例如聊天串），避免新訊息進入時導致滾動位置跳動。常見值為 0，但也可設為 1 以跳過載入指示器或其他不應固定位置的內容。

可選參數 `autoscrollToTopThreshold` 可在調整後，若使用者原本位於頂部閾值範圍內，則自動將內容滾動至頂部。這同樣適用於聊天類應用，希望新訊息平滑載入，但避免使用者已滾動較遠時造成干擾性跳動。

注意事項 1：啟用此功能時重新排序滾動視圖中的元件可能導致跳動或卡頓。此問題可修復，但目前無相關計畫。現階段請勿對使用此功能的滾動視圖或列表重新排序內容。

注意事項 2：此功能透過原生代碼的 `contentOffset` 和 `frame.origin` 計算可見性。遮擋、變形或其他複雜因素不會被納入內容是否「可見」的判斷。

| Type                                                                     |
| ------------------------------------------------------------------------ |
| object: `{minIndexForVisible: number, autoscrollToTopThreshold: number}` |

---

### `maximumZoomScale` <div class="label ios">iOS</div>

允許的最大縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

### `minimumZoomScale` <div class="label ios">iOS</div>

允許的最小縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

### `nestedScrollEnabled` <div class="label android">Android</div>

為 Android API 等級 21+ 啟用嵌套滾動。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onContentSizeChange`

當 ScrollView 的可滾動內容視圖大小變化時呼叫。

處理函式接收內容寬度和高度作為參數：`(contentWidth, contentHeight)`

其實作方式是透過附加到 ScrollView 渲染的內容容器的 onLayout 處理器。

| Type     |
| -------- |
| function |

---

### `onMomentumScrollBegin`

當動量滾動開始時呼叫（即 ScrollView 開始滑行時的滾動）。

| Type     |
| -------- |
| function |

---

### `onMomentumScrollEnd`

當動量滾動結束時呼叫（即 ScrollView 滑行至停止時的滾動）。

| Type     |
| -------- |
| function |

---

### `onScroll`

滾動時每幀最多觸發一次。事件頻率可透過 `scrollEventThrottle` 屬性控制。事件結構如下（所有值均為數字）：

```js
{
  nativeEvent: {
    contentInset: {bottom, left, right, top},
    contentOffset: {x, y},
    contentSize: {height, width},
    layoutMeasurement: {height, width},
    zoomScale
  }
}
```

| Type     |
| -------- |
| function |

---

### `onScrollBeginDrag`

當用戶開始拖動滾動視圖時觸發。

| Type     |
| -------- |
| function |

---

### `onScrollEndDrag`

當用戶停止拖動滾動視圖並使其停止或開始滑動時觸發。

| Type     |
| -------- |
| function |

---

### `onScrollToTop` <div class="label ios">iOS</div>

當滾動視圖因點擊狀態列而滾動至頂部時觸發。

| Type     |
| -------- |
| function |

---

### `overScrollMode` <div class="label android">Android</div>

用於覆寫默認的過度滾動模式值。

可能的值：

- `'auto'` - 僅當內容足夠大時才允許用戶過度滾動此視圖。
- `'always'` - 始終允許用戶過度滾動此視圖。
- `'never'` - 禁止用戶過度滾動此視圖。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'always'`, `'never'`) | `'auto'` |

---

### `pagingEnabled`

設為 true 時，滾動視圖會在滾動時停在視圖大小的整數倍位置。可用於實現水平分頁。

> 注意：Android 不支援垂直分頁。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `persistentScrollbar` <div class="label android">Android</div>

使滾動條在不使用時不會變透明。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `pinchGestureEnabled` <div class="label ios">iOS</div>

設為 true 時，允許使用捏合手勢來縮放 ScrollView。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `refreshControl`

RefreshControl 組件，用於為 ScrollView 提供下拉刷新功能。僅適用於垂直 ScrollView（`horizontal` 屬性必須為 `false`）。

參見 [RefreshControl](refreshcontrol)。

| Type    |
| ------- |
| element |

---

### `removeClippedSubviews`

Experimental: When `true`, offscreen child views (whose `overflow` value is `hidden`) are removed from their native backing superview when offscreen. This can improve scrolling performance on long lists.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `scrollEnabled`

設為 false 時，禁止透過觸摸交互滾動視圖。

注意：仍可透過調用 `scrollTo` 方法滾動視圖。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `scrollEventThrottle`

此參數控制滾動時觸發滾動事件的頻率（以毫秒為時間間隔）。數值越低，追蹤滾動位置的程式碼精確度越高，但可能因橋接傳輸大量資訊而導致滾動效能問題。由於 JavaScript 運行循環與螢幕刷新率同步，設定值在 1-16 之間時不會感受到差異。若不需要精確追蹤滾動位置，可提高此值以限制跨橋接傳輸的資訊量。預設值為 `0`，表示每次視圖滾動時僅觸發一次滾動事件。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `scrollIndicatorInsets` <div class="label ios">iOS</div>

滾動指示器與滾動視圖邊緣的內縮距離。通常應設為與 `contentInset` 相同的值。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `scrollPerfTag` <div class="label android">Android</div>

用於記錄此滾動視圖效能日誌的標籤。會強制啟用動量事件（參見 sendMomentumEvents）。預設無實際作用，需實作自訂原生 FpsListener 才能發揮效用。

| Type   |
| ------ |
| string |

---

### `scrollToOverflowEnabled` <div class="label ios">iOS</div>

設為 `true` 時，可透過程式將滾動視圖滾動至超出內容大小的範圍。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `scrollsToTop` <div class="label ios">iOS</div>

設為 `true` 時，點擊狀態列會使滾動視圖滾動至頂部。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsHorizontalScrollIndicator`

設為 `true` 時，顯示水平滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsVerticalScrollIndicator`

設為 `true` 時，顯示垂直滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToAlignment`

當設定 `snapToInterval` 時，`snapToAlignment` 會定義對齊方式與滾動視圖的關係。

可能的值：

- `'start'` 會將對齊點設於左側（水平）或頂部（垂直）。
- `'center'` 會將對齊點設於中央。
- `'end'` 會將對齊點設於右側（水平）或底部（垂直）。

| Type                                 | Default   |
| ------------------------------------ | --------- |
| enum(`'start'`, `'center'`, `'end'`) | `'start'` |

---

### `snapToEnd`

與 `snapToOffsets` 搭配使用。預設情況下，列表末端會視為對齊點。將 `snapToEnd` 設為 false 可禁用此行為，允許列表在末端與最後一個 `snapToOffsets` 對齊點之間自由滾動。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToInterval`

設定後，滾動視圖會在 `snapToInterval` 值的倍數處停止。適用於子項目長度小於滾動視圖的分頁效果。通常與 `snapToAlignment` 和 `decelerationRate="fast"` 搭配使用。會覆蓋較不靈活的 `pagingEnabled` 屬性。

| Type   |
| ------ |
| number |

---

### `snapToOffsets`

當設定此屬性時，滾動視圖會在指定的偏移位置停止。這可用於在滾動視圖中分頁瀏覽大小不一且長度小於滾動視圖的子元素。通常與 `decelerationRate="fast"` 搭配使用。此屬性會覆蓋較不靈活的 `pagingEnabled` 和 `snapToInterval` 屬性。

| Type            |
| --------------- |
| array of number |

---

### `snapToStart`

與 `snapToOffsets` 搭配使用。預設情況下，列表的開頭會視為一個吸附偏移位置。將 `snapToStart` 設為 `false` 可禁用此行為，讓列表能在開頭與第一個 `snapToOffsets` 偏移位置之間自由滾動。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `stickyHeaderHiddenOnScroll`

設為 `true` 時，黏性標頭會在向下滾動列表時隱藏，並在向上滾動時停靠在列表頂部。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `stickyHeaderIndices`

一個子元素索引陣列，決定哪些子元素在滾動時會固定在螢幕頂部。例如，傳入 `stickyHeaderIndices={[0]}` 會讓第一個子元素固定在滾動視圖頂部。也可使用 [x,y,z] 這樣的格式讓多個元素在頂部時保持黏性。此屬性不支援與 `horizontal={true}` 同時使用。

| Type            |
| --------------- |
| array of number |

---

### `zoomScale` <div class="label ios">iOS</div>

滾動視圖內容的當前縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

## 方法

### `flashScrollIndicators()`

```jsx
flashScrollIndicators();
```

短暫顯示滾動指示器。

---

### `scrollTo()`

```jsx
scrollTo(
  options?: { x?: number, y?: number, animated?: boolean } | number,
  deprecatedX?: number,
  deprecatedAnimated?: boolean,
);
```

滾動到指定的 x、y 偏移位置，可選擇立即執行或使用平滑動畫。

**範例：**

`scrollTo({ x: 0, y: 0, animated: true })`

> 注意：此函數簽名的奇怪設計是由於歷史原因，該函數也接受獨立參數作為選項物件的替代方案。由於存在歧義（y 在 x 之前），此用法已被棄用，不應繼續使用。

---

### `scrollToEnd()`

```jsx
scrollToEnd(([options]: {animated: boolean}));
```

如果是垂直滾動視圖，則滾動到底部；如果是水平滾動視圖，則滾動到右側。

使用 `scrollToEnd({ animated: true })` 可進行平滑動畫滾動，使用 `scrollToEnd({ animated: false })` 則會立即滾動。若未傳入選項，`animated` 預設為 `true`。