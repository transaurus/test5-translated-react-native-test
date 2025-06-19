---
id: scrollview
title: ScrollView
---

該元件封裝了平台的 ScrollView，同時提供與觸控鎖定「響應者」系統的整合。

請注意，ScrollView 必須具有固定的高度才能正常運作，因為它們將無限高度的子元件包含在一個有限容器中（透過滾動互動）。為了限制 ScrollView 的高度，可以直接設定視圖的高度（不建議）或確保所有父視圖都具有固定高度。忘記將 `{flex: 1}` 傳遞到視圖堆疊中可能會導致錯誤，而元素檢查器可以快速幫助除錯。

目前尚不支援其他包含的響應者阻止此滾動視圖成為響應者。

`<ScrollView>` 與 [`<FlatList>`](flatlist.md) 該使用哪一個？

`ScrollView` 會一次性渲染所有 React 子元件，但這會帶來性能上的缺點。

想像你有一個非常長的項目列表需要顯示，可能是好幾頁的內容。一次性為所有內容創建 JS 元件和原生視圖，其中許多可能根本不會顯示，這會導致渲染速度變慢並增加記憶體使用量。

這就是 `FlatList` 的用武之地。`FlatList` 會懶加載項目，僅在它們即將出現時渲染，並移除滾動到螢幕外的項目以節省記憶體和處理時間。

`FlatList` is also handy if you want to render separators between your items, multiple columns, infinite scroll loading, or any number of other features it supports out of the box.

## 範例

```SnackPlayer name=ScrollView
import React from 'react';
import {
  StyleSheet,
  Text,
  SafeAreaView,
  ScrollView,
  StatusBar,
} from 'react-native';

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
};

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

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view#props)。

---

### `StickyHeaderComponent`

一個 React 元件，用於渲染黏性標頭，應與 `stickyHeaderIndices` 一起使用。如果你的黏性標頭使用自訂變換（例如，當你想要列表具有動畫效果且可隱藏的標頭時），可能需要設定此元件。如果未提供元件，將使用預設的 [`ScrollViewStickyHeader`](https://github.com/facebook/react-native/blob/0.71-stable/Libraries/Components/ScrollView/ScrollViewStickyHeader.js) 元件。

| Type               |
| ------------------ |
| component, element |

---

### `alwaysBounceHorizontal` <div class="label ios">iOS</div>

When true, the scroll view bounces horizontally when it reaches the end even if the content is smaller than the scroll view itself.

| Type | Default                                               |
| ---- | ----------------------------------------------------- |
| bool | `true` when `horizontal={true}`<hr/>`false` otherwise |

---

### `alwaysBounceVertical` <div class="label ios">iOS</div>

When true, the scroll view bounces vertically when it reaches the end even if the content is smaller than the scroll view itself.

| Type | Default                                             |
| ---- | --------------------------------------------------- |
| bool | `false` when `vertical={true}`<hr/>`true` otherwise |

---

### `automaticallyAdjustContentInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整位於導覽列或標籤列/工具列後方的滾動視圖的內容內嵌。

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

控制 iOS 是否應自動調整滾動指示器的內邊距。詳見 Apple 的[屬性說明文件](https://developer.apple.com/documentation/uikit/uiscrollview/3198043-automaticallyadjustsscrollindica)。

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

設為 `false` 時，一旦開始追蹤觸控，即使移動觸點也不會觸發拖曳。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `centerContent` <div class="label ios">iOS</div>

設為 `true` 時，當內容小於滾動視圖邊界會自動置中；若內容大於滾動視圖則此屬性無效。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `contentContainerStyle`

此樣式會套用至包裹所有子視圖的滾動視圖內容容器。範例：

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

此屬性指定如何利用安全區域邊距調整滾動視圖內容區域。僅限 iOS 11 及以上版本。

| Type                                                           | Default   |
| -------------------------------------------------------------- | --------- |
| enum(`'automatic'`, `'scrollableAxes'`, `'never'`, `'always'`) | `'never'` |

---

### `contentOffset`

用於手動設定起始滾動偏移量。

| Type  | Default        |
| ----- | -------------- |
| Point | `{x: 0, y: 0}` |

---

### `decelerationRate`

此浮點數決定用戶鬆手後滾動視圖的減速速度。亦可使用字串快捷參數 `"normal"` 與 `"fast"`，分別對應 iOS 底層設定 `UIScrollViewDecelerationRateNormal` 和 `UIScrollViewDecelerationRateFast`。

- `'normal'`：iOS 為 0.998，Android 為 0.985
- `'fast'`：iOS 為 0.99，Android 為 0.9

| Type                               | Default    |
| ---------------------------------- | ---------- |
| enum(`'fast'`, `'normal'`), number | `'normal'` |

---

### `directionalLockEnabled` <div class="label ios">iOS</div>

When true, the ScrollView will try to lock to only vertical or horizontal scrolling while dragging.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableIntervalMomentum`

當設為 true 時，無論手勢速度多快，滾動視圖都會在釋放時對齊下一個索引位置（根據釋放時的滾動位置）。這可用於實現分頁效果，尤其當頁面寬度小於水平滾動視圖寬度或頁面高度小於垂直滾動視圖高度時。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableScrollViewPanResponder`

設為 true 時，會禁用滾動視圖預設的 JS 手勢響應器，將觸控事件的完全控制權交給子組件。此屬性特別適用於啟用 `snapToInterval` 的情況，因為該模式下滾動行為不符合常規觸控模式。請勿在未啟用 `snapToInterval` 的常規滾動視圖中使用，否則可能導致滾動時出現非預期觸控行為。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `endFillColor` <div class="label android">Android</div>

當滾動視圖空間大於內容區域時，此屬性會用指定顏色填充空白區域，避免設置背景色造成不必要的過度繪製。這是進階優化選項，一般情況下不需使用。

| Type            |
| --------------- |
| [color](colors) |

---

### `fadingEdgeLength` <div class="label android">Android</div>

使滾動內容邊緣產生漸隱效果。

If the value is greater than `0`, the fading edges will be set accordingly to the current scroll direction and position, indicating if there is more content to show.

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `horizontal`

When `true`, the scroll view's children are arranged horizontally in a row instead of vertically in a column.

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

控制黏性標頭是否固定在滾動視圖底部而非頂部，通常與反向滾動視圖（inverted ScrollView）搭配使用。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `keyboardDismissMode`

決定拖曳行為是否觸發鍵盤收起。

- `'none'` 拖曳時不收起鍵盤
- `'on-drag'` 開始拖曳時立即收起鍵盤

**僅限 iOS**

- `'interactive'` 鍵盤會隨拖曳互動式收起，向上拖曳可取消收起動作。Android 不支援此模式，行為將與 `'none'` 相同。

| Type                                                                                                                                                            | Default  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| enum(`'none'`, `'on-drag'`) <div className="label android">Android</div><hr />enum(`'none'`, `'on-drag'`, `'interactive'`) <div className="label ios">iOS</div> | `'none'` |

---

### `keyboardShouldPersistTaps`

決定點擊後鍵盤是否保持顯示狀態。

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

設定後，滾動視圖會調整滾動位置，使當前可見且位於或超過 `minIndexForVisible` 的第一個子元件保持位置不變。這適用於雙向載入內容的列表（例如聊天串），避免新訊息進入時導致滾動位置跳動。常見值為 0，但也可設為 1 以跳過不應固定位置的載入指示器或其他內容。

可選參數 `autoscrollToTopThreshold` 可在調整後，若使用者原本位於頂部閾值範圍內，則自動將內容滾動至頂部。這同樣適用於聊天類應用，希望新訊息平滑載入，但避免使用者已滾動較遠時造成干擾性跳動。

注意事項 1：啟用此功能時重新排序滾動視圖內的元件可能導致跳動問題。此問題可修復，但目前無相關計畫。現階段請勿對使用此功能的滾動視圖或列表重新排序內容。

注意事項 2：此功能透過原生代碼中的 `contentOffset` 和 `frame.origin` 計算可見性。遮擋、變形等複雜因素不會納入內容是否「可見」的判斷。

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

為 Android API 21+ 啟用嵌套滾動功能。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onContentSizeChange`

當 ScrollView 的可滾動內容視圖尺寸變化時呼叫。

處理函式會接收兩個參數：內容寬度與內容高度 `(contentWidth, contentHeight)`。

其實作方式是透過附加到 ScrollView 渲染的內容容器上的 onLayout 處理器。

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

滾動期間每幀最多觸發一次。事件頻率可透過 `scrollEventThrottle` 屬性控制。事件格式如下（所有值均為數字）：

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

- `'auto'` - 僅當內容足夠大時，才允許用戶對此視圖進行過度滾動。
- `'always'` - 始終允許用戶對此視圖進行過度滾動。
- `'never'` - 永不允許用戶對此視圖進行過度滾動。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'always'`, `'never'`) | `'auto'` |

---

### `pagingEnabled`

當設為 true 時，滾動視圖會在滾動時停在視圖大小的倍數位置。這可用於實現水平分頁。

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

當設為 true 時，ScrollView 允許使用捏合手勢進行縮放。

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

當設為 false 時，無法通過觸摸交互滾動視圖。

注意：仍可通過調用 `scrollTo` 方法滾動視圖。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `scrollEventThrottle`

此屬性控制滾動時觸發滾動事件的頻率（以毫秒為單位的時間間隔）。較小的數值能為追蹤滾動位置的代碼提供更高精度，但可能因橋接傳輸的數據量過大而導致滾動性能問題。由於 JS 運行循環與屏幕刷新率同步，設置在 1-16 之間的數值不會有明顯差異。若不需要精確追蹤滾動位置，可設置較高值以限制橋接傳輸的數據量。默認值為 `0`，表示每次視圖滾動時僅發送一次滾動事件。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `scrollIndicatorInsets` <div class="label ios">iOS</div>

滾動指示器與滾動視圖邊緣的內邊距。通常應設置為與 `contentInset` 相同的值。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `scrollPerfTag` <div class="label android">Android</div>

用於記錄此滾動視圖性能的標籤。會強制啟用動量事件（參見 sendMomentumEvents）。預設情況下此功能無效，需實作自訂的原生 FpsListener 才能發揮作用。

| Type   |
| ------ |
| string |

---

### `scrollToOverflowEnabled` <div class="label ios">iOS</div>

設為 `true` 時，可通過程式控制將滾動視圖滾動超出其內容大小。

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

當設置 `snapToInterval` 時，`snapToAlignment` 會定義對齊方式與滾動視圖的關係。

可能的值：

- `'start'` 將對齊左側（水平）或頂部（垂直）。
- `'center'` 將對齊中心。
- `'end'` 將對齊右側（水平）或底部（垂直）。

| Type                                 | Default   |
| ------------------------------------ | --------- |
| enum(`'start'`, `'center'`, `'end'`) | `'start'` |

---

### `snapToEnd`

Use in conjunction with `snapToOffsets`. By default, the end of the list counts as a snap offset. Set `snapToEnd` to false to disable this behavior and allow the list to scroll freely between its end and the last `snapToOffsets` offset.

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToInterval`

設置後，滾動視圖會在 `snapToInterval` 值的倍數處停止。可用於分頁瀏覽小於滾動視圖大小的子項。通常與 `snapToAlignment` 和 `decelerationRate="fast"` 搭配使用。會覆蓋較不靈活的 `pagingEnabled` 屬性。

| Type   |
| ------ |
| number |

---

### `snapToOffsets`

設置後，滾動視圖會在定義的偏移點停止。可用於分頁瀏覽大小不一且小於滾動視圖的子項。通常與 `decelerationRate="fast"` 搭配使用。會覆蓋較不靈活的 `pagingEnabled` 和 `snapToInterval` 屬性。

| Type            |
| --------------- |
| array of number |

---

### `snapToStart`

Use in conjunction with `snapToOffsets`. By default, the beginning of the list counts as a snap offset. Set `snapToStart` to `false` to disable this behavior and allow the list to scroll freely between its start and the first `snapToOffsets` offset.

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `stickyHeaderHiddenOnScroll`

當設為 `true` 時，滾動列表向下時會隱藏黏性標頭，向上滾動時則會固定在列表頂部。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `stickyHeaderIndices`

一個子元素索引陣列，決定哪些子元素在滾動時會固定在螢幕頂部。例如，傳入 `stickyHeaderIndices={[0]}` 會使第一個子元素固定在滾動視圖頂部。也可使用 [x,y,z] 格式讓多個項目在頂部時保持黏性。此屬性不支援與 `horizontal={true}` 同時使用。

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

```tsx
flashScrollIndicators();
```

短暫顯示滾動指示器。

---

### `scrollTo()`

```tsx
scrollTo(
  options?: {x?: number, y?: number, animated?: boolean} | number,
  deprecatedX?: number,
  deprecatedAnimated?: boolean,
);
```

滾動到指定的 x、y 偏移位置，可選擇立即執行或使用平滑動畫。

**範例：**

`scrollTo({x: 0, y: 0, animated: true})`

> 注意：此函數簽名的特殊設計是由於歷史原因，它同時接受獨立參數作為選項物件的替代方案。由於存在歧義（y 在 x 之前），此用法已被棄用，不應繼續使用。

---

### `scrollToEnd()`

```tsx
scrollToEnd(options?: {animated?: boolean});
```

如果是垂直 ScrollView 則滾動到底部；如果是水平 ScrollView 則滾動到右側。

使用 `scrollToEnd({animated: true})` 可啟用平滑動畫滾動，`scrollToEnd({animated: false})` 則為立即滾動。若未傳入選項，`animated` 預設為 `true`。