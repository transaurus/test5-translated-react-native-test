---
id: performance
title: Performance Overview
---

相較於基於 WebView 的工具，使用 React Native 的一個重要原因是能實現每秒 60 幀的流暢度，並為應用提供原生外觀與操作體驗。在可行情況下，我們希望 React Native 能自動處理優化，讓開發者能專注於應用功能而無需擔心效能問題。然而，目前仍有部分領域尚未達到此目標，還有一些情況 React Native（如同直接編寫原生代碼）無法自動為您決定最佳優化方案。此時便需要手動介入調整。我們致力於預設提供絲滑般的 UI 效能表現，但某些情況下可能仍無法完全實現。

本指南旨在傳授一些基礎知識，幫助您[診斷效能問題](profiling.md)，同時探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於影格的基本知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#accessing-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JavaScript 執行緒幀率

多數 React Native 應用的業務邏輯都在 JavaScript 執行緒上運行——這裡是 React 應用的核心所在，包含 API 呼叫、觸控事件處理等邏輯。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（若一切順利）趕在影格截止期限前發送至原生端。若 JavaScript 執行緒在某個影格期間無響應，就會被視為掉幀。例如，當您在複雜應用的根組件呼叫 `this.setState` 導致需要重新渲染計算密集的子組件樹時，可能耗費 200 毫秒並導致連續掉幀 12 次。此時所有由 JavaScript 控制的動畫都會出現凍結現象。任何超過 100 毫秒的延遲都會被使用者明顯感知。

這種情況常見於 `Navigator` 轉場時：當推送新路由時，JavaScript 執行緒需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常會耗費數個影格時間，導致由 JavaScript 執行緒控制的轉場動畫出現[卡頓](https://jankfree.org/)。有時組件在 `componentDidMount` 中執行額外工作，可能造成轉場過程中二次卡頓。

另一個典型例子是觸控響應：若 JavaScript 執行緒正在處理跨越多個影格的任務，您可能會注意到 `TouchableOpacity` 等組件的響應延遲。這是因為 JavaScript 執行緒繁忙而無法及時處理從主執行緒傳來的原始觸控事件，導致 `TouchableOpacity` 無法即時反應觸控動作並通知原生視圖調整透明度。

### 主執行緒幀率（UI 幀率）

許多人注意到 `NavigatorIOS` 的預設效能表現優於 `Navigator`，關鍵原因在於其轉場動畫完全在主執行緒執行，因此不會受 JavaScript 執行緒掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，你仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發至 JS 執行緒，但滾動行為的發生並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供完善的警告與錯誤訊息，運行時需執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能表現。

### 使用 `console.log` 語句

當運行打包後的應用時，這些語句會造成 JavaScript 執行緒的嚴重瓶頸。這包含來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，請確保在打包前移除它們。你也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。需先透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案正式版（生產環境）中的所有 `console.*` 呼叫。

即使專案中未使用 `console.*` 呼叫，仍建議使用此插件。第三方函式庫也可能呼叫這些方法。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具備顯著的效能提升，主要優勢在於無論行數多少，記憶體使用量幾乎保持恆定。

若你的 [`FlatList`](flatlist.md) 渲染速度緩慢，請確認已實作 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來大幅減少工作量。若採用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，可實作 `shouldComponentUpdate` 並明確指定元件需重新渲染的條件。若編寫純元件（渲染函式的返回值完全依賴於 props 和 state），可利用 PureComponent 自動處理。再次強調，不可變資料結構有助於保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場動畫卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是良好的解決方案，但若延遲動畫期間的工作會對使用者體驗造成過大影響，則可考慮採用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格，除非你[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，而 LayoutAnimation 則利用 Core Animation 實現，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於模態視窗的動畫（從頂部滑下並淡入半透明遮罩），同時初始化並可能接收多個網路請求的回應、渲染模態內容，以及更新開啟模態的原始視圖。詳見動畫指南以獲取更多關於 LayoutAnimation 的使用資訊。

注意事項：

- LayoutAnimation 僅適用於「一鍵觸發即忘」的動畫（「靜態」動畫）——若需中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

當文字帶有透明背景並疊加在圖片上方時尤為明顯，或任何需要逐幀重新繪製視圖的透明合成情境。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可顯著改善此問題。

注意避免過度使用，否則記憶體用量可能暴增。使用這些屬性時請務必分析效能與記憶體用量。若視圖不再需要移動，請關閉此屬性。

### 調整圖片尺寸動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬高時，系統會從原始圖片重新裁剪和縮放。對於大圖而言效能消耗極大。建議改用 `transform: [{scale}]` 樣式屬性實現尺寸動畫。典型應用場景如點擊圖片後放大至全螢幕。

### TouchableX 元件觸控反應遲鈍

若在調整觸控元件透明度或高亮效果的同一幀中執行操作，視覺效果將延遲至 `onPress` 函數執行完畢後才顯現。當 `onPress` 觸發的 `setState` 導致大量運算而掉幀時，此現象更為明顯。解決方案是將 `onPress` 處理函數內的操作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀需將新場景從起始屏外位置（假設 x 軸偏移 320）移動至目標位置（x 軸偏移 0）。若 JavaScript 執行緒被阻塞，無法即時傳送新偏移值至主執行緒，將導致動畫掉幀。

解決方案之一是將 JavaScript 動畫卸載至主執行緒執行。以前述場景為例，可在轉場開始時預先計算所有 x 軸偏移序列，並交由主執行緒以優化方式執行。釋放 JavaScript 執行緒後，即使渲染場景時偶爾掉幀，流暢的轉場動畫仍能維持良好使用者體驗。

Solving this is one of the main goals behind the new [React Navigation](navigation.md) library. The views in React Navigation use native components and the [`Animated`](animated.md) library to deliver 60 FPS animations that are run on the native thread.