---
title: 'React Native Monthly #3'
authors: [grabbou]
tags: [engineering]
---

React Native 每月會議持續進行中！由於多數團隊正忙於產品發布，本月會議稍顯精簡。下個月，我們將於波蘭弗羅茨瓦夫舉辦的 [React Native EU](https://react-native.eu/) 研討會上相聚，記得搶購門票並親臨現場！現在，讓我們看看各團隊的最新動態。

## 團隊參與

第三次會議共有 5 個團隊參與：

- [Callstack](https://github.com/callstack)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)

## 會議紀要

以下是各團隊的更新摘要：

### Callstack

- 近期開源了 [`react-native-material-palette`](https://github.com/callstack-io/react-native-material-palette)，該套件能從圖像提取主色調，協助打造視覺吸引力的應用程式。目前僅支援 Android，未來計劃擴展至 iOS。
- 我們已將熱模組替換（HMR）功能整合進 [`haul`](https://github.com/callstack-io/haul)，並發布多項更新！歡迎查看最新版本。
- React Native EU 2017 即將登場！下個月將是 React Native 與波蘭的主場！請把握最後購票機會[點此搶購](https://react-native.eu/)。

### Expo

- 在 [Snack](https://snack.expo.io) 上新增支援 npm 套件安裝功能（需符合 Expo 內建原生 API 限制）。我們正持續開發多檔案支援與資源上傳功能。[Satyajit](https://github.com/satya164) 將於 [React Native Europe](https://react-native.eu/) 研討會分享 Snack 相關內容。
- 發布 SDK20，新增相機、支付、安全儲存、磁力計、下載暫停/恢復功能，並優化啟動/載入畫面。
- 與 [Krzysztof](https://github.com/kmagiera) 合作開發 [react-native-gesture-handler](https://github.com/kmagiera/react-native-gesture-handler)，歡迎試用並回饋使用 PanResponder 或原生手勢識別器的遷移經驗。
- 實驗性開發 JSC 除錯協定，並處理 [Canny](https://expo.canny.io/feature-requests) 平台上的功能請求。

### Facebook

- 上個月我們討論了 GitHub 問題追蹤系統的管理策略，目標提升專案維護效率。
- 目前未解決問題數量穩定維持在 600 件左右。過去一個月，我們關閉了 690 個長期無活動（定義為 60 天內無回應）的議題，其中 58 件因維護者承諾修復或貢獻者提出有力論據而重新開啟。
- 我們將持續執行自動關閉陳舊議題的機制，目標是確保每個重要議題都能獲得處理。現階段亟需維護者協助分類議題，避免遺漏導致回歸錯誤或重大變更（特別是影響新專案的議題）。有意協助者可參考維護者指南學習分類技巧與 GitHub Bot 操作方式，並加入[議題特遣隊](https://github.com/facebook/react-native/blob/master/bots/IssueCommands.txt)！

### Microsoft

- 新版 Skype 應用程式基於 React Native 開發，以實現跨平台最大程度的程式碼共享。目前採用 React Native 的 Skype 應用已在 Android 和 iOS 應用商店上架。
- 在基於 React Native 開發 Skype 應用過程中，我們針對遇到的錯誤和缺失功能向 React Native 提交了 Pull Request。截至目前，我們已有約 [70 個 PR 被合併](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr%20author%3Arigdern%20)。
- React Native 讓我們能透過單一程式碼庫同時驅動 Android 和 iOS 版 Skype 應用。我們還希望用該程式碼庫支援 Skype 網頁版應用。為實現此目標，我們開發並開源了建構於 React/React Native 之上的輕量層 [ReactXP](https://microsoft.github.io/reactxp/blog/2017/04/06/introducing-reactxp.html)。ReactXP 提供一組跨平台元件，在 iOS/Android 平台會映射到 React Native，在網頁平台則映射到 react-dom。ReactXP 的目標與另一個開源庫 React Native for Web 相似。[ReactXP 常見問題](https://microsoft.github.io/reactxp/docs/faq.html) 中簡要說明了兩者方法的差異。

### Shoutem

- 我們持續努力改進並簡化開發者使用 [Shoutem](https://shoutem.github.io/) 建構應用的體驗。
- 曾開始將所有應用遷移至 react-navigation，但最終決定延後此計劃，直到更穩定版本發布或某個原生導航方案趨於穩定。
- 正在將所有 [擴充功能](https://github.com/shoutem/extensions) 和大多數開源庫（[動畫](https://github.com/shoutem/animation)、[主題](https://github.com/shoutem/theme)、[UI](https://github.com/shoutem/ui)）更新至 React Native 0.47.1 版本。

## 下次會議

下次會議定於 2017 年 9 月 13 日星期三舉行。由於這僅是我們的第三次會議，我們想了解這些紀錄如何讓 React Native 社群受益。若有任何改進會議產出的建議，歡迎透過 [Twitter](https://twitter.com/grabbou) 與我聯繫。