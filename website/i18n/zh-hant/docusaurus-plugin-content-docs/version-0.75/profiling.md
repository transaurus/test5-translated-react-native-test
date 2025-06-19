---
id: profiling
title: Profiling
---

使用內建的效能分析工具來獲取 JavaScript 執行緒與主執行緒工作細節的並列資訊。可透過 Debug 選單中的 Perf Monitor 來存取此功能。

在 iOS 上，Instruments 是不可或缺的工具；而在 Android 上，你應該學習使用 [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)。

但首先，[**請確保開發模式已關閉！**](performance.md#running-in-development-mode-devtrue) 你應該在應用程式日誌中看到 `__DEV__ === false, development-level warning are OFF, performance optimizations are ON`。

另一種分析 JavaScript 效能的方式是在除錯時使用 Chrome 的分析工具。由於程式碼是在 Chrome 中執行，這不會提供精確的結果，但能讓你大致了解可能的瓶頸所在。在 Chrome 的 `Performance` 分頁下執行分析工具。火焰圖會出現在 `User Timing` 下方。若要查看表格形式的詳細資訊，點擊下方的 `Bottom Up` 分頁，然後從左上角選單中選擇 `DedicatedWorker Thread`。

## Profiling Android UI Performance with `systrace`

Android 支援超過一萬種不同的手機，並普遍支援軟體渲染：框架架構及需要廣泛支援多種硬體目標的特性，意味著相對於 iOS，你能獲得的免費資源較少。但有時，仍有可以改進的地方——而且很多時候問題根本不在原生程式碼！

除錯卡頓的第一步是回答一個基本問題：在每個 16 毫秒的幀中，時間都花在哪裡。為此，我們將使用一個標準的 Android 分析工具 `systrace`。

`systrace` 是一個標準的基於標記的 Android 分析工具（安裝 Android platform-tools 套件時會一併安裝）。被分析的程式碼區塊會被開始/結束標記包圍，並以彩色圖表形式視覺化。Android SDK 和 React Native 框架都提供了可視覺化的標準標記。

### 1. 收集追蹤資料

首先，透過 USB 將出現卡頓問題的裝置連接到電腦，並讓它進入你想要分析的導航/動畫前的狀態。執行以下 `systrace` 指令：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

快速說明此指令的各部分：

- `time` 是追蹤資料收集的時間長度（秒）
- `sched`、`gfx` 和 `view` 是我們關注的 Android SDK 標記（標記集合）：`sched` 提供手機每個核心的執行資訊，`gfx` 提供圖形資訊（如幀邊界），`view` 提供測量、佈局和繪製過程的資訊
- `-a <your_package_name>` 啟用應用程式特定的標記，特別是 React Native 框架內建的標記。`your_package_name` 可在應用程式的 `AndroidManifest.xml` 中找到，格式如 `com.example.app`

開始收集追蹤資料後，執行你想要分析的動畫或互動。追蹤結束時，systrace 會提供一個可在瀏覽器中開啟的追蹤連結。

### 2. 閱讀追蹤資料

在瀏覽器（建議使用 Chrome）中開啟追蹤資料後，你應該會看到類似以下的內容：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用 WASD 鍵來平移和縮放。
:::

如果你的追蹤 .html 檔案無法正確開啟，請檢查瀏覽器控制台是否有以下錯誤：

![ObjectObserveError](/docs/assets/ObjectObserveError.png)

由於 `Object.observe` 在較新的瀏覽器中已被棄用，你可能需要透過 Google Chrome Tracing 工具來開啟檔案。操作步驟如下：

- 在 Chrome 中開啟分頁 `chrome://tracing`
- 選擇載入
- 選擇上一個指令生成的 html 檔案。

:::info[啟用 VSync 高亮顯示]
勾選畫面右上角的此核取方塊以突顯 16 毫秒的幀邊界：

![啟用 VSync 高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應會看到如上截圖中的斑馬條紋。若未顯示，請嘗試在其他裝置上進行分析：已知三星裝置可能無法正確顯示垂直同步信號，而 Nexus 系列通常較為可靠。
:::

### 3. 找到您的進程

滾動直到看見您的套件名稱（部分）。此例中，我分析的目標是 `com.facebook.adsmanager`，由於核心執行緒名稱長度限制，顯示為 `book.adsmanager`。

左側會顯示一組執行緒，對應右側時間軸的行。我們關注以下幾個執行緒：UI 執行緒（顯示您的套件名稱或「UI Thread」）、`mqt_js` 和 `mqt_native_modules`。若在 Android 5+ 上執行，還需關注 Render Thread。

- **UI Thread.** This is where standard android measure/layout/draw happens. The thread name on the right will be your package name (in my case book.adsmanager) or UI Thread. The events that you see on this thread should look something like this and have to do with `Choreographer`, `traversals`, and `DispatchUI`:

  ![UI Thread Example](/docs/assets/SystraceUIThreadExample.png)

- **JS Thread.** This is where JavaScript is executed. The thread name will be either `mqt_js` or `<...>` depending on how cooperative the kernel on your device is being. To identify it if it doesn't have a name, look for things like `JSCall`, `Bridge.executeJSCall`, etc:

  ![JS Thread Example](/docs/assets/SystraceJSThreadExample.png)

- **Native Modules Thread.** This is where native module calls (e.g. the `UIManager`) are executed. The thread name will be either `mqt_native_modules` or `<...>`. To identify it in the latter case, look for things like `NativeCall`, `callJavaModuleMethod`, and `onBatchComplete`:

  ![Native Modules Thread Example](/docs/assets/SystraceNativeModulesThreadExample.png)

- **Bonus: Render Thread.** If you're using Android L (5.0) and up, you will also have a render thread in your application. This thread generates the actual OpenGL commands used to draw your UI. The thread name will be either `RenderThread` or `<...>`. To identify it in the latter case, look for things like `DrawFrame` and `queueBuffer`:

  ![Render Thread Example](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應如下所示：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每次顏色變化代表一幀——請記住，要顯示一幀，所有 UI 工作需在 16 毫秒內完成。注意沒有任何執行緒的工作接近幀邊界。如此渲染的應用能以 60 FPS 運行。

若發現卡頓，可能會看到如下情況：

![JS 導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

可見 JS 執行緒幾乎全程運作，且跨越幀邊界！此應用未以 60 FPS 渲染。此時**問題出在 JS**。

您也可能看到以下情況：

![UI 導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

此例中，UI 和渲染執行緒的工作跨越了幀邊界。每幀嘗試渲染的 UI 需處理過多工作。此時**問題出於渲染的原生視圖**。

至此，您已獲得關鍵資訊來決定後續步驟。

## 解決 JavaScript 問題

若確認問題出在 JavaScript 端，請檢查執行的具體 JS 程式碼。在上述案例中，我們發現每幀都會多次呼叫 `RCTEventEmitter`。以下是從追蹤記錄中放大檢視的 JS 執行緒情況：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不正常。為何呼叫如此頻繁？這些是不同的事件嗎？答案通常取決於產品程式碼本身。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題出在原生 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 對 GPU 負載過高，或
2. 在動畫/互動過程中動態建立新 UI（例如滾動時載入新內容）。

### GPU 負載過高

第一種情境下，追蹤記錄會顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意 `DrawFrame` 耗時過長且跨越幀邊界，這表示 GPU 正在處理前一幀的命令緩衝區。

緩解方法包括：

- investigate using `renderToHardwareTextureAndroid` for complex, static content that is being animated/transformed (e.g. the `Navigator` slide/alpha animations)
- make sure that you are **not** using `needsOffscreenAlphaCompositing`, which is disabled by default, as it greatly increases the per-frame load on the GPU in most cases.

### 在 UI 執行緒建立新視圖

第二種情境的追蹤記錄會類似這樣：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

可觀察到 JS 執行緒先運算一段時間，接著原生模組執行緒處理工作，最後 UI 執行緒進行高成本的遍歷操作。

除非能延後建立新 UI 或簡化 UI 結構，否則暫無快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，將允許在非主執行緒建立和配置新 UI，確保互動流暢性。