---
id: performance
title: Performance Overview
---

選擇使用 React Native 而非基於 WebView 的工具，一個關鍵原因在於它能實現每秒 60 幀的流暢度與原生應用般的外觀與操作體驗。理想情況下，我們希望 React Native 能自動處理好效能優化，讓開發者專注於應用邏輯。但某些領域仍需手動介入——有些是框架尚未完善的環節，有些則如同直接編寫原生代碼般，無法自動判斷最佳優化方式。我們盡力預設提供如絲般順滑的 UI 效能，但這並非總是可行。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源與對應解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#accessing-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JavaScript 幀率（JavaScript 執行緒）

多數 React Native 應用的商業邏輯運行於 JavaScript 執行緒——這裡處理 React 組件生命週期、API 呼叫、觸控事件等。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下於幀截止期限前）發送至原生端。若 JavaScript 執行緒在某幀期間無響應，即視為掉幀。例如：當您在複雜應用的根組件呼叫 `this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗費 200 毫秒而導致丟失 12 幀，此時所有由 JavaScript 控制的動畫都會凍結。任何超過 100 毫秒的延遲都會被使用者感知。

此現象常見於 `Navigator` 轉場時：推送新路由後，JavaScript 執行緒需渲染場景所需的所有組件，才能向原生端發送建立對應視圖的指令。此過程常耗費數幀時間，造成[卡頓](http://jankfree.org/)，因為轉場動畫由 JavaScript 執行緒控制。有時組件在 `componentDidMount` 執行額外工作，可能導致轉場過程中出現二次卡頓。

另一個範例是觸控回應：若 JavaScript 執行緒持續多幀處理任務，您可能注意到 `TouchableOpacity` 的反應延遲。這是因為 JavaScript 執行緒忙碌而無法處理從主執行緒傳來的原始觸控事件，導致 `TouchableOpacity` 無法即時調控原生視圖的透明度。

### UI 幀率（主執行緒）

許多人注意到 `NavigatorIOS` 的預設效能優於 `Navigator`，關鍵在於其轉場動畫完全在主執行緒執行，因此不會受 JavaScript 執行緒掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被分發到 JS 執行緒，但滾動的發生並不依賴於這些事件的接收。

## 常見的效能問題來源

### 在開發模式下運行 (`dev=true`)

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他各種斷言），運行時需要做更多工作。請務必在[發行版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重的瓶頸。這包括來自調試庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的調用，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 調用。首先需要通過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝它，然後編輯項目目錄下的 `.babelrc` 文件，如下所示：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除項目發行（生產）版本中的所有 `console.*` 調用。

即使您的項目中沒有 `console.*` 調用，也建議使用此插件。第三方庫也可能會調用它們。

### `ListView` 初始渲染過慢或大型列表的滾動效能不佳

改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 組件。除了簡化 API 外，新的列表組件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保已實現 [`getItemLayout`](flatlist.md#getitemlayout)，通過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時，JS FPS 驟降

如果您使用 ListView，必須提供一個 `rowHasChanged` 函數，該函數可以通過快速判斷行是否需要重新渲染來減少大量工作。如果使用不可變數據結構，這可能只需要進行引用相等性檢查。

同樣地，您可以實現 `shouldComponentUpdate` 並指定您希望組件重新渲染的具體條件。如果您編寫純組件（渲染函數的返回值完全依賴於 props 和 state），可以利用 PureComponent 來為您完成這項工作。再次強調，不可變數據結構對於保持高效非常有用——如果您需要對大型對象列表進行深度比較，重新渲染整個組件可能會更快，而且無疑需要更少的代碼。

### 由於在 JavaScript 執行緒上同時執行大量工作而導致 JS 執行緒 FPS 下降

「導航器過渡緩慢」是最常見的表現，但其他時候也可能發生這種情況。使用 InteractionManager 可能是一個好方法，但如果延遲動畫期間的工作對用戶體驗影響太大，則可以考慮使用 LayoutAnimation。

除非您[設置 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，否則 Animated API 目前會在 JavaScript 執行緒上按需計算每個關鍵幀，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀的影響。

我曾使用這種方法的一個案例是：在初始化並可能接收多個網絡請求的回應、渲染模態框的內容以及更新打開模態框的視圖時，同時動畫顯示模態框（從頂部滑下並淡入半透明覆蓋層）。有關如何使用 LayoutAnimation 的更多信息，請參閱動畫指南。

注意事項：

- LayoutAnimation 僅適用於「一鍵觸發」的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生在文字帶有透明背景並疊加在圖片上方時，或任何需要在每一幀重新繪製視圖並進行 alpha 合成的場景。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可以顯著改善此問題。

但需謹慎使用，避免過度消耗記憶體。在使用這些屬性時，請務必分析效能和記憶體使用情況。如果不再需要移動視圖，請關閉此屬性。

### 調整圖片大小動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，系統會從原始圖片重新裁剪和縮放。這對大型圖片尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如，點擊圖片並將其放大至全螢幕時可使用此方法。

### TouchableX 視圖的響應速度不佳

有時，如果在同一幀中同時執行操作並調整回應觸控的元件透明度或高亮效果，這些效果可能要到 `onPress` 函數執行完畢後才會顯示。如果 `onPress` 觸發的 `setState` 導致大量工作並丟失幾幀，就可能出現此問題。解決方法是將 `onPress` 處理程序內的操作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」的場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）移動至左側，最終停在 x 偏移為 0 的位置。在此過程中，JavaScript 執行緒需將每一幀的新 x 偏移值傳送給主執行緒。若 JavaScript 執行緒卡住，則無法更新幀數，導致動畫卡頓。

解決方案之一是將基於 JavaScript 的動畫卸載到主執行緒處理。以上述例子為例，可以在轉場開始時預先計算新場景的所有 x 偏移值，並將其傳送給主執行緒以優化執行。如此一來，JavaScript 執行緒不再負責此任務，即使渲染場景時丟失幾幀也無妨——因為流暢的轉場動畫會讓用戶忽略這些問題。

新推出的 [React Navigation](navigation.md) 庫正是為解決此問題而設計。React Navigation 的視圖使用原生元件和 [`Animated`](animated.md) 庫，通過原生執行緒實現 60 FPS 的流暢動畫。