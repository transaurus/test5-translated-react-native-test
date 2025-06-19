---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的執行效能、資源使用狀況與行為模式，來識別潛在瓶頸或效率問題的過程。善用分析工具能確保應用程式在不同裝置與環境下流暢運作。

在iOS平台，Instruments是不可或缺的工具；而Android平台則建議掌握[`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)的使用技巧。

But first, [**make sure that Development Mode is OFF!**](performance.md#running-in-development-mode-devtrue) You should see `__DEV__ === false, development-level warning are OFF, performance optimizations are ON` in your application logs.

## Profiling Android UI Performance with `systrace`

Android 支援上萬種裝置且採用通用軟體渲染架構，這種框架設計與硬體兼容性需求，相較 iOS 會消耗更多系統資源。但仍有優化空間，且多數卡頓問題並非源自原生代碼！

診斷畫面卡頓的第一步，是確認每個16毫秒幀周期內的耗時分佈。我們將使用 Android 標準分析工具 `systrace`。

`systrace` 是 Android 標準的基於標記的分析工具（隨 Android platform-tools 套件安裝）。被分析的代碼區塊會被起始/結束標記包圍，並以彩色圖表可視化。Android SDK 與 React Native 框架都提供標準標記供分析。

### 1. 收集追蹤資料

首先透過 USB 連接會出現卡頓的裝置，並將應用程式導航至您想分析的動畫觸發前狀態。執行以下 `systrace` 指令：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

指令參數說明：

- `time` is the length of time the trace will be collected in seconds
- `sched`, `gfx`, and `view` are the android SDK tags (collections of markers) we care about: `sched` gives you information about what's running on each core of your phone, `gfx` gives you graphics info such as frame boundaries, and `view` gives you information about measure, layout, and draw passes
- `-a <your_package_name>` enables app-specific markers, specifically the ones built into the React Native framework. `your_package_name` can be found in the `AndroidManifest.xml` of your app and looks like `com.example.app`

追蹤開始後，立即執行您關注的動畫或交互操作。追蹤結束時，systrace 會生成可在瀏覽器開啟的分析報告連結。

### 2. 解讀追蹤報告

在瀏覽器（建議 Chrome）開啟報告後，您將看到類似下圖的視覺化資料：

![範例](/docs/assets/SystraceExample.png)

:::note[操作提示]
使用 WASD 鍵進行平移與縮放。
:::

若報告 .html 檔案無法正常開啟，請檢查瀏覽器控制台是否出現：

![ObjectObserve錯誤](/docs/assets/ObjectObserveError.png)

由於現代瀏覽器已棄用 `Object.observe`，您可能需要透過 Chrome Tracing 工具開啟：

- 在 Chrome 訪問 `chrome://tracing`
- 點擊「載入」按鈕
- 選擇先前指令生成的 html 檔案

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的這個核取方塊以高亮顯示16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應該會看到如上截圖中的斑馬條紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示垂直同步時會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到您的進程

滾動直到看到您套件名稱的部分。在這個例子中，我正在分析`com.facebook.adsmanager`，由於核心中執行緒名稱長度的限制，它顯示為`book.adsmanager`。

在左側，您會看到一組執行緒，對應右側時間軸上的行。我們關注的幾個執行緒包括：UI執行緒（顯示您的套件名稱或UI Thread）、`mqt_js`和`mqt_native_modules`。如果您在Android 5+上執行，我們還需要關注Render Thread。

- **UI執行緒。** 這是標準Android測量/佈局/繪製發生的地方。右側的執行緒名稱會是您的套件名稱（在我的例子中是book.adsmanager）或UI Thread。您在此執行緒上看到的事件應該與`Choreographer`、`traversals`和`DispatchUI`有關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒。** 這是JavaScript執行的地方。執行緒名稱會是`mqt_js`或`<...>`，取決於您裝置上的核心是否合作。如果沒有名稱，可以尋找`JSCall`、`Bridge.executeJSCall`等來識別它：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（例如`UIManager`）執行的地方。執行緒名稱會是`mqt_native_modules`或`<...>`。在後者的情況下，可以尋找`NativeCall`、`callJavaModuleMethod`和`onBatchComplete`來識別它：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：Render執行緒。** 如果您使用Android L（5.0）及以上版本，您的應用中還會有一個Render執行緒。這個執行緒生成用於繪製UI的實際OpenGL指令。執行緒名稱會是`RenderThread`或`<...>`。在後者的情況下，可以尋找`DrawFrame`和`queueBuffer`來識別它：

  ![Render執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該如下所示：

![流暢的動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的UI工作需要在16毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用是以60 FPS運行的。

如果您注意到卡頓，可能會看到類似這樣的情況：

![JS導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎一直在執行，並且跨越了幀邊界！這個應用沒有以60 FPS渲染。在這種情況下，**問題出在JS**。

您也可能會看到這樣的情況：

![UI導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI和Render執行緒的工作跨越了幀邊界。我們試圖在每一幀上渲染的UI需要完成太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，您將獲得一些非常有用的資訊來指導下一步操作。

## 解決 JavaScript 問題

若確認問題出在 JavaScript 端，請從執行的具體程式碼中尋找線索。在上述案例中，我們發現每幀都會多次呼叫 `RCTEventEmitter`。以下是從追蹤記錄中放大檢視的 JS 執行緒：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不正常。為何呼叫如此頻繁？這些真的是不同的事件嗎？答案通常取決於產品程式碼本身。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題出在原生 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 對 GPU 負擔過重，或
2. 在動畫/互動過程中動態建立新 UI（例如滾動時載入新內容）。

### GPU 負載過高

第一種情境下，追蹤記錄會顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意 `DrawFrame` 耗時過長且跨越幀邊界，這表示 GPU 正在等待前一幀的指令緩衝區清空。

緩解方法包括：

- investigate using `renderToHardwareTextureAndroid` for complex, static content that is being animated/transformed (e.g. the `Navigator` slide/alpha animations)
- make sure that you are **not** using `needsOffscreenAlphaCompositing`, which is disabled by default, as it greatly increases the per-frame load on the GPU in most cases.

### 在 UI 執行緒建立新視圖

In the second scenario, you'll see something more like this:

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

可觀察到 JS 執行緒先進行運算，接著原生模組執行緒處理任務，最後 UI 執行緒執行昂貴的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化新建 UI 的複雜度，否則沒有快速解決方案。React Native 團隊正在基礎架構層面開發解決方案，未來將允許在非主執行緒建立與配置新 UI，確保互動流暢度。