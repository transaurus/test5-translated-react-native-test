---
title: 'Use a framework to build React Native apps'
authors: [cortinico]
tags: [announcement]
date: 2024-06-25
---

在[React Conf](https://www.youtube.com/live/0ckOUBiuxVY?si=pU4qP4eB5iWfY0IG&t=2320)上，我們更新了關於開始構建React Native應用程式的最佳工具指南：**React Native框架**——一個包含所有必要API的工具箱，讓您能構建生產就緒的應用程式。

使用React Native框架（例如Expo）現在是創建新應用程式的**推薦**方法。

在這篇部落格文章中，我們將詳細介紹它們是什麼，以及對於您作為一個開始新專案的React Native開發者意味著什麼。

<!-- truncate  -->

## 什麼是React Native框架？

如果您曾經構建過生產環境的應用程式，您可能知道有一系列常見問題遲早需要解決。

無論是在網頁還是原生平台上構建任何應用程式，您可能希望用戶能夠在不同畫面之間導航、獲取數據並存儲用戶狀態。但對於原生應用程式來說，還有更多需要處理的事情：您需要工具來升級React Native版本之間的原生代碼、管理所有依賴項的兼容版本，以及處理原生構建工具。

沒有合適的工具，將一個應用程式從想法帶到生產環境需要付出巨大努力。

我們希望您專注於為用戶編寫精美的應用程式和功能，而不是反覆解決這些常見問題。

這就是為什麼我們認為，您體驗React Native的最佳方式是通過一個提供所有必要工具的工具箱框架來構建生產就緒的應用程式。

我們發現**您要麼使用一個框架，要麼正在構建自己的框架**。

構建自己的框架並沒有錯，通過為路由、導航、部署等問題打造自己的解決方案。像Meta和Microsoft這樣的大公司會在內部構建自己的框架，以深度整合到他們的棕地應用程式中。但我們相信大多數人使用現有框架會更好。

如果您曾經在網頁上使用過React，您可能對[生產級React框架](https://react.dev/learn/start-a-new-react-project#production-grade-react-frameworks)的類似概念很熟悉。

截至目前，唯一推薦的React Native社區框架是[Expo](https://docs.expo.dev/)。Expo團隊從React Native早期就開始投資於React Native生態系統，我們認為Expo提供的開發者體驗是目前最優秀的。

:::note

Expo框架是且將保持免費和開源，而Expo應用程式服務（EAS）是一個可選的付費服務。

:::

<!--alex ignore he-she retext-equality-->

如果您最近沒有使用過Expo，請不要錯過[Kadi在Expo的這場演講](https://www.youtube.com/live/0ckOUBiuxVY?si=N-WSfmAJSMfd6wDL&t=3888)，她在其中展示了2024年您可以用Expo做什麼。

我們還更新了網站上的[入門頁面](https://reactnative.dev/docs/environment-setup)以反映這一推薦。

## 框架將如何影響您？

如果您已經在使用推薦的框架（如Expo），那麼您已經準備就緒！

如果您想將現有應用程式遷移到Expo，可以在[Expo官方網站](https://docs.expo.dev/bare/overview/)上找到相關說明。Expo提供了許多好處，例如更簡單的[升級React Native版本](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/)方式、更好的開發者體驗等等。

不過，如果您無法或不願意遷移到 Expo，也沒關係。不透過官方框架使用 React Native 仍會持續獲得支援。您一直使用的工具，如 React Native Community CLI、Template 和 [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 將繼續照常運作。

The `react-native init` command has moved out of core and is now accessible via:

```
npx @react-native-community/cli@latest init
```

以及在 GitHub 上的 [react-native-community/cli](https://github.com/react-native-community/cli)。

如果您是 React Native 函式庫開發者，我們整理了一份關於該使用哪些 API 的建議清單。[詳見 RFC](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#what-do-we-recommend-to-react-native-library-developers)。

## 延伸閱讀

如果您想進一步了解此決策背後的考量，我們邀請您閱讀 [RFC0759: React Native Frameworks](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#what-do-we-recommend-to-react-native-library-developers)。此 RFC 是歷經數月努力、匯集 React Native 生態系中各方夥伴與參與者無數討論與腦力激盪的成果。

雖然目前 Expo 是唯一推薦的框架，但 RFC 中也包含了[指南](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#becoming-a-react-native-framework)，說明如何成為一個推薦框架，因為我們期待在此領域看到更多競爭與創新。

此外，您應該觀看在 App.js 2024 上的演講 [useFrameworks()](https://www.youtube.com/watch?v=lifGTznLBcw)，我們在短時間內介紹了此 RFC 及必要的變更。

我們相信，透過釐清 React Native Core 與框架各自的職責，能促進更健康的生態系，並推動 React Native 的成長與創新。