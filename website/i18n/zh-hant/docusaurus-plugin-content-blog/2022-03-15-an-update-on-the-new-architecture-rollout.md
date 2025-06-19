---
title: An update on the New Architecture Rollout
authors: [cortinico]
tags: [announcement]
date: 2022-03-15
---

各位好，
[如先前所宣布](/blog/2022/01/21/react-native-h2-2021-recap#the-new-architecture-rollout-and-releases)：

> 2022年將是開源新架構的元年

如果您尚未有時間了解React Native新架構（Fabric渲染器與TurboModule系統），**現在**正是最佳時機！

我們希望與社群分享一些我們準備的計畫與資源，確保所有人都能參與這項重要變革。

<!--truncate-->

### 工作小組

Recently, we launched the [React Native New Architecture Working Group](https://github.com/reactwg/react-native-new-architecture) on GitHub, a _discussion only_ repository to coordinate and support the rollout of the New Architecture across the ecosystem.

我們將此工作小組視為一個讓社群能**聚集**、分享**想法**，並**討論**新架構採用過程中挑戰的空間。此外，我們將透過此工作小組向更廣泛的社群**分享**資訊與更新，以確保透明度。

為保持討論聚焦，我們決定讓此工作小組**公開可讀**，但**僅限核准用戶可寫入**。

若您希望參與討論，可[填寫此表單](https://forms.gle/8emgdwFZXuzEpyyn9)來**申請或提名**您認為能為討論帶來價值的人選。

**歡迎所有人**申請加入討論。

如同所有討論論壇，我們要再次強調**尊重**與接納他人意見的重要性。請把握機會閱讀我們的[**行為準則**](https://github.com/reactwg/react-native-new-architecture/blob/main/CODE_OF_CONDUCT.md)（若您尚未閱讀）。

### 遷移指南

After several rounds of review & feedback, we finally merged **the Migration Guide** (f.k.a. _the Playbook_). You can find it [on the New Architecture working group.](https://github.com/reactwg/react-native-new-architecture#guides)

這份遷移指南將以逐步教學方式，向您展示**如何建立自訂Fabric元件或TurboModule**，同時也會說明如何**調整現有應用程式或函式庫**以採用新架構。

此外，我們要提醒您官網全新的[架構專區](/architecture/overview)。您可在該區找到多篇深入探討React Native內部機制的文章。特別是[Fabric章節](/architecture/fabric-renderer)能幫助您理解新架構下的渲染管線。

Finally, please consider **sharing your feedback** to this documentation material [on the working group](https://github.com/reactwg/react-native-new-architecture/discussions/7). We’re constantly looking for developer’s opinion, and we want to make sure we’re delivering the content that you find most useful.

未來幾個月，我們將持續完善並新增更多文件來進一步協助您。

### 新架構範本

React Native **0.68.0** is close to release. This version of React Native marks a crucial milestone in the New Architecture Rollout as it’s the first version to include an **opt-in switch** in the **new app template.**

這意味著您只需修改範本中的**一行代碼**即可試用新架構。我們還在範本中添加了大量**註解與文件**，確保您無需額外閱讀就能開箱即用。我們希望透過**減少需編寫的代碼量**來幫助您採用新架構。

<!-- alex ignore simple -->

在接下來的版本中，我們將持續更新模板，使其更加精簡且易於使用。

要在任一平台上啟用新架構，您可以：

- On iOS, run `RCT_NEW_ARCH_ENABLED=1 bundle exec pod install` inside the `ios` folder.
- On Android, set the `newArchEnabled` property to `true` by **either**:
  - Changing the corresponding line inside the `android/gradle.properties` file.
  - Set an environment variable `ORG_GRADLE_PROJECT_newArchEnabled=true`
  - Invoke Gradle with `-PnewArchEnabled=true`

Then you can **run your app** with `yarn react-native run-android` or `run-ios` and you’ll be running using Fabric and TurboModules enabled.

Please consider trying this new template, and [report any bug or unexpected behavior](https://github.com/reactwg/react-native-new-architecture/discussions/5) that you might face. Over the last months we worked hard to fix bugs and build failures that would have been **hard to catch** without the constant community feedback and testing.

### 第三方函式庫生態系統

若沒有**第三方函式庫作者與維護者**的全面支援，社群將無法順利遷移至新架構。

We understand how this can be a tedious process, and we understand the importance of supporting users on **both** old and New Architecture. Over the next months, we will focus on supporting our library developers to help them migrate over.

如果您是**函式庫開發者**，[我們邀請您在「新架構工作小組」中發布更新](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)，說明**您的函式庫狀態**。這將幫助您吸引早期採用者，並讓我們了解是否有函式庫遇到阻礙。

如果您是**函式庫使用者**，可以[在此發布訊息](https://github.com/reactwg/react-native-new-architecture/discussions/6)請求遷移某個函式庫。如果我們發現某個函式庫成為多數使用者的阻礙，我們將嘗試聯繫維護者，了解尚未遷移的原因。

最後，我們要特別感謝 Software Mansion 發布了支援雙架構的 [`react-native-screens`](https://github.com/software-mansion/react-native-screens) 新版本。此外，他們還發表了一篇部落格文章（[《將 Fabric 引入 react-native-screens》](https://blog.swmansion.com/introducing-fabric-to-react-native-screens-fd17bf18858e)），**分享他們的遷移歷程**。希望這個故事能激勵並幫助您完成遷移。

### 版本發布

0.68 預發布版本的工作實現了[我們在上半年定義的改進發布流程](/blog/2022/01/19/version-067#improvements-to-release-process)中的大部分內容。

我們很高興地宣布，在 0.68 版本中，我們成功實現了以下目標：

- 順利將發布工作移交至內部輪值機制。這主要得益於[改進的發布流程文件](/contributing/overview)，降低了發布流程的關鍵人員風險。
- 與合作夥伴展開討論，以支援[副駕駛輪值機制](https://github.com/reactwg/react-native-releases/blob/main/docs/roles-and-responsibilities.md)。我們希望此舉能提升流程透明度，並指引合作夥伴如何投資支援 React Native 發布與生態系統。
- [從社群中招募了多位發布支援者與測試者](https://github.com/reactwg/react-native-releases/discussions/11)。我們在上半年發出協助呼籲後，許多人挺身而出！測試者與支援者的回饋**幫助我們修復了關鍵錯誤**和回歸問題，尤其是新架構相關的部分，為即將發布的版本奠定了基礎。感謝所有參與測試的貢獻者！

在 React Native 0.69 版本中，我們將持續完善此流程，目標是讓合作夥伴能更早提供版本信號並完成副駕駛員的入職培訓。一如既往，[我們非常歡迎任何反饋意見](https://github.com/reactwg/react-native-releases/discussions)。若您有意加入成為版本測試員或支援人員，[請點此報名](https://forms.gle/fPuPE1MZRDGWNqpd6)。

### 邁向以 Hermes 作為預設引擎

One of the crucial point of the New Architecture Rollout is the adoption of the new JavaScript engine: **Hermes.**

隨著新 React Native 架構的推出，我們將**把 Hermes 設為預設引擎**。這意味著所有新文件和模板都會預設啟用 Hermes。

請注意，我們會持續與社群合作，確保**其他引擎**（如 JSC (JavaScript Core)）**仍受支援**。您仍可選擇使用偏好的引擎，但需**手動停用 Hermes**。

為提升 Hermes 的穩定性，我們正著手改變 Hermes 的**發佈模式**。具體而言，我們計劃讓 Hermes 的發佈流程**更貼近** React Native 的發佈流程。

這將讓我們能發佈與 JS 引擎**完全相容**的 React Native 版本。您無須再處理難以除錯與理解的執行時崩潰和 Hermes 相容性問題。

此外，這將**縮短**取得 Hermes **改進功能**和錯誤修復的週期，讓我們能更**迅速回應** React Native 使用者的需求。

未來幾個月我們將分享更多相關資訊。在此期間，歡迎[加入工作小組的討論](https://github.com/reactwg/react-native-new-architecture/discussions/4)。

若您尚未嘗試 Hermes，現在正是時候。也請務必回報您可能遇到的任何問題或阻礙。

以上就是本次的內容總結。

我要感謝 Andrei、Aleksandar、Dmitry、Eli、Luna、Héctor 和 Neil 審閱這篇部落格文章，並為這些工作提供寶貴貢獻。

期待**閱讀您的遷移經驗分享**。