---
title: React Native Open Source Update March 2019
authors: [cpojer]
tags: [announcement]
---

我們在2018年第四季宣布了[React Native開源路線圖](/blog/2018/11/01/oss-roadmap)，當時我們決定要更積極投入React Native開源社群。

在第一個里程碑階段，我們專注於識別並改善社群中最顯著的問題。目標是減少積壓的pull request、縮減專案範圍、找出主要使用者痛點，並建立社群管理準則。

過去兩個月，我們的進展超出預期。以下是詳細內容：

### Pull Requests

要建立健康的社群，必須快速回應程式碼貢獻。過去幾年我們未優先審核社群貢獻，導致積壓280個pull request（2018年12月）。在第一個里程碑期間，我們將未處理的pull request數量降至約65個。同時，每日平均新增的pull request從3.5個增加到7個，意味著我們在過去三個月處理了約[600個pull request](https://github.com/facebook/react-native/pulls?page=24&q=is%3Apr+closed%3A%3E2018-12-01&utf8=%E2%9C%93)。

我們合併了[近三分之二](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr+closed%3A%3E2018-12-01+label%3A%22Merged%22+)，並關閉了三分之一的pull request。若內容過時、品質不佳或不必要地增加專案範圍，我們會直接關閉而不合併。多數合併的pull request修復了錯誤、改善跨平台一致性或新增功能。值得注意的貢獻包括改進型別安全性，以及支援AndroidX的持續工作。

在Facebook內部，我們直接運行master分支的React Native，因此會先測試所有變更才會納入正式版本。在所有合併的pull request中，僅有六個造成問題：四個僅影響內部開發，兩個在發佈候選版本階段就被發現。

較顯著的社群貢獻之一是[更新版的「RedBox」錯誤畫面](https://github.com/facebook/react-native/pull/22242)，這充分展現社群如何讓開發者體驗更友善。

### Lean Core

React Native目前範圍過廣，包含許多Facebook較少使用的未維護抽象層。我們正努力縮減範圍，讓React Native更精簡，同時讓社群能更好地維護這些在Facebook較少使用的抽象層。

在第一個里程碑期間，[我們向社群尋求協助進行Lean Core專案](https://twitter.com/reactnative/status/1093171521114247171)。反應非常熱烈，我們幾乎跟不上進度。[查看不到一個月內完成的所有工作](https://github.com/facebook/react-native/issues/23313)！

最令人振奮的是，維護者們主動修復長期問題、新增測試，並支援期待已久的功能。這些模組獲得的支援遠超過在React Native核心時期，證明這對社群是重要的一步。例如[WebView](https://github.com/react-native-community/react-native-webview)在抽離後[收到大量pull request](https://twitter.com/titozzz/status/1101283928026034176)，以及現在[由社群成員維護的CLI工具](https://blog.callstack.io/the-react-native-cli-has-a-new-home-79b63838f0e6)獲得了急需的改進與修復。

### 主要使用者痛點

去年12月，我們詢問社群[對React Native的不滿之處](https://github.com/react-native-community/discussions-and-proposals/issues/64)。我們彙整回應並[逐一回覆每個問題](https://github.com/react-native-community/discussions-and-proposals/issues/104)。幸運的是，許多社群面臨的問題也是Facebook內部的痛點。在下個里程碑，我們計畫解決部分主要問題。

其中獲得最高票數的問題之一，是開發者在升級至新版 React Native 時的體驗問題。遺憾的是，由於我們直接從 master 分支運行 React Native，自身並未遇到此問題。值得慶幸的是，社群成員已主動著手解決這個問題：

- [Michał Pierzchała](https://github.com/thymikee) from Callstack [improved react-native upgrade](https://github.com/react-native-community/react-native-cli/pull/176/files) by using [rn-diff-purge](https://github.com/react-native-community/rn-diff-purge) under the hood. We also updated the website to remove outdated upgrade instructions.
- [We plan on recommending CocoaPods by default for iOS projects](https://github.com/facebook/react-native/pull/23563) which will reduce churn in project files when upgrading React Native. This will make it easier for people to install and link third-party modules which is even more important in the context of Lean Core as we expect projects to link more modules by default.

### 0.59 版本發布

若沒有 React Native 社群的協助，特別是 [Mike Grabowski](https://github.com/grabbou) 和 [Lorenzo Sciandra](https://github.com/kelset)，我們將無法順利發布版本。我們希望改善版本管理流程，並計劃從現在開始更積極參與：

- 我們將與社群成員合作，為每個主要版本撰寫部落格文章。
- 我們會在 CLI 中直接顯示版本升級時的破壞性變更。
- 我們將縮短發布版本所需的時間。目前正探索增加自動化測試的方法，並建立更完善的手動測試計畫。

這些計劃多數將納入即將發布的 [React Native 0.59 版本](https://github.com/facebook/react-native/releases/tag/v0.59.0-rc.3)。0.59 版將包含 React Hooks、Android 版 64 位元 JavaScriptCore，以及許多效能與功能改進。目前該版本已發布為候選版本，預計在兩週內推出穩定版。

### 後續步驟

接下來兩個月，我們將持續管理 pull request 以[維持進度](https://k03lwm00zo.codesandbox.io/)，同時開始減少未解決的 GitHub issue 數量。我們會透過 Lean Core 專案持續縮減 React Native 的範疇，並計劃解決 5 項最高票的社群問題。在完成社群準則後，我們將把注意力轉向官網與文件。

我們非常期待三月份能在 Facebook 倫敦辦公室接待十多位社群貢獻者，共同推動這些計畫。很高興您正在使用 React Native，希望您能看見並感受到我們在 2019 年的改進成果。幾個月後我們會再帶來更新，*期間也將持續合併您的 pull request！* ⚛️✌️