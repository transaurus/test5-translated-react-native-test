---
title: The State of the React Native Community in 2018
author: Lorenzo Sciandra
authorTitle: Core Maintainer & React Native Developer
authorURL: 'https://github.com/kelset'
authorImageURL: 'https://avatars2.githubusercontent.com/u/16104054?s=460&v=4'
authorTwitter: kelset
tags: [announcement]
---

2018年，React Native 社群對我們的開發與溝通方式進行了多項變革。我們相信，幾年後回顧時，這將成為 React Native 的轉捩點。

許多人對 React Native 架構的重寫（廣為人知的 [Fabric](https://github.com/react-native-community/discussions-and-proposals/issues/4)）感到興奮。這將解決 React Native 架構的根本限制，並與 [JSI 和 TurboModules](https://github.com/react-native-community/discussions-and-proposals/issues/40) 共同為 React Native 的未來奠定成功基礎。

2018年最大的轉變是賦能 React Native 社群。從一開始，Facebook 就鼓勵全球開發者參與 React Native 的開源專案。此後，湧現了一批核心貢獻者，負責處理包括發布流程在內的多項事務。

這些成員採取了幾項實質性措施，透過以下資源讓整個社群更有能力塑造專案的未來：

## [`react-native-releases`](https://github.com/react-native-community/react-native-releases) 📬

該存儲庫於一月創建，具有雙重目的：讓所有人能以更協作的方式追蹤新版本發布，並開放討論特定版本應包含哪些內容（例如 [0.57.8](https://github.com/react-native-community/react-native-releases/issues/71) 及其先前版本），任何人都可建議 cherry-pick。

這成為推動我們擺脫每月發布週期的動力，也是目前 0.57.x 版本採用「長期支援」方式的關鍵。

達成這些決策的功勞，有一半要歸於今年創建的另一個存儲庫：

## [`discussions-and-proposals`](https://github.com/react-native-community/discussions-and-proposals) 🗣

該存儲庫於七月創建，擴展了關於 React Native 的開放討論環境。此前，這類需求由主存儲庫中標記為 [`For Discussion`](https://github.com/facebook/react-native/labels/For%20Discussion) 的議題處理，但我們希望將此策略擴展為類似其他函式庫（如 React）的 RFC 流程。

此實驗立即在 React Native 生命週期中發揮作用。Facebook 團隊現正使用社群 RFC 流程來討論 [React Native 的改進方向](https://github.com/react-native-community/discussions-and-proposals/issues/64)，並協調 [Lean Core 專案](https://github.com/react-native-community/discussions-and-proposals/issues/6) 等相關工作的推進。

## [@ReactNativeComm](https://twitter.com/ReactNativeComm) 🐣

我們意識到，這些工作的溝通方式未達預期效果。為了讓大家更容易掌握 React Native 社群的最新動態（從版本發布到活躍討論），我們創建了新的 Twitter 帳號 [@ReactNativeComm](https://twitter.com/ReactNativeComm)，供您追蹤。

若您不使用該社交平台，請記住您仍可透過 GitHub 關注存儲庫；此功能近期已改進，現可設定僅接收發布通知，建議您多加利用。

## 未來展望 🎓

過去 7-8 個月間，核心貢獻者強化了 [React Native 社群 GitHub 組織](https://github.com/react-native-community)，以更自主地推動 React Native 開發，並加強與 Facebook 的協作。但此組織一直缺乏類似專案可能具備的正式結構。

該組織可為廣大開發者社群樹立典範：透過強制執行託管套件/存儲庫的統一標準，提供維護者互助的單一平台，貢獻符合社群共識標準的高品質程式碼。

2019年初，我們將實施這套新的指導方針。歡迎在[專屬討論區](https://github.com/react-native-community/discussions-and-proposals/issues/63)分享您的想法。

我們相信這些變革將使社群更加協作無間，當我們邁向1.0版本時，大家都能透過這份共同努力（甚至更進一步）打造出更出色的應用程式 🤗

---

希望您和我們一樣對社群的未來感到振奮。無論是透過上述儲存庫參與討論，或是產出精彩的程式碼，我們都期待看到各位的投入。

祝您編碼愉快！