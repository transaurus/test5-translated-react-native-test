---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個重要原因是為了達到每秒60幀的流暢度，並為應用提供原生般的視覺與操作體驗。在大多數情況下，我們希望React Native能自動處理優化工作，讓開發者能專注於應用邏輯而無需擔心效能問題。然而，仍存在某些領域尚未實現自動優化，或是像直接編寫原生代碼一樣，React Native無法自動判斷最佳優化方案。此時就需要手動介入調整。我們致力於預設提供如絲般順滑的UI表現，但某些情況下可能仍無法完全達成。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#accessing-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JavaScript幀率（JS線程）

多數React Native應用的業務邏輯都在JavaScript線程執行——這裡是React組件運作、API調用發送、觸控事件處理的所在。對原生視圖的更新會在事件循環的每次迭代結束時批量傳送至原生端（若一切順利）。若JS線程在某幀刷新週期內無響應，就會被視為掉幀。例如當您在複雜應用的根組件調用`this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗費200毫秒並造成12幀丟失，此時所有由JS控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常見於`Navigator`頁面轉場時：當推送新路由時，JS線程需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常會耗費數幀時間，由於轉場動畫由JS線程控制，就會導致[卡頓](https://jankfree.org/)。有時組件在`componentDidMount`中執行額外工作，可能造成轉場過程出現二次停頓。

另一個典型例子是觸控響應：若JS線程持續多幀處於忙碌狀態，您可能會注意到`TouchableOpacity`等組件的觸控反饋出現延遲。這是因為JS線程無法及時處理從主線程傳來的原始觸控事件，導致`TouchableOpacity`無法立即調節原生視圖的透明度。

### UI幀率（主線程）

Many people have noticed that performance of `NavigatorIOS` is better out of the box than `Navigator`. The reason for this is that the animations for the transitions are done entirely on the main thread, and so they are not interrupted by frame drops on the JavaScript thread.

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被分發到 JS 執行緒，但滾動的發生並不依賴於這些事件的接收。

## 常見效能問題來源

### 在開發模式下運行 (`dev=true`)

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他各種斷言），運行時需要執行更多工作。請務必在[發佈版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自調試庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的調用，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 調用。首先需要安裝它：`npm i babel-plugin-transform-remove-console --save-dev`，然後編輯專案目錄下的 `.babelrc` 文件，如下所示：

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

即使您的專案中沒有使用 `console.*` 調用，也建議使用此插件。因為第三方庫也可能會調用它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新的列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保已實作 [`getItemLayout`](flatlist.md#getitemlayout)，通過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時 JS FPS 驟降

如果您使用 ListView，必須提供 `rowHasChanged` 函數，該函數可以通過快速判斷行是否需要重新渲染來減少大量工作。如果使用不可變數據結構，這只需要進行引用相等性檢查。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。如果編寫純元件（渲染函數的返回值完全依賴於 props 和 state），可以利用 PureComponent 來實現這一點。再次強調，不可變數據結構有助於保持高效——如果需要對大型物件列表進行深度比較，重新渲染整個元件可能更快，且代碼量更少。

### 由於同時在 JavaScript 執行緒上執行大量工作導致 JS 執行緒 FPS 下降

「導航器過渡緩慢」是最常見的表現，但其他時候也可能發生這種情況。使用 InteractionManager 是一個不錯的解決方案，但如果延遲動畫期間的工作對用戶體驗影響太大，則可以考慮使用 LayoutAnimation。

目前，Animated API 會在 JavaScript 執行緒上按需計算每個關鍵幀，除非您[設置 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀的影響。

我曾遇到的一個案例是：在初始化並可能接收多個網絡請求的回應、渲染模態框內容以及更新模態框打開的視圖時，同時動畫顯示模態框（從頂部滑下並淡入半透明遮罩）。有關如何使用 LayoutAnimation 的更多信息，請參閱動畫指南。

注意事項：

- LayoutAnimation 僅適用於「一發不可收」的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生在文字帶有透明背景並疊加在圖片上方時，或任何需要在每一幀重新繪製視圖的 alpha 合成場景。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可以顯著改善此問題。

請謹慎使用這些屬性，否則記憶體用量可能會暴增。在使用這些屬性時，請務必分析效能和記憶體用量。如果不再需要移動視圖，請關閉這些屬性。

### 動畫調整圖片大小導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對大型圖片來說非常耗資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如，點擊圖片後將其放大至全螢幕的場景。

### 我的 TouchableX 視圖反應遲鈍

有時，如果我們在調整回應觸控的元件透明度或高亮效果的同一幀中執行動作，則在 `onPress` 函數返回之前不會看到效果。如果 `onPress` 觸發的 `setState` 導致大量工作並丟失幾幀，就可能發生這種情況。解決方法是將 `onPress` 處理程序內的所有動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫緩慢

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」的場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）移動到左側，最終停在 x 偏移為 0 的位置。在此轉場的每一幀，JavaScript 執行緒都需要向主執行緒發送新的 x 偏移值。如果 JavaScript 執行緒被鎖定，就無法完成此操作，導致該幀沒有更新，動畫就會卡頓。

一種解決方案是將基於 JavaScript 的動畫卸載到主執行緒。以上述例子為例，我們可以在開始轉場時計算新場景的所有 x 偏移值列表，並將其發送到主執行緒以優化方式執行。這樣 JavaScript 執行緒就無需承擔此責任，即使在渲染場景時丟失幾幀也無關緊要——你可能甚至不會注意到，因為漂亮的轉場動畫會吸引你的注意力。

解決此問題是新版 [React Navigation](navigation.md) 庫的主要目標之一。React Navigation 中的視圖使用原生元件和 [`Animated`](animated.md) 庫，以在主執行緒上實現 60 FPS 的流暢動畫。