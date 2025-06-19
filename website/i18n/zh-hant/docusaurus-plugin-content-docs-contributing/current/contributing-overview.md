---
id: overview
title: Contributing Overview
description: How to contribute to React Native
---

<!-- alex disable simple simply -->

感謝您對貢獻 React Native 感興趣！無論是評論與分類議題、審查或提交 Pull Request，我們歡迎所有形式的貢獻。
我們的目標是建立一個超越主 React Native GitHub 儲存庫、充滿活力且包容的[合作夥伴、核心貢獻者與社群生態系統](https://github.com/facebook/react-native/blob/main/ECOSYSTEM.md)。

The [Open Source Guides](https://opensource.guide/) website has a collection of resources for individuals, communities, and companies who want to learn how to run and contribute to an open source project.

無論是貢獻者或開源新手，以下指南特別實用：

- [如何貢獻開源專案](https://opensource.guide/how-to-contribute/)
- [建立友善社群](https://opensource.guide/building-community/)

### 行為準則

請注意，所有貢獻者均須遵守[行為準則](https://github.com/facebook/react-native/blob/HEAD/CODE_OF_CONDUCT.md)。

## 版本政策

為完整理解 React Native 的版本管理，建議您查閱[版本政策](/contributing/versioning-policy)頁面。
該頁面說明哪些 React Native 版本受支援、發布頻率，以及您應根據自身情況選擇的版本。

## 貢獻方式

若您渴望立即開始貢獻程式碼，我們有一份[適合新手的議題清單](https://github.com/facebook/react-native/labels/good%20first%20issue)，這些錯誤的範圍相對有限。
隨著經驗累積並展現對改進 React Native 的承諾，您可能獲得儲存庫的議題管理權限。

您也可以透過其他方式貢獻，無需撰寫任何程式碼。以下是幾種協助途徑：

1. **回覆與處理開放議題。**

   我們每天收到大量議題，部分可能缺乏必要資訊。您可以引導填寫議題模板、要求釐清細節，或指向與問題描述相符的現有議題。
   更多流程詳見[分類 GitHub 議題](/contributing/triaging-github-issues)頁面。

2. **審查文件更新的 Pull Request。**

   審查[文件更新](https://github.com/facebook/react-native-website/pulls)可簡單至檢查拼字與文法。
   若發現文件需更完善說明之處，點擊文件頁面頂部的**編輯**按鈕即可開始貢獻。

3. **協助撰寫測試計畫。**

   提交至主儲存庫的部分 Pull Request 可能缺乏適當測試計畫。這些計畫能幫助審查者理解變更如何測試，並加速貢獻被接受的流程。

每項任務都極具影響力，維護者將十分感謝您的協助。

### 我們的開發流程

我們使用 GitHub 議題與 Pull Request 追蹤社群錯誤報告與貢獻。Meta 工程師的所有變更會透過與 Meta 內部版本控制的橋接同步至 [GitHub](https://github.com/facebook/react-native)。社群變更則透過 GitHub Pull Request 處理。

GitHub 上的變更獲核准後，會先導入 Facebook 內部版本控制系統，並針對 Facebook 程式碼庫進行測試。合併至 Facebook 後，通過內部測試的變更最終會以單一提交同步回 GitHub。

您可透過以下文件進一步了解貢獻流程：

- [分類 GitHub 議題](/contributing/triaging-github-issues)
- [管理 Pull Request](/contributing/managing-pull-requests)

我們還有一個活躍的貢獻者社群，他們很樂意協助您進行設定。您可以透過 [@ReactNative](https://twitter.com/reactnative) 聯繫 React Native 團隊。

### 儲存庫

主儲存庫包含 React Native 框架本身，我們在這裡追蹤錯誤報告並管理拉取請求。

您可能需要熟悉以下幾個其他儲存庫：

- **React Native 網站** 包含網站的原始碼，包括文件，位於 [此儲存庫](https://github.com/facebook/react-native-website)。
- **版本發布** 的討論正在 [此討論儲存庫](https://github.com/reactwg/react-native-releases/discussions) 中進行。
- **變更日誌** 可以在 [這裡](https://github.com/facebook/react-native/blob/main/CHANGELOG.md) 找到。
- **討論** 關於 React Native 的內容位於 [討論與提案](https://github.com/react-native-community/discussions-and-proposals) 儲存庫。
- **討論** 關於 React Native 新架構的內容位於 [React Native 新架構工作組](https://github.com/reactwg/react-native-new-architecture) 儲存庫。
- **高品質插件** 可以在 [React Native 目錄](https://reactnative.directory) 網站中找到。

瀏覽這些儲存庫應該能讓您了解 React Native 開源專案的管理方式。

## GitHub 問題

我們使用 GitHub 問題來專門追蹤錯誤。我們在 [問題分類頁面](/contributing/triaging-github-issues) 中記錄了我們的問題處理流程。

### 安全性錯誤

Meta 有一個 [漏洞獎勵計劃](https://www.facebook.com/whitehat/) 用於安全地披露安全性錯誤。在這些情況下，請按照該頁面概述的流程進行操作，不要提交公開問題。

## 協助文件

React Native 文件是 React Native 網站儲存庫的一部分。該網站是使用 [Docusaurus](https://docusaurus.io/) 構建的。如果您想更改文件中的任何內容，可以點擊網站大多數頁面右上角的「編輯」按鈕開始。

如果您正在新增功能或引入行為變更，我們會要求您更新文件以反映您的變更。

### 貢獻部落格

React Native 部落格是 [從部落格的 Markdown 原始檔](https://github.com/facebook/react-native-website/tree/HEAD/website/blog) 生成的。

請在 React Native 網站儲存庫中開一個問題，或在 Twitter 上標記我們 [@ReactNative](https://twitter.com/reactnative)，並在撰寫專為 React Native 部落格準備的文章之前獲得維護者的同意。
在大多數情況下，您可能希望在自己的部落格或寫作平台上分享您的文章。不過，詢問一下是值得的，以防我們發現您的文章適合部落格。

我們建議參考 `react-native-website` 儲存庫的 [Readme 文件](https://github.com/facebook/react-native-website#-contributing) 以了解更多關於貢獻網站的資訊。

## 貢獻程式碼

對 React Native 的程式碼層級貢獻通常以 [拉取請求](https://help.github.com/en/articles/about-pull-requests) 的形式進行。這些是通過分叉儲存庫並在本地進行變更來完成的。

### 逐步指南

每當您準備好貢獻程式碼時，請查看我們的 [發送第一個拉取請求的逐步指南](/contributing/how-to-open-a-pull-request)，或閱讀 [如何貢獻程式碼](/contributing/how-to-contribute-code) 頁面以獲取更多詳細資訊。

### 測試

測試能幫助我們防止程式碼庫引入回歸問題。GitHub 儲存庫會透過 CircleCI 持續進行測試，測試結果可透過 [commits](https://github.com/facebook/react-native/commits/HEAD) 和 pull requests 上的 Checks 功能查看。

您可以在[如何執行與撰寫測試](/contributing/how-to-run-and-write-tests)頁面了解更多關於執行與撰寫測試的資訊。

## 社群貢獻

對 React Native 的貢獻不僅限於 GitHub。您可以透過撰寫部落格文章、在研討會上演講，或是在 Twitter 上分享使用 React Native 的經驗並標記 [@ReactNative](https://twitter.com/reactnative) 來幫助其他人。