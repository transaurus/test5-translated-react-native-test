---
id: scrollview
title: ScrollView
---

封裝平台原生 ScrollView 的組件，同時提供與觸控鎖定「響應者」系統的整合功能。

請注意，ScrollView 必須具有固定高度才能正常運作，因為它們將無限高度的子元素裝載到有限容器中（透過滾動互動）。若要限制 ScrollView 的高度，可以直接設定視圖高度（不建議），或確保所有父視圖都具有固定高度。忘記將 `{flex: 1}` 向下傳遞至視圖結構可能導致錯誤，此時元素檢查器能快速協助除錯。

目前尚不支援其他內含的響應者阻止此滾動視圖成為主響應者。

`<ScrollView>` 與 [`<FlatList>`](flatlist.md) 該如何選擇？

`ScrollView` 會一次性渲染所有 React 子組件，但這會帶來效能上的缺點。

假設您需要顯示一個非常長的項目列表，內容可能橫跨多個螢幕。若一次性為所有內容建立 JS 組件和原生視圖（其中許多可能根本不會顯示），將導致渲染速度變慢並增加記憶體使用量。

此時 `FlatList` 便能發揮作用。`FlatList` 會延遲渲染項目（僅在即將顯示時才渲染），並移除遠離螢幕的可見區域的項目以節省記憶體和處理時間。

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

# 參考文獻

## 屬性

### [視圖屬性](view.md#props)

繼承 [視圖屬性](view#props)。

---

### `StickyHeaderComponent`

用於渲染黏性標頭的 React 組件，需與 `stickyHeaderIndices` 搭配使用。若您的黏性標頭使用自訂變形效果（例如希望列表具有可動畫顯示/隱藏的標頭），則可能需要設定此組件。若未提供組件，將使用預設的 [`ScrollViewStickyHeader`](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Components/ScrollView/ScrollViewStickyHeader.js) 組件。

| Type               |
| ------------------ |
| component, element |

---

### `alwaysBounceHorizontal` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，滾動視圖在水平滾動到底部時仍會彈跳。

| Type | Default                                               |
| ---- | ----------------------------------------------------- |
| bool | `true` when `horizontal={true}`<hr/>`false` otherwise |

---

### `alwaysBounceVertical` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，滾動視圖在垂直滾動到底部時仍會彈跳。

| Type | Default                                             |
| ---- | --------------------------------------------------- |
| bool | `false` when `vertical={true}`<hr/>`true` otherwise |

---

### `automaticallyAdjustContentInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整位於導覽列或標籤列/工具列後方的滾動視圖的內容插入邊距。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `automaticallyAdjustKeyboardInsets` <div class="label ios">iOS</div>

控制當鍵盤大小變更時，ScrollView 是否應自動調整其 `contentInset` 和 `scrollViewInsets`。

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

當設為 `true` 時，手勢操作可驅動縮放超出最小/最大值，並在手勢結束時動畫回彈至限制值；否則縮放不會超過限制範圍。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `canCancelContentTouches` <div class="label ios">iOS</div>

設為 `false` 時，一旦開始追蹤觸控，即使觸控點移動也不會嘗試拖曳。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `centerContent` <div class="label ios">iOS</div>

設為 `true` 時，當內容尺寸小於滾動視圖邊界時會自動置中；若內容大於滾動視圖則此屬性無效。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `contentContainerStyle`

這些樣式將套用至包裹所有子視圖的滾動視圖內容容器。範例：

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

滾動視圖內容與其邊緣的內縮距離。

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

用於手動設定起始滾動偏移量。

| Type  | Default        |
| ----- | -------------- |
| Point | `{x: 0, y: 0}` |

---

### `decelerationRate`

此浮點數決定用戶鬆開手指後滾動視圖的減速速度。亦可使用字串快捷參數 `"normal"` 和 `"fast"`，分別對應 iOS 底層設定 `UIScrollViewDecelerationRateNormal` 和 `UIScrollViewDecelerationRateFast`。

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

When true, the scroll view stops on the next index (in relation to scroll position at release) regardless of how fast the gesture is. This can be used for pagination when the page is less than the width of the horizontal ScrollView or the height of the vertical ScrollView.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableScrollViewPanResponder`

When true, the default JS pan responder on the ScrollView is disabled, and full control over touches inside the ScrollView is left to its child components. This is particularly useful if `snapToInterval` is enabled, since it does not follow typical touch patterns. Do not use this on regular ScrollView use cases without `snapToInterval` as it may cause unexpected touches to occur while scrolling.

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `endFillColor` <div class="label android">Android</div>

當滾動視圖佔用的空間大於內容實際填充的範圍時，此屬性會以指定顏色填充剩餘區域，避免設定背景色而導致不必要的過度繪製。這是一項進階優化功能，通常情況下不需要使用。

| Type            |
| --------------- |
| [color](colors) |

---

### `fadingEdgeLength` <div class="label android">Android</div>

淡化滾動內容的邊緣。

若值大於 `0`，會根據當前滾動方向與位置設定淡出邊緣效果，提示使用者還有更多內容可查看。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `horizontal`

設為 `true` 時，滾動視圖的子元件會以水平排列（橫向）而非垂直排列（縱向）。

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

設定黏性標頭是否應固定在滾動視圖底部而非頂部。通常與反向滾動視圖（inverted ScrollViews）搭配使用。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `keyboardDismissMode`

決定拖曳行為是否觸發鍵盤收起。

- `'none'`，拖曳時不會收起鍵盤。
- `'on-drag'`，開始拖曳時立即收起鍵盤。

**僅限 iOS**

- `'interactive'`，鍵盤會隨拖曳手勢互動式收起，並與觸控點同步移動，向上拖曳可取消收起動作。Android 不支援此模式，行為將與 `'none'` 相同。

| Type                                                                                                                                                            | Default  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| enum(`'none'`, `'on-drag'`) <div className="label android">Android</div><hr />enum(`'none'`, `'on-drag'`, `'interactive'`) <div className="label ios">iOS</div> | `'none'` |

---

### `keyboardShouldPersistTaps`

決定點擊後鍵盤是否應保持顯示。

- `'never'` tapping outside of the focused text input when the keyboard is up dismisses the keyboard. When this happens, children won't receive the tap.
- `'always'`, the keyboard will not dismiss automatically, and the scroll view will not catch taps, but children of the scroll view can catch taps.
- `'handled'`, the keyboard will not dismiss automatically when the tap was handled by children of the scroll view (or captured by an ancestor).
- `false`, **_deprecated_**, use `'never'` instead
- `true`, **_deprecated_**, use `'always'` instead

| Type                                                      | Default   |
| --------------------------------------------------------- | --------- |
| enum(`'always'`, `'never'`, `'handled'`, `false`, `true`) | `'never'` |

---

### `maintainVisibleContentPosition`

設定此屬性後，滾動視圖會調整滾動位置，使當前可見且位於或超過 `minIndexForVisible` 的第一個子元件保持位置不變。這適用於雙向載入內容的列表（例如聊天串），否則新訊息的加入可能導致滾動位置跳動。常見值為 0，但也可使用 1 等值來跳過不應保持位置的載入指示器或其他內容。

可選參數 `autoscrollToTopThreshold` 可在調整後，若使用者原本位於頂部閾值範圍內，則自動將內容滾動至頂部。這同樣適用於聊天類應用，希望新訊息能平滑滾入視圖，但若使用者已向上滾動一段距離，則避免造成干擾性的大幅滾動。

注意事項 1：啟用此功能時若重新排序滾動視圖中的元件，可能會導致跳動或卡頓。此問題可修復，但目前無相關計畫。現階段請勿對使用此功能的滾動視圖或列表重新排序內容。

注意事項 2：此功能在原生代碼中使用 `contentOffset` 和 `frame.origin` 計算可見性。遮擋、變形或其他複雜因素不會被納入判斷內容是否「可見」的考量。

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

為 Android API 21+ 啟用嵌套滾動。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onContentSizeChange`

當 ScrollView 的可滾動內容視圖大小變更時呼叫。

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

滾動期間每幀最多觸發一次。事件格式如下（所有值均為數字）：

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

當用戶開始拖動滾動視圖時調用。

| Type     |
| -------- |
| function |

---

### `onScrollEndDrag`

當用戶停止拖動滾動視圖且滾動視圖停止或開始滑動時調用。

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

用於覆寫預設的過度滾動模式值。

可能的值：

- `'auto'` - 僅當內容足夠大時，才允許用戶對此視圖進行過度滾動。
- `'always'` - 始終允許用戶對此視圖進行過度滾動。
- `'never'` - 永不允許用戶對此視圖進行過度滾動。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'always'`, `'never'`) | `'auto'` |

---

### `pagingEnabled`

當設為 true 時，滾動視圖會在滾動時停在視圖大小的倍數位置。這可用於實現水平分頁效果。

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

當設為 true 時，ScrollView 允許使用捏合手勢來縮放內容。

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

當設為 false 時，無法透過觸摸交互滾動視圖。

請注意，仍可透過調用 `scrollTo` 方法滾動視圖。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `scrollEventThrottle`

限制滾動事件觸發的頻率，以毫秒為單位指定時間間隔。當滾動時需執行耗費資源的操作時，此屬性可能很有用。值 ≤ `16` 會停用節流，無論設備的刷新率為何。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `scrollIndicatorInsets` <div class="label ios">iOS</div>

滾動指示器與滾動視圖邊緣的內嵌距離。通常應設為與 `contentInset` 相同的值。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `scrollPerfTag` <div class="label android">Android</div>

用於記錄此滾動視圖滾動性能的標籤。將強制啟用動量事件（參見sendMomentumEvents）。此功能預設不生效，需實現自定義的原生FpsListener才能發揮作用。

| Type   |
| ------ |
| string |

---

### `scrollToOverflowEnabled` <div class="label ios">iOS</div>

當設為`true`時，可通過程式控制將滾動視圖滾動至超出其內容尺寸的位置。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `scrollsToTop` <div class="label ios">iOS</div>

當設為`true`時，點擊狀態列會使滾動視圖滾動至頂部。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsHorizontalScrollIndicator`

當設為`true`時，顯示水平滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsVerticalScrollIndicator`

當設為`true`時，顯示垂直滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToAlignment`

當設置`snapToInterval`時，`snapToAlignment`將定義對齊方式與滾動視圖的關係。

可能的值：

- `'start'` 將對齊至左側（水平）或頂部（垂直）。
- `'center'` 將對齊至中心。
- `'end'` 將對齊至右側（水平）或底部（垂直）。

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

設置後，滾動視圖將在`snapToInterval`值的倍數處停止。可用於分頁瀏覽尺寸小於滾動視圖的子項。通常與`snapToAlignment`和`decelerationRate="fast"`配合使用。會覆蓋配置較少的`pagingEnabled`屬性。

| Type   |
| ------ |
| number |

---

### `snapToOffsets`

設置後，滾動視圖將在定義的偏移點處停止。可用於分頁瀏覽尺寸各異且小於滾動視圖的子項。通常與`decelerationRate="fast"`配合使用。會覆蓋配置較少的`pagingEnabled`和`snapToInterval`屬性。

| Type            |
| --------------- |
| array of number |

---

### `snapToStart`

與`snapToOffsets`配合使用。預設情況下，列表起始處會作為吸附偏移點。將`snapToStart`設為`false`可禁用此行為，允許列表在起始處與第一個`snapToOffsets`偏移點之間自由滾動。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `stickyHeaderHiddenOnScroll`

當設為`true`時，向下滾動列表時黏性標題會隱藏，向上滾動時會固定在列表頂部。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `stickyHeaderIndices`

一個子元素索引陣列，決定哪些子元素在滾動時會固定在螢幕頂部。例如，傳入 `stickyHeaderIndices={[0]}` 會讓第一個子元素固定在滾動視圖頂部。你也可以使用像 [x,y,z] 這樣的格式來讓多個項目在到達頂部時保持固定。此屬性不支援與 `horizontal={true}` 同時使用。

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

滾動到指定的 x、y 偏移位置，可以立即執行或使用平滑動畫。

**範例：**

`scrollTo({x: 0, y: 0, animated: true})`

> 注意：這個奇怪的函數簽名是由於歷史原因，該函數也接受單獨的參數作為選項物件的替代方案。由於存在歧義（y 在 x 之前），這種用法已被棄用，不應再使用。

---

### `scrollToEnd()`

```tsx
scrollToEnd(options?: {animated?: boolean});
```

如果是垂直 ScrollView，則滾動到底部；如果是水平 ScrollView，則滾動到右側。

使用 `scrollToEnd({animated: true})` 進行平滑動畫滾動，使用 `scrollToEnd({animated: false})` 立即滾動。如果未傳入選項，`animated` 預設為 `true`。