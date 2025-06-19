---
id: performance
title: Performance Overview
---

使用 React Native 而非基於 WebView 的工具，一個關鍵原因在於能實現至少每秒 60 幀的流暢度，並為應用提供原生外觀與體驗。我們盡可能讓 React Native 自動處理優化，使開發者能專注於應用邏輯而無需擔心效能問題。但在某些領域，我們尚未完全實現這個目標，也有些情況（如同直接編寫原生代碼時）React Native 無法自動判斷最佳優化方式，此時就需要手動介入。我們預設追求絲滑般的 UI 效能，但偶爾仍可能無法達成。

本指南旨在傳授基礎知識，協助您[排查效能問題](profiling.md)，並探討[常見問題根源與建議解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

Your grandparents' generation called movies ["moving pictures"](https://www.youtube.com/watch?v=F1i40rnpOsA) for a reason: realistic motion in video is an illusion created by quickly changing static images at a consistent speed. We refer to each of these images as frames. The number of frames that is displayed each second has a direct impact on how smooth and ultimately life-like a video (or user interface) seems to be. iOS and Android devices display at least 60 frames per second, which gives you and the UI system at most 16.67ms to do all of the work needed to generate the static image (frame) that the user will see on the screen for that interval. If you are unable to do the work necessary to generate that frame within the allotted time slot, then you will "drop a frame" and the UI will appear unresponsive.

Now to confuse the matter a little bit, open up the [Dev Menu](debugging.md#opening-the-dev-menu) in your app and toggle `Show Perf Monitor`. You will notice that there are two different frame rates.

![效能監測器截圖](/docs/assets/PerfUtil.png)

### JS 幀率（JavaScript 執行緒）

多數 React Native 應用的業務邏輯都在 JavaScript 執行緒上運行——這裡是 React 組件樹的所在，也是處理 API 呼叫、觸控事件等核心邏輯的地方。對原生視圖的更新會批量處理，並在事件循環的每次迭代結束時（理想情況下趕在幀截止期限前）發送到原生端。若 JavaScript 執行緒在某幀期間無響應，就會被視為掉幀。例如，當您在複雜應用的根組件設置新狀態，導致需要重新渲染計算密集的子組件樹時，可能耗費 200 毫秒而導致丟失 12 幀。此時所有由 JavaScript 控制的動畫都會明顯凍結。若掉幀過多，使用者將明顯感知卡頓。

以觸控回應為例：若 JavaScript 執行緒持續處理跨越多幀的任務，您可能會注意到 `TouchableOpacity` 等組件對觸控的反應延遲。這是因為 JavaScript 執行緒忙碌而無法及時處理從主執行緒傳來的原始觸控事件，導致 `TouchableOpacity` 無法通知原生視圖調整透明度。

### UI 幀率（主執行緒）

您可能已觀察到，原生堆疊導航器（如 React Navigation 提供的 [@react-navigation/native-stack](https://reactnavigation.org/docs/native-stack-navigator)）預設效能優於 JavaScript 實現的導航器。這是因為轉場動畫直接在原生主 UI 執行緒執行，不會受 JavaScript 執行緒掉幀影響。

Similarly, you can happily scroll up and down through a `ScrollView` when the JavaScript thread is locked up because the `ScrollView` lives on the main thread. The scroll events are dispatched to the JS thread, but their receipt is not necessary for the scroll to occur.

## 常見效能問題根源

### 處於開發模式（`dev=true`）

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[發佈版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

當運行打包後的應用程式時，這些語句可能會在 JavaScript 執行緒中造成嚴重的瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。你也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需要安裝它：`npm i babel-plugin-transform-remove-console --save-dev`，然後編輯專案目錄下的 `.babelrc` 文件如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案發佈（生產）版本中的所有 `console.*` 呼叫。

即使專案中沒有使用 `console.*` 呼叫，也建議使用此插件。第三方函式庫也可能會呼叫它們。

### `FlatList` 渲染過慢或大型列表的滾動效能不佳

如果 [`FlatList`](flatlist.md) 渲染速度過慢，請確保已實作 [`getItemLayout`](flatlist.md#getitemlayout) 以通過跳過渲染項目的測量來優化渲染速度。

還有其他針對效能優化的第三方列表函式庫，包括 [FlashList](https://github.com/shopify/flash-list) 和 [Legend List](https://github.com/legendapp/legend-list)。

### 由於同時在 JavaScript 執行緒上執行大量工作導致 JS 執行緒 FPS 下降

「導航器過渡緩慢」是最常見的表現，但也有其他情況可能發生。使用 [`InteractionManager`](interactionmanager.md) 可能是一個好方法，但如果延遲動畫期間的工作對使用者體驗成本過高，則可以考慮使用 [`LayoutAnimation`](layoutanimation.md)。

The [`Animated API`](animated.md) currently calculates each keyframe on-demand on the JavaScript thread unless you [set `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app), while [`LayoutAnimation`](layoutanimation.md) leverages Core Animation and is unaffected by JS thread and main thread frame drops.

One case for using this is animating in a modal (sliding down from top and fading in a translucent overlay) while initializing and perhaps receiving responses for several network requests, rendering the contents of the modal, and updating the view where the modal was opened from. See the [Animations guide](animations.md) for more information about how to use `LayoutAnimation`.

**注意事項：**

- `LayoutAnimation` 僅適用於「一次性」動畫（「靜態」動畫）——如果需要中斷動畫，則需要使用 [`Animated`](animated.md)。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

在 Android 上尤其如此，當你有文字帶有透明背景並位於圖片上方，或任何其他需要在每一幀重新繪製視圖的 alpha 合成情況時。你會發現啟用 `renderToHardwareTextureAndroid` 可以顯著改善此問題。對於 iOS，`shouldRasterizeIOS` 預設已啟用。

請注意不要過度使用此功能，否則記憶體使用量可能會急劇上升。在使用這些屬性時，請分析效能和記憶體使用情況。如果不再移動視圖，請關閉此屬性。

### 動畫調整圖片大小導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 [`Image` 元件](image.md) 的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對於大型圖片來說可能非常耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫效果。一個典型的使用場景是點擊圖片後將其放大至全螢幕顯示。

### 我的 TouchableX 元件觸控反應遲鈍

有時，如果我們在調整元件透明度或高亮效果的同一幀中執行觸控響應操作（這些視覺效果是對觸控的回應），那麼在 `onPress` 函數執行完畢前，這些視覺變化可能不會立即顯現。這種情況通常發生在 `onPress` 觸發了會導致大量重新渲染的狀態更新，進而造成數幀丟失時。解決方法是將 `onPress` 處理函數內的所有操作包裹在 `requestAnimationFrame` 中：

```tsx
function handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```