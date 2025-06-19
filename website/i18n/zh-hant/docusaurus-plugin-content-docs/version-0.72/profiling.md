---
id: profiling
title: Profiling
---

使用內建的效能分析工具來獲取 JavaScript 執行緒與主執行緒工作負載的詳細並列資訊。可透過 Debug 選單中的「Perf Monitor」啟用該功能。

在 iOS 上，Instruments 是不可或缺的工具；而在 Android 平台，你應該學習使用 [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)。

但首先，[**務必關閉開發者模式！**](performance.md#running-in-development-mode-devtrue) 你應該在應用程式日誌中看到 `__DEV__ === false, development-level warning are OFF, performance optimizations are ON` 的提示。

Another way to profile JavaScript is to use the Chrome profiler while debugging. This won't give you accurate results as the code is running in Chrome but will give you a general idea of where bottlenecks might be. Run the profiler under Chrome's `Performance` tab. A flame graph will appear under `User Timing`. To view more details in tabular format, click at the `Bottom Up` tab below and then select `DedicatedWorker Thread` at the top left menu.

## Profiling Android UI Performance with `systrace`

Android 支援上萬種裝置且採用通用軟體渲染架構：這種框架設計與硬體兼容性需求，意味著相較 iOS 你能獲得的原生優化較少。但有時仍有改進空間——許多時候問題甚至與原生程式碼無關！

偵測卡頓的第一步是釐清每個 16ms 幀的時間消耗分佈。我們將使用 Android 標準分析工具 `systrace`。

`systrace` 是 Android 標準的基於標記的分析工具（隨 Android platform-tools 套件安裝）。被分析的程式區塊會被開始/結束標記包圍，並以彩色圖表視覺化呈現。Android SDK 與 React Native 框架均提供可視化的標準標記。

### 1. 收集追蹤數據

首先透過 USB 連接出現卡頓問題的裝置，並將裝置狀態調整至即將進行要分析的導航/動畫操作前。執行以下 `systrace` 指令：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

指令參數說明：

- `time` is the length of time the trace will be collected in seconds
- `sched`, `gfx`, and `view` are the android SDK tags (collections of markers) we care about: `sched` gives you information about what's running on each core of your phone, `gfx` gives you graphics info such as frame boundaries, and `view` gives you information about measure, layout, and draw passes
- `-a <your_package_name>` enables app-specific markers, specifically the ones built into the React Native framework. `your_package_name` can be found in the `AndroidManifest.xml` of your app and looks like `com.example.app`

追蹤開始後，立即執行要分析的動畫或互動操作。追蹤結束時，systrace 會提供可在瀏覽器開啟的追蹤檔案連結。

### 2. 解讀追蹤數據

在瀏覽器（建議使用 Chrome）開啟追蹤檔案後，你會看到類似下圖的視覺化結果：

![範例](/docs/assets/SystraceExample.png)

:::note[操作提示]
使用 WASD 鍵平移與縮放視圖。
:::

若追蹤 .html 檔案無法正確開啟，請檢查瀏覽器控制台是否出現以下錯誤：

![ObjectObserveError](/docs/assets/ObjectObserveError.png)

由於近期瀏覽器已棄用 `Object.observe`，你可能需要透過 Google Chrome Tracing 工具開啟檔案。操作步驟：

- Opening tab in chrome `chrome://tracing`
- Selecting load
- Selecting the html file generated from the previous command.

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的這個核取方塊，以高亮顯示16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

你應該會看到如上截圖中的斑馬條紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示垂直同步時會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到你的進程

滾動直到看到你的套件名稱（部分）。在這個例子中，我正在分析 `com.facebook.adsmanager`，由於核心中執行緒名稱長度的限制，它顯示為 `book.adsmanager`。

在左側，你會看到一組執行緒，對應右側時間軸上的行。對於我們的目的來說，有幾個執行緒是重要的：UI執行緒（顯示你的套件名稱或「UI Thread」）、`mqt_js` 和 `mqt_native_modules`。如果你在Android 5+上執行，我們還需要關心渲染執行緒。

- **UI執行緒。** 這是標準Android測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是book.adsmanager）或「UI Thread」。你在這個執行緒上看到的事件應該類似這樣，並與 `Choreographer`、`traversals` 和 `DispatchUI` 有關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒。** 這是JavaScript執行的位置。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你裝置上的核心是否合作。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等來識別它：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（例如 `UIManager`）執行的位置。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。在後者的情況下，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 來識別它：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 如果你使用Android L（5.0）及以上版本，你的應用程式中還會有一個渲染執行緒。這個執行緒生成用於繪製UI的實際OpenGL指令。執行緒名稱會是 `RenderThread` 或 `<...>`。在後者的情況下，可以透過尋找 `DrawFrame` 和 `queueBuffer` 來識別它：

  ![渲染執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題所在

流暢的動畫應該看起來像這樣：

![流暢的動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的UI工作需要在16毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用程式是以60 FPS運行的。

然而，如果你注意到卡頓，可能會看到這樣的情況：

![由JS引起的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎一直在執行，並且跨越了幀邊界！這個應用程式沒有以60 FPS渲染。在這種情況下，**問題出在JS**。

你也可能會看到這樣的情況：

![由UI引起的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀上渲染的UI需要完成太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你將獲得一些非常有用的資訊來指導你的下一步行動。

## 解決 JavaScript 問題

若確認問題出在 JavaScript 端，請從執行的具體 JS 程式碼中尋找線索。在上述案例中，我們發現每幀都會多次呼叫 `RCTEventEmitter`。以下是從前述追蹤記錄中放大檢視的 JS 執行緒：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不合理。為何呼叫如此頻繁？這些真的是不同的事件嗎？答案通常取決於產品程式碼本身。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題出在原生 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 對 GPU 負載過重，或
2. 在動畫/互動過程中動態建立新 UI（例如滾動時載入新內容）。

### GPU 負載過高

第一種情境下，追蹤記錄會顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意跨越幀邊界的 `DrawFrame` 長時間執行。這表示 GPU 正在等待清空前一幀的命令緩衝區。

緩解方法包括：

- investigate using `renderToHardwareTextureAndroid` for complex, static content that is being animated/transformed (e.g. the `Navigator` slide/alpha animations)
- make sure that you are **not** using `needsOffscreenAlphaCompositing`, which is disabled by default, as it greatly increases the per-frame load on the GPU in most cases.

### 在 UI 執行緒建立新視圖

第二種情境的追蹤記錄會呈現如下模式：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

可觀察到 JS 執行緒先進行運算，接著原生模組執行緒處理任務，最後 UI 執行緒執行高成本的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化新建 UI 的複雜度，否則目前無快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，未來將允許在非主執行緒建立與配置新 UI，確保互動流暢性。