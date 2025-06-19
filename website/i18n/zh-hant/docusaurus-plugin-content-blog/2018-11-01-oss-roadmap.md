---
title: Open Source Roadmap
author: Héctor Ramos
authorTitle: Engineer at Facebook
authorURL: 'https://hectorramos.com/about'
authorImageURL: 'https://s.gravatar.com/avatar/f2223874e66e884c99087e452501f2da?s=128'
authorTwitter: hectorramos
tags: [announcement]
---

![](/blog/assets/oss-roadmap-hero.jpg)

今年，React Native 團隊專注於大規模的 [React Native 重新架構](https://github.com/react-native-community/discussions-and-proposals/issues/4)。正如 Sophie 在 [《React Native 現狀 2018》](/blog/2018/06/14/state-of-react-native-2018) 中所提到的，我們已擬定計劃以更好地支持 Facebook 外部蓬勃發展的 React Native 用戶和協作者群體。現在是時候分享我們的工作細節了。在此之前，我想先闡明我們對開源版 React Native 的長期願景。

我們對 React Native 的願景是...

- **健康的 GitHub 倉庫。** 問題和拉取請求能在合理時間內得到處理。
  - 提高測試覆蓋率。
  - 從 Facebook 代碼倉庫同步的提交不應破壞開源測試。
  - 更大規模的社區有意義貢獻。
- **穩定的 API，** 讓與開源依賴的交互更輕鬆。
  - Facebook 使用與開源相同的公共 API。
  - React Native 發布遵循語意化版本控制。
- **活躍的生態系統。** 由社區維護的高質量 ViewManager、原生模組和多平台支持。
- **優秀的文檔。** 專注於幫助用戶創造高質量體驗，以及最新的 API 參考文檔。

我們已確定以下重點領域來幫助實現這一願景。

## ✂️ 精簡核心

我們的目標是通過移除非核心和未使用的組件來 [減少 React Native 的表面積](https://github.com/react-native-community/discussions-and-proposals/issues/6)。我們將把非核心組件移交給社區，讓社區能更快地發展。減少表面積將使管理對 React Native 的貢獻變得更容易。

[`WebView`](https://github.com/react-native-community/discussions-and-proposals/blob/master/proposals/0001-webview.md) 是我們移交給社區的一個組件示例。我們正在開發一個工作流程，讓內部團隊在我們從倉庫中移除這些組件後仍能繼續使用它們。我們已確定 [數十個其他組件](https://github.com/react-native-community/discussions-and-proposals/issues/6)，將交由社區負責。

## 🎁 開源內部工具和 🛠 更新工具鏈

Facebook 產品團隊的 React Native 開發體驗可能與開源社區大不相同。開源社區中流行的工具在 Facebook 內部可能並未使用。某些情況下，Facebook 團隊已習慣使用一些外部不存在的工具。這些差異可能對我們開源即將推出的架構工作構成挑戰。

我們將努力發布部分內部工具，並改進對開源社區流行工具的支持。以下是我們將處理的部分項目：

- 開源 JSI 並讓社區能使用自己的 JavaScript 虛擬機，取代現有初始版本中的 JavaScriptCore。我們將在未來文章中詳細介紹 JSI，在此期間你可以從 [Parashuram 在 React Conf 的演講](https://www.youtube.com/watch?v=UcqRXTriUVI) 中了解更多。
- 支持 Android 上的 64 位庫。
- 在新架構下啟用調試功能。
- 改進對 CocoaPods、Gradle、Maven 和新 Xcode 構建系統的支持。

## ✅ 測試基礎設施

當 Facebook 工程師發布代碼時，如果通過所有測試則被認為可以安全合併。這些測試能識別變更是否可能破壞我們自己的 React Native 功能。然而，Facebook 使用 React Native 的方式存在差異，這讓我們可能在不知情的情況下破壞開源版 React Native。

我們將加強內部測試，確保它們在盡可能接近開源的環境中運行。這將有助於防止破壞這些測試的代碼進入開源版本。我們還將致力於基礎設施建設，以便更好地在 GitHub 上測試核心倉庫，讓未來的拉取請求能輕鬆包含測試。

結合精簡後的程式碼範圍，這將讓貢獻者能更有信心地快速合併拉取請求(pull requests)。

## 📜 公開API

Facebook將透過與開源社群相同的公開API來使用React Native，以減少非預期的破壞性變更。我們已開始轉換內部呼叫點來解決此問題。我們的目標是建立穩定的公開API，並在1.0版本中採用語意化版本控制(semantic versioning)。

## 📣 溝通

React Native是GitHub上[貢獻者數量最多的開源專案](https://octoverse.github.com/#top-and-trending-projects)之一。這讓我們感到非常高興，並希望能持續保持。我們將繼續推動提高透明度與開放討論等能促進貢獻者參與的計畫。文件是新接觸React Native的人首先會接觸到的內容，但過去卻非優先事項。我們希望改善這點，從恢復自動生成的API參考文件開始，新增專注於建立[優質使用者體驗](/docs/improvingux)的內容，並改進我們的[版本發布說明](https://github.com/react-native-community/react-native-releases/issues/47)。

## 時間軸

我們計劃在接下來一年左右陸續完成這些專案。部分工作已在進行中，例如[JSI已開源並合併](https://github.com/facebook/react-native/compare/e337bcafb0b017311c37f2dbc24e5a757af0a205...8427f64e06456f171f9df0316c6ca40613de7a20)。其他如精簡程式碼範圍等專案則需要更長時間完成。我們將盡力向社群同步進度。歡迎加入[討論與提案](https://github.com/react-native-community/discussions-and-proposals)儲存庫參與討論，這個由React Native社群發起的平台已促成本路線圖中多項計畫的誕生。