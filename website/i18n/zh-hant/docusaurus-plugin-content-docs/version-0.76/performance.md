---
id: performance
title: Performance Overview
---

選擇使用 React Native 而非基於 WebView 的工具，一個關鍵原因在於能實現至少每秒 60 幀的流暢度，並為應用程式提供原生外觀與操作體驗。在可行情況下，我們希望 React Native 能自動處理優化，讓開發者能專注於應用邏輯而無需擔心效能問題。然而，目前仍有部分領域尚未達到此理想狀態，也有些情況 React Native（如同直接編寫原生代碼）無法自動為您決定最佳優化方式。此時便需要手動介入調整。我們預設會提供如絲般順滑的 UI 效能表現，但某些情況下可能無法完全實現。

本指南旨在傳授基礎知識，協助您[排查效能問題](profiling.md)，同時探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display at least 60 frames per second, which gives you and the UI system at most 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted time slot, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#opening-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JavaScript 幀率（JavaScript 執行緒）

多數 React Native 應用的業務邏輯都運行於 JavaScript 執行緒。這裡是 React 應用核心所在，包含 API 呼叫、觸控事件處理等... 對原生視圖的更新會批量處理，並在事件循環每次迭代結束時（理想情況下於幀截止時間前）發送至原生端。若 JavaScript 執行緒在單幀期間無響應，就會被視為掉幀。例如，當您在複雜應用的根元件呼叫 `this.setState` 導致需要重新渲染計算密集的子元件樹時，可能耗費 200 毫秒並造成 12 幀丟失。此時所有由 JavaScript 控制的動畫都會出現凍結現象。任何超過 100 毫秒的延遲都會被使用者明顯感知。

這種情況常見於 `Navigator` 轉場動畫：當推送新路由時，JavaScript 執行緒需渲染場景所需的所有元件，才能向原生端發送正確指令來建立對應視圖。此過程常會耗費數幀時間，由於轉場由 JavaScript 執行緒控制，會導致[畫面卡頓](https://jankfree.org/)。有時元件在 `componentDidMount` 執行額外工作，可能造成轉場過程中再次出現停頓。

另一個範例是觸控回應：若 JavaScript 執行緒持續進行跨越多幀的作業，您可能會注意到 `TouchableOpacity` 等元件對觸控的反應延遲。這是因為 JavaScript 執行緒忙碌而無法及時處理從主執行緒傳送的原始觸控事件，導致 `TouchableOpacity` 無法即時調整原生視圖的透明度。

### UI 幀率（主執行緒）

許多人注意到 `NavigatorIOS` 的預設效能優於 `Navigator`，關鍵原因在於其轉場動畫完全在主執行緒執行，因此不會受 JavaScript 執行緒掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發到 JS 執行緒，但滾動行為的發生並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[正式版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在打包應用程式時，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [Babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

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

即使專案中沒有使用 `console.*` 呼叫，仍建議使用此插件，因為第三方函式庫也可能呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件重新渲染的條件。若編寫純元件（渲染函式的返回值完全依賴於 props 和 state），可透過 PureComponent 自動處理。再次強調，不可變資料結構有助於保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場過慢」是最常見的表現，但其他情況也可能發生。使用 InteractionManager 是良好的解決方案，但若延遲動畫期間的工作對使用者體驗影響過大，則可考慮改用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格，除非您[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於模態視窗動畫（從頂部滑下並淡入半透明遮罩），同時初始化多個網路請求、渲染模態內容並更新開啟模態的原始視圖。詳見動畫指南以獲取更多 LayoutAnimation 的使用資訊。

注意事項：

- LayoutAnimation 僅適用於一次性動畫（「靜態」動畫）——如果需要中斷動畫，則需要使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生在文字帶有透明背景並疊加在圖片上，或任何需要在每一幀重新繪製視圖的 alpha 合成場景。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 可以顯著改善此問題。

請謹慎使用這些屬性，否則記憶體使用量可能會暴增。在使用這些屬性時，請務必分析效能和記憶體使用情況。如果不再需要移動視圖，請關閉此屬性。

### 調整圖片大小動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，系統會從原始圖片重新裁剪和縮放。這對於大圖來說非常耗資源。建議改用 `transform: [{scale}]` 樣式屬性來實現大小動畫。例如，點擊圖片後將其放大至全螢幕的場景。

### 我的 TouchableX 視圖反應遲鈍

有時，如果我們在調整觸控回應元件的透明度或高亮效果的同一幀中執行動作，這些效果將在 `onPress` 函數返回後才會顯示。如果 `onPress` 中的 `setState` 導致大量工作並丟失幾幀，就可能發生這種情況。解決方法是將 `onPress` 處理程序中的任何動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器過場動畫緩慢

如前所述，`Navigator` 的動畫由 JavaScript 執行緒控制。以「從右側推入」的場景過場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）移動到左側，最終停在 x 偏移為 0 的位置。在此過場的每一幀中，JavaScript 執行緒需要將新的 x 偏移發送到主執行緒。如果 JavaScript 執行緒被鎖定，就無法完成此操作，導致該幀沒有更新，動畫就會卡頓。

解決方法之一是將基於 JavaScript 的動畫卸載到主執行緒。如果採用這種方式處理上述範例，我們可能會在開始過場時計算新場景的所有 x 偏移值，並將其發送到主執行緒以優化的方式執行。這樣 JavaScript 執行緒就無需承擔此責任，即使在渲染場景時丟失幾幀也無關緊要——你可能根本不會注意到，因為漂亮的過場效果會吸引你的注意力。

解決此問題是新版 [React Navigation](navigation.md) 庫的主要目標之一。React Navigation 中的視圖使用原生元件和 [`Animated`](animated.md) 庫，以確保動畫在原生執行緒上運行，並達到至少 60 FPS 的流暢效果。