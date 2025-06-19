---
title: 'React Native Core Contributor Summit 2024 Recap'
authors: [thymikee, szymonrybczak, mojavad, stmoy]
image: https://raw.githubusercontent.com/facebook/react-native-website/9915d9c0b32ef348958c8119f6e83e571c1c0ba3/website/static/blog/assets/react-native-core-contributor-summit-2024-1.jpeg
tags: [engineering]
date: 2025-02-03
---

每年，React Native 社群的核心貢獻者都會與 React Native 團隊齊聚一堂，共同規劃這個專案的發展方向。

去年也不例外——只有一個小變化。我們通常會在 [React Universe Conf](https://www.reactuniverseconf.com)（前身為 React Native EU）前一天於 [Callstack](https://www.callstack.com/open-source) 位於弗羅茨瓦夫的總部舉行會議。2024 年，我們從過往經驗中學習，將峰會延長為連續兩天，以便有更多非結構化的交流時間。

![all-participants](/blog/assets/react-native-core-contributor-summit-2024-1.jpeg)

<!--truncate-->

這項年度傳統已成為貢獻者分享見解、表達關切的寶貴機會，同時也讓核心團隊能分享計畫並收集來自 React Native 生態系關鍵貢獻者的反饋——包括合作企業、獨立函式庫作者與夥伴們。

我們將峰會分為兩大議程軌，涵蓋以下主題：

- [版本發布](#releases)
- [新架構之後的下一步？](#whats-next-after-the-new-architecture)
- [原生模組的 Web API 規範](#web-apis-for-native-modules)
- [LeanCore 2.0](#leancore-20)
- [Nitro 模組 - 透過將 props 暴露為 jsi::Value 來解除視圖元件限制](#nitro-modules---unblocking-view-components-by-exposing-props-as-jsivalues)
- [樹外平台與 CocoaPods](#out-of-tree-platforms--cocoapods)
- [桌面端的 React Native](#react-native-on-desktop)

在這篇部落格中，我們想讓您先睹為快這次聚會的成果。

## 版本發布

我們針對 React Native 的發布流程進行了深入討論。核心團隊認同讓 Meta 外部貢獻者參與版本發布的價值，並強調每日構建（nightly releases）的重要性——這對樹外平台（如 React Native visionOS）、函式庫維護者（Reanimated）和框架（Expo）特別有益。我們討論了發布頻率，部分與會者希望加快發布節奏以更快推送修復，其他人則對第三方函式庫的影響與升級成本表示擔憂。

我們也集思廣益如何減少非預期的破壞性變更，並改善 React Native 與第三方依賴套件相容性的溝通。

這場會議顯示了管理 React Native 版本發布的複雜性，以及這個議題的敏感性——畢竟需要考量生態系中所有不同的環節。

## 新架構之後的下一步？

在新架構已穩定發布的此刻，我們討論了接下來該聚焦的方向。下一個重大里程碑可能是什麼？討論圍繞以下主題展開：

- **網頁相容性** - 討論圍繞著 React Strict DOM 專案的方向，該專案應被視為暫時性的 polyfill，同時 Xplat 團隊將在 React Native 核心中實現適當的跨平台功能。
- **穩定核心 API** - 我們發現需要更多共識來釐清這對應用程式開發者、函式庫作者和樹外平台意味著什麼。例如可能需要從共享的 C++ 程式碼庫中提取 iOS 和 Android 的平台原生邏輯。這部分內容與 LeanCore 2.0 的討論有所重疊。
- **舊架構支援** - 正如預期，團隊確認基於並行渲染的 React 19 新功能將無法在舊架構中運作。新功能主要針對新架構設計。由於 React 19 發布時程的阻礙，目前尚不清楚新舊架構之間的功能支持界線應劃在何處。
- **React Native 第三方函式庫** - 現今函式庫作者可以使用 TurboModules、ExpoModules，以及最近的 NitroModules 來實現橋接原生平台功能的相同目標。我們需要更好的文件來說明如何完善這項工作。
- **Brownfield 文件** - 在峰會舉行時，官方關於將 React Native 整合到原生應用中的文件已相當過時。此後團隊已跟進發布了更新且更簡潔的 Android 和 iOS 文件。
- **Metro web 的 Tree-shaking** - 核心 Metro 團隊願意合併 Expo 團隊在這方面的工作。

## 原生模組的 Web API

本場會議專注於微軟提出的 RFC，其核心思想是將 Web API 的子集引入 React Native。這旨在通過利用熟悉的 API 來增強 React Native 的可擴展性並吸引更多網頁開發者，從而開放大量現有開源網頁函式庫的使用，這些函式庫目前沒有明確的 React Native 支持。

![web-apis](/blog/assets/react-native-core-contributor-summit-2024-2.jpeg)

基於 Web API 規範進行標準化不僅有益，而且對 React Native 的成長至關重要，這與我們的「多平台」願景和 react-strict-dom 專案高度契合。網頁通過其規範提供了一個統一的介面，而這正是 React Native 社群模組目前所缺乏的。微軟已識別出約 200 個基本的 Web API，可優先為他們支持的平台實現：iOS、Android、Windows 和 macOS。

我們鼓勵函式庫開發者在可能的情況下使其 API 與網頁規範保持一致，這種標準化將提升跨平台程式碼的可攜性和開發者體驗。

雖然這項提案對 React Native 的未來看似有益，但我們仍在集思廣益下一步該如何推進。我們注意到的一個問題是 API 的治理，以及它們是否需要與平台實現分開存放在不同的儲存庫中。另一個問題是當特定平台允許不符合 W3C 規範的行為時，該如何處理與官方規範的分歧。我們需要找出方法來避免捆綁不必要的模組，例如通過 Babel 插件。更不用說這項計劃的範圍相當龐大。

會議結論強化了兩個關鍵點：首先，React Native 社群在可能的情況下採用網頁相容規範方面有強烈共識。其次，我們需要為這些 Web API 實現建立清晰的技術策略，說明如何針對不同平台分別維護。微軟與 Callstack 可以合作完善原始 RFC，並作為社群倡議為少量 API 製作概念驗證實現。這種漸進式方法將幫助我們在擴大範圍前驗證設計和開發者體驗。

## LeanCore 2.0

2019 年，React Native 團隊啟動了 LeanCore 計劃，目標是處理 React Native 核心的覆蓋範圍，減少過時和遺留的 API 與元件。自那時起，React Native 的元件和 API 覆蓋範圍早已需要另一輪清理。

現今有許多元件沒有積極維護，且有更好的社群替代方案。此外，有些元件存在重複，最終應進行整合以提高可維護性。

在 API 方面，許多 JS 層 API 與原生 iOS 和 Android 實現綁定，而非真正的平台無關。例如 Pressable 有像 `android_disableSound` 和 `android_ripple` 這樣的屬性。理想情況下，React Native 元件應具有最小的 API 覆蓋範圍，且不與任何特定平台綁定。

隨著樹外平台（Out-of-Tree platforms）的成長並被生態系更多採用，需要有一條途徑來減少 React Native 核心的元件與 API 表面積，減輕 React Native 核心團隊的負擔，同時讓樹外平台與函式庫維護者能更容易跟上最新進展。

另一個好處是，這將使初學的應用程式開發者更容易上手 React Native，因為需要學習的重複元件和「陷阱」減少了。當有更好的社群替代方案時，可以引導開發者並鼓勵他們使用現有的社群替代方案。

在會議期間，我們討論了：

- Lean Core 的高層次動機以及對相關各方（開發者、函式庫維護者、Meta）的好處
- 一些真實生產環境中 React Native 應用所使用的元件彙總視圖
- 判斷哪些候選項目應從核心中移除的標準
- 執行 Lean Core 2.0 的明確行動計劃，包括：
  - 棄用的高層次流程
  - 處理 Meta 內部使用但已有更好社群替代方案的元件的情況

作為下一步，一群核心貢獻者將著手收集更多遙測數據、評估社群替代方案，並擬定一份 RFC 詳細說明提議的變更。

## Nitro 模組 - 透過將 props 暴露為 jsi::Values 來解除視圖元件的阻塞

近期，Marc Rousavy 引入了 Nitro 模組作為創建原生模組的另一種方法。Nitro 模組利用了實驗性的 C++ Swift 互操作性，並整合了一系列增強功能，在某些情境下可帶來效能提升。然而，在此次會議中，我們討論了 Nitro 模組與現有 Turbo 模組之間的各種權衡取捨。

雖然 Nitro 模組提供了一些效能優勢，但它們也有需要解決的限制和考量。例如，使用實驗性的互操作性功能可能會引入 Turbo 模組中不存在的複雜性或相容性問題。我們的討論聚焦於這些權衡取捨，以及將 Nitro 模組的部分改進上遊至 React Native 核心的可能性，這可能讓所有開發者都能受益於更高性能的模組。

## 樹外平台與 CocoaPods

樹外平台展現了 React Native 的完整能力，讓我們能在行動裝置、桌面甚至 VR/XR 裝置等不同平台上共享同一份 JS 程式碼庫。目前創建這樣的平台並非最簡單的過程，實際上並沒有關於應如何創建、開發和維護的指南。此外，React Native 核心在某種程度上仍與 Android 和 iOS 平台綁定。未來我們可以朝向所有平台被平等對待並透過相同 API 與 C++/JS 核心整合的目標邁進。

![oot-platforms](/blog/assets/react-native-core-contributor-summit-2024-3.jpeg)

在此次會議中，不同平台的維護者討論了他們面臨的問題、困難以及應如何統一創建和維護新樹外平台的流程。

會議的另一個面向是討論 CocoaPods 及與管理原生依賴相關的未來計劃。近期 CocoaPods 團隊宣布他們已進入維護模式，將不再發布重大的改進或新功能。會中我們討論了各種替代方案的優缺點，以及遷移可能面臨的情況。

## 桌面端的 React Native

來自 Microsoft 的 Steven 和 Saad（react-native-windows 和 react-native-macos 的維護者）主持了一場會議，聽取並收集貢獻者關於桌面平台的意見。討論的主題包括如何提高 React Native 在桌面端的採用率（例如在 Visual Studio 中提供專用工作流程，或將桌面端作為 Nx 的一部分公開），以及如何支援 Expo（這一直是提高採用率的一大痛點）。

macOS 與 Windows 在社群模組的可用性上存在顯著差異，主要原因在於 iOS 程式碼大多能與 macOS 相容，而 React Native for Windows (RNW) 則需要專門的實作。當團隊為 React Native for Windows 的新架構進行開發時，他們發現 C++ 模組具有潛力，能進一步實現跨平台程式碼共享，這有望減輕針對桌面平台的開發負擔。值得注意的是，在社群方面，Software Mansion 正致力於為其最受歡迎的模組（如 React Native Screens、Gesture Handler 和 Reanimated）添加桌面支援。

---

我們仍對短短幾天的密集交流所產生的知識共享與創意激盪感到驚嘆。在此次高峰會中，我們為多項倡議播下種子，這些倡議將協助我們改進並重塑 React Native 生態系。

若您有意參與 React Native 的開發，請務必加入我們的開放倡議並閱讀官網上的[貢獻指南](https://reactnative.dev/contributing/overview)。期待未來也能與您線下相會！