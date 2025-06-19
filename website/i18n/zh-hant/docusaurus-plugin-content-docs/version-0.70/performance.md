---
id: performance
title: Performance Overview
---

使用 React Native 而非基於 WebView 的工具，一個關鍵原因在於能實現每秒 60 幀的流暢度與原生應用般的用戶體驗。理想情況下，React Native 會自動處理效能優化，讓您專注於應用開發。但某些領域我們尚未完善，也有些情況（如同直接編寫原生代碼時）需要手動介入才能達到最佳優化。我們預設會盡力提供絲滑般的 UI 效能，但有時仍無法完全避免問題。

本指南旨在傳授基礎知識，幫助您[排查效能問題](profiling.md)，並探討[常見問題根源與解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS devices display 60 frames per second, which gives you and the UI system about 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted 16.67ms, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the developer menu in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![](/docs/assets/PerfUtil.png)

### JS 幀率（JavaScript 線程）

多數 React Native 應用的業務邏輯運行於 JavaScript 線程。這裡處理著 React 組件生命週期、API 調用、觸控事件等邏輯。對原生視圖的更新會批量處理，並在事件循環每次迭代結束時（理想情況下於幀截止時間前）發送至原生端。若 JavaScript 線程在單幀時間內無響應，就會被視為掉幀。例如：當您在複雜應用的根組件調用 `this.setState` 導致需要重新渲染計算密集型子組件樹時，可能耗時 200 毫秒，造成 12 幀丟失——此時所有由 JavaScript 控制的動畫都將凍結。任何超過 100 毫秒的延遲都會被用戶明顯感知。

這種情況常見於 `Navigator` 轉場時：當推送新路由時，JavaScript 線程需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常會耗費數幀時間，造成[界面卡頓](http://jankfree.org/)（因轉場動畫由 JavaScript 線程控制）。有時組件在 `componentDidMount` 中執行額外工作，可能導致轉場過程中出現二次卡頓。

另一個典型例子是觸控響應：若 JavaScript 線程持續多幀處於忙碌狀態，您可能會注意到 `TouchableOpacity` 等組件的響應延遲。這是因為 JavaScript 線程無法及時處理從主線程傳來的原始觸控事件，導致 `TouchableOpacity` 無法立即調節原生視圖的透明度。

### UI 幀率（主線程）

許多人注意到 `NavigatorIOS` 的預設性能優於 `Navigator`，其根本原因在於前者的轉場動畫完全在主線程執行，因此不會受 JavaScript 線程掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發至 JS 執行緒，但滾動行為的執行並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下執行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供完善的警告與錯誤訊息（例如驗證 propTypes 及其他斷言），運行時需執行更多工作。請務必在[正式版本](running-on-device.md#building-your-app-for-production)中測試效能表現。

### 使用 `console.log` 語句

在執行打包後的應用程式時，這些語句會嚴重阻塞 JavaScript 執行緒。這包含來自偵錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。需先透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，接著編輯專案目錄下的 `.babelrc` 檔案如下：

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

即使專案中未使用 `console.*` 呼叫，仍建議使用此插件，因為第三方函式庫可能也會呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具備顯著的效能優化，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

若您的 [`FlatList`](flatlist.md) 渲染速度緩慢，請確認是否實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量步驟來優化渲染速度。

### 重新渲染幾乎未變動的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來大幅減少工作量。若採用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需重新渲染的條件。若編寫純元件（即渲染函式的返回值完全依賴於 props 和 state），可透過 PureComponent 自動實現此功能。再次強調，不可變資料結構有助於保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場動畫卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是可行的解決方案，但若延遲動畫期間的工作會對使用者體驗造成過大影響，則可考慮改用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 則直接利用 Core Animation，不受 JS 執行緒與主執行緒掉幀影響。

我曾將此技術應用於模態視窗的動畫（從頂部滑下並淡入半透明遮罩），同時初始化並接收多個網路請求的回應、渲染模態內容，以及更新觸發模態的原始視圖。詳見動畫指南以獲取更多關於 LayoutAnimation 的使用資訊。

注意事項：

- LayoutAnimation 僅適用於「一觸即發」的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生於當您將具有透明背景的文字置於圖片上方，或任何需要在每一幀重新繪製視圖並進行 alpha 合成的場景。您會發現啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 能顯著改善此問題。

請注意不要過度使用這些屬性，否則記憶體使用量可能會暴增。在使用這些屬性時，請務必分析效能和記憶體使用情況。如果不再需要移動視圖，請關閉此屬性。

### 動畫調整圖片大小導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對於大型圖片來說非常耗資源。相反地，請使用 `transform: [{scale}]` 樣式屬性來動畫調整大小。例如，當您點擊圖片並將其放大至全螢幕時，可以採用這種方式。

### 我的 TouchableX 視圖反應不靈敏

有時，如果我們在調整回應觸控的元件的不透明度或高亮效果的同一幀中執行動作，則在 `onPress` 函數返回之前，我們不會看到該效果。如果 `onPress` 執行 `setState` 導致大量工作和幾幀丟失，則可能會發生這種情況。解決方法是將 `onPress` 處理程序內的任何動作包裝在 `requestAnimationFrame` 中：

```jsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器過渡動畫緩慢

如上所述，`Navigator` 動畫由 JavaScript 執行緒控制。想像「從右側推入」的場景過渡動畫：每一幀，新場景從右向左移動，從螢幕外開始（假設 x 偏移為 320），最終在場景位於 x 偏移為 0 時穩定下來。在此過渡的每一幀中，JavaScript 執行緒需要向主執行緒發送新的 x 偏移值。如果 JavaScript 執行緒被鎖定，則無法執行此操作，因此在該幀上不會發生更新，動畫會卡頓。

解決此問題的一種方法是允許將基於 JavaScript 的動畫卸載到主執行緒。如果我們採用這種方法來處理上述示例，我們可能會在開始過渡時計算新場景的所有 x 偏移值列表，並將其發送到主執行緒以優化的方式執行。現在 JavaScript 執行緒從此責任中解放出來，即使在渲染場景時丟失幾幀也沒關係——您可能甚至不會注意到，因為您會被漂亮的過渡動畫所吸引。

解決此問題是新 [React Navigation](navigation.md) 庫的主要目標之一。React Navigation 中的視圖使用原生元件和 [`Animated`](animated.md) 庫，以在原生執行緒上運行的 60 FPS 動畫來呈現流暢的過渡效果。