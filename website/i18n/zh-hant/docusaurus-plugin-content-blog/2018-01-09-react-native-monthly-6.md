---
title: 'React Native Monthly #6'
author: Tomislav Tenodi
authorTitle: Founder at Speck
authorURL: 'https://twitter.com/TomislavTenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

React Native 每月會議持續熱烈進行中！請務必查看本文底部關於下次會議的註記。

### Expo

- 恭喜 [Devin Abbott](https://github.com/dabbott) 和 [Houssein Djirdeh](https://twitter.com/hdjirdeh) 預先發布了《Full Stack React Native》一書！該書透過構建多個小型應用程式來引導你學習 React Native。
- 發布了首個（實驗性）版本的 [reason-react-native-scripts](https://github.com/react-community/reason-react-native-scripts)，幫助人們輕鬆嘗試 [ReasonML](https://reasonml.github.io/)。
- Expo SDK 24 已[發布](https://blog.expo.io/expo-sdk-v24-0-0-is-now-available-bfcac3b50d51)！它使用了 [React Native 0.51](https://github.com/facebook/react-native/releases/tag/v0.51.0)，並包含了一系列新功能和改進：在獨立應用中打包圖片（無需首次載入時緩存！）、圖片處理 API（裁剪、調整大小、旋轉、翻轉）、人臉檢測 API、新的發布頻道功能（設置特定頻道的活動發布並回滾）、用於追蹤獨立應用構建的網頁儀表板，以及修復了 Android 多任務處理中 OpenGL 實現的長期錯誤，僅舉幾例。
- 我們將從今年一月開始為 React Navigation 分配更多資源。我們堅信，僅使用 React 元件和如 Animated 和 `react-native-gesture-handler` 這樣的原始功能來構建 React Native 導航是可能且理想的，我們對計劃中的一些改進感到非常興奮。如果你想為社區貢獻，請查看 [react-native-maps](https://github.com/react-community/react-native-maps) 和 [react-native-svg](https://github.com/react-native-community/react-native-svg)，它們都需要一些幫助！

### Infinite Red

- 我們已確定 [Chain React 大會](https://infinite.red/ChainReactConf) 的主題演講者：[Kent C. Dodds](https://twitter.com/kentcdodds) 和 [Tracy Lee](https://twitter.com/ladyleet)。我們將很快開放徵稿。
- [社區聊天](https://community.infinite.red/) 現有 1600 人參與。
- [React Native 新聞通訊](https://reactnative.cc/) 現有 8500 名訂閱者。
- 目前正在研究使 React Native 具備抗崩潰能力的最佳實踐，後續將發布報告。
- 正在為 [Solidarity](https://shift.infinite.red/effortless-environment-reports-d129d53eb405) 添加報告功能。
- 發布了關於 [React Native 和 Android](https://shift.infinite.red/simple-react-native-android-releases-319dc5e29605) 發布的 HOW-TO 指南。

### Microsoft

- 已開始一個[拉取請求](https://github.com/Microsoft/react-native-windows/pull/1419)，將核心 React Native Windows 橋接遷移到 .NET Standard，使其實際上與操作系統無關。希望許多其他 .NET Core 平台可以擴展該橋接，提供自己的線程模型、JavaScript 運行時和 UIManager（例如 JavaScriptCore、Xamarin.Mac、Linux Gtk# 和 Samsung Tizen 選項）。

### Wix

- [Detox](https://github.com/wix/detox)
  - 為了讓我們的端對端測試能夠擴展，我們希望最小化在 CI 上花費的時間，目前正在為 Detox 開發平行化支援。
  - 提交了一個 [pull request](https://github.com/facebook/react-native/pull/16948) 來啟用自訂版本風味的支援，以便更好地支援端對端測試中的模擬。
- [DetoxInstruments](https://github.com/wix/DetoxInstruments)
  - DetoxInstruments 的殺手級功能開發證明是一項非常具有挑戰性的任務，在任何給定時間獲取 JavaScript 回溯需要一個自訂的 JSCore 實現來支援 JS 線程暫停。在 Wix 的應用上內部測試分析器揭示了關於 JS 線程的有趣見解。
  - 該項目目前還不夠穩定以供一般使用，但正在積極開發中，我們希望很快能宣布它。
- [React Native Navigation](https://github.com/wix/react-native-navigation)
  - V2 的開發速度已經大幅提升，到目前為止，我們只有一名開發人員用他 20% 的時間來開發，現在我們有三名開發人員全職投入！
- Android 性能
  - 將 RN 中捆綁的舊版 JSCore 替換為最新版本（基於 webkitGTK 項目的尖端版本，並帶有自訂的 JIT 配置），在 JS 線程上產生了 40% 的性能提升。接下來是編譯它的 64 位版本。這項努力基於 [JSC build scripts for Android](https://github.com/SoftwareMansion/jsc-android-buildscripts)。可以在此跟蹤其當前狀態 [here](https://github.com/DanielZlotin/jsc-android-buildscripts/tree/tip)。

## 下一次會議

已經有一些關於重新利用這次會議來討論一個單一且具體的主題（例如導航、將 React Native 模組移動到單獨的存儲庫、文檔等）的討論。這樣我們覺得我們可以為 React Native 社區做出最好的貢獻。這可能會在下次會議中進行。歡迎在推特上告訴我們你希望看到哪些主題被涵蓋。