---
title: 'React Native Monthly #5'
author: Tomislav Tenodi
authorTitle: Founder at Speck
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

React Native 月度會議持續進行中！來看看我們的團隊最近在忙些什麼。

### Callstack

- 我們一直在改進 React Native 的持續整合（CI）系統。最重要的是，我們已從 Travis 遷移至 CircleCI，讓 React Native 擁有統一且單一的 CI 流程。
- 我們舉辦了 [Hacktoberfest - React Native 版本](https://blog.callstack.io/announcing-hacktoberfest-7313ea5ccf4f)，與參與者一起嘗試提交許多開源專案的 Pull Request。
- 我們持續開發 [Haul](https://github.com/callstack/haul)。上個月，我們發布了兩個新版本，包括支援 webpack 3。我們計劃新增 [CRNA](https://github.com/react-community/create-react-native-app) 和 [Expo](https://github.com/expo/expo) 的支援，並改進熱模組替換（HMR）功能。我們的開發路線圖已公開在 issue 追蹤器中。如果您有任何建議或反饋，歡迎告訴我們！

### Expo

- 發布了 [Expo SDK 22](https://blog.expo.io/expo-sdk-v22-0-0-is-now-available-7745bfe97fc6)（使用 React Native 0.49），並更新了 [CRNA](https://github.com/react-community/create-react-native-app) 以支援此版本。
  - 包含改進的啟動畫面 API、基礎 ARKit 支援、「DeviceMotion」API、iOS11 的 SFAuthenticationSession 支援，以及[更多功能](https://blog.expo.io/expo-sdk-v22-0-0-is-now-available-7745bfe97fc6)。
- 您的 [Snack](https://snack.expo.io) 現在可以包含多個 JavaScript 檔案，並且只需將圖片或其他資源拖曳至編輯器即可上傳。
- 貢獻至 [react-navigation](https://github.com/react-community/react-navigation)，新增對 iPhone X 的支援。
- 專注於解決使用 Expo 開發大型應用程式時的痛點。例如：
  - 提供對多環境部署（如 staging、production 及任意頻道）的一流支援。頻道將支援回滾及設定特定頻道的當前版本。如果您想成為早期測試者，請聯繫 [@expo_io](https://twitter.com/expo_io)。
  - 我們也在改進獨立應用程式的建置基礎設施，並新增在獨立應用程式中打包圖片及其他非程式碼資源的功能，同時保留透過空中下載（OTA）更新資源的能力。

### Facebook

- 改進 RTL（從右至左）支援：
  - 我們引入了一系列方向感知的樣式：
    - 定位：
      - (left|right) → (start|end)
    - 邊距：
      - margin(Left|Right) → margin(Start|End)
    - 內距：
      - padding(Left|Right) → padding(Start|End)
    - 邊框：
      - borderTop(Left|Right)Radius → borderTop(Start|End)Radius
      - borderBottom(Left|Right)Radius → borderBottom(Start|End)Radius
      - border(Left|Right)Width → border(Start|End)Width
      - border(Left|Right)Color → border(Start|End)Color
  - 在 RTL 模式下，「left」和「right」的意義在定位、邊距、內距及邊框樣式中會被交換。幾個月後，我們將移除此行為，讓「left」始終表示「左」，「right」始終表示「右」。這項破壞性變更目前隱藏在標誌下。您可以在 React Native 元件中使用 `I18nManager.swapLeftAndRightInRTL(false)` 來啟用這些變更。
- 正在為我們的內部原生模組進行 [Flow](https://github.com/facebook/flow) 類型檢查，並使用這些類型生成 Java 的介面及 Objective-C 的協議，原生實作必須遵循這些規範。我們希望這項程式碼生成工具能在明年開源，最早的話。

### Infinite Red

- 新的開源工具，用於協助 React Native 及其他專案。詳情請見[這裡](https://shift.infinite.red/solidarity-the-cli-for-developer-sanity-672fa81b98e9)。
- 正在翻新 [Ignite](https://github.com/infinitered/ignite)，準備新的樣板版本（代號：Bowser）。

### Shoutem

- 改進 Shoutem 上的開發流程。我們希望從創建應用程式到第一個自訂畫面的過程能夠更加流暢，讓新接觸 React Native 的開發者更容易上手，從而降低入門門檻。我們準備了幾場工作坊來測試新功能。同時也改進了 [Shoutem CLI](https://github.com/shoutem/cli) 以支援新的流程。
- [Shoutem UI](https://github.com/shoutem/ui) 進行了一些元件改進和錯誤修復。我們也檢查了與最新版 React Native 的相容性。
- Shoutem 平台迎來了一些重要更新，新的整合功能已作為[開源擴充專案](https://github.com/shoutem/extensions)的一部分推出。我們非常高興看到其他開發者對 Shoutem 擴充功能的積極開發。我們會主動聯繫並提供關於他們擴充功能的建議和指導。

## 下次會議

下次會議定於 2017 年 12 月 6 日星期三舉行。如果你對如何改進會議成果有任何建議，歡迎[在 Twitter 上聯繫我](https://twitter.com/TomislavTenodi)。