---
id: performance
title: Performance Overview
---

相較於基於 WebView 的工具，使用 React Native 的一個重要原因是為了達到每秒 60 幀的流暢度，並為應用提供原生的外觀和體驗。在可行情況下，我們希望 React Native 能自動處理優化，讓開發者能專注於應用本身，無需擔心效能問題。然而，目前仍有部分領域尚未達到此目標，還有一些情況 React Native（與直接編寫原生代碼類似）無法自動為您選擇最佳優化方案。此時就需要手動介入。我們致力於預設提供絲般順滑的 UI 效能，但有時可能無法完全實現。

本指南旨在教授一些基礎知識，幫助您[排查效能問題](profiling.md)，並討論[常見問題來源及其建議解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的基礎知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#accessing-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JS 幀率（JavaScript 線程）

對於大多數 React Native 應用，業務邏輯運行在 JavaScript 線程上。這裡是 React 應用的核心所在：API 調用、觸控事件處理等均在此執行。對原生視圖的更新會被批量處理，並在事件循環每次迭代結束時（若一切順利）在幀截止期限前發送到原生端。如果 JavaScript 線程在某一幀期間無響應，就會被視為掉幀。例如，在複雜應用的根組件上調用 `this.setState` 可能導致計算密集型子樹重新渲染，假設這耗時 200 毫秒，則會造成 12 幀丟失。這段時間內所有由 JavaScript 控制的動畫都會出現凍結。任何超過 100 毫秒的操作都會被使用者感知。

這種情況常發生於 `Navigator` 轉場時：當推送新路由時，JavaScript 線程需要渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程的工作量通常會耗費數幀時間，導致[卡頓](https://jankfree.org/)，因為轉場由 JavaScript 線程控制。有時組件會在 `componentDidMount` 中執行額外工作，可能造成轉場過程中的二次卡頓。

另一個例子是觸控響應：如果在 JavaScript 線程上執行跨越多幀的工作，您可能會注意到 `TouchableOpacity` 等組件的響應延遲。這是因為 JavaScript 線程繁忙，無法處理從主線程發送的原始觸控事件。結果導致 `TouchableOpacity` 無法響應觸控事件並通知原生視圖調整透明度。

### UI 幀率（主線程）

許多人注意到 `NavigatorIOS` 的預設效能優於 `Navigator`。這是因為其轉場動畫完全在主線程執行，因此不會受 JavaScript 線程掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發至 JS 執行緒，但滾動動作的執行並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下執行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他各種斷言），運行時需要進行更多工作。請務必在[正式版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案正式（生產）版本中的所有 `console.*` 呼叫。

即使專案中沒有使用 `console.*` 呼叫，仍建議使用此插件。因為第三方函式庫也可能會呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖時 JS FPS 驟降

若使用 ListView，您必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。若編寫純元件（渲染函式的返回值完全依賴於 props 和 state），可透過 PureComponent 自動處理。再次強調，不可變資料結構有助於保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場過慢」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是不錯的解決方案，但若延遲動畫期間的工作對使用者體驗影響過大，則可考慮改用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術應用於模態視窗的動畫（從頂部滑下並淡入半透明遮罩），同時初始化多個網路請求、渲染模態內容及更新開啟模態的原始視圖。詳見動畫指南以獲取更多關於 LayoutAnimation 的使用資訊。

注意事項：

- LayoutAnimation 僅適用於「一次性觸發即忘」的動畫（「靜態」動畫）——若需中斷動畫，則必須使用 `Animated`。

### 在畫面上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

此情況尤其容易發生於文字帶有透明背景並疊加在圖片上方時，或任何需要透過 alpha 合成逐幀重繪視圖的情境。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可顯著改善此問題。

請注意避免過度使用這些屬性，否則記憶體用量可能暴增。使用時務必分析效能與記憶體用量。若視圖不再需要移動，請關閉此屬性。

### 動態調整圖片尺寸導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，系統會從原始圖片重新裁剪與縮放。這對大型圖片尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片放大至全螢幕的場景即適用此技巧。

### TouchableX 元件觸控反應遲鈍

有時若在調整觸控回應元件的透明度或高亮效果的同一幀中執行操作，這些視覺效果會等到 `onPress` 函式執行完畢後才會顯現。若 `onPress` 觸發的 `setState` 導致大量運算而掉幀，便可能發生此狀況。解決方法是將 `onPress` 處理程序內的操作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導覽器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀中，新場景需從起始的螢幕外位置（假設 x 軸偏移 320）移動至 x 軸偏移 0 的目標位置。此過程中，JavaScript 執行緒需逐幀將新偏移值傳送給主執行緒。若 JavaScript 執行緒阻塞，動畫將因無法更新而卡頓。

解決方案之一是將 JavaScript 動畫卸載至主執行緒處理。以前述場景為例，我們可在轉場開始時預先計算所有 x 軸偏移值序列，並交由主執行緒以優化方式執行。如此釋放 JavaScript 執行緒後，即便場景渲染過程掉幀也影響甚微——流暢的轉場動畫將讓使用者忽略輕微延遲。

Solving this is one of the main goals behind the new [React Navigation](navigation.md) library. The views in React Navigation use native components and the [`Animated`](animated.md) library to deliver 60 FPS animations that are run on the native thread.