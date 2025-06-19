---
title: Helping migrate React Native libraries to the New Architecture
authors: [cipolleschi]
tags: [announcement]
date: 2022-06-16
---

**簡要說明**：我們正在完善支援 React Native 新架構的資源。我們已發布兩個資源庫協助遷移應用程式（[RNNewArchitectureApp](https://github.com/react-native-community/RNNewArchitectureApp)）與函式庫（[RNNewArchitectureLibraries](https://github.com/react-native-community/RNNewArchitectureLibraries)），同時重構官網的[新架構指南](https://github.com/facebook/react-native-website/pull/3037)，並成立[GitHub 工作小組](https://github.com/reactwg/react-native-new-architecture/discussions)解答相關問題。

<!--truncate-->

## 引言

In this post we are sharing an update on tools and resources to help you migrate your **Native Modules** and **Native Components** to their **New Architecture** equivalents, **TurboModule** and **Fabric Components**.

React Native 開發者廣泛使用開源函式庫建置應用程式。為確保生態系統完整且一致，這些函式庫需完成遷移，方能讓所有人受益於新架構帶來的效能提升與功能擴展。

以下是我們為支援函式庫開發者遷移所推進的工作：

- **文件擴充**：我們正在官網擴充[新架構指南](https://github.com/facebook/react-native-website/pull/3037)，涵蓋更多新架構概念與元件開發方法。  
- **遷移範例**：建立兩個資源庫分別示範如何遷移應用程式至新架構（[RNNewArchitectureApp](https://github.com/react-native-community/RNNewArchitectureApp)），以及如何開發同時相容新舊架構的**Fabric元件**與**TurboModule**（[RNNewArchitectureLibraries](https://github.com/react-native-community/RNNewArchitectureLibraries)）。  
- **支援體系**：今年初成立專屬的[GitHub 工作小組](https://github.com/reactwg/react-native-new-architecture/discussions)，集中討論新架構相關問題。

本文將深入解析這些資源，並說明如何高效運用。最後我們將提供熱門 React Native 函式庫的當前遷移狀態概覽。

### 文件資源

過去半年間，我們新增了[新架構採用指南](https://github.com/reactwg/react-native-new-architecture#guides)與 Fabric 的[架構深度解析](/architecture/overview)。未來計劃擴充至 TurboModule 開發、CodeGen 解析等主題，預計在 0.70 版本發布時更新內容。

當前**新架構指南**涵蓋如何正確遷移[應用程式](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/enable-apps.md)與[函式庫](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/enable-libraries-prerequisites.md)至新架構。

若您關注指南進展或有意反饋，可追蹤[此](https://github.com/facebook/react-native-website/pull/3037)拉取請求。

### 遷移範例

針對偏好實作參考的開發者，我們準備了兩個範例資源庫。

#### RNNewArchitectureApp

[此資源庫](https://github.com/react-native-community/RNNewArchitectureApp)展示如何將 React Native 0.67 版本的應用程式、原生模組與元件從舊架構遷移至新架構及最新版 React Native。每個提交對應一個獨立遷移步驟。

<figure>
    <img src="/blog/assets/new-arch-example-steps-to-migrate-an-app.png" alt="Example steps to migrate an app" />
    <figcaption>Commit list for a migration in the RNNewArchitectureApp repository</figcaption>
</figure>

資源庫結構如下：

- A **main** branch has no code but a README.md which advertises other branches.
- Several migration branches which show a migration from a specific version of RN to another.

部分遷移分支還包含一個 **RUN.md** 文件，以更人性化的方式描述每個提交中應用的具體步驟。

我們計劃讓此範例與最新的穩定版本保持同步，針對每個即將發布的 React Native 小版本新增遷移步驟。如果您發現任何步驟存在問題，請在該存儲庫中提交 issue。此做法將持續到我們合理認為大多數 React Native 用戶已遷移至新架構為止。

#### RNNewArchitectureLibraries

類似地，[此存儲庫](https://github.com/react-native-community/RNNewArchitectureLibraries)提供了逐步指南，說明如何創建 **TurboModule** 和 **Fabric Component**，重點在於確保新架構與舊架構之間的向後兼容性。

該存儲庫的組織方式與前一個類似：

- A **main** branch has no code but a README.md which advertises other branches.
- Other branches to show how to develop **TurboModules** and **Fabric Components**.

我們計劃讓此範例隨 React Native 的新版本更新，特別是影響庫開發的版本，並添加更多關於如何使用進階功能（例如：實現命令、事件發射器、自定義狀態）的範例。如果您發現錯誤，請在範例存儲庫中提交 issue。

### 支援

我們創建了一個專屬的[工作組](https://github.com/reactwg/react-native-new-architecture)，為社區提供提問和獲取新架構更新的空間。如果您是庫維護者，這是一個寶貴的資源，可以找到問題的答案，並讓我們了解您的需求。如需加入，請遵循[這些說明](https://github.com/reactwg/react-native-new-architecture#how-to-join-the-working-group)。歡迎所有人參與。

工作組分為以下幾個部分：

- [公告](https://github.com/reactwg/react-native-new-architecture/discussions/categories/announcements)：分享 RN 新架構推出的里程碑和重要更新
- [深度探討](https://github.com/reactwg/react-native-new-architecture/discussions/categories/deep-dive)：討論深度技術主題
- [文檔](https://github.com/reactwg/react-native-new-architecture/discussions/categories/documentation)：討論新架構文檔和遷移材料
- [庫](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)：討論第三方庫及其遷移至新架構的進展
- [問答](https://github.com/reactwg/react-native-new-architecture/discussions/categories/q-a)：向社區尋求新架構主題的幫助
- [發布](https://github.com/reactwg/react-native-new-architecture/discussions/categories/releases)：討論特定版本的錯誤和構建問題

有效使用此工作組的方法：

- **確保您的庫列在[庫](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)部分中**。這將幫助我們分享您的庫遷移狀態更新，並了解庫作者面臨的困難以提供更好的支援。
- **如果您遇到阻礙並需要支援，請利用[問答](https://github.com/reactwg/react-native-new-architecture/discussions/categories/q-a)部分**。我們的團隊和社區專家將盡力提供支援。
- **關注其他部分中可能影響您的主題**。新版本可能正好引入了您需要的 API。您可以通過 GitHub 訂閱特定討論。

我們計劃支援此工作組，直到**新架構**默認啟用且所有主要庫均已遷移至該架構為止。

### 熱門庫的遷移狀態

庫維護者一直在[工作組中](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)與我們分享他們的遷移進展，我們希望為您提供一個快速概覽：

- [react-native-gesture-handler](https://github.com/reactwg/react-native-new-architecture/discussions/15): ✅ 已完成遷移
- [react-native-navigation](https://github.com/reactwg/react-native-new-architecture/discussions/17): 🏃‍♂️ 進行中
- [react-native-pager-view](https://github.com/reactwg/react-native-new-architecture/discussions/16): 🏃‍♂️ 進行中
- [react-native-reanimated](https://github.com/reactwg/react-native-new-architecture/discussions/14): ✅ 已完成遷移。正在進行效能測試與分析
- [react-native-screens](https://github.com/reactwg/react-native-new-architecture/discussions/13): 🏃‍♂️ 進行中
- [react-native-slider](https://github.com/reactwg/react-native-new-architecture/discussions/38): 🎬 已開始
- [react-native-template-new-architecture](https://github.com/reactwg/react-native-new-architecture/discussions/21): ✅ 已完成遷移。逐步採用/測試更多配套函式庫
- [react-native-template-typescript](https://github.com/reactwg/react-native-new-architecture/discussions/22): ✅ 已完成遷移
- [react-native-webview](https://github.com/reactwg/react-native-new-architecture/discussions/19): 🎬 已開始

## 後續步驟

我們致力於支持 React Native 社群採用新架構。具體而言，我們將持續：

- Offer best-effort support in the **Working Group.**
- Provide more examples about how to achieve amazing results with the New Architecture in the **RNNewArchitecture** repositories.
- Provide clear and up-to-date documentation on the **New Architecture**.
- Track the migration status of essential React Native libraries in the **Working Group**.
- Simplify the migration path for developers

此外，React Native 0.69 將為採用新架構的應用程式和函式庫開發者帶來改進的開發體驗。您可以在[此處](https://github.com/reactwg/react-native-releases/discussions/21)找到更多關於 0.69.0 版本的資訊。

我們對於將與**新架構**共同建構的未來感到興奮！