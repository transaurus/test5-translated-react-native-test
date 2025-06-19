---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的效能、資源使用情況和行為來識別潛在瓶頸或效率問題的過程。善用效能分析工具能確保您的應用在不同裝置和條件下流暢運作。

在iOS平台上，Instruments是不可或缺的工具；而在Android上，您應該學習使用[Android Studio Profiler](profiling.md#profiling-android-ui-performance-with-system-tracing)。

但首先，[**請確認開發者模式已關閉！**](performance.md#running-in-development-mode-devtrue) 您應在應用日誌中看到`__DEV__ === false, development-level warning are OFF, performance optimizations are ON`的提示。

## 使用系統追蹤分析Android UI效能

Android支援上萬種不同手機型號，並採用通用化設計以兼容軟體渲染：這種框架架構與硬體兼容性需求，意味著相較iOS平台，開發者需自行處理更多效能優化工作——但許多時候問題根源並非原生程式碼！

偵測卡頓問題的第一步，是釐清每個16毫秒幀的執行時間分配。我們將使用[Android Studio內建的System Tracing分析工具](https://developer.android.com/studio/profile)。

### 1. 收集追蹤資料

首先透過USB連接出現卡頓問題的裝置至電腦。在Android Studio中開啟專案的`android`資料夾，於右上角面板選擇裝置後，[以可分析模式執行專案](https://developer.android.com/studio/profile#build-and-run)。

當應用以可分析模式建置並在裝置上運行時，導航至欲分析的動畫/操作觸發前，點擊Android Studio Profiler面板中的["Capture System Activities"任務](https://developer.android.com/studio/profile#start-profiling)。

開始記錄後，執行目標動畫或互動操作，接著點擊「停止記錄」。您可[直接在Android Studio檢視追蹤資料](https://developer.android.com/studio/profile/jank-detection)，或於「Past Recordings」面板選擇記錄檔後點擊「Export recording」，透過[Perfetto](https://perfetto.dev/)等工具開啟分析。

### 2. 解讀追蹤資料

在Android Studio或Perfetto中開啟追蹤檔後，您將看到類似以下畫面：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用WASD鍵進行平移和縮放。
:::

不同工具的介面可能略有差異，但下列說明均適用。

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的核取方塊以標示16毫秒幀邊界：

![啟用垂直同步高亮](/docs/assets/SystraceHighlightVSync.png)

如上方截圖所示，您應看到斑馬條紋狀標記。若未顯示，請嘗試在其他裝置上分析：三星裝置可能無法正常顯示垂直同步標記，而Nexus系列通常較可靠。
:::

### 3. 定位您的進程

滾動直至看見您的套件名稱（部分顯示）。本例中分析的`com.facebook.adsmanager`因系統核心執行緒名稱限制，顯示為`book.adsmanager`。

左側面板顯示的執行緒清單對應右側時間軸。我們需關注以下執行緒：UI執行緒（顯示您的套件名稱或UI Thread）、`mqt_js`和`mqt_native_modules`。若在Android 5+裝置上運行，還需注意Render Thread。

- **UI 執行緒。** 這是標準 Android 測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是 book.adsmanager）或 UI Thread。你在這個執行緒上看到的事件應該類似以下內容，並與 `Choreographer`、`traversals` 和 `DispatchUI` 相關：

  ![UI Thread Example](/docs/assets/SystraceUIThreadExample.png)

- **JS 執行緒。** 這是執行 JavaScript 的地方。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你設備的內核行為。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等來識別：

  ![JS Thread Example](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是執行原生模組呼叫（例如 `UIManager`）的地方。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。在後者情況下，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 來識別：

  ![Native Modules Thread Example](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 如果你使用 Android L（5.0）或更高版本，你的應用還會有一個渲染執行緒。這個執行緒生成用於繪製 UI 的實際 OpenGL 指令。執行緒名稱會是 `RenderThread` 或 `<...>`。在後者情況下，可以透過尋找 `DrawFrame` 和 `queueBuffer` 來識別：

  ![Render Thread Example](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該看起來像這樣：

![Smooth Animation](/docs/assets/SystraceWellBehaved.png)

每個顏色變化代表一幀——記住，為了顯示一幀，我們所有的 UI 工作需要在 16 毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用是以 60 FPS 運行的。

如果你注意到卡頓，可能會看到類似這樣的情況：

![Choppy Animation from JS](/docs/assets/SystraceBadJS.png)

注意 JS 執行緒幾乎一直在執行，而且跨越了幀邊界！這個應用沒有以 60 FPS 渲染。在這種情況下，**問題出在 JS**。

你也可能會看到這樣的情況：

![Choppy Animation from UI](/docs/assets/SystraceBadUI.png)

在這種情況下，UI 和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀渲染的 UI 需要完成太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你已經掌握了一些非常有用的信息來指導下一步行動。

## 解決 JavaScript 問題

如果你確認是 JS 問題，可以在執行的具體 JS 代碼中尋找線索。在上面的情境中，我們看到 `RCTEventEmitter` 在每一幀被多次呼叫。以下是從上述追蹤中放大的 JS 執行緒：

![Too much JS](/docs/assets/SystraceBadJS2.png)

這看起來不太對。為什麼它被呼叫這麼頻繁？它們真的是不同的事件嗎？這些問題的答案可能取決於你的產品代碼。很多時候，你需要查看 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。

## 解決原生 UI 問題

如果你確認是原生 UI 問題，通常有兩種情況：

1. 你試圖在每一幀繪製的 UI 在 GPU 上涉及太多工作，或者
2. 你在動畫/交互過程中構建了新的 UI（例如在滾動時加載新內容）。

### GPU 負載過高

在第一種情境中，您會看到追蹤記錄顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 過載](/docs/assets/SystraceBadUI.png)

請注意 `DrawFrame` 中耗費大量時間且跨越影格邊界的情況。這段時間是在等待 GPU 清空前一影格的指令緩衝區。

為緩解此問題，您應：

- investigate using `renderToHardwareTextureAndroid` for complex, static content that is being animated/transformed (e.g. the `Navigator` slide/alpha animations)
- make sure that you are **not** using `needsOffscreenAlphaCompositing`, which is disabled by default, as it greatly increases the per-frame load on the GPU in most cases.

### 在 UI 執行緒建立新視圖

第二種情境中，您會看到類似以下的狀況：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

注意 JS 執行緒先運算一段時間，接著原生模組執行緒處理部分工作，最後 UI 執行緒進行昂貴的遍歷操作

除非能延遲建立新 UI 至互動結束後，或簡化建立的 UI 結構，否則沒有快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，將允許在非主執行緒建立與配置新 UI，保持互動流暢度

### 找出原生 CPU 熱點

若問題似乎出在原生端，可使用 [CPU 熱點分析器](https://developer.android.com/studio/profile/record-java-kotlin-methods)獲取詳細資訊。開啟 Android Studio Profiler 面板並選擇「Find CPU Hotspots (Java/Kotlin Method Recording)」

:::info[選擇 Java/Kotlin 記錄]

請務必選擇「Find CPU Hotspots **(Java/Kotlin Recording)**」而非「Find CPU Hotspots (Callstack Sample)」。兩者圖示相似但功能不同
:::

執行互動後點擊「Stop recording」。記錄過程會消耗大量資源，請保持互動簡短。您可在 Android Studio 檢查結果追蹤，或匯出後用 [Firefox Profiler](https://profiler.firefox.com/) 等線上工具開啟

與系統追蹤不同，CPU 熱點分析速度較慢，無法提供精確測量。但能顯示哪些原生方法被呼叫，以及各影格時間的分配比例