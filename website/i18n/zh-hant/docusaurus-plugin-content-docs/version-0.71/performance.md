---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個重要原因是為了實現每秒60幀的流暢度以及應用程式的原生外觀與操作體驗。在可能的情況下，我們希望React Native能夠自動處理好這些細節，讓開發者能專注於應用功能而非效能優化。然而，目前仍有部分領域尚未達到理想狀態，還有一些情況（如同直接編寫原生程式碼時）React Native無法自動判斷最佳優化方式，此時就需要手動介入調整。我們盡力預設提供如絲般順滑的UI效能，但有時這並不可行。

本指南旨在傳授一些基礎知識，幫助您[排查效能問題](profiling.md)，並討論[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the developer menu in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

多數React Native應用的業務邏輯都在JavaScript執行緒上運行——這裡是React組件運作、API呼叫執行、觸控事件處理的地方。對原生視圖的更新會被批量處理，並在事件循環每次迭代結束時（若一切順利）趕在幀截止期限前發送到原生端。若JavaScript執行緒在某一幀期間無響應，就會被視為掉幀。例如，當您在複雜應用的根組件上呼叫`this.setState`，導致需要重新渲染計算量龐大的子組件樹時，可能會耗費200毫秒，造成12幀丟失。此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常發生在`Navigator`轉場時：當推送新路由時，JavaScript執行緒需要渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程通常會耗費數幀時間，導致[卡頓](http://jankfree.org/)，因為轉場動畫是由JavaScript執行緒控制的。有時組件會在`componentDidMount`中執行額外工作，可能造成轉場過程出現二次卡頓。

另一個典型例子是觸控響應：若JavaScript執行緒正在處理跨越多幀的任務，您可能會注意到`TouchableOpacity`等組件的響應延遲。這是因為JavaScript執行緒繁忙而無法及時處理從主執行緒傳來的原始觸控事件，導致`TouchableOpacity`無法立即響應觸控並通知原生視圖調整透明度。

### UI幀率（主執行緒）

許多開發者注意到`NavigatorIOS`的預設效能優於`Navigator`，關鍵原因在於前者將轉場動畫完全放在主執行緒處理，因此不會受JavaScript執行緒掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 位於主執行緒上。滾動事件會被分派到 JS 執行緒，但滾動的發生並不依賴於這些事件的接收。

## 常見效能問題來源

### 在開發模式下運行 (`dev=true`)

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他各種斷言），運行時需要做更多工作。請務必在[發佈版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自調試庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的調用，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 調用。首先需要安裝它：`npm i babel-plugin-transform-remove-console --save-dev`，然後編輯專案目錄下的 `.babelrc` 文件如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案發佈（生產）版本中的所有 `console.*` 調用。

即使專案中沒有 `console.*` 調用，也建議使用此插件。第三方庫也可能會調用它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 組件。除了簡化 API 外，新的列表組件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持不變。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保已實作 [`getItemLayout`](flatlist.md#getitemlayout)，通過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時 JS FPS 驟降

如果使用 ListView，您必須提供 `rowHasChanged` 函數，該函數可以通過快速判斷行是否需要重新渲染來減少大量工作。如果使用不可變數據結構，這只需要進行引用相等性檢查。

同樣地，您可以實作 `shouldComponentUpdate` 並指定組件重新渲染的確切條件。如果編寫純組件（渲染函數的返回值完全依賴於 props 和 state），可以利用 PureComponent 來實現這一點。再次強調，不可變數據結構有助於保持高效——如果需要對大型對象列表進行深度比較，重新渲染整個組件可能更快，且代碼量更少。

### 由於同時在 JavaScript 執行緒上執行大量工作導致 JS 執行緒 FPS 下降

「Navigator 過渡動畫緩慢」是最常見的表現形式，但其他時候也可能發生這種情況。使用 InteractionManager 是一個不錯的方法，但如果延遲動畫期間的工作對用戶體驗成本太高，則可以考慮使用 LayoutAnimation。

除非您[設置 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，否則 Animated API 目前會在 JavaScript 執行緒上按需計算每個關鍵幀，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀的影響。

我曾使用此方法的一個案例是：在初始化並可能接收多個網絡請求的回應、渲染模態框內容以及更新打開模態框的視圖時，同時動畫顯示模態框（從頂部滑下並淡入半透明遮罩）。有關如何使用 LayoutAnimation 的更多信息，請參閱動畫指南。

注意事項：

- LayoutAnimation 僅適用於「一觸即發」的動畫（「靜態」動畫）——若需中斷動畫，則必須使用 `Animated`。

### 移動畫面中的視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

當文字具有透明背景並疊加在圖片上，或任何需要每幀重新繪製視圖的 alpha 合成情境時，此問題尤其明顯。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可顯著改善效能。

請謹慎使用這些屬性，避免記憶體用量暴增。使用時務必分析效能與記憶體消耗。若視圖不再需要移動，請關閉此屬性。

### 調整圖片尺寸動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，系統會從原始圖片重新裁剪和縮放。對於大圖而言，此操作成本極高。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片後放大至全螢幕的場景。

### TouchableX 元件響應遲鈍

若在調整觸控回應元件的透明度或高亮效果的同一幀中執行操作，可能需等到 `onPress` 函數返回後才會看到效果。若 `onPress` 觸發的 `setState` 導致大量運算和幀數下降，就會發生此狀況。解決方案是將 `onPress` 處理程序內的操作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導覽器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移量為 320）向左移動，最終停駐在 x 偏移量 0 的位置。若 JavaScript 執行緒被阻塞，無法發送新偏移量，就會導致動畫掉幀。

解決方案之一是將 JavaScript 動畫卸載到主執行緒執行。例如預先計算新場景所有 x 偏移量並交由主執行緒優化執行。如此釋放 JavaScript 執行緒後，即使渲染場景時掉幀，流暢的轉場動畫也能轉移用戶注意力。

Solving this is one of the main goals behind the new [React Navigation](navigation.md) library. The views in React Navigation use native components and the [`Animated`](animated.md) library to deliver 60 FPS animations that are run on the native thread.