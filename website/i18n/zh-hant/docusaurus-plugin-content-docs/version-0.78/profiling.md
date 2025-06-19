---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的效能、資源使用情況及行為，來識別潛在瓶頸或效率問題的過程。善用效能分析工具能確保應用程式在不同裝置與條件下流暢運作。

在iOS平台，Instruments是不可或缺的工具；而在Android平台，則建議學習使用[Android Studio Profiler](profiling.md#profiling-android-ui-performance-with-system-tracing)。

但首先，[**請確認開發者模式已關閉！**](performance.md#running-in-development-mode-devtrue) 您應在應用程式日誌中看到`__DEV__ === false, development-level warning are OFF, performance optimizations are ON`的提示。

## 使用系統追蹤分析Android UI效能

Android支援上萬種不同手機型號，並採用通用化設計以兼容軟體渲染：其框架架構與跨硬體平台的通用性需求，意味著相較iOS平台，開發者需自行處理更多效能優化工作——但許多時候問題根源並非原生程式碼！

偵測卡頓問題的第一步，是釐清每個16毫秒幀周期內的時間消耗分佈。我們將使用[Android Studio內建的System Tracing效能分析工具](https://developer.android.com/studio/profile)。

### 1. 收集追蹤資料

首先透過USB連接會出現卡頓問題的裝置至電腦。在Android Studio中開啟專案的`android`資料夾，於右上角面板選擇裝置，並[以可分析模式執行專案](https://developer.android.com/studio/profile#build-and-run)。

當應用程式以可分析模式建置並在裝置上運行後，導航至欲分析的動畫/互動觸發前狀態，於Android Studio Profiler面板啟動[「擷取系統活動」任務](https://developer.android.com/studio/profile#start-profiling)。

開始記錄追蹤後，執行欲分析的動畫或互動操作，隨後點擊「停止記錄」。您可立即[在Android Studio中檢視追蹤資料](https://developer.android.com/studio/profile/jank-detection)，或於「過往記錄」面板選擇記錄檔後點擊「匯出記錄」，使用[Perfetto](https://perfetto.dev/)等工具開啟分析。

### 2. 解讀追蹤資料

在Android Studio或Perfetto中開啟追蹤檔後，您將看到類似以下畫面：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用WASD鍵進行平移與縮放。
:::

不同工具的介面可能略有差異，但以下說明均適用。

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的此選項以標記16毫秒幀邊界：

![啟用垂直同步高亮](/docs/assets/SystraceHighlightVSync.png)

您應看到如上圖所示的斑馬條紋。若未顯示，請嘗試在其他裝置上分析：三星裝置已知可能無法正常顯示垂直同步標記，而Nexus系列通常較可靠。
:::

### 3. 定位您的進程

滾動直至看見您的套件名稱（部分）。本例中分析的`com.facebook.adsmanager`因系統核心執行緒名稱長度限制，顯示為`book.adsmanager`。

左側面板顯示的執行緒清單對應右側時間軸的各軌道。我們需重點關注以下執行緒：UI執行緒（顯示您的套件名稱或UI Thread）、`mqt_js`與`mqt_native_modules`。若在Android 5+系統運行，還需關注Render Thread。

- **UI 執行緒。** 這是標準 Android 測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是 book.adsmanager）或 UI Thread。你在這個執行緒上看到的事件應該類似以下內容，並與 `Choreographer`、`traversals` 和 `DispatchUI` 相關：

  ![UI 執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS 執行緒。** 這是 JavaScript 執行的地方。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你設備的內核行為。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等內容來識別：

  ![JS 執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（例如 `UIManager`）執行的地方。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。如果沒有名稱，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 等內容來識別：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 如果你使用的是 Android L（5.0）或更高版本，你的應用還會有一個渲染執行緒。這個執行緒生成實際用於繪製 UI 的 OpenGL 指令。執行緒名稱會是 `RenderThread` 或 `<...>`。如果沒有名稱，可以透過尋找 `DrawFrame` 和 `queueBuffer` 等內容來識別：

  ![渲染執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該看起來像這樣：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的 UI 工作必須在 16 毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用是以 60 FPS 運行的。

如果你注意到卡頓，可能會看到類似這樣的內容：

![JS 導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意 JS 執行緒幾乎一直在執行，而且跨越了幀邊界！這個應用沒有以 60 FPS 渲染。在這種情況下，**問題出在 JS**。

你也可能會看到這樣的內容：

![UI 導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI 和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀渲染的 UI 需要太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你已經掌握了一些非常有用的資訊來指導下一步。

## 解決 JavaScript 問題

如果你確定是 JS 問題，請在執行的特定 JS 中尋找線索。在上面的情境中，我們看到 `RCTEventEmitter` 每幀被呼叫多次。以下是從上述追蹤中放大的 JS 執行緒：

![過多的 JS](/docs/assets/SystraceBadJS2.png)

這看起來不太對。為什麼它被呼叫這麼頻繁？它們真的是不同的事件嗎？這些問題的答案可能取決於你的產品代碼。很多時候，你需要查看 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。

## 解決原生 UI 問題

如果你確定是原生 UI 問題，通常有兩種情況：

1. 你試圖在每一幀繪製的 UI 在 GPU 上需要太多工作，或者
2. 你在動畫/互動期間構建新的 UI（例如在滾動時載入新內容）。

### GPU 工作負載過高

在第一種情境中，您會看到追蹤記錄顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 過載](/docs/assets/SystraceBadUI.png)

請注意跨越影格邊界的 `DrawFrame` 長時間執行。這表示 GPU 正在等待清空前一個影格的指令緩衝區。

為緩解此問題，您應：

- investigate using `renderToHardwareTextureAndroid` for complex, static content that is being animated/transformed (e.g. the `Navigator` slide/alpha animations)
- make sure that you are **not** using `needsOffscreenAlphaCompositing`, which is disabled by default, as it greatly increases the per-frame load on the GPU in most cases.

### 在 UI 執行緒建立新視圖

在第二種情境中，您會看到類似以下的狀況：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

注意 JS 執行緒先運算一段時間，接著原生模組執行緒執行部分工作，隨後 UI 執行緒進行耗時的遍歷操作

除非能延遲建立新 UI 至互動結束後，或簡化要建立的 UI 結構，否則沒有快速解決方案。React Native 團隊正在基礎架構層面開發解決方案，未來將允許在主執行緒外建立與配置新 UI，保持互動流暢度

### 找出原生 CPU 熱點

若問題似乎出在原生端，可使用 [CPU 熱點分析工具](https://developer.android.com/studio/profile/record-java-kotlin-methods)獲取詳細資訊。開啟 Android Studio Profiler 面板並選擇「Find CPU Hotspots (Java/Kotlin Method Recording)」

:::info[選擇 Java/Kotlin 記錄]

請確認選擇「Find CPU Hotspots **(Java/Kotlin Recording)**」而非「Find CPU Hotspots (Callstack Sample)」。兩者圖示相似但功能不同
:::

執行互動操作後點擊「Stop recording」。記錄過程會消耗大量資源，請保持互動簡短。您可在 Android Studio 檢視結果追蹤，或匯出後透過 [Firefox Profiler](https://profiler.firefox.com/) 等線上工具開啟

與系統追蹤不同，CPU 熱點分析速度較慢，無法提供精確測量值。但能顯示哪些原生方法被呼叫，以及各影格期間的時間比例分配狀況