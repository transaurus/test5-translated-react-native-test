---
title: React Native Performance in Marketplace
author: Aaron Chiu
authorTitle: Software Engineer at Facebook
authorURL: 'https://www.facebook.com/aaronechiu'
authorFBID: 1057500063
authorTwitter: AaaChiuuu
tags: [engineering]
---

React Native 被廣泛應用於 Facebook 旗下多款應用程式的多個功能中，包括 Facebook 主應用程式中的頂層標籤頁。本文將重點討論一個高曝光率的產品——[Marketplace](https://newsroom.fb.com/news/2016/10/introducing-marketplace-buy-and-sell-with-your-local-community/)。該服務已在十餘個國家上線，讓使用者能探索其他使用者提供的商品與服務。

2017 年上半年，透過 Relay 團隊、Marketplace 團隊、Mobile JS Platform 團隊與 React Native 團隊的共同努力，我們將 Marketplace 在 [Year Class 2010-11 裝置](https://code.facebook.com/posts/307478339448736/year-class-a-classification-system-for-android/)（Facebook 歷來歸類為低階 Android 裝置）上的互動時間（TTI）縮短了一半。這類裝置在任何平台或裝置類型中都具有最慢的 TTI 表現。

典型的 React Native 啟動流程如下：

![](/blog/assets/RNPerformanceStartup.png)

> 免責聲明：圖中比例僅為示意，實際數值會因 React Native 配置與使用方式而異。

我們會先初始化 React Native 核心（即「Bridge」），接著執行產品特定的 JavaScript 程式碼，這段程式碼決定了 React Native 將在「原生處理時間」中渲染哪些原生視圖。

### 不同的方法論

我們早期犯的一個錯誤是過度依賴 [Systrace 和 CTScan](https://code.facebook.com/posts/747457662026706/performance-instrumentation-for-android-apps/) 來驅動效能優化。這些工具在 2016 年幫助我們發現了許多低垂的果實，但後來我們發現這兩種工具都**無法真實反映生產環境情境**，也無法模擬實際使用場景中的狀況。時間分佈比例經常不準確，有時甚至嚴重偏離事實。最極端的情況下，某些我們預期僅需幾毫秒的操作實際耗時竟達數百甚至數千毫秒。不過 CTScan 仍有其價值，我們發現它能攔截約三分之一的效能衰退問題，避免其進入生產環境。

在 Android 平台上，這些工具的局限性主要源自：1) React Native 是多執行緒框架；2) Marketplace 與 Newsfeed 等多個複雜視圖及其他頂層標籤頁共存；3) 運算時間波動極大。因此本季度我們幾乎完全依據生產環境的實際測量數據與細分指標來驅動決策與優先級排序。

### 生產環境監測的實踐之路

表面上看，生產環境監測似乎很簡單，但實際操作卻異常複雜。由於從提交代碼到 master 分支、推播應用至 Play Store、再到收集足夠的生產環境樣本以驗證結果，每個迭代週期需耗時 2-3 週。每個週期都需要確認：監測指標是否準確、顆粒度是否適當、各項細分時間是否能正確累加為總時長。我們無法依賴 alpha 和 beta 版本，因為它們無法代表一般用戶的使用情境。本質上，我們透過極其繁瑣的過程，基於數百萬份樣本的聚合數據，建立了高度精確的生產環境追蹤系統。

我們之所以嚴格驗證每毫秒細分數據是否能正確累加至父級指標，是因為早期就發現監測系統存在盲區。事實證明，最初的細分指標未計入執行緒切換造成的停滯時間。執行緒切換本身成本不高，但切換至正在處理任務的繁忙執行緒時代價極高。我們最終透過在關鍵節點插入 `Thread.sleep()` 呼叫的方式在本地重現了這些阻塞問題，並透過以下方法解決：

1. 移除對 AsyncTask 的依賴
2. 取消強制在 UI 執行緒初始化 ReactContext 和 NativeModules
3. 移除初始化階段測量 ReactRootView 的依賴

這些執行緒阻塞問題的解決，總計使啟動時間縮短了 25% 以上。

生產環境的指標數據也挑戰了我們先前的一些假設。例如，我們過去會在啟動路徑上預加載許多 JavaScript 模組，認為將這些模組集中打包能降低初始化成本。然而，預加載和集中這些模組的代價遠超過其帶來的效益。通過重新配置我們的內聯 require 黑名單並從啟動路徑中移除 JavaScript 模組，我們成功避免了加載不必要的模組（例如當只需要 [Relay Modern](https://relay.dev/docs/new-in-relay-modern) 時卻加載了 Relay Classic）。如今，我們的 `RUN_JS_BUNDLE` 分解階段速度提升了超過 75%。

我們還通過研究產品特定的原生模組發現了優化空間。例如，通過延遲注入某個原生模組的依賴項，我們將該模組的成本降低了 98%。此外，通過減少 Marketplace 啟動與其他產品之間的競爭，我們等效地減少了啟動時間。

最棒的是，這些改進中有許多可以廣泛應用於所有使用 React Native 構建的畫面。

## 結論

人們通常認為 React Native 的啟動效能問題是由 JavaScript 運行緩慢或網絡時間過長所導致。雖然加速 JavaScript 等操作確實能顯著降低 TTI（Time to Interaction），但這些因素對 TTI 的影響比例實際上比過去認為的要小得多。

目前學到的教訓是：_測量、測量、再測量！_ 有些優化來自於將運行時成本轉移到構建時，例如 Relay Modern 和 [Lazy NativeModules](https://github.com/facebook/react-native/commit/797ca6c219b2a44f88f10c61d91e8cc21e2f306e)。其他優化則來自於通過更智能地並行化代碼或移除無用代碼來避免工作。還有一些優化來自於 React Native 的大型架構變更，例如清理線程阻塞問題。效能優化沒有萬能解，長期的效能提升將來自於逐步的檢測和改進。切勿讓認知偏差影響決策，而應仔細收集和解讀生產環境數據來指導未來工作。

## 未來計劃

長期來看，我們希望 Marketplace 的 TTI 能與使用原生技術構建的類似產品相當，並讓 React Native 的整體效能與原生效能持平。此外，儘管本季度我們已將 React Native 橋接層的啟動成本大幅降低了約 80%，但我們計劃通過 [Prepack](https://prepack.io/) 等項目和更多的構建時處理，將橋接層成本進一步趨近於零。