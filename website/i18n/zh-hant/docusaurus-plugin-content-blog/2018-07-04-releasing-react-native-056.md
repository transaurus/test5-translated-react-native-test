---
title: Releasing 0.56
author: Lorenzo Sciandra
authorTitle: Core Maintainer & React Native Developer at Drivetribe
authorURL: 'https://github.com/kelset'
authorImageURL: 'https://avatars2.githubusercontent.com/u/16104054?s=460&v=4'
authorTwitter: kelset
tags: [announcement, release]
---

眾所期待的 React Native 0.56 版本現已發布 🎉。本篇部落格文章將重點介紹此新版本中的一些[變更](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md#highlights)。我們也想藉此機會說明自三月以來我們忙碌的事項。

### 破壞性變更的兩難，或「何時發布？」

The [Contributor's Guide](https://github.com/facebook/react-native/blob/master/CONTRIBUTING.md) explains the integration process that all changes to React Native go through. The project has is composed by [many different tools](https://github.com/facebook/react-native-website/issues/370), requiring coordination and constant support to keep everything working properly. Add to this the vibrant open source community that contributes back to the project, and you will get a sense of the mind-bending scale of it all.

由於 React Native 的廣泛採用，破壞性變更必須非常謹慎地進行，且流程不如我們所希望的那麼順暢。我們決定跳過四月和五月的發布，以便核心團隊能夠整合和測試一系列新的破壞性變更。過程中使用了[專屬的社群溝通](https://github.com/react-native-community/react-native-releases/issues/14)管道，以確保 2018 年六月（`0.56.0`）的發布對於耐心等待穩定版本的用戶來說能夠盡可能順利地採用。

Is `0.56.0` perfect? No, as every piece of software out there: but we reached a point where the tradeoff between "waiting for more stability" versus "testing led to successful results so we can push forward" that we feel ready to release it. Moreover, we are aware [of](https://github.com/facebook/react-native/issues/19955) [a](https://github.com/facebook/react-native/issues/19827) [few](https://github.com/facebook/react-native/issues/19763) [issues](https://github.com/facebook/react-native/issues/19859) that are not solved in the final `0.56.0` release. Most developers should have no issues upgrading to `0.56.0`. For those that are blocked by the aforementioned issues, we hope to see you around in our discussions and we are looking forward to working with you on a solution to these issues.

您可以將 `0.56.0` 視為邁向更穩定框架的基礎構建塊：可能需要一兩周的廣泛採用後，所有邊緣案例才會被磨平，但這將帶來更好的 2018 年七月（`0.57.0`）版本。

我們想在本節結束時感謝[所有 67 位貢獻者，他們總共提交了 818 次提交](https://github.com/facebook/react-native/compare/v0.55.4...v0.56.0-rc.4) (!)，這些提交將幫助您的應用變得更好 👏。

現在，事不宜遲...

## 重大變更

### Babel 7

如您所知，讓我們能夠使用 JavaScript 最新和最強大功能的轉譯工具 Babel 即將[升級到 v7](https://babeljs.io/blog/2017/12/27/nearing-the-7.0-release)。由於此新版本帶來了一些重要變更，我們認為現在是升級的好時機，讓 [Metro](https://github.com/facebook/metro) 能夠[利用其改進](https://github.com/facebook/metro/issues/92)。

如果您在升級過程中遇到困難，請參考[相關的說明文件](https://new.babeljs.io/docs/en/next/v7-migration.html)。

### 現代化 Android 支援

在 Android 方面，周邊工具鏈已大幅更新。我們已升級至 [Gradle 3.5](https://github.com/facebook/react-native/commit/699e5eebe807d1ced660d2d2f39b5679d26925da)、[Android SDK 26](https://github.com/facebook/react-native/commit/065c5b6590de18281a8c592a04240751c655c03c)、[Fresco 至 1.9.0 版及 OkHttp 至 3.10.0 版](https://github.com/facebook/react-native/commit/6b07602915157f54c39adbf0f9746ac056ad2d13)，甚至將 [NDK API 目標版本設為 API 16](https://github.com/facebook/react-native/commit/5ae97990418db613cd67b1fb9070ece976d17dc7)。這些變更應能無縫銜接，並帶來更快的建置速度。更重要的是，這將協助開發者符合下個月生效的 [Play 商店新規範](https://android-developers.googleblog.com/2017/12/improving-app-security-and-performance.html)。

在此特別感謝 [Dulmandakh](https://github.com/dulmandakh) 提交大量 PR 促成這些更新 👏。

未來還有更多相關工作需要推進，您可透過 [專屬議題](https://github.com/facebook/react-native/issues/19297)（以及 [JSC 相關的獨立議題](https://github.com/facebook/react-native/issues/19737)）參與後續 Android 支援更新的規劃討論。

### 新版 Node、Xcode、React 與 Flow 登場！

Node 8 現已成為 React Native 的標準環境。雖然先前已進行過測試，但隨著 Node 6 進入維護模式，我們正式全面採用。React 也同步更新至 16.4 版，帶來大量錯誤修正。

我們將停止支援 iOS 8，[改以 iOS 9 作為最低支援版本](https://github.com/facebook/react-native/commit/f50df4f5eca4b4324ff18a49dcf8be3694482b51)。由於所有能運行 iOS 8 的裝置皆可升級至 iOS 9，此變更應無相容性問題。這也讓我們能移除專為 iOS 8 舊裝置設計的罕用相容性程式碼。

持續整合工具鏈已更新為 [採用 Xcode 9.4](https://github.com/facebook/react-native/commit/c55bcd6ea729cdf57fc14a5478b7c2e3f6b2a94d)，確保所有 iOS 測試皆在 Apple 最新開發工具環境下執行。

我們升級至 [Flow 0.75](https://github.com/facebook/react-native/commit/6264b6932a08e1cefd83c4536ff7839d91938730) 以使用 [廣受開發者好評](https://twitter.com/dan_abramov/status/998610821096857602) 的新錯誤格式，同時為更多元件加入型別定義。若您的專案尚未導入靜態型別檢查，建議使用 Flow 在編碼階段即時發現問題，而非等到執行時期。

### 以及其他眾多改進...

例如 YellowBox 已被 [替換](https://github.com/facebook/react-native/commit/d0219a0301e59e8b0ef75dbd786318d4b4619f4c) 為新實作版本，大幅提升除錯體驗。

完整版本說明請參閱 [更新日誌](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md)，並務必查閱 [升級指南](/docs/upgrading) 以避免移轉至新版本時發生問題。

---

最後提醒：從本週開始，React Native 核心團隊將恢復每月例會。我們會確保讓大家及時了解會議討論內容，並將您的反饋納入未來會議考量。

祝各位編碼愉快！

[Lorenzo](https://twitter.com/Kelset)、[Ryan](https://github.com/turnrye) 以及整個 [React Native 核心團隊](https://twitter.com/reactnative)

**附註：** 一如既往，我們要提醒大家 React Native 仍處於 0.x 版本階段，意味著仍有許多變更正在進行——因此升級時請記住，某些功能可能仍會崩潰或存在問題。在提交 issue 和 PR 時請保持互助精神，並遵守 [行為準則](https://code.fb.com/codeofconduct/)：請記住螢幕另一端始終是人類同仁。