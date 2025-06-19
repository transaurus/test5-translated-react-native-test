---
title: 'React Native Monthly #1'
author: Tomislav Tenodi
authorTitle: Product Manager at Shoutem
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

在 [Shoutem](https://shoutem.github.io/)，我們很幸運能從最初就參與 React Native 的開發。我們決定從第一天起就成為這個優秀社群的一份子。但很快我們發現，幾乎不可能跟上社群飛速成長與進步的節奏。因此我們決定每月舉辦一次會議，讓所有主要的 React Native 貢獻者能簡短分享他們的努力成果與未來計劃。

## 每月會議

我們在 2017 年 6 月 14 日舉行了首次月度會議。React Native 月度會議的使命簡單明確：**提升 React Native 社群**。透過各團隊分享工作進展，促進線下的跨團隊協作。

## 參與團隊

首次會議共有 8 個團隊參與：

- [Airbnb](https://github.com/airbnb)
- [Callstack](https://github.com/callstack-io)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [GeekyAnts](https://github.com/GeekyAnts)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)
- [Wix](https://github.com/wix)

我們期待未來會議能有更多核心貢獻者加入！

## 會議紀要

由於各團隊的計劃可能對更廣泛的讀者有益，我們將在 React Native 官方部落格分享這些內容。以下是各團隊的更新：

### Airbnb

- 計劃為 `View` 和原生模組 `AccessibilityInfo` 新增無障礙功能 (A11y) API
- 研究在 Android 原生模組新增 API 以指定執行緒
- 持續研究初始化效能優化方案
- 探索在 "unbundle" 基礎上更進階的打包策略

### Callstack

- 計劃採用 [Detox](https://github.com/wix/detox) 進行端對端測試來改善發布流程，相關 PR 即將提交
- 已完成 Blob 功能的 PR 合併，後續 PR 將陸續推出
- 在內部專案擴大採用 [Haul](https://github.com/callstack-io/haul)，與 [Metro Bundler](https://github.com/facebook/metro-bundler) 進行效能比較，並與 webpack 團隊合作提升多線程效能
- 內部已建立更完善的開源專案管理基礎設施，預計數週內公開更多成果
- React Native Europe 會議籌備中，誠摯邀請大家參與！
- 暫時減少對 [react-navigation](https://github.com/react-community/react-navigation) 的投入，轉而研究替代方案（特別是原生導航方案）

### Expo

- 正在為 [Snack](https://snack.expo.io/) 新增 npm 模組安裝功能，方便函式庫在文檔中添加範例
- 與 [Krzysztof](https://github.com/kmagiera) 及 [Software Mansion](https://github.com/software-mansion) 團隊合作開發 Android 版 JSC 更新與手勢處理函式庫
- [Adam Miskiewicz](https://github.com/skevy) 將重心轉向 [react-navigation](https://github.com/react-community/react-navigation) 開發
- [Create React Native App](https://github.com/react-community/create-react-native-app) 已納入官方文檔的[入門指南](/docs/getting-started)。Expo 鼓勵函式庫作者明確標註是否相容 CRNA，並提供設定說明

### Facebook

- React Native 的打包工具現已獨立為 [Metro Bundler](https://github.com/facebook/metro) 專案。倫敦的 Metro Bundler 團隊正積極解決社群需求，提升模組化以支援 React Native 以外的使用場景，並加快對問題與 PR 的響應速度。
- 未來幾個月，React Native 團隊將著重優化基礎元件的 API 設計，預計改善佈局異常、無障礙功能與 Flow 類型檢查等問題。
- 團隊今年也計劃透過重構核心模組，全面支援第三方平台如 Windows 和 macOS。

### GeekyAnts

- 團隊正在開發代號為 Builder 的 UI/UX 設計工具，可直接操作 `.js` 檔案，目前專注支援 React Native，功能類似 Adobe XD 和 Sketch。
- 致力實現「在編輯器中載入現有 React Native 應用 → 視覺化修改設計 → 直接保存變更至 JS 檔案」的工作流程。
- 目標是縮小設計師與開發者之間的鴻溝，讓兩者能在同個代碼庫協作。
- 此外，[NativeBase](https://github.com/GeekyAnts/NativeBase) 近期已突破 5,000 GitHub 星標。

### Microsoft

- [CodePush](https://github.com/Microsoft/code-push) has now been integrated into [Mobile Center](https://mobile.azure.com/). This is the first step in providing a much more integrated experience with distribution, analytics and other services. See their announcement [here](https://microsoft.github.io/code-push/articles/CodePushOnMobileCenter.html).
- [VS Code](https://github.com/Microsoft/vscode) has a bug with debugging, they are working on fixing that right now and will have a new build.
- Investigating [Detox](https://github.com/wix/detox) for Integration testing, looking at JSC Context to get variables alongside crash reports.

### Shoutem

- 簡化 Shoutem 應用與 React Native 生態工具的整合，未來可使用標準 React Native 指令運行 [Shoutem](https://shoutem.github.io/) 建立的應用。
- 研究 React Native 效能分析工具，將分享配置過程中遇到的問題與解決方案。
- 開發更流暢的 React Native 與既有原生應用整合方案，計劃公開內部研發的架構概念以獲取社群反饋。

### Wix

- 內部全面採用 [Detox](https://github.com/wix/detox) 實現 Wix 應用的「零人工 QA」流程，數十名開發者在生產環境中密集使用，促使工具快速成熟。
- 擴展 [Metro Bundler](https://github.com/facebook/metro) 功能，支援在構建時覆蓋任意副檔名（如 "e2e" 或 "detox"），現有實驗性庫 [react-native-repackager](https://github.com/wix/react-native-repackager) 正進行 PR 改進。
- 開發開源工具 [DetoxInstruments](https://github.com/wix/DetoxInstruments) 實現效能測試自動化。
- 與 KPN 貢獻者合作推進 Detox 的 Android 支援與實機測試功能。
- 規劃「Detox 平台化」，支援構建如 [Storybook](https://github.com/storybooks/react-native-storybook) 等需自動化操作模擬器/設備的工具。

## 下次會議

會議將每四周舉行一次。下一次會議定於2017年7月12日。由於我們才剛開始舉辦此會議，我們想知道這些筆記如何對React Native社群有所助益。如果您對接下來會議應涵蓋的內容，或是我們應如何改進會議成果有任何建議，歡迎[在Twitter上聯繫我](https://twitter.com/TomislavTenodi)。